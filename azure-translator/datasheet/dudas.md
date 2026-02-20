# Dudas - Azure Translator API

## Tiers deprecados (S2, S3, S4)

Los tiers S2, S3 y S4 han sido **deprecados** por Microsoft. Se mencionan en la documentación de service limits con sus límites horarios (40M, 120M, 200M chars/hour respectivamente), pero en la página de pricing se indica:

> "Please note that Instances S2, S3, and S4 instances have been deprecated. For discounted rates on standard translation, please use Commitment Tiers."

No se han creado datasheets separadas para estos tiers ya que están obsoletos.

## Commitment Tiers vs Instance Tiers (C2-C4)

Existen dos mecanismos de descuento por volumen:
- **S1 Commitment Tiers**: Descuentos en Standard Translation (no aplican a Custom Translation).
- **C2-C4 Instance Tiers**: Descuentos en Custom Translation (no aplican a Standard Translation).

Para obtener descuento en ambos tipos, se necesitan dos recursos Azure separados (uno S1 con commitment tier + otro C2-C4). No se han creado datasheets separadas para los commitment tiers al tratarse de variaciones de descuento sobre el mismo S1.

## Modelo de cobro basado en caracteres, no en requests

A diferencia de la mayoría de APIs REST, Azure Translator **no cobra por petición** sino **por carácter traducido**. Esto implica que el concepto de "rate limit" en esta API se mide en caracteres/hora en lugar del habitual requests/segundo o requests/minuto. El campo `maxPower` en la datasheet refleja este modelo usando characters/hour.

## Información adicional incluida: precios y límites de tamaño de archivos

Se han añadido a las datasheets datos que van más allá del rate limit habitual, como los **precios por operación** (text translation, document translation, custom translation) y los **límites de tamaño de archivos** (peso máximo de documentos, número de ficheros por batch, tamaño de glossarios, etc.).

Aunque estos campos no forman parte del modelo de rate limiting clásico (requests/segundo), resultan especialmente relevantes en esta API porque:

- El cobro se basa en **caracteres**, no en peticiones, por lo que entender el coste por carácter es clave para estimar el consumo real.
- Los límites de **tamaño de archivo** (40 MB async, 10 MB sync, 250 MB por batch) y **número de documentos** (1.000 por batch) condicionan directamente cómo se diseña la integración con la API.
- Estos límites complementan al rate limit horario y, en la práctica, son igualmente restrictivos para el consumidor de la API.
