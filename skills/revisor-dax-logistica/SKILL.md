---
name: revisor-dax-logistica
description: Revisa medidas DAX de Power BI para logística (fill rate, OTIF, rotación, exactitud de inventario). Detecta totales mal calculados, contexto de filtro heredado, promedios de porcentajes, granos mal definidos (línea vs pedido vs SKU) y relaciones invertidas; propone la medida DAX corregida y la prueba numérica que la confirma. También explica por qué una medida «oficial» no cuadra con el piso. No conecta a Power BI ni ejecuta DAX; trabaja con las medidas y los datos que el usuario pega o adjunta.
license: LicenseRef-Heroldo-1.0
---

# Revisor de medidas DAX para logística

Pre-revisa medidas, cruza contra los datos del usuario y arma evidencia. **No sustituye al contador, al agente aduanal ni al SAT** (y no dictamina sobre impuestos, aduanas ni obligaciones fiscales); tampoco certifica un tablero ni «aprueba» un número para la junta. El resultado es una tabla de hallazgos con la medida corregida y su prueba, lista para que quien modela la valide y aplique.

## Recursos y límites

Las rutas `references/` y `ejemplos/` son relativas a la raíz del paquete: en el ZIP están junto a este archivo; en el repositorio, dos niveles arriba. Antes de revisar lee:

- [Checklist de fallas DAX](references/checklist-fallas-dax.md): catálogo de los errores que esta skill caza, con su síntoma y su patrón.
- [Fuentes](references/fuentes.md): URL, fecha de consulta y procedencia de cada referencia.
- [Caso de práctica](ejemplos/entrada-anden-norte.md) y [resultado esperado](ejemplos/salida-esperada.md), cuando se pida una demostración o no haya datos propios.

## 1. Recibir y delimitar

Acepta medidas pegadas como texto, captura de pantalla transcrita, descripción del modelo (tablas, columnas, relaciones, dirección de filtro) y exportes de datos (CSV, tabla, Excel). Identifica:

- Qué mide cada medida declarada (fill rate, OTIF, rotación, exactitud, productividad, recibo) y su definición oficial en la empresa, si existe.
- El **grano** de cada tabla citada: línea de pedido, pedido, SKU-día, movimiento, conteo.
- Relaciones declaradas: de qué tabla a cuál, y dirección del filtro (uno → varios).
- Qué número «oficial» se contradice y quién lo reporta.

No pidas el archivo `.pbix` ni acceso al tenant; basta con el texto de las medidas, el esquema y una muestra de datos. Nunca solicites contraseñas, cuentas de servicio ni llaves. No ejecutes ni «corras» DAX: verifica con aritmética a mano sobre la muestra.

## 2. Reconstruir el cálculo esperado

Para cada medida bajo revisión:

1. Escribe la fórmula **correcta a priori** según la definición acordada (ver [checklist](references/checklist-fallas-dax.md), sección Definiciones de referencia). Fill rate, OTIF, rotación y exactitud tienen definición fija: la medida del cliente debe cuadrar con esa definición o declararse explícitamente como otra cosa.
2. Determina el grano en que la pregunta tiene sentido: fill rate se suma en líneas o cajas; OTIF se evalúa **por pedido** (todas las líneas completas y a tiempo); rotación se calcula **por SKU y periodo**; la exactitud de inventario se evalúa **por ubicación/SKU contado**.
3. Toma la muestra de datos y calcula a mano el valor esperado con total transparencia: muestra cada suma, conteo y división con sus unidades.

## 3. Cazar las fallas

Recorre el catálogo del checklist y, para cada medida del usuario, registra `medida | falla | evidencia en la fórmula | efecto en el número | corrección DAX`. Fallas prioritarias en logística:

- **Total mal calculado / promedio de porcentajes:** `AVERAGE` sobre una columna de porcentajes, o sumar porcentajes entre segmentos; el total ya no es el cociente de los totales.
- **Contexto de filtro heredado:** la medida no responde al segmentador (relación invertida o dirección de filtro cruzada omitida), o hereda un filtro de una tabla dimensión que no toca el hecho.
- **Grano mal definido:** medir OTIF por línea en vez de por pedido; contar pedidos con `COUNTROWS` de líneas; `DISTINCTCOUNT` de pedidos en un numerador que suma líneas.
- **Comparación de fechas y blancos:** `== 0` (igualdad estricta) contra `BLANK()`; con `=` DAX trata BLANK como 0. `>=` donde el compromiso admite el mismo día; una línea embarcada el día del compromiso es puntual, no tarde.
- **DIVIDE ausente:** división directa que devuelve error o infinito cuando el denominador es cero en algún segmento.
- **Relaciones de dimensión:** SKU→líneas debe ir de dim (uno) a fact (varios); con la flecha al revés el filtro no llega y la tarjeta se congela.

Si la fórmula es correcta pero el número no cuadra con el «oficial», la discrepancia está en los datos o en la definición: dilo y pide la definición escrita que rige, no la infieras.

## 4. Proponer y probar

Por cada hallazgo entrega:

1. **La medida corregida en DAX**, completa y pegable en Power BI (nombre y todo), con el cambio resaltado.
2. **La prueba numérica:** sobre la muestra del usuario (o sobre el caso AN-DAX-01), el valor antes, el valor esperado calculado a mano, y el valor después; con las cuentas explícitas para que se pueda comprobar sin Power BI.
3. **El síntoma de regresión:** cómo se ve en el tablero si la corrección se estructura mal otra vez (qué tarjeta lo delata).

Formato de salida — sin introducciones ni relleno:

- **Tabla de hallazgos:** `ID | medida | falla | evidencia | corrección (resumen) | prioridad`.
- **Medidas corregidas:** bloque DAX por medida, listo para pegar.
- **Pruebas:** tabla `medida | valor antes | esperado (con cuenta a mano) | valor después | veredicto`.
- **Pendientes:** lo que no se pudo verificar sin más datos (muestra incompleta, relaciones no declaradas) — marcado como NO VERIFICABLE, nunca como aprobado.

Termina siempre con el límite de autoridad del primer párrafo y la atribución exigida por LICENSE: «Basado en una skill de Heroldo Escobedo — heroldo.org». No prometas ahorro medido, certificación del tablero ni que la junta acepte el número: entrega evidencia reproducible.
