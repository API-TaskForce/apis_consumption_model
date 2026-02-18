# Notas sobre la Documentación de Bitbucket Cloud API

Este documento registra las dificultades encontradas al modelar los límites de la API de Bitbucket Cloud debido a la naturaleza fragmentada y a veces confusa de la documentación oficial, así como las decisiones tomadas para estructurar los datasheets.

## 1. Puntos de Confusión e Inconsistencias

La documentación de Atlassian presenta varios desafíos para un modelado limpio:

*   **Mezcla de Conceptos de Red vs. API:** La documentación mezcla límites de llamadas REST estándar (JSON) con operaciones de Git (Clone/Push/Pull) que pueden ocurrir vía HTTPS o SSH. Aunque ambos consumen recursos, sus métricas y límites son drásticamente diferentes (1,000/h vs 60,000/h), lo que dificulta definir un "Max Power" único.
*   **Definiciones Vagas sobre "Scaled Limits":** Los límites escalados aplican a un "collective known as the api resource group". La documentación no enumera exhaustivamente qué endpoints pertenecen a este grupo, indicando solo que "están sujetos a cambios". Esto impide modelar con precisión qué llamadas se benefician del escalado y cuáles no.
*   **Reglas Generales vs. Excepciones:** Se establece un límite general de 1,000 peticiones/hora, pero inmediatamente se listan excepciones para "Raw Files", "Invitations" y "Application Properties" que tienen sus propios contadores. No queda 100% claro si estos contadores son *adicionales* al límite global o si el límite global es solo para lo que no está listado explícitamente.
*   **Anonymous Access:** El acceso anónimo está muy restringido (60 req/h) y limitado a datos públicos, pero la documentación sobre el alcance exacto (scope) está dispersa entre varias páginas de soporte y no centralizada en la hoja de límites.

## 2. Decisiones de Modelado

Para aportar claridad y utilidad a los datasheets, se han tomado las siguientes decisiones de diseño:

### A. División en 3 Datasheets Específicos
En lugar de un único archivo gigante lleno de condicionales, hemos separado el modelo en tres ficheros según el **contexto del consumidor**:

1.  **`anonymous-datasheet.yml`:**
    *   **Propósito:** Modelar el acceso más restrictivo y básico (IP-based).
    *   **Decisión:** Se ha limpiado de ruido, dejando solo el límite de 60/h y eliminando referencias a scopes complejos que no aplican.

2.  **`authenticated-datasheet.yml`:**
    *   **Propósito:** Representar el uso estándar de un desarrollador o script con credenciales básicas.
    *   **Decisión:** Se ha mantenido el límite base de 1,000/h como el "Max Power" principal.
    *   **Decisión:** Los límites específicos (Git, Raw Files, Webhooks) se han incluido como `extraFields` (o podrían moverse a una lista de políticas detallada) para indicar que son excepciones a la regla general. Se asume que son contadores independientes.

3.  **`scaled-datasheet.yml`:**
    *   **Propósito:** Modelar el escenario Enterprise/Pago donde los límites son dinámicos.
    *   **Decisión:** Se ha extraído la fórmula de cálculo (`1,000 + (Usuarios - 100) * 10`) como la pieza central de este datasheet, ya que es la diferencia fundamental con el modelo autenticado estándar.

### B. Segmentación
Se ha clarificado explícitamente la segmentación para cada caso:
*   **IP** para anónimos.
*   **User ID** para autenticados estándar.
*   **Workspace/Repository** para los límites escalados (ya que el pago y los usuarios se agregan a nivel de cuenta/workspace).

