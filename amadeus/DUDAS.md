# Dudas y Definiciones - Amadeus API

## 1. Definición de "Split Limit" (Límite Dividido)
En los datasheets `test-datasheet.yml` y `production-datasheet.yml`, hemos usado el término **"Split Limit"** bajo la sección `maxPower`.

**Explicación:**
Esto se refiere a que Amadeus **no aplica un límite de velocidad único** para todas sus APIs. En su lugar, el límite se "divide" o diferencia en dos categorías principales:

1.  **APIs Estándar (The rest of Self-Service APIs):**
    *   Tienen un rendimiento más alto (40 TPS en Prod, 10 TPS en Test).
    *   Permiten paralelismo total en Producción.
    *   Tienen una ventana de *throttling* de 100ms.

2.  **APIs de IA y Partners (AI & Partner APIs):**
    *   Incluye: *Tours and Activities, Prediction, Air Traffic, etc.*
    *   Tienen un límite más estricto y fijo de **20 TPS** tanto en Test como en Producción.
    *   Tienen un *throttling* más agresivo de 1 petición cada 50ms.

**Decisión de Modelado:**
Se usó "Split Limit" para indicar rápidamente que hay que mirar las `policies` o el `throttling` para ver el número exacto según la API que se use.

## 2. Configurabilidad en Enterprise
Se ha marcado `capacity` y `maxPower` como `.inf` (Infinito/Custom) en el plan Enterprise para denotar que estos límites duros de 2000 requests/mes o 40 TPS no aplican por defecto, sino que se negocian contratos a medida.
