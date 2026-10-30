# Revisor de medidas DAX para logística

Revisa las medidas de Power BI que mandan en el servicio de tu CEDIS — fill rate, OTIF, rotación, exactitud — y te dice cuáles mienten, por qué, y la fórmula corregida con su prueba. Precio previsto del paquete empresarial: **USD 20**. Es un paquete de instrucciones para un asistente de IA; no incluye software, no se conecta a tu Power BI ni ejecuta DAX.

Su valor está en el oficio de los números: separar un total mal calculado (promedio de porcentajes) de un grano mal definido (OTIF medido por línea en vez de por pedido), de un contexto de filtro heredado que congela las tarjetas, y dejarte la medida corregida lista para pegar más la cuenta a mano que la prueba.

## Uso en 3 pasos

1. **Conéctala:** súbela como skill a tu asistente (en claude.ai: Configuración → Capacidades → Skills → Subir el ZIP) o conéctala desde Agensi. No hay nada que instalar, configurar ni programar.
2. **Pégale tus medidas y datos:** copia el texto de tus medidas DAX, el esquema de tus tablas (nombres, columnas, relaciones) y una muestra de tus datos (tabla o CSV). No compartas contraseñas, cuentas de servicio ni llaves.
3. **Pide la revisión:** «revísame estas medidas y dime cuáles no cuadran y por qué». Recibes la tabla de hallazgos, las medidas corregidas listas para pegar y la prueba numérica; revísala y aplícala tú en Power BI.

Sin datos propios, pide que corra el caso de práctica AN-DAX-01 (abajo).

## Qué recibes

- **Tabla de hallazgos:** `medida | falla | evidencia en la fórmula | efecto en el número | corrección | prioridad`.
- **Medidas DAX corregidas**, completas y listas para pegar en Power BI.
- **La prueba numérica:** valor antes, valor esperado calculado a mano sobre tus datos (o sobre el ejemplo), valor después, y el síntoma de regresión que delata si la corrección se estructura mal otra vez.
- **Lo no verificable marcado como tal**, con el insumo que falta pedir — nunca dado por aprobado.

## Prueba sin datos propios

El paquete trae un caso completo con datos **FICTICIOS de Andén Norte**: seis líneas de pedido donde el «fill rate oficial» de 95.83 % es un promedio de porcentajes (el correcto es 94.87 %), el «OTIF» de 66.67 % está medido por línea (por pedido es 33.33 %), el `Pedidos OTIF` da 0 por un `BLANK` comparado con `== 0` (igualdad estricta), y el segmentador no filtra por una relación invertida. La [entrada](ejemplos/entrada-anden-norte.md) y la [salida esperada](ejemplos/salida-esperada.md) se siguen a mano con una calculadora; pídele a tu asistente que reproduzca la salida antes de usarla con tus datos.

## Qué no hace

- No se conecta a Power BI, no abre `.pbix`, no ejecuta DAX ni valida el despliegue: trabaja con lo que le pegas o adjuntas.
- No certifica tableros ni «aprueba» un número para la junta: entrega evidencia reproducible que tú validas.
- No revisa rendimiento (tiempos de consulta, materialización), RLS ni conectores.
- No revisa reglas fiscales, aduanales ni de autoridad, y no cita ninguna.
- **No sustituye al contador, al agente aduanal ni al SAT:** pre-revisa medidas, cruza contra tus datos y arma evidencia; las decisiones las tomas tú.

Las referencias —documentación oficial de funciones DAX de Microsoft, la guía de precios de Agensi y la ley que cita LICENSE— con URL y fecha de consulta (2026-10-29) están en [references/fuentes.md](references/fuentes.md).

## Estado de esta entrega

Revisada y cerrada por cuatro IA de frontera por separado sobre esta versión. Página: https://heroldo.org/skills/revisor-dax-logistica/ · a la venta en [Agensi](https://agensi.io/skills/revisor-dax-logistica).

---

## Licencia

**Propiedad intelectual de Heroldo Escobedo**, con el código a la vista ([Licencia de uso de Heroldo 1.0](LICENSE)):

- **Uso personal: gratis.** Léelo, pruébalo y adáptalo para aprender o en tus propios datos.
- **Uso en una empresa o con fines de lucro: solo con compra.** Cómpralo en [Agensi](https://agensi.io/skills/revisor-dax-logistica)
  o escribe a heroldo@lolo-enterprises.com. La compra cubre a tu empresa.
- **Siempre se cita la fuente:** «Basado en una skill de Heroldo Escobedo — heroldo.org».
- No se revende ni se vuelve a publicar.

> **In English:** Source-available, not open source. Free for personal use. Business use requires purchase on
> [Agensi](https://agensi.io/skills/revisor-dax-logistica). Attribution to Heroldo Escobedo — heroldo.org is always required.
