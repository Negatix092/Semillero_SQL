# Ejercicio práctico 29 · Las metas a lo largo

**Estudiante:** Daniel Troya  
**Fecha:** 29 de septiembre de 2026  
**Herramienta:** Power BI Desktop  
**Archivos de entrega:** `Ejercicio29_Troya_Daniel.md`, `clase29-calidad.png` y `clase29-metas.png`

---

## Parte A · El modelo sin metas

### A3. Medidas de cosecha

```dax
Kilos = SUM( h_cosecha[kg] )
```
* **Resultado:** Medida base de kilos cosechados. En la tabla de control (2026, meses 1–4) suma 30 550.

```dax
Cosechas = COUNTROWS( h_cosecha )
```
* **Resultado:** Conteo de transacciones de cosecha registradas. En la tabla de control (2026, meses 1–4) suma 9.

### A4. Tabla de control inicial (dim_tiempo[anio] = 2026, dim_tiempo[mes] = 1 a 4)

| dim_finca[finca] | Kilos | Cosechas |
| :--- | :---: | :---: |
| Agricola La Union | 2 100 | 2 |
| Finca El Guayabo | 14 250 | 3 |
| Hacienda Santa Rosa | 14 200 | 4 |
| **Total** | **30 550** | **9** |

### A5. Preguntas previas con metas_planeacion.csv

* **¿Cuántas filas tiene que tener h_meta para guardar una meta por finca y por mes?**  
  Debe tener exactamente **36 filas** ($3\text{ fincas} \times 12\text{ meses}$).
* **¿Y cuánto tiene que sumar el año, según el correo?**  
  Tiene que sumar **47 000 kg**.

---

## Parte B · Anular dinamización

* **B1. Dimensiones al abrir metas_planeacion.csv:**  
  4 filas y 15 columnas (`finca_id`, `finca`, 12 columnas mensuales y la columna `Total`).

* **B2. Filas tras Anular dinamización de otras columnas:**  
  Quedan **52 filas** ($4\text{ filas} \times 13\text{ columnas desdinamizadas}$).  
  Fórmula literal del paso:
  ```powerquery
  = Table.UnpivotOtherColumns(#"Encabezados promovidos", {"finca_id", "finca"}, "Atributo", "Valor")
  ```

* **B4. Error al tipificar fecha_mes a Fecha:**  
  Salen **4 celdas con Error**.  
  En el paso previo al cambio de tipo, la columna `fecha_mes` traía el valor `"Total"` en esas cuatro filas:
  1. `finca_id: 1` | `finca: Hacienda Santa Rosa` | `kg_meta: 20000`
  2. `finca_id: 2` | `finca: Finca El Guayabo` | `kg_meta: 19000`
  3. `finca_id: 3` | `finca: Agricola La Union` | `kg_meta: 8000`
  4. `finca_id: null` | `finca: Total` | `kg_meta: 47000`

* **B5. Filas resultantes tras desmarcar "Total":**  
  Quedan **48 filas** y **0 errores**.

### B7. Medidas de meta y cumplimiento

```dax
Meta = SUM( h_meta[kg_meta] )
```
* **Resultado:** Suma de metas asignadas en el contexto. En la tabla con la fila Total suma 48 880; sin la fila duplicada suma 24 440.

```dax
Cumplimiento = DIVIDE( [Kilos] , [Meta] )
```
* **Resultado:** Porcentaje de cumplimiento. Con la duplicación marca 62,50 %; corregido marca 125,00 %. Formato configurado en porcentaje (`%`) con 2 decimales.

```dax
Filas de meta = COUNTROWS( h_meta )
```
* **Resultado:** Conteo de granularidad de metas. Con duplicado en meses 1–4 marca 16; corregido marca 12.

---

## Parte C · La finca que se llamaba Total

### C1. Tabla de control con fila resumen (mes 1 a 4)

| dim_finca[finca] | Kilos | Cosechas | Meta | Cumplimiento | Filas de meta |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Agricola La Union | 2 100 | 2 | 5 000 | 42,00 % | 4 |
| Finca El Guayabo | 14 250 | 3 | 9 440 | 150,95 % | 4 |
| Hacienda Santa Rosa | 14 200 | 4 | 10 000 | 142,00 % | 4 |
| (En blanco) | — | — | 24 440 | — | 4 |
| **Total** | **30 550** | **9** | **48 880** | **62,50 %** | **16** |

### C2. Calidad de columna en finca_id
* **Válido:** 75 % (36 filas)
* **Error:** 0 %
* **Vacío:** 25 % (12 filas)  
*(Se adjunta captura `clase29-calidad.png`)*

### C3. Inspección del filtro finca_id = (null)
* Quedan **12 filas** y la columna `finca` dice `"Total"`.
* Los doce valores de `kg_meta` son:  
  `4040, 5200, 6900, 8300, 4560, 4000, 3200, 3700, 2500, 2100, 1500, 1000`.
* Corresponden idénticamente a la última fila resumen (`Total`) de `metas_planeacion.csv`.

### C4. Columna Total vs Fila Total
La columna Total dio Error porque la palabra `"Total"` era un texto no convertible a tipo Fecha al cambiar `fecha_mes`. La fila Total no dio ningún error porque en esa columna tenía fechas válidas (`2026-01-01`, etc.) y números enteros en los kilos; su único detalle era tener `finca_id` nulo.

### C5. Año completo sin filtro de mes (anio = 2026)
* **Total de la tabla:** 30 550 / 94 000 / 32,50 %
* **¿Por qué 94 000 si el correo dice 47 000?**  
  Porque el modelo suma las metas individuales de las 3 fincas ($47\,000$) y también la fila resumen de totales ($47\,000$), duplicando la meta anual de la empresa.

### C6. Quitar por nombre vs Quitar errores
Hoy habría dado el mismo resultado numérico porque los únicos 4 errores provenían de la palabra `"Total"`. Si una celda de un mes real hubiera traído un error de formato auténtico, `Quitar errores` la habría eliminado de forma silenciosa, perdiendo datos válidos de meta.

---

## Parte D · El segundo arreglo obvio

### D2. Las tres pruebas tras filtrar (En blanco) en el objeto visual

| Prueba | Valor obtenido |
| :--- | :--- |
| La fila (En blanco) | ya no sale |
| Total de la tabla de control | 30 550 / 24 440 / 125,00 % |
| [Filas de meta] en el total | 12 |

### D3. Tarjeta con [Meta]
* **Valor:** **48 880** (el filtro del objeto visual no afecta al resto de objetos del informe).

### D4. Medida de auditoría

```dax
Meta sin finca = CALCULATE( [Meta] , ISBLANK( h_meta[finca_id] ) )
```
* **Resultado en tarjeta:** **24 440**

### D5. Tabla por mes (mes 1 a 4)

| dim_tiempo[nombre_mes] | Kilos | Meta | Cumplimiento |
| :--- | :---: | :---: | :---: |
| Enero | — | 8 080 | — |
| Febrero | — | 10 400 | — |
| Marzo | 10 800 | 13 800 | 78,26 % |
| Abril | 19 750 | 16 600 | 118,98 % |
| **Total** | **30 550** | **48 880** | **62,50 %** |

* **¿Aparece alguna fila (En blanco)? ¿Por qué no?**  
  No aparece ninguna fila (En blanco) porque todas las filas de la meta tienen una fecha válida relacionada con `dim_tiempo`. La fila de total se absorbe dentro de cada mes y duplica su meta.

### D6. Metas reales calculadas a mano (meses 1 a 4)

| Mes | Santa Rosa | El Guayabo | La Union | Meta del mes | Kilos | Cumplimiento |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| Enero | 1 500 | 1 440 | 1 100 | 4 040 | — | — |
| Febrero | 2 000 | 2 000 | 1 200 | 5 200 | — | — |
| Marzo | 3 000 | 2 500 | 1 400 | 6 900 | 10 800 | 156,52 % |
| Abril | 3 500 | 3 500 | 1 300 | 8 300 | 19 750 | 237,95 % |
| **Total** | **10 000** | **9 440** | **5 000** | **24 440** | **30 550** | **125,00 %** |

### D7. Diagnóstico del filtro visual
El filtro de D1 solo ocultó visualmente la fila huérfana en una tabla concreta, pero no quitó la fila Total del modelo. La prueba del número viejo pasó en esa tabla porque `dim_finca` excluyó las filas sin clave foránea, mientras que en la dimensión de tiempo y en las tarjetas la meta seguía cobrándose dos veces.

---

## Parte E · El arreglo, en Power Query

* **E2. Filas restantes tras filtrar finca_id desmarcando (null):**  
  Quedan exactamente **36 filas** y la calidad de columna marca **0 % vacío / 100 % válido**.

### E3. Tabla de control limpia (sin filtros en el visual, mes 1 a 4)

| dim_finca[finca] | Kilos | Cosechas | Meta | Cumplimiento | Filas de meta |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Agricola La Union | 2 100 | 2 | 5 000 | 42,00 % | 4 |
| Finca El Guayabo | 14 250 | 3 | 9 440 | 150,95 % | 4 |
| Hacienda Santa Rosa | 14 200 | 4 | 10 000 | 142,00 % | 4 |
| **Total** | **30 550** | **9** | **24 440** | **125,00 %** | **12** |

* **Tarjetas:** `[Meta] = 24 440` y `[Meta sin finca]` está vacía (`(En blanco)`).  
*(Se adjunta captura `clase29-metas.png`)*

### E4. Tabla por mes corregida (mes 1 a 4)

| dim_tiempo[nombre_mes] | Kilos | Meta | Cumplimiento |
| :--- | :---: | :---: | :---: |
| Enero | — | 4 040 | — |
| Febrero | — | 5 200 | — |
| Marzo | 10 800 | 6 900 | 156,52 % |
| Abril | 19 750 | 8 300 | 237,95 % |
| **Total** | **30 550** | **24 440** | **125,00 %** |

* Coinciden exactamente con los cálculos a mano de la Parte D6.

### E5. Tabla anual por finca (anio = 2026 completo)

| dim_finca[finca] | Kilos | Meta | Cumplimiento | Filas de meta |
| :--- | :---: | :---: | :---: | :---: |
| Hacienda Santa Rosa | 14 200 | 20 000 | 71,00 % | 12 |
| Finca El Guayabo | 14 250 | 19 000 | 75,00 % | 12 |
| Agricola La Union | 2 100 | 8 000 | 26,25 % | 12 |
| **Total** | **30 550** | **47 000** | **65,00 %** | **36** |

* Cada `[Meta]` coincide exactamente con la columna `Total` de la hoja original: 20 000, 19 000 y 8 000 kg, totalizando 47 000 kg.

### E6. Verificación con A5
Sí, acertaron las dos respuestas: 36 filas en la tabla de hechos y 47 000 kg de meta total anual.

---

## Parte F · La fórmula del paso, y preguntas de cierre

### F1. Línea literal de Table.UnpivotOtherColumns
```powerquery
= Table.UnpivotOtherColumns(#"Encabezados promovidos", {"finca_id", "finca"}, "Atributo", "Valor")
```
* **Columnas que nombra:** Nombra **las columnas que se quedan fijas** (`finca_id` y `finca`), no las que se desdinamizan.

### F2. Nuevas columnas en 2027
Al usar «otras columnas», al actualizar la consulta las nuevas columnas de 2027 se desdinamizan solas porque todo lo que no sea `finca_id` ni `finca` se convierte en filas. Si se hubiera usado «Anular solo dinamización de columnas seleccionadas», las columnas nuevas habrían quedado ignoradas como columnas sueltas sin incorporarse a las filas.

### F3. Preguntas de cierre
1. **¿Qué hace Anular dinamización, dicho en filas y columnas?**  
   Convierte nombres de columnas repetidas en valores de fila dentro de una sola columna de atributos, aumentando la cantidad de filas y reduciendo las columnas.
2. **Regla de detección del día:**  
   «Una hoja que trae subtotales o totales calculados no agregó resumen: agregó duplicados al desdinamizar».
3. **Filtro del objeto visual vs Quitar duplicados:**  
   Ambos son parches que atacan el síntoma y no el origen; el filtro de objeto visual deja el error dentro del modelo tabular distorsionando cualquier otro cálculo o visualización.
4. **Problema con la fila de total idéntica a la suma:**  
   Fue el problema porque, al desdinamizarla y conservarla en la tabla de hechos, el modelo sumó tanto los componentes como el total ya consolidado, sumando los kilos dos veces.
5. **Columna Observaciones al final:**  
   Aparecería dentro de la columna `fecha_mes` como un atributo más, y te avisaría al generar `Error` inmediato al intentar convertir la columna a tipo Fecha.