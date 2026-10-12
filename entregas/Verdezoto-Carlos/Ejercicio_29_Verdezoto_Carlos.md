# Ejercicio 29 · Las metas a lo largo

## PARTE A - EL MODELO SIN METAS

**A5.** 
* ¿Cuántas filas tiene que tener `h_meta` para guardar una meta por finca y por mes? **36 filas** (3 fincas × 12 meses).
* ¿Cuánto tiene que sumar el año, según el correo? **47 000 kg**.

---

## PARTE B - ANULAR DINAMIZACIÓN

**B1.** En Power Query, abajo a la izquierda dice que la tabla tiene **4 filas** y **15 columnas** (12 meses, 2 de identificación y 1 de Total).

**B2.** Quedan **52 filas**. 
Fórmula literal del paso:
```powerquery
= Table.UnpivotOtherColumns(#"Encabezados promovidos", {"finca_id", "finca"}, "Atributo", "Valor")

```

**B4.** Hay **4** celdas con `Error`. En el paso anterior (antes de cambiar tipo), `fecha_mes` decía **`Total`** en esas cuatro filas. Su finca era Santa Rosa, El Guayabo, La Union y Total, y su `kg_meta` eran los totales anuales (20000, 19000, 8000, 47000).

---

## PARTE C - LA FINCA QUE SE LLAMABA TOTAL

**C3.** Quedan **12 filas** con `finca_id` vacío. La columna `finca` dice **`Total`**. Los doce valores de `kg_meta` (4040, 5200, 6900, 8300...) son exactamente idénticos a los meses de la última fila en `metas_planeacion.csv`.

**C4.** La **columna** Total dio error porque su encabezado (la palabra "Total") se convirtió en el valor de la columna `fecha_mes`, y la palabra "Total" no puede transformarse al tipo Fecha. La **fila** Total no dio error en la fecha porque cruzaba con los meses ("2026-01-01"), los cuales sí son fechas válidas; su problema era que `finca_id` venía en blanco, no el tipo de la fecha.

**C5.** Suma **94 000** en vez de 47 000 porque la tabla dinámica incluye tanto las metas individuales de las 3 fincas (47 000) como la fila adicional de "Total" (47 000), sumándolas y duplicando exactamente el presupuesto anual de toda la empresa.

**C6.** Si hubieras usado `Quitar errores`, **sí** habría dado lo mismo hoy, porque solo habría eliminado la columna de totales anuales, pero la fila `Total` (que no tenía error en fecha) habría sobrevivido intacta duplicando los meses. Si una celda de verdad hubiera traído un error tipográfico, `Quitar errores` habría borrado esa meta mensual válida permanentemente, destruyendo datos.

---

## PARTE D - EL SEGUNDO ARREGLO OBVIO

### D4 · Meta sin finca

```dax
Meta sin finca = CALCULATE( [Meta] , ISBLANK( h_meta[finca_id] ) )

```

**D6. Cálculo a mano de la meta de la empresa (solo las tres fincas):**

| Mes | Santa Rosa | El Guayabo | La Union | Meta del mes | Kilos | Cumplimiento |
| --- | --- | --- | --- | --- | --- | --- |
| Enero | 1 500 | 1 440 | 1 100 | **4 040** | — | — |
| Febrero | 2 000 | 2 000 | 1 200 | **5 200** | — | — |
| Marzo | 3 000 | 2 500 | 1 400 | **6 900** | 10 800 | **156,52 %** |
| Abril | 3 500 | 3 500 | 1 300 | **8 300** | 19 750 | **237,95 %** |
| **Total** | **10 000** | **9 440** | **5 000** | **24 440** | **30 550** | **125,00 %** |

**D7.** El filtro de objeto visual arregló **la apariencia** del gráfico (escondió la fila en blanco y corrigió el subtotal de ese visual), pero **no arregló** el modelo de datos (las medidas globales y el resto de páginas seguían calculando 48 880 como meta). La prueba del número viejo pasó porque el total sumado en ese visual específico filtrado coincidía matemáticamente con la respuesta esperada, ocultando el error subyacente.

---

## PARTE E - EL ARREGLO EN POWER QUERY

**E6.** Sí, acerté las dos respuestas de A5. Tras filtrar `finca_id` sin `(null)` en Power Query, quedaron **36 filas** exactas, y al quitar el segmentador de mes, la suma total del año volvió a cuadrar con los **47 000 kg** esperados por la gerencia.

---

## PARTE F - LA FÓRMULA DEL PASO, Y PREGUNTAS DE CIERRE

**F1.** Línea literal: `Table.UnpivotOtherColumns(#"Encabezados promovidos", {"finca_id", "finca"}, "Atributo", "Valor")`. La fórmula nombra a las columnas **que se quedan fijas** (las que NO se dinamizan).

**F2.** Al actualizar, la consulta capturará y dinamizará automáticamente las nuevas columnas de 2027 hacia abajo (porque la fórmula dice "unpivot *other* columns"). Si hubieras usado "Anular solo la dinamización de las columnas seleccionadas", la fórmula tendría quemados en duro ("hardcoded") solo los nombres de los 12 meses de 2026, ignorando y dejando fuera del modelo todas las columnas nuevas de 2027.

**F3. Preguntas de cierre:**

1. **¿Qué hace Anular dinamización?** Transforma columnas anchas en filas largas; agarra los encabezados de varias columnas y los convierte en valores de una sola columna ("Atributo"), y sus datos los apila en otra ("Valor").


2. **Regla de detección:** "Si una meta o presupuesto suma exactamente el doble de lo planeado al cargar un archivo nuevo, la hoja origen traía una fila de totales calculados que se importó como un registro más".


3. **Filtro vs Quitar duplicados:** Se parecen en que ambos son "arreglos estéticos" que ocultan un problema de base. El filtro de objeto visual deja el error **dentro del modelo**, contaminando cualquier tarjeta, KPI o nueva página que no herede explícitamente ese filtro manual.


4. **Fila Total en clase 21 vs clase 29:** En la clase 21 la fila total no sumaba lo mismo porque era un promedio ponderado o porcentaje general distinto a la suma aritmética. Hoy el problema fue precisamente que **sí** era la suma exacta, causando que el modelo agregara 100% de duplicidad matemática pura a los resultados.


5. **Columna "Observaciones":** Con "otras columnas", `Observaciones` se tratará de anular y aparecerá en la nueva columna `fecha_mes`. Te avisaría porque al paso siguiente, donde cambiaste `fecha_mes` a tipo Fecha, generará un **Error** visible en cada fila donde diga el texto de la observación.