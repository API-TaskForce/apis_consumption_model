# Dudas y Decisiones de Modelado - Yelp Places API

## 1. Interpretación de "5,000 Free API Calls"
**Contexto:** La web dice "Receive 5,000 free API calls during the 30-day trial period".
**Decisión:** Se ha modelado un límite específico `trialCalls` que tiene un periodo de **30 días** y no se renueva (`autoRecharge: never` en datasheet).
**Razón:** Estas llamadas NO son mensuales recurrentes. Son un incentivo de una sola vez al inicio.

## 2. Cuota Mensual vs Llamadas Incluidas
**Contexto:** Los planes tienen un precio fijo (ej. Base: $229/mo) y luego dice "plus $5.91 per additional 1,000 API calls".
**Duda:** ¿El precio de $229 incluye alguna llamada gratuita recurrente?
**Decisión:** Se asume que **NO**. Se ha modelado `limitApiCalls` con valor **0**.
**Razón:** El texto usa "plus... per additional", lo que en pricing suele indicar un modelo de tarifa base (acceso/licencia) + consumo variable desde la primera unidad (pay-as-you-go). Por tanto, la cuota mensual paga el acceso a los datos (Atributos, Fotos, Reviews) y las llamadas se pagan aparte vía Add-ons.

## 3. Niveles de Atributos y Filtros en Enhanced
**Duda:** El plan Enhanced dice "Enhanced attributes only" y "Enhanced parameters" pero no especifica "Search Filters" como Premium.
**Decisión:** Se ha mantenido en `Base` para Search Filters en el plan Enhanced, asumiendo que solo Premium tiene "Premium search filters" y Base tiene "Base search filters".

## 4. Features adicionales en Documentación vs Pricing
**Contexto:** En https://docs.developer.yelp.com/docs/plans aparecen más features listadas, pero no aparecen en el pricing.
**Duda:** ¿Se deben añadir también estas features al modelo?
## 5. Discrepancia en Límites de Trial (5,000 total vs 300/día)
**Contexto:** El pricing menciona "5,000 free API calls during the 30-day trial period". Sin embargo, la documentación de Rate Limiting menciona un "Starter Plan" limitado a **300 API calls per 24 hours**.
**Duda:** ¿Coexisten ambos límites? ¿Es el Starter Plan una opción distinta al Trial de los planes de pago?
**Decisión provisional:** Se ha mantenido el modelado de 5,000 en el Trial de los planes `BASE`, `ENHANCED` y `PREMIUM`, pero se ha registrado el límite diario de 300 en la datasheet de Rate Limiting para futuras aclaraciones.
