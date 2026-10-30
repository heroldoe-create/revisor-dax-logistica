# Caso AN-DAX-01 — resultado esperado y comprobación manual

**Datos FICTICIOS de Andén Norte.** Revisión 2026-10-29 de las medidas y la muestra de 6 líneas (3 pedidos). **CON HALLAZGOS e INCOMPLETO**: no hay certificación del tablero ni dictamen sobre definiciones fiscales. Este resultado puede seguirse a mano con la tabla de la entrada; no requiere Power BI.

## Cálculo esperado a mano (la vara)

| Cuenta | Operación | Resultado |
|---|---|---:|
| Fill rate (cajas) | (10+5+6+4+10+2) surtidas ÷ (10+5+8+4+10+2) pedidas = 37 ÷ 39 | **94.87 %** |
| Fill rate (líneas completas) | líneas con surtidas = pedidas: L1, L2, L4, L5, L6 = 5 ÷ 6 | **83.33 %** |
| Pedidos | P-101, P-102, P-103 | **3** |
| On Time (por pedido) | pedidos sin línea embarcada después del compromiso: P-102, P-103 = 2 ÷ 3 | **66.67 %** |
| In Full (por pedido) | pedidos sin faltante de cajas: P-101, P-103 = 2 ÷ 3 | **66.67 %** |
| OTIF | pedidos completos y puntuales: solo P-103 = 1 ÷ 3 | **33.33 %** |

Notas de grano: P-101 es **completo pero tarde** (L2 embarcó 10-06 contra compromiso 10-05); P-102 es **puntual pero incompleto** (L3 con faltante de 2 cajas). Embarcar el mismo día del compromiso (L1, L4, L5, L6) es puntual, no tarde.

## Hallazgos

| ID | Medida | Falla (checklist) | Evidencia | Corrección (resumen) | Prioridad |
|---|---|---|---|---|---|
| H1 | `Fill Rate Prom` | F1 · total mal calculado: promedio de porcentajes | `AVERAGE(pct_linea)` = (100+100+75+100+100+100)/6 = **95.83 %**; el total correcto es cociente de totales | Recalcular como `DIVIDE ( [Cajas Surtidas], [Cajas Pedidas] )` | Corregir antes de la junta |
| H2 | `OTIF %` | F3 · grano mal definido: OTIF medido por línea | 4 líneas «a tiempo y completas» ÷ 6 = 66.67 %; por pedido solo P-103 = 33.33 %. Además usa `<=`, que es aceptable aquí por los datos, pero la condición correcta es por pedido | Patrón `ADDCOLUMNS(VALUES(pedido))` de referencia | Corregir antes de la junta |
| H3 | `Pedidos OTIF v2` | F4 · `BLANK` contra `0` con `==` | `COUNTROWS` sobre un filtro vacío devuelve `BLANK()`, y el operador estricto `[@tarde] == 0` es FALSE contra un blanco (con `=` sería TRUE): ningún pedido pasa y el resultado es **0** | Probar con `ISBLANK ( [@tarde] )` | Corregir antes de la junta |
| H4 | Tarjetas vs segmentador | F2 · relación invertida | La relación declarada pone el lado **uno** en `fact_lineas_pedido` y el **varios** en `dim_sku`: el filtro no puede llegar al hecho | Invertir: `dim_sku[sku]` (uno) → `fact_lineas_pedido[sku]` (varios) | Corregir en el modelo |
| H5 | Rotación | — | No se aportó fórmula ni datos de inventario | **NO VERIFICABLE**; pedir fórmula y existencias | Pedir insumo |

## Medidas corregidas (listas para pegar)

```dax
Cajas Pedidas = SUM ( fact_lineas_pedido[cajas_pedidas] )

Cajas Surtidas = SUM ( fact_lineas_pedido[cajas_surtidas] )

Fill Rate Cajas = DIVIDE ( [Cajas Surtidas], [Cajas Pedidas] )

Fill Rate Lineas =
DIVIDE (
    COUNTROWS ( FILTER ( fact_lineas_pedido, fact_lineas_pedido[cajas_surtidas] = fact_lineas_pedido[cajas_pedidas] ) ),
    COUNTROWS ( fact_lineas_pedido )
)

Pedidos = DISTINCTCOUNT ( fact_lineas_pedido[pedido] )

Pedidos OTIF =
VAR porPedido =
    ADDCOLUMNS (
        VALUES ( fact_lineas_pedido[pedido] ),
        "@faltante", CALCULATE ( SUMX ( fact_lineas_pedido, fact_lineas_pedido[cajas_pedidas] - fact_lineas_pedido[cajas_surtidas] ) ),
        "@tarde", CALCULATE ( COUNTROWS ( FILTER ( fact_lineas_pedido, fact_lineas_pedido[fecha_embarque] > fact_lineas_pedido[fecha_compromiso] ) ) )
    )
RETURN
    COUNTROWS ( FILTER ( porPedido, [@faltante] = 0 && ISBLANK ( [@tarde] ) ) )

OTIF % = DIVIDE ( [Pedidos OTIF], [Pedidos] )
```

Y en el modelo: relación `dim_sku[sku]` → `fact_lineas_pedido[sku]`, dirección de filtro simple (dim filtra a fact).

## Pruebas

| Medida | Antes | Esperado (cuenta a mano) | Después | Veredicto |
|---|---:|---:|---:|---|
| Fill rate | 95.83 % | 94.87 % (37÷39) | 94.87 % | Pasa |
| OTIF % | 66.67 % | 33.33 % (1÷3) | 33.33 % | Pasa |
| Pedidos OTIF | 0 | 1 (P-103) | 1 | Pasa |
| Fill rate con segmentador «Secos» | no cambia | 33 ÷ 35 = **94.29 %** (quita L4: 4 pedidas y 4 surtidas) | 94.29 % | Pasa |
| Rotación | — | — | — | NO VERIFICABLE (H5) |

Síntomas de regresión: si `OTIF %` vuelve a «casar» casi con `On Time %`, alguien volvió a medir por línea; si `Pedidos OTIF` da 0 de nuevo, alguien comparó un `BLANK()` con `== 0`; si el total del fill rate no es cociente de totales, volvió el `AVERAGE` de porcentajes.

## Límite de autoridad

Estas correcciones verifican la **aritmética y el grano** de las medidas contra la definición documentada; no certifican el tablero, no validan los datos de origen y no sustituyen al contador, al agente aduanal ni al SAT. Basado en una skill de Heroldo Escobedo — heroldo.org.
