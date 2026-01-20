# Dudas y Discusión - ZenHub API

## 1. Clasificación de 'Capacity' vs 'Max Power'
En el datasheet `free-datasheet.yml`, hemos definido:
- **Capacity (Quota):** 90s/min por API Key
- **Max Power (Rate Limit):** 90s/min por API Key

**Duda:**
Normalmente, "Capacity" se refiere a una cuota de volumen a largo plazo (ej. mensual, diaria) que se agota y requiere recarga o pago extra. "Max Power" es la velocidad instantánea o a corto plazo (throttling).
Dado que el límite de Zenhub es una ventana rodante de 1 minuto (90 segundos de *processing time* por minuto), técnicamente encaja mejor como un **Rate Limit** (Max Power). Al ponerlo también en Capacity, ¿estamos duplicando la información intencionalmente para indicar que *no hay* una cuota mensual adicional, o deberíamos indicar "Unlimited" en Capacity (como volumen total mensual) y dejar la restricción solo en Max Power?

## 2. API Key vs Token
Hemos unificado el concepto de "Personal API Key" y "Token". Zenhub menciona ambos. Asumimos que el límite es por token generado. Si un usuario genera múltiples tokens, ¿se comparten los límites o son independientes?
*Hipótesis actual:* Límites por Personal API Key (Token).

## 3. Límites REST vs GraphQL
El límite de "90s de processing time" parece específico de GraphQL (o computación compleja). La API REST antigua suele tener límites más simples (ej. 100 req/min).
En el datasheet actual hemos generalizado a "90s/min". ¿Deberíamos distinguir explícitamente REST (req/min) vs GraphQL (processing time) si ambos están en uso?
*Estado actual:* Enfocados en el modelo de GraphQL que parece ser el principal/moderno documentado.
