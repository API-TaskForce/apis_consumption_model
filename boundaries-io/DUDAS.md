# Dudas

**Rate Limit en Planes Superiores (Pro, Ultra, Mega)**

En el plan BASIC existe un `hourlyRateLimit` de 1000/hora. En los planes superiores este límite no aplica (es ilimitado).

Mi duda es qué enfoque es el correcto para el modelado:
1.  **Usar `.inf`**: Definir el límite explícitamente como infinito (`value: .inf`).
2.  **Usar `false`**: Deshabilitar la feature (`rateLimit: false`).

**Problema observado:**
Si lo dejo como `false` (feature deshabilitada), me preocupa que la herramienta de visualización siga mostrando el valor por defecto del usage limit (1000) aunque la feature esté apagada, o si existe una forma más limpia de ocultarlo completamente sin arrastrar el valor default.