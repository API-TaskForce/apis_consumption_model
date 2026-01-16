# Dudas durante la transformación a YAML - SoundCloud

Este documento registra dudas surgidas durante el modelado del datasheet de SoundCloud.

## Endpoint-Specific Rate Limits: Policies vs Segmentation
**Pregunta:** Si un Rate Limit aplica únicamente a un endpoint concreto (como `/tracks/:id/stream`), ¿dónde se debe explicar esa restricción: en `Policies` o en `Segmentation`?

**Contexto:** SoundCloud aplica un límite de 15,000 requests/day específicamente para "Play Requests".

**Posible Solución:**
Explicarlo en `Policies` parece describir mejor la naturaleza de la regla ("Solo aplica a play requests"). `Segmentation` suele referirse más al ámbito de aplicación del contador (por Client ID, por IP, por Token), aunque si el contador es exclusivo para ese endpoint, podría considerarse un segmento funcional.
