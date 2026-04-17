# Plan de Pruebas: Rate Limits y Quotas de APIs

Este documento recopila todos los límites de frecuencia (*rate limits*) y cuotas (*quotas*) extraídos de los datasheets del repositorio, e indica qué pruebas deben realizarse contra cada API para verificar que los límites se cumplen tal y como describe la documentación.

---

## Índice

1. [API-Football](#1-api-football)
2. [SoundCloud](#2-soundcloud)
3. [Clarifai](#3-clarifai)
4. [Boundaries.io](#4-boundariesio)
5. [ZenHub](#5-zenhub)
6. [Amadeus Self-Service](#6-amadeus-self-service)
7. [Foursquare Places API](#7-foursquare-places-api)
8. [Spotify Web API](#8-spotify-web-api)
9. [Yelp APIs](#9-yelp-apis)
10. [Vimeo API](#10-vimeo-api)
11. [Dailymotion API](#11-dailymotion-api)
12. [Bitbucket Cloud API](#12-bitbucket-cloud-api)
13. [Spoonacular Food API](#13-spoonacular-food-api)
14. [Azure Translator (Azure AI Services)](#14-azure-translator-azure-ai-services)
15. [AccuWeather API](#15-accuweather-api)
16. [ADS-B Exchange API](#16-ads-b-exchange-api)
17. [Azure AI Search](#17-azure-ai-search)
18. [CurrencyLayer](#18-currencylayer)
19. [DHL Location Finder](#19-dhl-location-finder)
20. [Twilio SendGrid Email API](#20-twilio-sendgrid-email-api)
21. [MailerSend](#21-mailersend)
22. [Resend](#22-resend)

---

## 1. API-Football

**Fuente:** rama `7-api-football` · `api-football/datasheet/basic-datasheet.yml`  
**URL de referencia:** https://rapidapi.com/api-sports/api/api-football/pricing

### Límites documentados

| Tipo | Valor |
|------|-------|
| Cuota diaria | 100 requests/día (se reinicia diariamente) |
| Cuota mensual de datos | 10 240 MB/mes (excedente: $0.001/MB) |
| Rate limit | 1 000 requests/hora |

### Pruebas recomendadas

1. **Verificar respuesta al superar la cuota diaria:** enviar 101 peticiones en el mismo día y comprobar que la petición 101 devuelve HTTP 429. Verificar que al día siguiente el contador se reinicia.
2. **Verificar el rate limit horario:** enviar 1 001 peticiones en menos de una hora y confirmar que se recibe HTTP 429 antes de llegar a la petición 1 001.
3. **Verificar cabeceras de límite:** revisar que las respuestas incluyen cabeceras de cuota restante (`X-RateLimit-Requests-Remaining`, `X-RateLimit-Requests-Reset` o similares).
4. **Verificar el excedente de datos:** superar los 10 240 MB de transferencia mensual y confirmar que se aplica el cargo adicional de $0.001/MB sin bloquear el acceso.

---

## 2. SoundCloud

**Fuente:** rama `15-soundcloud` · `soundcloud/datasheet/datasheet.yml`  
**URL de referencia:** https://developers.soundcloud.com/docs/api/rate-limits

### Límites documentados

| Tipo | Valor |
|------|-------|
| Rate limit (Play Requests) | 15 000 requests/día (ventana deslizante de 24 h) |
| Segmentación | Por Client ID de la aplicación |
| Cooldown | Dinámico (campo `reset_time` en el cuerpo JSON de la respuesta 429) |

### Pruebas recomendadas

1. **Verificar el límite diario en play requests:** enviar 15 001 peticiones al endpoint `/tracks/:id/stream` dentro de una ventana de 24 horas y confirmar que la última devuelve HTTP 429.
2. **Verificar la ventana deslizante:** confirmar que el límite no se reinicia en un momento fijo del día sino en relación con las peticiones más antiguas de la ventana de 24 h.
3. **Verificar el campo `reset_time`:** al recibir una respuesta 429, comprobar que el cuerpo JSON contiene el campo `reset_time` con la fecha/hora en que se restablecerá el acceso.
4. **Verificar aislamiento por Client ID:** utilizar dos Client IDs distintos y verificar que cada uno tiene su propio contador independiente.

---

## 3. Clarifai

**Fuente:** rama `17-clarifai` · `clarifai/datasheet/`  
**URL de referencia:** https://docs.clarifai.com/resources/api-overview/rate-limits

### Límites documentados

#### Plan Pay-As-You-Go

| Tipo | Valor |
|------|-------|
| Cuota mensual (Essential) | 30 000 operaciones/mes |
| Cuota mensual (Professional) | 100 000 operaciones/mes |
| Rate limit | 15 requests/segundo |
| Cooldown | 15 segundos de back-off al recibir error 429/11005 (`CONN_THROTTLED`) |
| Gasto máximo | Configurable (por defecto $0) |

#### Plan Enterprise

| Tipo | Valor |
|------|-------|
| Cuota mensual | Ilimitada (negociada) |
| Rate limit | 15 RPS por defecto, configurable hasta 1 000+ RPS |
| Políticas | SLAs y soporte dedicado |

### Pruebas recomendadas

1. **Verificar el rate limit de 15 RPS:** enviar 16 peticiones en menos de 1 segundo y confirmar que alguna devuelve error `CONN_THROTTLED` (11005).
2. **Verificar la estrategia de back-off:** al recibir el error 11005, esperar 15 segundos y comprobar que las peticiones se retoman correctamente.
3. **Verificar el límite mensual de operaciones:** enviar peticiones hasta agotar la cuota del plan (30 000 o 100 000) y confirmar que se bloquean las peticiones siguientes hasta el reinicio mensual.
4. **Verificar el límite de gasto máximo:** configurar un límite de gasto mensual y comprobar que al alcanzarlo la API devuelve el error correspondiente sin cobrar adicional.

---

## 4. Boundaries.io

**Fuente:** rama `20-boundariesio` · `boundaries-io/datasheet/`  
**URL de referencia:** https://rapidapi.com/VanitySoft/api/boundaries-io-1/pricing

### Límites documentados

| Plan | Cuota (por grupo de endpoints) | Rate limit |
|------|-------------------------------|------------|
| Basic (Trial 7 días) | 50 requests/endpoint group | 1 000 requests/hora |
| Pro | 5 000 requests/endpoint group (excedente $0.01/req) | Ilimitado |
| Ultra | 30 000 requests/endpoint group (excedente $0.01/req) | Ilimitado |
| Mega | 100 000–200 000 requests/endpoint group (variable; excedente $0.01/req) | Ilimitado |

**Política especial:** prohibición estricta de *harvesting* (descarga masiva); viola la política de uso puede resultar en bloqueo de la cuenta.

### Pruebas recomendadas

1. **Verificar la cuota por endpoint group (plan Basic):** enviar 51 peticiones al mismo grupo de endpoints y confirmar que la 51 devuelve HTTP 429 o 403.
2. **Verificar el rate limit horario (plan Basic):** enviar 1 001 peticiones en menos de una hora y confirmar el bloqueo.
3. **Verificar independencia de cuotas entre grupos de endpoints:** comprobar que el consumo en un grupo no afecta la cuota de otro grupo.
4. **Verificar el excedente:** en planes Pro/Ultra/Mega superar la cuota incluida y comprobar que se aplica el cargo $0.01/request sin cortar el acceso.
5. **Verificar el uptime garantizado (99.9%):** monitorizar la disponibilidad durante un período representativo.

---

## 5. ZenHub

**Fuente:** rama `25-zenhub-api` · `zenhub/datasheet/`  
**URL de referencia:** https://developers.zenhub.com/graphql-api-docs/rate-limiting

### Límites documentados

#### Free/Teams (2026)

| Tipo | Valor |
|------|-------|
| Rate limit | 90 segundos de tiempo de procesamiento/minuto por API Key |
| Concurrencia máxima | 30 peticiones concurrentes por token |
| Algoritmo | Token Bucket (relleno continuo) |
| Límite de complejidad | 200 puntos por consulta |

#### Enterprise (2026)

| Tipo | Valor |
|------|-------|
| Rate limit | 90 segundos/minuto por defecto (configurable) |
| Concurrencia | 30 por defecto (configurable) |
| Complejidad | 200 puntos por defecto (configurable) |

#### Cabeceras de respuesta

`X-RateLimit-Limit`, `X-RateLimit-Used`, `X-RateLimit-Remaining`

### Pruebas recomendadas

1. **Verificar el límite de tiempo de procesamiento:** enviar consultas costosas que consuman más de 90 segundos de CPU por minuto y comprobar que se devuelve HTTP 429.
2. **Verificar la concurrencia máxima:** lanzar 31 peticiones simultáneas y confirmar que alguna es rechazada o encolada.
3. **Verificar el límite de complejidad por consulta:** construir una consulta GraphQL con más de 200 puntos de complejidad y confirmar que es rechazada.
4. **Verificar las cabeceras de rate limit:** comprobar en cada respuesta la presencia y valores correctos de `X-RateLimit-Limit`, `X-RateLimit-Used` y `X-RateLimit-Remaining`.
5. **Verificar el relleno continuo del Token Bucket:** tras recibir un 429, esperar menos de un minuto y verificar que las peticiones se retoman conforme se rellena el bucket.

---

## 6. Amadeus Self-Service

**Fuente:** rama `28-amadeus` · `amadeus/datasheet/` y rama actual · `new_datasheets/amadeus.yaml`  
**URL de referencia:** https://developers.amadeus.com/self-service/apis-docs/guides/developer-guides/api-rate-limits/

### Límites documentados

#### Entorno Test (Sandbox)

| Tipo | Valor |
|------|-------|
| Rate limit (APIs estándar) | 10 TPS (1 request cada 100 ms) |
| Rate limit (APIs AI/Partner) | 20 TPS (1 request cada 50 ms) |
| Cuotas mensuales (ejemplo) | 2 000 req/mes para Flight Offers; 3 000 para Pricing; hasta 10 000 para Hotels/Airlines |
| Coste excedente | Sin coste (error 429 al superar la cuota) |

#### Entorno Production

| Tipo | Valor |
|------|-------|
| Rate limit (APIs estándar) | 40 TPS (full parallelización) |
| Rate limit (APIs AI/Partner) | 20 TPS |
| Cuotas mensuales | Igual que Test pero con overage pay-as-you-go |
| Coste por petición según tier | Tier 1: €0.04; Tier 2: €0.025; Tier 3: €0.015; Tier 4: €0.0025; Tier 5: €0.0015 |

### Pruebas recomendadas

1. **Verificar rate limit estándar en Test:** enviar 11 peticiones en menos de 1 segundo al endpoint `v2/shopping/flight-offers` y confirmar que se recibe HTTP 429.
2. **Verificar rate limit AI/Partner en Test:** enviar peticiones con menos de 50 ms de separación a un endpoint AI (p.ej. `v1/shopping/activities`) y confirmar el error 429.
3. **Verificar cuota mensual en Test:** alcanzar la cuota mensual de un endpoint concreto (por ejemplo 2 000 requests en `v2/shopping/flight-offers`) y confirmar que las siguientes peticiones devuelven 429 hasta el reinicio mensual.
4. **Verificar el incremento de cuota en Production:** verificar que el coste por petición se registra correctamente en el panel de Amadeus según el tier del endpoint utilizado.
5. **Verificar que la cuota es compartida entre apps de la misma cuenta:** crear dos apps bajo la misma cuenta y comprobar que el consumo de una afecta al contador de la otra.

---

## 7. Foursquare Places API

**Fuente:** rama `33-foursquare---places-api` · `foursquare/datasheet/`  
**URL de referencia:** https://docs.foursquare.com/developer/reference/rate-limits

### Límites documentados

| Plan | Cuota incluida | Rate limit |
|------|----------------|------------|
| Pay-As-You-Go | 10 000 Pro Calls/mes gratis, luego pay-as-you-go | 50 QPS (total entre todos los endpoints) |
| Enterprise | Cuota negociada | 100 QPS |

**Precios por petición (Pay-as-you-go):**

| Tier (Pro) | Rango | Precio |
|------------|-------|--------|
| Tier 1 | 10k–100k | $0.015/req |
| Tier 2 | 100k–500k | $0.012/req |
| Tier 3 | 500k–1M | $0.009/req |

### Pruebas recomendadas

1. **Verificar el rate limit de 50 QPS:** enviar 51 peticiones por segundo y confirmar que alguna recibe HTTP 429.
2. **Verificar que el rate limit es global entre endpoints:** distribuir 50 peticiones/segundo entre varios endpoints (Search + Details) y comprobar que el límite aplica al total, no por endpoint individual.
3. **Verificar las 10 000 llamadas gratuitas:** llevar el contador a exactamente 10 000 peticiones y verificar que la llamada 10 001 genera cargo en el dashboard.
4. **Verificar los precios escalonados:** consumir más de 100 000 peticiones en un mes y confirmar que el precio por petición baja al Tier 2.
5. **Verificar el límite de 100 QPS para Enterprise:** con credenciales Enterprise, enviar 101 peticiones/segundo y confirmar la limitación.

---

## 8. Spotify Web API

**Fuente:** rama `35-spotify-api` · `spotify/datasheet/`  
**URL de referencia:** https://developer.spotify.com/documentation/web-api/concepts/rate-limits

### Límites documentados

#### Development Mode (2026)

| Tipo | Valor |
|------|-------|
| Rate limit | Ventana deslizante de 30 segundos (valor exacto no publicado) |
| Usuarios autenticados | Máximo 25 usuarios (allowlist manual) |
| Cooldown | Cabecera `Retry-After` en la respuesta 429 |

#### Extended Quota Mode (2026)

| Tipo | Valor |
|------|-------|
| Rate limit | Aumentado (ventana deslizante de 30 s, valores negociados) |
| Usuarios | Ilimitados |
| Requisitos | Aprobación de Spotify, entidad legal, mínimo 250k MAUs |

### Pruebas recomendadas

1. **Verificar el límite de usuarios en Development Mode:** intentar autenticar un usuario número 26 y confirmar que se devuelve HTTP 403.
2. **Verificar el rate limit por aplicación:** enviar peticiones rápidas hasta recibir HTTP 429 y leer la cabecera `Retry-After` para determinar el tiempo de espera.
3. **Verificar que el límite es por aplicación (Client ID) y no por usuario:** con múltiples usuarios autenticados bajo la misma app, verificar que la suma de sus peticiones cuenta contra el límite global de la aplicación.
4. **Verificar `Retry-After` en respuesta 429:** confirmar que al recibir 429 el campo `Retry-After` está presente y que esperar ese tiempo permite continuar.
5. **Verificar ventana deslizante de 30 segundos:** confirmar que el contador no se reinicia en intervalos fijos sino de forma deslizante.

---

## 9. Yelp APIs

**Fuente:** rama `37-yelp-apis-places-insight-ai-api` · `yelp/datasheet/`

### 9.1 Yelp Places API (Fusion)

**URL de referencia:** https://docs.developer.yelp.com/docs/fusion-rate-limiting

| Tipo | Valor |
|------|-------|
| Cuota diaria (Starter) | 300 llamadas/día |
| Rate limit por segundo | TBD (HTTP 429 `TOO_MANY_REQUESTS_PER_SECOND`) |
| Cooldown | `RateLimit-ResetTime` en cabeceras |
| Cabeceras | `RateLimit-DailyLimit`, `RateLimit-Remaining`, `RateLimit-ResourceDailyLimit`, `RateLimit-ResetTime` |

### 9.2 Yelp AI API

| Tipo | Valor |
|------|-------|
| Cuota | TBD (mínimo 1 000 llamadas/día en planes standalone) |
| Rate limit | TBD |

### 9.3 Yelp Insights API

| Tipo | Valor |
|------|-------|
| Acceso | Restringido a partners contratados |
| Límites | Negociados por contrato |

### Pruebas recomendadas

1. **Verificar la cuota diaria de 300 llamadas:** enviar 301 peticiones en el mismo día UTC y confirmar `ACCESS_LIMIT_REACHED` (HTTP 429).
2. **Verificar la cabecera `RateLimit-Remaining`:** comprobar en cada respuesta que el contador decrece correctamente y llega a 0 al agotar la cuota.
3. **Verificar `RateLimit-ResetTime`:** al recibir 429, leer `RateLimit-ResetTime` y confirmar que después de esa hora se restablece el acceso.
4. **Verificar el rate limit por segundo:** enviar varias peticiones en un mismo segundo hasta recibir `TOO_MANY_REQUESTS_PER_SECOND` y medir el QPS máximo permitido.
5. **Verificar límites por endpoint con `RateLimit-ResourceDailyLimit`:** si la cabecera está presente en algún endpoint, comprobar que tiene su propio sub-límite diario independiente.

---

## 10. Vimeo API

**Fuente:** rama `42-vimeo-api` · `vimeo/datasheet/vimeo.yaml`  
**URL de referencia:** https://developer.vimeo.com/guidelines/rate-limiting

### Límites documentados

| Plan | Rate limit (general) | Rate limit (con field filtering) |
|------|---------------------|----------------------------------|
| Free | 25 req/min | 50 req/min |
| Starter | 125 req/min | 250 req/min |
| Standard | 250 req/min | 500 req/min |
| Advanced | 750 req/min | 1 500 req/min |
| Enterprise | 2 500 req/min | 5 000 req/min |

**Cooldown:** hasta que termine el período actual de 60 segundos (HTTP 429, error code 9000; cabecera `X-RateLimit-Reset`).  
**Segmentación:** por token (unauthenticated → cuota del dueño de la app; authenticated → cuota propia por usuario).

### Pruebas recomendadas

1. **Verificar el límite por minuto según plan:** enviar N+1 peticiones en menos de 60 segundos (donde N es el límite del plan) y confirmar HTTP 429 con código de error 9000.
2. **Verificar el doble de límite con field filtering:** utilizar el parámetro `fields` en la petición y comprobar que el límite efectivo se duplica respecto al sin filtro.
3. **Verificar la cabecera `X-RateLimit-Reset`:** al recibir 429, leer la cabecera `X-RateLimit-Reset` y confirmar que tras esa fecha/hora el acceso se restaura.
4. **Verificar independencia de cuotas entre usuarios autenticados:** con dos usuarios diferentes bajo la misma app, confirmar que el rate limit de uno no consume el del otro.
5. **Verificar que el token no autenticado hereda el límite del plan del dueño de la app:** hacer peticiones sin autenticación y comprobar que el límite corresponde al plan del propietario de la aplicación.

---

## 11. Dailymotion API

**Fuentes:**  
- Rama `43-dailymotion-api` · `dailymotion/datasheet/dailymotion.yaml`  
- Rama actual · `new_datasheets/dailymotion.yaml`  
**URL de referencia:** https://developers.dailymotion.com/reference/rate-limiting / https://pro.dailymotion.com/en/pricing/

### Límites documentados

#### Límites de subida de vídeos (por nivel de creador)

Los siguientes límites aplican a la subida de contenido según el tipo de cuenta de creador:

| Plan de creador | Vídeos/día | Duración máx./día | Cooldown |
|----------------|-----------|-------------------|---------|
| Standard (usuarios, creadores estándar, Pro Trial/Starter/Advanced) | 15 vídeos | 10 horas | Suspensión 24 h |
| Premium (creadores Premium, Pro Enterprise) | 96 vídeos (ampliable contactando Dailymotion) | Sin límite | Suspensión 24 h |

#### Cuotas de API y servicio (planes de suscripción Pro)

Los siguientes límites aplican a las cuotas de API y ancho de banda según el plan de suscripción Pro:

| Plan Pro | Límite de API | Bandwidth VOD/mes | Almacenamiento |
|----------|--------------|-------------------|---------------|
| Starter | 1 000 req/h | 2 000 GB | 100 GB |
| Advanced | 1 000 req/h | 2 000 GB (excedente $0.0000072/MB) | 1 000 GB |
| Enterprise | Personalizado | Personalizado | Personalizado |

#### Rate limits adicionales

| Límite | Valor |
|--------|-------|
| Upload diario por hora global | 4 vídeos/hora |
| Stream URL retrieval | 50 req/día |
| Resultados por búsqueda | Máximo 1 000 |
| Items por página | Máximo 100 |

### Pruebas recomendadas

1. **Verificar el límite diario de subida (Standard):** intentar subir el vídeo número 16 en el mismo día y confirmar el rechazo.
2. **Verificar el límite de duración total diaria (Standard):** subir vídeos cuya duración acumulada supere 10 horas y confirmar que se bloquea la subida.
3. **Verificar la suspensión de 24 horas:** tras superar el límite de subida, confirmar que ninguna subida es posible durante las siguientes 24 horas.
4. **Verificar el rate limit global de API (1 000 req/h):** enviar 1 001 peticiones en menos de una hora y confirmar HTTP 429.
5. **Verificar el límite de stream URL retrieval (50/día):** acceder 51 veces al campo de URL de streaming en el mismo día y confirmar el bloqueo.
6. **Verificar la paginación máxima (100 items/página):** solicitar más de 100 resultados por página y confirmar que se trunca o devuelve error.

---

## 12. Bitbucket Cloud API

**Fuente:** rama `44-bitbucket-api` · `bitbucket/datasheet/`  
**URL de referencia:** https://support.atlassian.com/bitbucket-cloud/docs/api-request-limits/

### Límites documentados

| Tipo de acceso | Rate limit | Algoritmo |
|----------------|------------|-----------|
| Anónimo (por IP) | 60 req/hora | Rolling Window |
| Autenticado (por usuario) | 1 000 req/hora | Rolling Window |
| Scaled (>100 usuarios pagados) | Hasta 10 000 req/hora (fórmula: 1 000 + (usuarios_pagados − 100) × 10) | Rolling Window |

#### Límites especiales (autenticado)

| Recurso | Límite |
|---------|--------|
| Git Operations (HTTPS/SSH) | 60 000 req/hora |
| Raw Files | 5 000 req/hora |
| Archive Files | 1 500 archivos/hora |
| Webhook Management | 1 000 req/hora |
| Application Properties | 2 000 req/hora |
| Invitations | 100 req/minuto |

**Cabeceras:** `X-RateLimit-Limit`, `X-RateLimit-Resource`, `X-RateLimit-NearLimit`

### Pruebas recomendadas

1. **Verificar el límite anónimo de 60 req/hora:** desde una IP sin autenticación, enviar 61 peticiones en menos de una hora y confirmar HTTP 429.
2. **Verificar el límite autenticado de 1 000 req/hora:** con un usuario autenticado, enviar 1 001 peticiones en menos de una hora y confirmar el bloqueo.
3. **Verificar la cabecera `X-RateLimit-NearLimit`:** comprobar que al superar el 80% del límite, la cabecera está presente en las respuestas.
4. **Verificar el límite de Invitations (100 req/min):** enviar 101 invitaciones en menos de un minuto y confirmar el rechazo.
5. **Verificar el límite de Raw Files (5 000 req/hora):** descargar más de 5 000 archivos raw en menos de una hora y confirmar la limitación.
6. **Verificar el cálculo del límite Scaled:** con una organización con 150 usuarios pagados, comprobar que el límite es 1 000 + (150 − 100) × 10 = 1 500 req/hora.

---

## 13. Spoonacular Food API

**Fuente:** rama `45-spoonacular-api` · `spoonacular/datasheet/`  
**URL de referencia:** https://spoonacular.com/food-api/docs#Quotas

### Límites documentados

| Plan | Cuota diaria (puntos) | Rate limit | Excedente |
|------|----------------------|------------|-----------|
| Free | 150 puntos/día | 60 req/min (1 req/s) | No (error 402) |
| Cook | Cuota superior | Mayor | Sí |
| Chef | 10 000 puntos/día | 20 req/s | $0.002/punto |
| Enterprise | Personalizado | Personalizado | Negociado |

**Sistema de puntos:** cada petición cuesta ≈1 punto + 0.01 puntos por resultado devuelto.  
**Reinicio:** medianoche UTC.  
**Cabeceras:** `X-API-Quota-Request`, `X-API-Quota-Used`, `X-API-Quota-Left`

### Pruebas recomendadas

1. **Verificar el límite de rate (60 req/min en Free):** enviar 61 peticiones en menos de un minuto y confirmar HTTP 429.
2. **Verificar que la cuota se agota con el sistema de puntos:** hacer peticiones con resultados grandes (que consuman más de 1 punto) y comprobar que la cuota se reduce más rápidamente de lo esperado si solo se contaran peticiones.
3. **Verificar el reinicio a medianoche UTC:** agotar la cuota diaria y confirmar que al llegar la medianoche UTC se restablece.
4. **Verificar las cabeceras de cuota:** comprobar en cada respuesta que `X-API-Quota-Used`, `X-API-Quota-Left` y `X-API-Quota-Request` tienen valores correctos y coherentes.
5. **Verificar el error 402 al agotar la cuota (Free):** en el plan Free, confirmar que al llegar a 0 puntos se devuelve HTTP 402 (no 429) y que no se admiten más llamadas hasta el reinicio.

---

## 14. Azure Translator (Azure AI Services)

**Fuente:** rama `46-azure-translator-api` · `azure-translator/datasheet/`  
**URL de referencia:** https://learn.microsoft.com/en-us/azure/ai-services/translator/service-limits

### Límites documentados

#### Tier F0 (Gratis)

| Tipo | Valor |
|------|-------|
| Cuota mensual de caracteres | 2 000 000 caracteres/mes |
| Rate limit | 2 000 000 caracteres/hora (≤33 300 caracteres/minuto) |
| Límite por petición (`/translate`) | 50 000 caracteres, hasta 1 000 elementos en el array |

#### Tier S1 (Pay-as-you-go)

| Tipo | Valor |
|------|-------|
| Rate limit | 40 000 000 caracteres/hora (≤666 666 caracteres/minuto) |
| Coste traducción de texto | $10 / 1 000 000 caracteres |
| Coste traducción de documentos | $15 / 1 000 000 caracteres |
| Latencia máxima (modelos estándar) | 15 segundos |

#### Límites por operación

| Operación | Tamaño máx. por elemento | Máx. elementos | Tamaño máx. petición |
|-----------|--------------------------|----------------|----------------------|
| Translate | 50 000 chars | 1 000 | 50 000 chars |
| Transliterate | 5 000 chars | 10 | 5 000 chars |
| Detect | 50 000 chars | 100 | 50 000 chars |
| BreakSentence | 50 000 chars | 100 | 50 000 chars |
| DictionaryLookup | 100 chars | 10 | 1 000 chars |

### Pruebas recomendadas

1. **Verificar el límite de caracteres por hora (F0):** enviar traduccciones hasta superar 2 000 000 caracteres en una hora y confirmar la respuesta *out-of-quota*.
2. **Verificar los límites por petición:** enviar una petición de traducción con un array de 1 001 elementos y confirmar el rechazo. Repetir con un texto de 50 001 caracteres.
3. **Verificar el límite de `DictionaryLookup`:** enviar un array de 11 palabras y confirmar el error.
4. **Verificar que no hay límite de peticiones concurrentes:** lanzar múltiples peticiones simultáneas y confirmar que el único límite es el total de caracteres, no el número de peticiones.
5. **Verificar la latencia máxima de 15 segundos (S1):** enviar traducciones de larga duración y confirmar que siempre responden en ≤15 segundos con modelos estándar.
6. **Verificar la cuota mensual de F0:** al llegar a 2 000 000 caracteres en el mes, confirmar que se devuelve el error de cuota superada hasta el mes siguiente.

---

## 15. AccuWeather API

**Fuente:** rama `accuweather` · `accuweather/datasheet/`  
**URL de referencia:** https://developer.accuweather.com/pricing

### Límites documentados

| Plan | Cuota | Rate limit | Algoritmo |
|------|-------|------------|-----------|
| Free Trial | 500 llamadas/24 h (rolling) | 1 req/s | Fixed Window |
| Starter | 15 000 llamadas/mes (calendario) | 5 req/s | Sliding Window Counter |
| Standard | 225 000 llamadas/mes | 10 req/s | Sliding Window Counter |
| Prime | 1 800 000 llamadas/mes | 50 req/s | Sliding Window Counter |
| Elite | Personalizado | Personalizado | — |

**Nota:** Free Trial está limitado a 1 API Key por desarrollador.  
**Cooldown:** cabecera `Retry-After` en respuestas 429.

### Pruebas recomendadas

1. **Verificar la cuota diaria del Free Trial (500 llamadas/24 h):** enviar la llamada número 501 dentro de una ventana de 24 horas y confirmar HTTP 429.
2. **Verificar el rate limit por segundo según plan:** para el plan Starter, enviar 6 peticiones en menos de 1 segundo y confirmar el rechazo.
3. **Verificar la cabecera `Retry-After`:** al recibir 429, leer `Retry-After` y confirmar que esperar ese tiempo restablece el acceso.
4. **Verificar la cuota mensual (calendario):** para Starter, enviar más de 15 000 peticiones antes del fin del mes natural y confirmar el bloqueo.
5. **Verificar el excedente de llamadas ($0.25 CPM para Starter):** superar la cuota mensual incluida y comprobar que se registra el coste adicional en el panel.
6. **Verificar que Free Trial solo permite 1 API Key por cuenta:** intentar crear una segunda API Key y confirmar que se rechaza.

---

## 16. ADS-B Exchange API

**Fuente:** rama `adsb-exchange` · `adsb-exchange/datasheet/adsb-exchange-basic.yml`  
**URL de referencia:** https://rapidapi.com/adsbx/api/adsbexchange-com1

### Límites documentados (Plan Basic)

| Tipo | Valor |
|------|-------|
| Cuota mensual | 10 000 llamadas/mes |
| Rate limit | No especificado explícitamente |
| Excedente | $0.0015/petición |
| Cooldown | Hard block al superar el límite mensual |

### Pruebas recomendadas

1. **Verificar la cuota mensual de 10 000 llamadas:** enviar la llamada número 10 001 en el mismo mes y confirmar el bloqueo.
2. **Verificar el cargo por excedente:** superar las 10 000 llamadas y confirmar en el dashboard de RapidAPI que se aplica el cargo $0.0015 por petición adicional.
3. **Verificar el hard block:** confirmar que al agotar la cuota sin habilitar overage no es posible realizar más llamadas hasta el próximo mes.
4. **Verificar la segmentación por API Key:** comprobar que cada API Key tiene su propio contador independiente.

---

## 17. Azure AI Search

**Fuente:** rama `azure-search` · `azure-search/datasheet/`  
**URL de referencia:** https://azure.microsoft.com/en-us/pricing/details/search/

### Límites documentados

#### Tier Free

| Tipo | Valor |
|------|-------|
| Almacenamiento | 50 MB |
| Índices | Máx. 3 |
| Documentos por índice | Máx. 1 000 |
| Rate limit (operaciones de gestión) | 200 req/min |
| Límite de instancias por suscripción | 1 |

#### Tier Basic

| Tipo | Valor |
|------|-------|
| Almacenamiento | 15 GB/partición (hasta 45 GB con 3 particiones) |
| Índices | Máx. 15 |
| Rate limit (operaciones de gestión) | 200 req/min |
| Escalado | Hasta 9 Search Units |

#### Tiers Standard (S1/S2/S3) y Storage Optimized (L1/L2)

Capacidad escalable en función de las Search Units (SU); el throughput (QPS) escala linealmente con el número de réplicas.

### Pruebas recomendadas

1. **Verificar el límite de almacenamiento (Free):** indexar datos hasta superar 50 MB y confirmar que se devuelve HTTP 403.
2. **Verificar el límite de índices (Free):** intentar crear un 4.º índice en el tier Free y confirmar el rechazo.
3. **Verificar el límite de 1 000 documentos (Free):** intentar indexar el documento número 1 001 y confirmar el error.
4. **Verificar el rate limit de 200 req/min en gestión:** enviar 201 operaciones de control (crear/actualizar índices) en menos de un minuto y confirmar HTTP 429.
5. **Verificar que el throughput de búsqueda escala con réplicas:** comparar el QPS máximo con 1 réplica vs. 3 réplicas en el tier Basic y confirmar el incremento proporcional.
6. **Verificar el límite de 1 instancia Free por suscripción:** intentar crear un segundo servicio Free en la misma suscripción y confirmar el error.

---

## 18. CurrencyLayer

**Fuente:** rama `currencylayer` · `currencylayer/datasheets/`  
**URL de referencia:** https://currencylayer.com/product

### Límites documentados

| Plan | Cuota mensual | Frecuencia de actualización de datos |
|------|---------------|--------------------------------------|
| Free | 100 req/mes | Diaria |
| Basic | 10 000 req/mes (excedente $0.002998/req) | Horaria |
| Starter | — | — |
| Enterprise | — | — |
| Enterprise Plus | — | — |

**Nota:** al superar la cuota, la API devuelve el código de error 104.

### Pruebas recomendadas

1. **Verificar la cuota mensual del plan Free (100 req):** enviar la petición número 101 en el mismo mes natural y confirmar el error 104.
2. **Verificar el reinicio mensual:** al comenzar el nuevo mes, confirmar que el contador de peticiones se ha reiniciado a 0.
3. **Verificar la frecuencia de actualización de datos (Free: diaria):** comparar los datos de tipo de cambio obtenidos en dos peticiones con menos de 24 horas de diferencia y confirmar que son los mismos (datos cacheados).
4. **Verificar el cargo por excedente (Basic):** superar las 10 000 peticiones y confirmar en el panel de CurrencyLayer que se aplica el cargo $0.002998 por petición adicional.
5. **Verificar la segmentación por API Access Key:** usar dos API Keys distintas y confirmar que cada una tiene su propio contador.

---

## 19. DHL Location Finder

**Fuente:** rama `dhl-location-finder` · `dhl-location-finder/datasheets/`  
**URL de referencia:** https://developer.dhl.com/api-reference/location-finder-unified

### Límites documentados

#### Entorno por defecto (Default)

| Tipo | Valor |
|------|-------|
| Cuota diaria | 500 llamadas/día (rolling 24 h desde la primera petición) |
| Rate limit | 1 llamada/segundo (spike arrest) |
| Cooldown | Inmediato (esperar siguiente segundo para rate limit; siguiente día para cuota) |

#### Entorno Production

| Tipo | Valor |
|------|-------|
| Cuota diaria | Variable (proporcional al volumen de envíos del cliente) |
| Rate limit | Custom (alto throughput previa revisión técnica) |

### Pruebas recomendadas

1. **Verificar la cuota diaria de 500 llamadas:** enviar la llamada número 501 dentro de una ventana de 24 horas desde la primera petición y confirmar HTTP 429 o 403.
2. **Verificar el spike arrest de 1 llamada/segundo:** enviar 2 peticiones simultáneas en el mismo segundo y confirmar que la segunda es rechazada o retrasada.
3. **Verificar el reinicio de la ventana rolling de 24 h:** tras recibir el error de cuota, esperar que transcurran 24 h desde la primera petición del día y confirmar que el acceso se restablece.
4. **Verificar la segmentación por API Key:** comprobar que el límite aplica por API Key vinculada a la aplicación en el portal de desarrolladores DHL.

---

## 20. Twilio SendGrid Email API

**Fuente:** rama `sendgrid-email-twilio` · `sendgrid-email-twilio/datasheet/`  
**URL de referencia:** https://sendgrid.com/en-us/solutions/email-api/pricing

### Límites documentados

| Plan | Cuota de emails | Webhooks |
|------|----------------|----------|
| Free Trial | 100 emails/día (rolling) · 60 días de duración | 1 webhook |
| Essentials | Variable según volumen contratado | — |
| Pro | 2 500 000 emails/mes + 2 500 validaciones | 5 webhooks |
| Premiere | Plan de gran volumen | — |

### Pruebas recomendadas

1. **Verificar la cuota diaria de 100 emails (Free Trial):** intentar enviar el email número 101 en el mismo día y confirmar el rechazo.
2. **Verificar la duración del trial de 60 días:** intentar enviar un email una vez caducado el período de prueba y confirmar que se requiere cambio de plan.
3. **Verificar el límite de 1 webhook (Free Trial):** intentar crear un segundo event webhook y confirmar el rechazo.
4. **Verificar las 2 500 validaciones de email incluidas en Pro:** consumir las 2 500 validaciones y confirmar que las siguientes generan cargo adicional.
5. **Verificar el historial de actividad de email:** en Free Trial (3 días de historial) y en Pro (7 días), comprobar que no se pueden consultar actividades anteriores al período correspondiente.

---

## 21. MailerSend

**Fuente:** rama `mailersend` / rama actual · `new_datasheets/mailersend.yaml`  
**URL de referencia:** https://www.mailersend.com/pricing

### Límites documentados

#### Cuotas

| Plan | Emails/mes | Req/día (API general) | SMS/mes |
|------|-----------|----------------------|---------|
| Free | 500 | 100 | — |
| Hobby | 5 000 (excedente $1.20/1 000) | 1 000 | — |
| Starter | 50 000 (excedente $0.95/1 000) | 100 000 | 100 (excedente $1.40/100) |
| Professional | 50 000 (excedente $0.95/1 000) | 500 000 | 150 (excedente $1.40/100) |

#### Rate limits

| Endpoint | Reputación saludable | Reputación en revisión |
|----------|---------------------|------------------------|
| `v1/email` | 120 req/min | 10 req/min |
| `v1/bulk` | 10 req/min | 1 req/min |
| Todos los demás (`/*`) | 60 req/min | 60 req/min |

### Pruebas recomendadas

1. **Verificar el rate limit general de 60 req/min (`/*`):** enviar 61 peticiones en menos de un minuto a cualquier endpoint y confirmar HTTP 429.
2. **Verificar el rate limit de email con reputación saludable (120 req/min):** enviar 121 peticiones al endpoint `v1/email` en menos de un minuto y confirmar el bloqueo.
3. **Verificar el rate limit reducido con reputación en revisión (10 req/min en `v1/email`):** simular el estado de cuenta en revisión y confirmar que el límite baja a 10 req/min.
4. **Verificar la cuota diaria de requests (100 req/día en Free):** enviar la petición número 101 en el mismo día UTC y confirmar el rechazo.
5. **Verificar la cuota mensual de emails (500 en Free):** enviar el email número 501 en el mismo mes y confirmar el error.
6. **Verificar el excedente de emails (Hobby):** superar las 5 000 emails/mes y confirmar el cobro de $1.20/1 000 emails adicionales.

---

## 22. Resend

**Fuente:** rama actual · `new_datasheets/resend.yaml`  
**URL de referencia:** https://resend.com/pricing

### Límites documentados

#### Rate limits

| Plan | Rate limit |
|------|------------|
| Free | 2 req/segundo (pool por equipo) |
| Pro | 2 req/segundo |
| Scale | 2 req/segundo |
| Enterprise | Custom |

**Límites adicionales:** máximo 100 emails por lote (`/emails/batch`); máximo 50 destinatarios por email individual.

#### Cuotas mensuales

| Plan | Emails/mes | Cuota diaria (solo Free) | Contactos marketing |
|------|-----------|--------------------------|---------------------|
| Free | 3 000 | 100 emails/día | 1 000 |
| Pro | 50 000 (excedente $0.90/1 000) | — | 5 000 |
| Scale | 100 000 (excedente $0.90/1 000) | — | 5 000 |

#### Límites de infraestructura

| Recurso | Free | Pro | Scale |
|---------|------|-----|-------|
| Dominios verificados | 1 | 10 | 1 000 |
| Endpoints webhook | 1 | 10 | — |
| Retención de logs | 1 día | 3 días | 7 días |

### Pruebas recomendadas

1. **Verificar el rate limit de 2 req/segundo:** enviar 3 peticiones en menos de 1 segundo y confirmar HTTP 429.
2. **Verificar el límite de 100 emails por lote:** enviar una petición a `v1/emails/batch` con 101 objetos de email y confirmar el rechazo.
3. **Verificar el límite de 50 destinatarios por email individual:** enviar un email con 51 destinatarios (entre `to`, `cc` y `bcc`) y confirmar el error.
4. **Verificar la cuota diaria de 100 emails (Free):** enviar el email número 101 en el mismo día UTC y confirmar el bloqueo.
5. **Verificar la cuota mensual (Free: 3 000 emails):** enviar el email número 3 001 en el mismo mes y confirmar el error.
6. **Verificar el límite de dominios verificados (1 en Free):** intentar verificar un segundo dominio con una cuenta Free y confirmar el rechazo.
7. **Verificar la retención de logs:** en el plan Free (1 día), confirmar que no se pueden consultar eventos de email de hace más de 1 día.
8. **Verificar el excedente de emails (Pro/Scale):** superar la cuota mensual incluida y confirmar el cobro de $0.90/1 000 emails adicionales.

---

*Documento generado a partir de los datasheets disponibles en las ramas del repositorio a fecha 2026-03-05.*
