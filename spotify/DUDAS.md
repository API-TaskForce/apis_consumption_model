# Dudas y Decisiones de Modelado - Spotify Web API

## 1. Coste por Request (Per Request Cost)
**Duda:** La documentación no especifica un precio por llamada API.
**Decisión:** Se ha modelado con `perRequestCost: 0.00`.
**Razón:** Spotify Web API opera tradicionalmente bajo un modelo gratuito para desarrolladores/partners standard. El coste es el acceso restringido (Development Mode) hasta que se cumple con los requisitos para escalar.

## 2. Recarga de Capacidad (AutoRecharge)
**Duda:** ¿Cómo definir la recarga de la cuota en una "rolling window"?
**Decisión:** Se ha establecido `autoRecharge: dynamic`.
**Razón:** Al ser una ventana deslizante de 30 segundos, la capacidad no se reinicia en una fecha fija (como el día 1 del mes), sino que se libera continuamente segundo a segundo conforme pasan los 30 segundos desde las peticiones anteriores.

## 3. Barreras de Entrada (Extended Quota Mode)
**Nota Importante:** Aunque técnicamente es un "Nivel Superior" de API, se ha documentado específicamente en `extraFields` los requisitos de negocio (250k MAUs, Empresa Registrada).
**Impacto:** Esto significa que para la gran mayoría de desarrolladores indies o proyectos pequeños, el límite real es el "Development Mode" (25 usuarios máx), no solo el Rate Limit técnico.

## 4. Rate Limits: Standard vs High
**Duda:** ¿Cuáles son los valores exactos (RPS/QPS) para "Standard Rate Limit" y "High Rate Limit"?
**Respuesta:** Spotify **no publica cifras oficiales** (ej. 10 Requests/segundo).
**Decisión:** Se usan los términos cualitativos "Standard" y "High".
*   **Standard (Dev Mode):** Límite base restringido, diseñado para pruebas y pequeño uso privado. Calculado sobre ventana de 30s.
*   **High (Extended Mode):** La documentación oficial cita textualmente "much higher than apps in development mode". Se asume que escala según el volumen de la aplicación aprobada (>250k MAUs).
