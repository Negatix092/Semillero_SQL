# Ejercicio 29 · Las metas a lo largo

- **Alumno:** Axel Cortez
- **Entorno:** Power BI Desktop / Power Query / DAX

---

# Parte A · El modelo sin metas

## A3. Medidas DAX de Cosecha

### Medida Kilos

```dax
Kilos = SUM( h_cosecha[kg] )
```

- **Resultado (Meses 1-4, 2026):** 30 550.

### Medida Cosechas

```dax
Cosechas = COUNTROWS( h_cosecha )
```

- **Resultado (Meses 1-4, 2026):** 9.

## A4. Tabla de control inicial (sin metas)

**Filtros:** `anio = 2026`, `mes entre 1 y 4`.

| Finca | Kilos | Cosechas |
|---|---:|---:|
| Santa Rosa | 14 200 | 4 |
| El Guayabo | 14 250 | 3 |
| La Union | 2 100 | 2 |
| **Total** | **30 550** | **9** |

## A5. Análisis previo de `metas_planeacion.csv`

### ¿Cuántas filas tiene que tener `h_meta` para guardar una meta por finca y por mes?

Debe tener **36 filas**:

```text
3 fincas reales × 12 meses = 36 filas
```

### ¿Cuánto tiene que sumar el año según el correo de planeación?

Debe sumar exactamente **47 000 kg**.

---

# Parte B · Anular dinamización

## B1. Dimensiones iniciales del archivo

El archivo contiene inicialmente:

- **5 filas**
- **15 columnas**

## B2. Filas tras Anular dinamización de otras columnas

Después de aplicar **Anular dinamización de otras columnas**, la tabla pasó a tener **52 filas**.

### Fórmula literal generada

```powerquery
= Table.UnpivotOtherColumns(
    #"Encabezados promovidos",
    {"finca_id", "finca"},
    "Atributo",
    "Valor"
)
```

## B4. Filas que dieron Error al tipificar `fecha_mes` a Fecha

Se produjeron errores en **4 celdas**.

Los valores presentes en el paso anterior eran:

| finca_id | finca | fecha_mes | kg_meta |
|---:|---|---|---:|
| 1 | Santa Rosa | `"Total"` | 20 000 |
| 2 | El Guayabo | `"Total"` | 19 000 |
| 3 | La Union | `"Total"` | 8 000 |
| `(null)` | Total | `"Total"` | 47 000 |

## B5. Estado tras filtrar `"Total"` y tipificar

Después de filtrar los valores `"Total"` y aplicar correctamente los tipos de datos:

- **Filas:** 48
- **Errores:** 0

## B7. Medidas DAX de Metas

### Medida Meta

```dax
Meta = SUM( h_meta[kg_meta] )
```

### Medida Cumplimiento

```dax
Cumplimiento = DIVIDE( [Kilos], [Meta] )
```

### Medida Filas de meta

```dax
Filas de meta = COUNTROWS( h_meta )
```

---

# Parte C · La finca que se llamaba Total

## C1. Tabla de control con la fila Total presente

**Segmentadores:** Meses 1-4.

| Finca | Kilos | Cosechas | Meta | Cumplimiento | Filas de meta |
|---|---:|---:|---:|---:|---:|
| Santa Rosa | 14 200 | 4 | 10 000 | 142,00 % | 4 |
| El Guayabo | 14 250 | 3 | 9 440 | 150,95 % | 4 |
| La Union | 2 100 | 2 | 5 000 | 42,00 % | 4 |
| (En blanco) | — | — | 24 440 | — | 4 |
| **Total** | **30 550** | **9** | **48 880** | **62,50 %** | **16** |

## C2. Calidad de columna en `finca_id`

- **Válido:** 75 %
- **Error:** 0 %
- **Vacío:** 25 %

## C3. Inspección del valor `(null)`

Al filtrar `(null)`, quedan **12 filas** correspondientes a:

```text
finca = "Total"
```

Los valores de `kg_meta` para los meses corresponden exactamente a la suma mensual consolidada de la última fila de la hoja original.

## C4. ¿Por qué la columna Total dio Error y la fila Total no?

La **columna Total** dio Error porque Power Query intentó convertir la cadena de texto:

```text
"Total"
```

en un tipo de dato `Date`.

En cambio, la **fila Total** no dio error porque sus valores en `fecha_mes` eran fechas legítimas:

```text
2026-01-01
2026-02-01
...
```

y sus valores en `kg_meta` eran enteros válidos.

Por lo tanto, su inconsistencia era **dimensional/estructural** (`finca_id` nulo), no de tipo de dato.

## C5. Comportamiento anual sin segmentador de mes

El total de la tabla fue:

- **Kilos:** 30 550
- **Meta:** 94 000
- **Cumplimiento:** 32,50 %

### Causa de los 94 000 kg

Se está sumando:

```text
Meta de las tres fincas = 47 000 kg
+
Fila de total consolidado = 47 000 kg
=
94 000 kg
```

Planeación incluyó en el CSV tanto las metas individuales de las tres fincas como una fila de total consolidado, provocando que el modelo duplique la meta global de la empresa.

## C6. Filtrado explícito vs. Quitar errores

En este caso, ambas opciones habrían dado el mismo resultado visual porque solamente esas **4 celdas** fallaban.

Sin embargo, si un mes legítimo hubiese venido mal formateado, por ejemplo:

```text
2026/02/31
```

**Quitar errores** habría eliminado silenciosamente una meta válida de producción sin alertar al desarrollador.

---

# Parte D · El segundo arreglo obvio

## D2. Pruebas tras aplicar filtro visual en finca

Se aplicó un filtro visual para excluir los valores en blanco.

Los resultados fueron:

- La fila **(En blanco)** ya no aparece en la tabla.
- **Total de la tabla de control:** 30 550 / 24 440 / 125,00 %.
- **[Filas de meta] en el total:** 12.

## D3 y D4. Tarjetas globales en el lienzo

### Tarjeta `[Meta]`

```text
48 880
```

La tarjeta mantiene la fila Total porque el filtro aplicado anteriormente solamente afectaba al objeto visual.

### Medida Meta sin finca

```dax
Meta sin finca =
CALCULATE(
    [Meta],
    ISBLANK( h_meta[finca_id] )
)
```

### Tarjeta `[Meta sin finca]`

```text
24 440
```

## D5. Tabla por mes sin filtro de finca

| nombre_mes | Kilos | Meta | Cumplimiento |
|---|---:|---:|---:|
| Enero | — | 8 080 | — |
| Febrero | — | 10 400 | — |
| Marzo | 10 800 | 13 800 | 78,26 % |
| Abril | 19 750 | 16 600 | 118,98 % |
| **Total** | **30 550** | **48 880** | **62,50 %** |

### ¿Aparece alguna fila (En blanco)? ¿Por qué no?

No aparece ninguna fila **(En blanco)** porque todas las filas de `h_meta`, incluidas las que no tienen finca, poseen un `fecha_mes` válido que se relaciona al **100 %** con `dim_tiempo[fecha]`.

El total huérfano de finca se distribuye transparentemente en cada mes.

## D6. Conciliación manual de metas de la empresa

**Periodo:** Meses 1-4.

| Mes | Santa Rosa | El Guayabo | La Union | Meta del mes | Kilos | Cumplimiento |
|---|---:|---:|---:|---:|---:|---:|
| Enero | 1 600 | 1 440 | 1 000 | 4 040 | — | — |
| Febrero | 2 100 | 2 100 | 1 000 | 5 200 | — | — |
| Marzo | 2 800 | 2 600 | 1 500 | 6 900 | 10 800 | 156,52 % |
| Abril | 3 500 | 3 300 | 1 500 | 8 300 | 19 750 | 237,95 % |
| **Total** | **10 000** | **9 440** | **5 000** | **24 440** | **30 550** | **125,00 %** |

## D7. Diagnóstico del parche visual

### ¿Qué arregló y qué no arregló?

El filtro visual ocultó la fila huérfana en un visual específico, pero dejó contaminado el modelo global de datos.

Por lo tanto, cualquier tarjeta, gráfico o reporte por mes que no tuviera ese mismo filtro seguiría contabilizando el doble de meta.

### ¿Por qué la prueba del número viejo pasó?

La prueba pasó porque la suma de las tres fincas en los meses 1 a 4 daba:

```text
24 440 kg
```

Al ocultar la fila en blanco, este valor coincidía con el total visual esperado.

---

# Parte E · El arreglo en Power Query

## E2. Estado tras filtrar `(null)` en `finca_id`

Después de eliminar las filas cuyo `finca_id` era `(null)`, el estado de la tabla fue:

- **Filas:** 36
- **Vacíos:** 0 %

## E3. Validación de la tabla de control

**Sin filtros de objeto visual.**

| Finca | Kilos | Cosechas | Meta | Cumplimiento | Filas de meta |
|---|---:|---:|---:|---:|---:|
| Santa Rosa | 14 200 | 4 | 10 000 | 142,00 % | 4 |
| El Guayabo | 14 250 | 3 | 9 440 | 150,95 % | 4 |
| La Union | 2 100 | 2 | 5 000 | 42,00 % | 4 |
| **Total** | **30 550** | **9** | **24 440** | **125,00 %** | **12** |

### Tarjetas de validación

- **Tarjeta `[Meta]`:** 24 440
- **Tarjeta `[Meta sin finca]`:** (En blanco / vacía)

## E5. Conciliación anual de metas

**Periodo:** 2026 completo.

| Finca | Kilos | Meta | Cumplimiento | Filas de meta | Total en hoja CSV |
|---|---:|---:|---:|---:|---:|
| Santa Rosa | 14 200 | 20 000 | 71,00 % | 12 | 20 000 |
| El Guayabo | 14 250 | 19 000 | 75,00 % | 12 | 19 000 |
| La Union | 2 100 | 8 000 | 26,25 % | 12 | 8 000 |
| **Total** | **30 550** | **47 000** | **65,00 %** | **36** | **47 000** |

## E6. Cotejo con A5

Sí, se confirmaron exactamente:

- **36 filas finales**
- **47 000 kg** estipulados por planeación

Esto coincide con la validación planteada inicialmente en la Parte A.

---

# Parte F · La fórmula del paso y preguntas de cierre

## F1. Línea literal en el Editor Avanzado

```powerquery
= Table.UnpivotOtherColumns(
    #"Encabezados promovidos",
    {"finca_id", "finca"},
    "Atributo",
    "Valor"
)
```

La fórmula nombra las columnas que se quedan fijas:

- `finca_id`
- `finca`

Todas las demás columnas se convierten en pares **atributo-valor**.

## F2. Comportamiento ante columnas de 2027

Al utilizar `Table.UnpivotOtherColumns`, cualquier columna nueva, como los meses correspondientes a **2027**, se desdinamizará automáticamente hacia abajo sin necesidad de modificar el código.

Si se hubiese utilizado **Anular solo las columnas seleccionadas**, las nuevas columnas de 2027 habrían sido ignoradas por completo al actualizar.

## F3. Respuestas de cierre

### Definición de Anular dinamización

**Anular dinamización** convierte encabezados de columna en valores de fila, transformando una matriz ancha en una tabla normalizada vertical.

### Regla de detección del día

> **«Si la tabla de hechos tiene más filas que el producto de sus dimensiones, no importaste datos: importaste subtotales de la hoja.»**

### Filtro visual vs. Quitar duplicados

Ambos barren el síntoma visual dejando el error dentro del modelo.

El filtro visual mantiene el registro, por lo que este continúa inflando medidas en cualquier otro gráfico que no comparta ese filtro.

### Impacto de la fila totalizadora

En modelos tabulares relacionales, el motor DAX totaliza sumando las filas base.

Si el archivo fuente ya contiene una fila **Total** que suma las demás, el modelo vuelve a incluir esa fila dentro de la agregación y, por lo tanto, duplica la métrica:

```text
Filas individuales + fila Total = métrica duplicada
```

### Columna Observaciones con texto

Si apareciera una columna adicional llamada `Observaciones` con contenido textual, esta también sería afectada por `Table.UnpivotOtherColumns`.

Como consecuencia, podría:

- Caer dentro de la columna `fecha_mes`, generando errores al intentar convertir texto a fecha.
- Generar fallos masivos de conversión numérica al intentar convertir el contenido textual en `kg_meta`.

Por lo tanto, cualquier nueva columna descriptiva que no deba ser desdinamizada tendría que considerarse dentro de la estructura de columnas fijas de la transformación.