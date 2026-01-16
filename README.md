# Pricing2YAML + Datasheet

Al modelar cualquier API (comercial o no), tenemos que acudir a estos dos documentos:

1. Pricing2YAML
2. Datasheet

Ninguno de los dos en su estado actual es capaz de representar al 100% el modelo de consumo de una API, pero sí que proporcionarán un esqueleto básico para ello.

A continuación se explica cómo se construyen y deben organizarse todos los proveedores que vamos a estudiar:

1. Elegir qué API queremos representar, existe en la literatura un dataset muy estudiado y utilizado, disponible en este [repositorio](https://github.com/isa-group/2023-paper-pricing-modeling-framework-for-apis-dataset/tree/main/models). Adicionalmente, si encontráseis alguna API interesante también podéis usarla.
2. Cuando se elija una API podemos encontrarnos en 3 casos diferentes (a priori):

   - La API corresponde a un SaaS al 100%, y la API es una simple `feature` para los desarrolladores. Un ejemplo claro de esto sería [**Github**](https://github.com/pricing). En este caso, estamos hablando de que el proveedor ofrece, en esencia, un SaaS. Aquí es donde Pricing2YAML brilla, ya que está pensado específicamente para estos casos de SaaS. Cuando nos encontremos en este caso, podemos acudir a 3 vías para extraer el Pricing2YAML:
     - Usar el repositorio [SPHERE](https://sphere.score.us.es/), donde ya tenemos muchos SaaS disponibles.
     - Usar A-MINT, una API disponible en [HARVEY](https://github.com/isa-group/Pricing-Intelligence-Interpretation-Process) que extrae un Pricing2YAML a partir de una URL.
     - Usar vuestra inteligencia natural para extraerlo, a partir de la documentación disponible [aquí](https://sphere-docs.vercel.app/docs/2.0.1/category/pricing2yaml)
       > Nota: Si os encontráis en este caso, intentad hacer el esfuerzo de acudir a esta última opción contrastándola con las anteriores. Cuando os encontréis comodos con el lenguaje podréis acudir a la primera o a la segunda opción.
   - La API es realmente el servicio que el proveedor ofrece a los desarrolladores, es su producto principal. Sin embargo por el paradigma actual acostumbran a ofrecerlo vía Pricing como si fuera un SaaS, sin embargo la API es realmente la principal funcionalidad, dejando el Frontend con características reducidas o a un simple Dashbord de control. Casos de este tipo de proveedores serían APIs de correo como [**Mailersend**](https://mailersend.com/pricing) o [**Resend**](https://resend.com/pricing). En este caso, modelar el Pricing2YAML es un poco más complejo, ya que habrá características que se pueden asociar más a consumo de API (Datasheet), que a una característica del Pricing (Pricing2YAML).
     - Un ejemplo claro sería cuando en el Pricing aparecen características como, `Rate` o `Cuota`. Aquí, el consejo que damos es que os limitéis a representar el Pricing en Pricing2YAML, modelando `rates`, `cuotas` o `costes por exceso` como features del Pricing. Si alguna característica se repitiera en la Datasheet y en el Pricing, no sería ningún inconveniente.
   - El último caso que os podéis encontrar es cuando la API no es **comercial**, es decir, que no se ofrece vía Pricing. Ejemplos de este tipo de APIs serían la API de [**DBLP**](https://dblp.org/faq/How+to+use+the+dblp+search+API.html) o alguna API que se ofreza vía Github como [**Quotable**](https://github.com/lukePeavey/quotable). En estos casos, el Pricing2YAML no es aplicable y lo único que se pide es modelar la Datasheet, la cual en el mayor de los casos será complicada de modelar o tendrá muy pocos valores.

3. Una vez extraido el Pricing2YAML, pasamos a extraer la Datasheet, la cual no tiene aún una estructura definida pero que se puede modelar siguiendo el siguiente esquema:

```yaml
## Campos (explicación breve)

- **Associated SaaS:** Nombre del servicio externo y URL de referencia.
  - **Type:** `Full SaaS` / `Partial SaaS` / `API-First` (según aplique).
- **Capacity (Quota):** Límite temporal (ej. 100.000 emails/month). Indicar ventana (mensual/diaria/hora).
  - **Auto-Recharge:** Período de recarga del límite (monthly, daily, etc.).
  - **Extra Charge:** Coste adicional por exceso de uso (ej. $0.90/1.000 emails).
- **Max Power (Rate Limit):** Límite de tasa de peticiones.
  - **Value:** Número de peticiones permitidas por ventana (ej. 2 requests/second).
  - **Throttling:** Tipo de throttling aplicado (sliding window, token bucket, fixed window, etc.).
  - **Policies:** Políticas adicionales de rate limiting.
- **Per-Request Cost:** Coste por petición en créditos/unidades (si las peticiones varían en función de los créditos consumidos).
- **Cooling Period:** Tiempo de enfriamiento / backoff recomendado. Indicar si existe cabecera Retry-After en respuestas 429.
- **Segmentation:** Segmentación de límites por diferentes niveles y qué limita (API Key, tenant, endpoint, etc.)
  - Nota: La segmentación puede variar entre rate limit y quota. Por ejemplo, un endpoint puede tener cuota diferente pero compartir el mismo rate limit a nivel de cuenta.
- **Shared Limits:** Si hay límites compartidos entre endpoints/tenants/API keys.
- **Extra Fields:** Cualquier descubrimiento adicional que no encaje en los campos anteriores.

---

## API Template

- **Associated SaaS:** [Nombre del SaaS](URL)
  - Type: TBD (Full SaaS / Partial SaaS / API-First)
- **Capacity (Quota):**
  - Value: TBD
  - Auto-Recharge: TBD
  - Extra Charge: TBD
- **Max Power (Rate Limit):**
  - Value: TBD
  - Throttling: TBD
  - Policies: TBD
- **Per-Request Cost:** TBD
- **Cooling Period:** TBD
- **Segmentation:**
  - Rate Limit Level: TBD
  - Quota Level: TBD
- **Shared Limits:** TBD
- **Extra Fields:** TBD

```

    Este es un esquema muy preliminar, aplicable a algunas APIS, pero para otras tendréis que intentar adaptarlo como podáis. Hemos decidio que se realizará una Datasheet para cada **PLAN** que se ofrezca. Es decir en el caso de GitHub tendréis que ver los casos de uso de cada plan (configuraciones) y extraer Datasheet para este caso. Habrá campos que se repitan, pero va a ser más sencillo ver los diferentes casos de consumo y si los proveedores hacen distinción entre ellos.

4. Crear una carpeta en este repositorio y alojar ambos archivos cuando se analicen. También se pueden dejar evidencias de capturas de pantalla para alguna información útil o extra que hayáis encontrado y que esté relacionada. (Por ejemplo, una captura al Pricing, a el rate limit, alguna política de consumo... etc.)

### Consejos adicionales.

- Al modelar Pricing2YAML podéis obviar los campos `expression` y `serverExpression` y el campo `type` de las Features también puede ser complicado, en caso de duda poned **DOMAIN**.

- Los Pricings se van subiendo a SPHERE, ejemplo de [Resend](https://sphere.score.us.es/pricings/rgavira/Resend?collectionName=undefined)