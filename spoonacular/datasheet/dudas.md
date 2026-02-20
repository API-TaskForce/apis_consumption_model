# Dudas - Spoonacular Food API

## Plan "Starter" mencionado en Rate Limiting pero inexistente en Pricing

En la sección de [Rate Limiting & Quotas](https://spoonacular.com/food-api/docs#Quotas) de la documentación oficial se listan los siguientes niveles de rate limit:

- **Free:** 60 requests in 1 minute
- **Starter:** 120 requests in 1 minute
- **Cook:** 5 requests per second
- **Culinarian:** 10 requests per second
- **Chef:** 20 requests per second

Sin embargo, en la [página de pricing](https://spoonacular.com/food-api/pricing) y en la FAQ solo se mencionan **5 planes**: Free, Cook, Culinarian, Chef y Enterprise. El plan **"Starter"** no aparece como opción de suscripción.

Posibles explicaciones:
- Plan descontinuado que sigue referenciado en la documentación de rate limits.
- Plan disponible únicamente a través de marketplaces de terceros (RapidAPI, APILayer) con nomenclatura diferente.
- Tier intermedio entre Free y Cook no visible en la página principal de pricing.

**No se ha creado datasheet para el plan Starter** al no poder verificar su existencia ni sus condiciones concretas desde la documentación oficial.

---

## Discrepancia en el Rate Limit del plan Free: ¿por minuto o por segundo?

En la sección de [Rate Limiting](https://spoonacular.com/food-api/docs#Quotas) de la documentación se indica:

> **Free:** 60 requests in 1 minute

Sin embargo, en la [página de pricing](https://spoonacular.com/food-api/pricing) el rate limit del plan Free aparece como:

> **1 request/second**

Ambos valores son numéricamente equivalentes (60 req/min ≈ 1 req/s), pero la **ventana de throttling** implícita es diferente:

- **60 req/min** → podría permitir ráfagas (e.g. 60 peticiones en los primeros segundos y luego esperar el resto del minuto).
- **1 req/s** → implica un límite estricto por segundo, sin posibilidad de ráfagas.

Es importante clarificar cuál es el comportamiento real, ya que afecta directamente al tipo de throttling (Fixed Window por minuto vs. Fixed Window por segundo) y al modelado en la datasheet.
