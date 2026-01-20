## Introducción
Parece que la **API DHL Location Finder Unified** no sigue un modelo de suscripción SaaS tradicional (donde pagas una cuota mensual por tramos), sino que utiliza un modelo de acceso por niveles basado en la relación comercial.

1. Nivel Exploratory (Datasheet Default)

- Propósito: Evaluación técnica y desarrollo (Sandbox).

- Justificación: DHL otorga este nivel de forma automática a cualquier desarrollador registrado. El límite de 500 llamadas/día y 1 req/seg actúa como un *"freno de seguridad"* para evitar el **uso indebido o el scraping masivo sin una relación comercial establecida**.

- Comportamiento técnico: Es una infraestructura compartida con prioridad baja. Si se excede el límite, el bloqueo es automático (Error 429).

---

2. Nivel Approved (Datashet Prod)
Propósito: Uso operativo en aplicaciones de producción.

- Justificación: Para pasar a este nivel, *DHL realiza una revisión técnica y comercial*. Aunque sigue costando $0, el "pago" es indirecto: **debes ser cliente de DHL (tener un Shipping Account activo)**.

- Comportamiento técnico: Los límites de ráfaga (burst) son significativamente más altos para permitir procesos logísticos en tiempo real (ej. un checkout de un e-commerce con miles de pedidos).