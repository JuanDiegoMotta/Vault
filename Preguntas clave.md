Problemas técnicos y no técnicos a los que me he enfrentado y cómo lo he solucionado.
Técnicos:
- Tiempos de respuesta altos que daban timeout -> Optimización de llamadas a base de datos, en vez de llamar en bucle, hacer una sola llamada en batch y luego iterar en memoria. + Creación de indice en BBDD para reducir el tiempo de consulta en una tabla con muchos registros.
- Código espaguetti ininteligible -> Usar patrón de factoría para evitar los infinitos condicionales nesteados.
- Integraciones con APIs externas de Bbva -> Leer mucha documentación y prueba y error hasta dar con la tecla. Esto sobre todo al principio cuando empecé..