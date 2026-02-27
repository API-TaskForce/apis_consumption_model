# Dudas durante la transformación a YAML - API Football

Este documento sirve para registrar dudas, preguntas y decisiones pendientes sobre la estructura y contenido del archivo `api-football-2026.yml`.

## Feature: Players
**Pregunta:** ¿Cómo se debe representar la feature que agrupa Players, TopScorers, Transfers y Coachs?

**Contexto:** En la web aparecían listados juntos o como un grupo. En el YAML actual está definido como `players`.

**Posible Solución:**
Concatenar los nombres de las features. Y documentar este caso al aparecer seguidos de ",".

```yaml
  playersTopScorersTransfersCoachs:
    valueType: BOOLEAN
    defaultValue: true
    type: DOMAIN
```

## Fuente de Documentación: RapidAPI vs Web Oficial
**Pregunta:** Si una API existe tanto en RapidAPI como en su web oficial (como es el caso de API Football), ¿cuál se debe documentar o cómo se procede?

**Contexto:** Los planes, precios y rate limits pueden diferir entre la versión directa del proveedor y la versión ofrecida a través del marketplace de RapidAPI. En este caso estamos documentando la versión de RapidAPI.

**Posible Solución:**
Si se expresa que la api es sobre rapidapi, entonces se documenta la versión de rapidapi (ya que deberia ser la mas restrictiva y tiene un proposito claro que es facilitar el uso antes de la api directa del proveedor). Si no se expresa, se documenta la versión directa del proveedor.

## Rate Evidence Images: Formato y Naming
**Pregunta:** ¿Las fotos de la "rate evidence" deben tener un formato o nombre específico?

**Contexto:** Se están guardando capturas para justificar los límites, pero no hay convención establecida.

**Posible Solución:**
Podría estandarizarse como `<plan>-<restriction>.png` o similar, y ubicarse en la carpeta `datasheet`.

## Type Definition: Partial SaaS vs API First
**Pregunta:** ¿Cuándo se debe usar `type: Partial SaaS` y cuándo `API First`?

**Contexto:** En el datasheet actual (`basic-datasheet.yml`) se ha definido como `Partial SaaS` (heredado de ejemplos anteriores como Resend), pero API Football es un producto cuyo valor principal es la API en sí misma.

**Posible Solución:**
Definir si `Partial SaaS` se refiere a servicios que son parte de una suite o si `API First` es la denominación correcta para productos que son puramente APIs. Pendiente de aclarar la taxonomía de `type`.

## AutoRecharge Frequency for 'Requests/Day' Limits
**Pregunta:** Si se especifica un límite de "100 requests/day", ¿se debe asumir que el `autoRecharge` es `daily` aunque no esté explícitamente documentado?

**Contexto:** En RapidAPI se indica "100 requests/day" pero no se especifica si es una recarga diaria fija (00:00 UTC) o ventana móvil de 24h.

**Posible Solución:**
La convención habitual es asumir recarga diaria (`autoRecharge: daily`) cuando la unidad es "day", salvo que se especifique lo contrario.
