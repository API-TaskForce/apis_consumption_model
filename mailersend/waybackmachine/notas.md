## Notas

El pricing de mailersend técnicamente lleva definido desde 2013, pero este presenta un formato de pricing personalizado a medida

![alt text](mailersend-weayback-pricing2013.png)

En el wayback machine tenemos constancia desde 2013, pero este formato de pricing no es válido para nuestro estudio.

Sin embargo mailersend aplica una versión temprana de un pricing adaptable a nuestro estudio en 2014.

![alt text](mailersend-wayback-pricing2014.png)

Este pricing si aplica.

## Notas sobre las datasheets

A la hora de determinar las datasheets, es un proceso complicado corroborar si existen límites aplicables para nuestro estudio. La wayback machine no permite una navegación real entre páginas, por lo que buscar y corroborar rate limits no aplica en estos casos.

De momento dejaremos las datasheets con valores TBD.

Ejemplo:
```yml
associatedSaaS: MailerSend
planReference: FREE
type: Partial SaaS
capacity:
  - value: 500 emails
    type: QUOTA
    windowType: CALENDAR_MONTH
    resetCondition: "First day of the calendar month"
    extraCharge: "None (Hard Limit)"
  - value: 100 requests
    type: QUOTA
    windowType: ROLLING_24H
    throttling: "Daily quota"
maxPower:
  value: 60 requests per minute (https://www.mailersend.com/help/rate-limits-how-to-reduce-403-422-429-errors)
  type: RATE_LIMIT
  description: "Shared infrastructure burst limit"
segmentation:
  - Key Level: Limits applied per MailerSend account API key.
```