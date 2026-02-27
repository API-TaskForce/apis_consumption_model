# Dudas y Decisiones de Modelado - Foursquare Places API

## 1. Modelado de "Limit Premium Endpoints"
**Pregunta:** ¿De dónde sale `limitPremiumEndpoints` si no hay un límite explícito en la documentación?

**Decisión:**
Se ha introducido `limitPremiumEndpoints` como una construcción lógica en el modelo `Pricing2YAML` para representar correctamente la facturación de los endpoints Premium (Tips, Photos, Rich Attributes).

*   **Razón:** A diferencia de los "Pro Endpoints" que tienen un tier gratuito de 0 a 10,000 llamadas, los "Premium Endpoints" **se cobran desde la primera llamada** (Tier 1: 0-100,000 a $18.75 CPM).
*   **Implementación:** Se define un límite de uso (`usageLimits`) con un valor por defecto de **0**.
*   **Efecto:** Esto fuerza a que cualquier consumo de endpoints Premium active inmediatamente los "Add-ons" de pago (`extraPremiumCallsTierX`), reflejando así el modelo de precios "Pay-as-you-go" sin franquicia gratuita para esta categoría.
