# API Datasheets — v0.1

> Esquema de trabajo para modelar el consumo de APIs, combinando **Pricing2YAML** y **Datasheet YAML**.

---

## 1. Objetivo

Al modelar cualquier API (comercial o no) necesitamos producir hasta **dos documentos** complementarios:

| Documento | ¿Qué describe? | ¿Cuándo aplica? |
|---|---|---|
| **Pricing2YAML** | Planes, precios, features y add-ons del SaaS | Siempre que la API sea comercial / tenga planes |
| **Datasheet YAML** | Cuotas, caudales, costes por petición y políticas de consumo a nivel de endpoint | Siempre (comercial o no) |

Ninguno de los dos, por separado, representa el 100% del modelo de consumo, pero juntos proporcionan un esqueleto sólido para comenzar a iterar.

---

## 2. Clasificación de APIs

Antes de empezar, identifica en qué caso se encuentra la API que quieres modelar:

### Caso A — SaaS puro (la API es una feature más)

La API es una funcionalidad secundaria dentro de un producto SaaS completo.  
**Ejemplo:** [GitHub](https://github.com/pricing).

- El **Pricing2YAML** es el documento principal — el lenguaje está diseñado para estos casos.
- La **Datasheet** complementa con rate limits, cuotas de API y políticas de consumo.

**Fuentes para el Pricing2YAML:**

1. [SPHERE](https://sphere.score.us.es/) — repositorio con muchos SaaS ya modelados.
2. [A-MINT / HARVEY](https://github.com/isa-group/Pricing-Intelligence-Interpretation-Process) — extracción automática desde URL.
3. Escritura manual siguiendo la [documentación de Pricing2YAML](https://sphere-docs.vercel.app/docs/2.0.1/category/pricing2yaml).

> **Recomendación:** Empieza por la opción 3 para familiarizarte con el lenguaje; después valida con SPHERE o A-MINT.

### Caso B — API-First / Partial SaaS (la API es el producto principal)

El proveedor vende la API como servicio principal, pero la empaqueta en planes tipo SaaS. El frontend suele reducirse a un dashboard de control.  
**Ejemplos:** [Mailersend](https://mailersend.com/pricing), [Resend](https://resend.com/pricing), [Dailymotion](https://pro.dailymotion.com/en/pricing/).

- El **Pricing2YAML** modela los planes y precios. Las características de consumo (rates, cuotas, sobrecoste) se incluyen como features del Pricing.
- La **Datasheet** detalla el comportamiento real a nivel de endpoint.
- **Es esperable** que algunos campos se repitan en ambos documentos — no es problema.

### Caso C — API no comercial (sin pricing)

La API no se ofrece vía pricing.  
**Ejemplos:** [DBLP](https://dblp.org/faq/How+to+use+the+dblp+search+API.html), [Quotable](https://github.com/lukePeavey/quotable).

- **No aplica Pricing2YAML.**
- Solo se modela la **Datasheet**, que probablemente tendrá pocos valores o será difícil de completar.

---

## 3. Esquema de la Datasheet YAML

La Datasheet se estructura en **cuatro bloques principales**:

```
┌─────────────────────────────────────┐
│  Cabecera (metadata del servicio)   │
├─────────────────────────────────────┤
│  capacity   → Cuotas (volumen)      │
├─────────────────────────────────────┤
│  max_power  → Caudales (rate limit) │
├─────────────────────────────────────┤
│  plans      → Asignación por plan   │
│    └ endpoints → Detalle por ruta   │
├─────────────────────────────────────┤
│  metadata   → Fuentes y provenance  │
└─────────────────────────────────────┘
```

### 3.1 Cabecera

| Campo | Descripción | Valores posibles |
|---|---|---|
| `associated_saas` | Nombre del servicio | Texto libre |
| `type` | Clasificación según § 2 | `full_saas` · `partial_saas` · `api_first` |
| `url` | URL de referencia del pricing / documentación | URL |
| `syntax_version` | Versión del esquema | `0.1` |
| `date` | Fecha de extracción | `YYYY-MM-DD` |

### 3.2 `capacity` — Cuotas (Volumen)

Define los **límites de volumen** que se asignan posteriormente a los planes. Cada cuota es un bloque con clave descriptiva.

| Campo | Descripción | Ejemplo |
|---|---|---|
| `value` | Valor numérico (o `custom` si es negociable) | `50000` |
| `unit` | Unidad de medida | `requests` · `emails` · `sms` · `records` · `gb` · `tokens` |
| `period` | Ventana temporal | `1 month` · `1 day` · `1 hour` |
| `window_type` | Tipo de reinicio | `rolling` · `fixed_utc_midnight` · `fixed_billing_cycle` |
| `overage_cost` | _(opcional)_ Coste por exceso | Objeto con `price` y `value` (o `unit`) |

### 3.3 `max_power` — Caudales (Rate Limit)

Define los **límites de frecuencia** (velocidad de acceso). Misma estructura que `capacity`, sin `overage_cost`.

| Campo | Descripción | Ejemplo |
|---|---|---|
| `value` | Peticiones permitidas por ventana | `120` |
| `unit` | Unidad | `requests` · `records` |
| `period` | Ventana temporal | `1 min` · `1 second` · `1 hour` |

### 3.4 `plans` — Planes

Cada plan agrupa un precio, un periodo de facturación y la asignación concreta de cuotas/caudales a nivel global y por endpoint.

| Campo | Descripción |
|---|---|
| `price` | Precio del plan (numérico o `custom`) |
| `billing_period` | Periodo de facturación (`1 month`) |
| `quota` | _(opcional)_ Cuota global del plan (referencia a clave de `capacity`) |
| `rate` | _(opcional)_ Caudal global del plan (referencia a clave de `max_power`) |
| `endpoints` | Mapa de endpoints con detalle individual |

**Cada endpoint** puede contener:

| Campo | Descripción |
|---|---|
| `rate` | Referencia a un caudal definido en `max_power` |
| `quota` | Referencia a una cuota definida en `capacity` |
| `cost_per_request` | _(opcional)_ Descripción del coste por petición en texto libre |

### 3.5 `metadata` — Fuentes y Provenance

| Campo | Descripción |
|---|---|
| `sources_visited` | Lista de URLs consultadas |
| `data_provenance` | Lista de afirmaciones clave extraídas, con referencia a la fuente entre corchetes cuando sea posible |

---

## 4. Convenciones de nombres y formatos

> Estas reglas están documentadas en detalle en `glosario_tecnico.txt` y `diccionario_api_semantica.txt`.

### Naming convention para claves

```
[tipo]_[recurso]_[contexto]_[plan]
```

| Segmento | Valores típicos | 
|---|---|
| `tipo` | `quota` · `rate` |
| `recurso` | `email` · `sms` · `requests` · `bandwidth` · `storage` · `encoding` |
| `contexto` | `healthy` · `under_review` · `bulk` · `search` · `global` · `transactional` |
| `plan` | `free` · `hobby` · `starter` · `pro` · `enterprise` |

> **Regla:** NUNCA incluir valores numéricos en el nombre de la clave (usar `rate_bulk_pro`, no `rate_30_min`).

### Formatos estándar

- **`period`:** Siempre `[número] [unidad]` → `1 min`, `1 day`, `1 month`, `100 ms`.
- **`window_type`:** `rolling` | `fixed_utc_midnight` | `fixed_billing_cycle`.
- **`unit`:** `requests` · `emails` · `sms` · `tokens` · `records` · `gb` · `hours` · `minutes`.

---

## 5. Template (esqueleto vacío)

Copiar este fichero como punto de partida al crear una nueva Datasheet:

```yaml
associated_saas: # Nombre del servicio
type: # full_saas | partial_saas | api_first
url: # URL de referencia
syntax_version: 0.1
date: # YYYY-MM-DD

capacity:
  # Definir aquí todas las cuotas (volumen) que luego se referencian en los planes.
  # Nombre siguiendo la convención: quota_[recurso]_[contexto]_[plan]
  #
  # quota_ejemplo:
  #   value:
  #   unit:
  #   period:
  #   window_type:
  #   overage_cost:        # opcional
  #     price:
  #     value:              # o unit

max_power:
  # Definir aquí todos los caudales (rate limits) que luego se referencian en los planes.
  # Nombre siguiendo la convención: rate_[recurso]_[contexto]_[plan]
  #
  # rate_ejemplo:
  #   value:
  #   unit:
  #   period:

plans:
  # Un bloque por cada plan del proveedor.
  #
  # nombre_plan:
  #   price:
  #   billing_period:
  #   quota:               # opcional — cuota global del plan
  #   rate:                # opcional — caudal global del plan
  #   endpoints:
  #     ruta/del/endpoint:
  #       rate:            # referencia a clave de max_power
  #       quota:           # referencia a clave de capacity
  #       cost_per_request: # opcional — descripción libre

```

## 6. Consejos adicionales

- Al modelar **Pricing2YAML**, podéis obviar los campos `expression`, `serverExpression` y, en caso de duda, poner `DOMAIN` en el campo `type` de las Features.
- Si una característica aparece tanto en Pricing2YAML como en la Datasheet, **no es ningún inconveniente** — es esperable en APIs de tipo Partial SaaS / API-First.
- Se genera **una Datasheet por cada plan** del proveedor, facilitando la comparación de los diferentes casos de consumo.
- El esquema es **preliminar (v0.1)**: hay APIs donde habrá que adaptar o extender los campos según lo que se descubra en la documentación.
- Documentad siempre las **fuentes** en `metadata` — es clave para la trazabilidad y para futuras revisiones.

---
