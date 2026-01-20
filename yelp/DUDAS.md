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
