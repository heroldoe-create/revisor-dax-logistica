# Checklist de fallas DAX en KPIs de logística

Catálogo de las fallas que esta skill caza en medidas de fill rate, OTIF, rotación y exactitud, con síntoma, causa y corrección tipo. Referencias externas en [fuentes.md](fuentes.md).

## Definiciones de referencia (el numerador y el denominador correctos)

| KPI | Definición fija | Numerador | Denominador | Grano de evaluación |
|---|---|---|---|---|
| Fill rate (cajas) | cajas surtidas ÷ cajas pedidas | `SUM ( fact[cajas_surtidas] )` | `SUM ( fact[cajas_pedidas] )` | se suma por línea; el total es cociente de totales |
| Fill rate (líneas completas) | líneas surtidas igual a lo pedido ÷ líneas pedidas | `COUNTROWS ( FILTER ( fact, fact[cajas_surtidas] = fact[cajas_pedidas] ) )` | `COUNTROWS ( fact )` | línea |
| OTIF | pedidos con todas sus líneas completas y ninguna línea embarcada después del compromiso ÷ pedidos | `Pedidos OTIF` (ver abajo) | `DISTINCTCOUNT ( fact[pedido] )` | **pedido** |
| On Time | pedidos sin línea embarcada después del compromiso ÷ pedidos | conteo por pedido | `DISTINCTCOUNT ( fact[pedido] )` | **pedido** |
| In Full | pedidos sin faltante de cajas ÷ pedidos | conteo por pedido | `DISTINCTCOUNT ( fact[pedido] )` | **pedido** |
| Rotación de inventario | consumo/salidas del periodo ÷ inventario promedio | `SUM ( salidas )` del periodo | promedio de existencias (inicial+final)/2, por SKU | **SKU × periodo** |
| Exactitud de inventario | ubicaciones/SKU contados sin diferencia ÷ contados | `COUNTROWS ( FILTER ( conteos, ajuste = 0 ) )` | `COUNTROWS ( conteos )` | **conteo (SKU × ubicación)** |

`Pedidos OTIF` de referencia:

```dax
Pedidos OTIF =
VAR porPedido =
    ADDCOLUMNS (
        VALUES ( fact[pedido] ),
        "@faltante", CALCULATE ( SUMX ( fact, fact[cajas_pedidas] - fact[cajas_surtidas] ) ),
        "@tarde", CALCULATE ( COUNTROWS ( FILTER ( fact, fact[fecha_embarque] > fact[fecha_compromiso] ) ) )
    )
RETURN
    COUNTROWS ( FILTER ( porPedido, [@faltante] = 0 && ISBLANK ( [@tarde] ) ) )
```

## Catálogo de fallas

### F1 · Total mal calculado: promedio de porcentajes

- **Síntoma:** el total del KPI no es el cociente de los totales; cada segmento «promedia bien» y el conjunto miente.
- **Causa típica:** `AVERAGE ( fact[fill_rate_linea] )` donde la columna ya era un porcentaje por línea; o `SUM` de porcentajes entre familias.
- **Efecto:** en muestras con volúmenes desiguales, el total se desvía del valor real en puntos visibles; los segmentos chicos inflan o hunden el promedio.
- **Corrección:** recalcular el KPI como cociente de sumas (`DIVIDE ( [Surtidas], [Pedidas] )`), nunca como promedio de cocientes.
- **Prueba:** valor esperado = suma de numeradores ÷ suma de denominadores, a mano, sobre la muestra.

### F2 · Contexto de filtro heredado: el segmentador no filtra

- **Síntoma:** al filtrar por familia de SKU o por mes, la tarjeta no cambia.
- **Causa típica:** relación invertida (la flecha va de fact a dim) o tabla dimensión sin relación al hecho; medida calculada sobre toda la tabla.
- **Corrección:** relación de `dim[clave]` (lado uno) a `fact[clave]` (lado varios), dirección de filtro simple; si se necesita bidireccional, justificarlo y aislarlo con `CROSSFILTER`/`ALL` explícitos, no por omisión.
- **Prueba:** congelar un filtro conocido (una sola familia) y verificar que la tarjeta cambia al valor de esa familia calculado a mano.

### F3 · Grano mal definido: OTIF por línea en vez de por pedido

- **Síntoma:** `On Time %` y `In Full %` casi coinciden y la historia «una línea tarde tumba el pedido» no aparece; o un conteo de pedidos mezcla `COUNTROWS` de líneas con `DISTINCTCOUNT` de pedidos (resultados > 100 %).
- **Causa típica:** evaluar la condición por línea y luego promediar; o `DIVIDE ( COUNTROWS ( líneas ), DISTINCTCOUNT ( pedidos ) )`.
- **Corrección:** el patrón `ADDCOLUMNS ( VALUES ( fact[pedido] ), ... )` por pedido, como en la definición de referencia.
- **Prueba:** un pedido de 10 líneas con 1 tarde: On Time por líneas ≈ 90 %, On Time por pedidos = 0 % para ese pedido.

### F4 · BLANCO contra cero y comparación de fechas

- **Síntoma:** `Pedidos OTIF` = 0 aunque hay pedidos completos; o `On Time %` desplomado por debajo de lo razonable.
- **Causa típica:** `[@tarde] == 0` (igualdad estricta) donde `COUNTROWS` sobre un filtro vacío devuelve `BLANK()`: `BLANK() == 0` es FALSE. Ojo: con `=` DAX trata BLANK como 0 y `BLANK() = 0` es TRUE, así que `= 0` no es la falla; `ISBLANK` deja la intención explícita; o `fecha_embarque >= fecha_compromiso` marcando como tarde al embarque del mismo día.
- **Corrección:** probar con `ISBLANK ( [@tarde] )`; comparar con `>` estricto.
- **Prueba:** un pedido embarcado el mismo día del compromiso cuenta como puntual y completo.

### F5 · División sin DIVIDE

- **Síntoma:** en segmentos sin denominador (SKU sin pedidos en el mes) la tarjeta muestra error o un valor infinito que contamina totales.
- **Causa típica:** operador `/` directo.
- **Corrección:** `DIVIDE ( numerador, denominador )` en todo KPI que pueda tener denominador cero.
- **Prueba:** un segmento con denominador 0 debe dar BLANK(), no error.

### F6 · Rotación y exactitud con grano cruzado

- **Síntoma:** rotación a nivel empresa «promedia» rotaciones de SKU sin ponderar por inventario; exactitud contada por línea de ajuste en vez de por ubicación contada.
- **Causa típica:** `AVERAGE` de una columna de rotación por SKU, o `COUNTROWS` sobre movimientos en vez de sobre conteos.
- **Corrección:** la rotación del agregado = salidas totales del agregado ÷ inventario promedio del agregado; la exactitud = conteos sin diferencia ÷ conteos, al grano del conteo.
- **Prueba:** recalcular el agregado como cociente de totales; si el valor «oficial» difiere, la diferencia es el peso de los SKU grandes.

## Lo que este catálogo NO cubre

- Rendimiento (variables, `SUMMARIZE` vs `ADDCOLUMNS`, medidas materializadas): fuera de alcance; se menciona solo cuando una corrección lo exige.
- Certificación o validación oficial del tablero, dictámenes fiscales y obligaciones ante autoridad: **NO VERIFICABLE** aquí; derivar al responsable correspondiente.
- Conectores, RLS ni despliegue: no se revisan sin acceso, y este paquete nunca pide acceso.
