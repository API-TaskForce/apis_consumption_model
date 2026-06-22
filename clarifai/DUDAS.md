# Dudas durante la transformación a YAML - Clarifai

Este documento registra dudas surgidas durante el modelado del pricing de Clarifai.

## Límite de Requests Per Second (RPS) en Enterprise
**Pregunta:** ¿Qué valor exacto poner para `requestsPerSecond` en el plan Enterprise cuando la documentación dice "1000+"?

**Contexto:** En la tabla de precios se indica "1000+" requests per second. YAML requiere un valor numérico o una convención clara.

**Posible Solución:**
Se ha puesto `1000` como valor base más restrictivo ("ponemos el límite más restrictivo" según comentario en el código), pero la notación `.inf` o un valor más alto podría ser más apropiado para Enterprise. Queda pendiente definir si "1000+" implica verdaderamente ilimitado/negociable o un suelo garantizado.

## Campo para Notas y Observaciones de la Documentación
**Observación:** Sería interesante añadir en el datasheet un campo estándar para capturar notas importantes que aparecen en la documentación original.

**Ejemplo:**
"Note: Contact support to request a higher limit." (visto en el límite de gasto mensual del plan Pay-As-You-Go).

**Propuesta:** Capturar estas notas en una sección `notes` o `extraFields` para no perder contexto comercial u operativo importante que no encaja directamente en los campos estructurados numéricos.

## Inclusión de Precios de Hardware Dedicado
**Duda:** ¿Es estrictamente necesario o apropiado incluir el precio de alquiler de nodos dedicados (Compute/Hardware) en el modelo de consumo de la API?
**Contexto:** Clarifai ofrece "Dedicated Node Pricing" (alquiler de GPUs/CPUs por minuto). Aunque es un coste relacionado, es infraestructura más que consumo de API puro (requests). Se ha incluido como `addOns`, pero podría ensuciar el modelo si el foco es puramente API.

## Visualización de Precios Bajos (< $0.01)

**Problema:** Los precios inferiores a 0.01$ (por ejemplo, $0.0098) no se muestran o renderizan correctamente en algunas vistas o herramientas.

**Contexto:** Algunos nodos (ej. NVIDIA A16, AMD EPYC) tienen costes por minuto muy bajos.

**Observación:** Verificar si el sistema de renderizado o la especificación YAML soporta y muestra correctamente hasta 4 decimales o si redondea, lo cual daría precios erróneos (0.00$ en lugar de 0.00XX$).

## Representación de Precios "Contact Us" (Enterprise / Custom)
**Duda:** ¿Cómo representar items como "Model training containers (multi-GPU)" donde el precio es "Contact us"?

**Contexto:** Actualmente no se ha incluido en el YAML porque `price` espera un valor numérico.
**Posible solución:** No incluirlo en `addOns` estructurados si no tiene precio público, o usar un flag/valor especial si el esquema lo permite (ej. `price: -1` o no definir precio y usar descripción).

## Agrupación de Add-ons (Tags/Categorías)
**Sugerencia:** Sería muy útil tener una opción para agrupar `addOns` que sean del mismo tipo o estén relacionados, similar a cómo se usan los `tags` en las `features`.

**Contexto:** En el archivo de precios de Clarifai hay muchos add-ons de diferentes tipos (Inference, Compute/Dedicated Nodes, Training, etc.) y la lista plana se hace difícil de manejar. Poder etiquetarlos o agruparlos visualmente mejoraría la legibilidad y organización.

## Modelado de Alta Personalización en Planes Enterprise
**Observación:** Se ha detectado un nivel muy alto de personalización en los planes Enterprise de Clarifai (Límites de Rate, SLAs, Precios por volumen, Hardware dedicado, etc.), lo cual es difícil de plasmar en un esquema rígido.

**Contexto:** Prácticamente todos los límites (RPS, Quotas) y precios pueden ser negociados o personalizados. Poner valores fijos o "default" puede llevar a confusión sobre la verdadera capacidad del plan.

**Sugerencia de Mejora:** Estudiar campos para indicar explícitamente `negotiable: true`, `customizable: true` o rangos de valores (ej. `15 - 1000+ RPS`) para que el consumidor de la API de precios entienda que los valores listados son solo puntos de partida.