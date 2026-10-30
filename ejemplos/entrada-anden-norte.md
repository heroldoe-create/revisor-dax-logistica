# Caso AN-DAX-01 — entrada ficticia

**Todos los nombres, folios, SKU y valores son FICTICIOS de Andén Norte.** Fecha simulada: octubre 2026. Revisión: 2026-10-29.

Solicitud: «Revisa estas medidas de mi tablero de servicio. El fill rate oficial marca 95.83 %, el OTIF 66.67 % y el `Pedidos OTIF v2` me da 0. No sé por qué no cuadran con el piso».

## Modelo declarado por el usuario

- Tabla `fact_lineas_pedido` (grano: línea de pedido). Columnas: `pedido`, `sku`, `cajas_pedidas`, `cajas_surtidas`, `fecha_compromiso`, `fecha_embarque`, `pct_linea` (columna calculada = surtidas÷pedidas por línea).
- Tabla `dim_sku` (`sku`, `familia`). Relación declarada: `fact_lineas_pedido[sku]` (lado **uno**) → `dim_sku[sku]` (lado **varios**).
- Medidas declaradas:

```dax
Fill Rate Prom = AVERAGE ( fact_lineas_pedido[pct_linea] )

OTIF % =
DIVIDE (
    COUNTROWS ( FILTER ( fact_lineas_pedido,
        fact_lineas_pedido[cajas_surtidas] = fact_lineas_pedido[cajas_pedidas]
        && fact_lineas_pedido[fecha_embarque] <= fact_lineas_pedido[fecha_compromiso] ) ),
    COUNTROWS ( fact_lineas_pedido )
)

Pedidos OTIF v2 =
VAR porPedido =
    ADDCOLUMNS ( VALUES ( fact_lineas_pedido[pedido] ),
        "@faltante", CALCULATE ( SUMX ( fact_lineas_pedido, fact_lineas_pedido[cajas_pedidas] - fact_lineas_pedido[cajas_surtidas] ) ),
        "@tarde", CALCULATE ( COUNTROWS ( FILTER ( fact_lineas_pedido, fact_lineas_pedido[fecha_embarque] > fact_lineas_pedido[fecha_compromiso] ) ) ) )
RETURN
    COUNTROWS ( FILTER ( porPedido, [@faltante] = 0 && [@tarde] == 0 ) )
```

## Muestra de datos (las 6 líneas completas del periodo)

`pct_linea` ya viene calculada en la tabla como surtidas÷pedidas por línea.

| Línea | Pedido | SKU | Familia | Cajas pedidas | Cajas surtidas | pct_linea | Compromiso | Embarque |
|---|---|---|---|---:|---:|---:|---|---|
| L1 | P-101 | SKU-A | Secos | 10 | 10 | 100 % | 2026-10-05 | 2026-10-05 |
| L2 | P-101 | SKU-B | Secos | 5 | 5 | 100 % | 2026-10-05 | 2026-10-06 |
| L3 | P-102 | SKU-A | Secos | 8 | 6 | 75 % | 2026-10-05 | 2026-10-04 |
| L4 | P-102 | SKU-C | Frío | 4 | 4 | 100 % | 2026-10-05 | 2026-10-05 |
| L5 | P-103 | SKU-B | Secos | 10 | 10 | 100 % | 2026-10-03 | 2026-10-03 |
| L6 | P-103 | SKU-A | Secos | 2 | 2 | 100 % | 2026-10-03 | 2026-10-03 |

Observaciones del usuario: al filtrar por la familia «Secos» en el segmentador, las tarjetas no cambian. La medida de rotación existe en el tablero pero no se aporta su fórmula ni datos de inventario.
