# Ejercicio práctico 30 · La carpeta de la báscula

**Estudiante:** Daniel Troya  
**Archivo de entrega:** `Ejercicio30_Troya_Daniel.md`  

---

## Parte A · El modelo sin cosechas

### A1 & A2. Relaciones del modelo inicial
Se cargaron los archivos CSV `dim_tiempo`, `dim_finca` y `h_meta` con la detección automática de relaciones desactivada. Las relaciones creadas manualmente en el modelo fueron:
* `h_meta[finca_id]` (Muchos) → `dim_finca[finca_id]` (Uno)
* `h_meta[fecha_mes]` (Muchos) → `dim_tiempo[fecha]` (Uno)

### A3. Calendario y Medida de Meta
Se marcó `dim_tiempo` como tabla de fechas usando la columna `fecha`, y se ordenó `nombre_mes` por la columna numérica `mes`.

```dax
Meta = SUM( h_meta[kg_meta] )
```
* **Resultado anotado:** Total de **24 440 kg** para el período enero–abril de 2026.

#### Tabla de control inicial (Enero–Abril 2026)
| dim_finca[finca] | [Meta] |
| :--- | :--- |
| El Guayabo | 9 440 |
| La Union | 5 000 |
| Santa Rosa | 10 000 |
| **Total** | **24 440** |

### A4. Respuestas de inspección previa en el Explorador de Archivos
* **¿Cuántos archivos hay en `C:\agrodb30\bascula\`?:** Hay **15 archivos** CSV en total.
* **¿Cuántas filas tiene que tener `h_cosecha`, según el correo?:** Debe tener **25 filas** (correspondientes a las 25 cosechas históricas reales).
* **¿Cuántos kilos en 2025?:** Deben ser **47 000 kg** (según lo registrado en la clase 16).

---

## Parte B · Combinar la carpeta

### B2. Archivo de ejemplo
* **Primer archivo seleccionado:** `bascula_2025-01.csv`.
* **Columnas mostradas en la vista previa:** `cosecha_id`, `fecha`, `finca_id`, `kg`.

### B3. Panel de consultas y estructura inicial
* **Elementos generados en el panel de consultas:**
  * Carpeta auxiliar **Transformar archivo de bascula**:
    1. `Archivo de ejemplo` (archivo binario de referencia).
    2. `Parámetro1` (parámetro de archivo conectado al binario).
    3. `Transformar archivo de ejemplo` (consulta que define la transformación aplicada a cada archivo).
    4. `Transformar archivo` (función invocada para procesar cada archivo individual).
  * Consulta principal: `bascula` (renombrada a `h_cosecha`).
* **Dimensiones iniciales de la consulta `bascula`:** **31 filas** y **7 columnas** (contando columnas adicionales generadas por metadatos/atributos como `calidad` y `cultivo_id` junto con los campos base de cosecha).
* **Nombre de la primera columna:** `Source.Name`.

### B4 & B5. Limpieza, relaciones y medidas de cosecha
Se validaron los tipos de datos (`fecha` como Fecha y `kg` como Número entero), conservando las 5 columnas requeridas del modelo (`Source.Name`, `cosecha_id`, `fecha`, `finca_id`, `kg`) mediante el paso de quitar otras columnas. Se renombró la consulta a `h_cosecha` y se crearon las relaciones:
* `h_cosecha[finca_id]` (Muchos) → `dim_finca[finca_id]` (Uno)
* `h_cosecha[fecha]` (Muchos) → `dim_tiempo[fecha]` (Uno)

```dax
Kilos = SUM( h_cosecha[kg] )
```
* **Resultado inicial (con copia):** 50 300 kg en el control de enero–abril 2026.

```dax
Cosechas = COUNTROWS( h_cosecha )
```
* **Resultado inicial (con copia):** 15 cosechas en el control de enero–abril 2026 (31 cosechas en total).

```dax
Cumplimiento = DIVIDE( [Kilos] , [Meta] )
```
* **Resultado inicial (con copia):** 205,81 % en el control de enero–abril 2026.

---

## Parte C · La copia de abril

### C1. Tabla de control antes del filtro (Enero–Abril 2026)
| finca | Meta | Kilos | Cumplimiento | Cosechas |
| :--- | :--- | :--- | :--- | :--- |
| El Guayabo | 9 440 | 28 500 | 301,91 % | 9 |
| La Union | 5 000 | 3 000 | 60,00 % | 1 |
| Santa Rosa | 10 000 | 18 800 | 188,00 % | 5 |
| **Total** | **24 440** | **50 300** | **205,81 %** | **15** |

### C2. Tabla por `Source.Name` (Enero–Abril 2026)
*(Referencia visual para la captura `clase30-archivos.png`)*

| Source.Name | Kilos | Cosechas |
| :--- | :--- | :--- |
| `bascula_2026-01.csv` | 4 800 | 2 |
| `bascula_2026-02.csv` | 6 000 | 2 |
| `bascula_2026-03.csv` | 10 800 | 3 |
| `bascula_2026-04 - copia.csv` | 19 750 | 6 |
| `bascula_2026-04.csv` | 19 750 | 6 |
| **Total** | **41 350** | **15** |

### C3. Intento de Quitar duplicados con Ctrl + A
* **Filas resultantes:** Quedan exactamente las mismas **31 filas**.
* **¿Por qué no las quita?:** Porque la columna `Source.Name` está incluida en la selección evaluada y su valor es diferente en cada fila (`bascula_2026-04.csv` frente a `bascula_2026-04 - copia.csv`), por lo que a nivel de fila completa ningún registro es un duplicado idéntico.
* *(Paso borrado inmediatamente después de la comprobación).*

### C4. Medida de detección de duplicados
```dax
Cosechas repetidas = [Cosechas] - DISTINCTCOUNT( h_cosecha[cosecha_id] )
```
* **Resultado en tarjeta (antes del filtro):** **6**.

### C5. Filtrado del archivo duplicado
En Power Query, se aplicó el filtro para descartar la copia:
```powerquery
= Table.SelectRows(#"Otras columnas quitadas", each not Text.Contains([Source.Name], "copia"))
```
* **Filas resultantes:** **25 filas**.
* **Tabla de control actualizada (Enero–Abril 2026):** **30 550 / 24 440 / 125,00 % / 9**.
* **Tarjeta `[Cosechas repetidas]`:** **0**.

### C6. Respuesta sobre la detección de copias futuras
No lo quita, porque el filtro de texto busca de manera estricta la subcadena `"copia"` y no atrapará sufijos como `(2)`. Sin embargo, la medida `[Cosechas repetidas]` lo atraparía inmediatamente en el informe marcando un valor mayor a 0 al encontrar identificadores de cosecha repetidos.

---

## Parte D · El mes sin kilos

### D1. Tabla por Año (Sin segmentadores, antes de corregir julio)
| anio | Kilos | Cosechas |
| :--- | :--- | :--- |
| 2025 | 44 150 | 16 |
| 2026 | 30 550 | 9 |
| **Total** | **74 700** | **25** |

*Comparación con A4:* El número de cosechas (16) concuerda con lo esperado, pero los kilos suman 44 150 kg en lugar de los 47 000 kg previstos (hay una discrepancia exacta de 2 850 kg faltantes).

### D2. Tabla mensual de 2025
| anio_mes | Kilos | Cosechas |
| :--- | :--- | :--- |
| 2025-01 | 4 200 | 1 |
| 2025-02 | 3 500 | 1 |
| 2025-03 | 5 100 | 2 |
| 2025-04 | 4 000 | 1 |
| 2025-05 | 4 200 | 1 |
| 2025-06 | 3 100 | 1 |
| 2025-07 | *(en blanco)* | 2 |
| 2025-08 | 5 600 | 2 |
| 2025-09 | 3 450 | 1 |
| 2025-10 | 4 100 | 1 |
| 2025-11 | 3 000 | 1 |
| 2025-12 | 3 900 | 2 |
| **Total** | **44 150** | **16** |

* **Mes afectado:** **Julio de 2025 (`2025-07`)** registra 2 cosechas pero sus kilos aparecen vacíos.

### D3. Calidad de Columna en `kg`
* **Válido:** 92 %
* **Error:** 0 %
* **Vacío:** 8 % (2 de 25 filas).
* **Filas al filtrar `(null)`:** Quedan 2 filas, ambas provenientes de `Source.Name = bascula_2025-07.csv`.
* *(Paso de filtro borrado después de la comprobación).*

### D4. Comparación de encabezados en el Bloc de notas
* **Primera línea de `bascula_2025-07.csv`:**
  `cosecha_id,fecha,finca_id,kg_neto`
* **Primera línea de `bascula_2025-06.csv`:**
  `cosecha_id,fecha,finca_id,kg`
* **Explicación técnica:** El encabezado cambió de `kg` a `kg_neto`. Power Query no genera un `Error` porque al expandir las tablas busca la lista fija de columnas del archivo de ejemplo (`cosecha_id`, `fecha`, `finca_id`, `kg`); al no encontrar el campo `kg` en julio, rellena la columna con valores `null` e ignora `kg_neto` por no estar incluido en la lista de expansión.

### D5. Prueba del segundo arreglo obvio (Filtrar `(null)`)
Resultados tras desmarcar `(null)` en la columna `kg`:
| Prueba | Resultado obtenido |
| :--- | :--- |
| Calidad de columna de `kg` | 100 % válido |
| Control de enero–abril | 30 550 / 125,00 % |
| La tabla de D1, 2025 | 44 150 / 14 cosechas |
| La tabla de D1, total | 74 700 / 23 cosechas |

```dax
Cosechas sin kilos = COUNTROWS( FILTER( h_cosecha , ISBLANK( h_cosecha[kg] ) ) )
```
* **Tarjeta con el filtro de `(null)` activo:** Vacía (`BLANK`).
* **Tarjeta al borrar el filtro de `(null)`:** **2**.

### D6. Cuadre a mano vs. Power BI (Mayo a Agosto de 2025)
| Archivo | Filas | Kilos en el archivo | `[Kilos]` en Power BI | `[Cosechas]` en Power BI |
| :--- | :--- | :--- | :--- | :--- |
| `bascula_2025-05.csv` | 1 | 4 200 | 4 200 | 1 |
| `bascula_2025-06.csv` | 1 | 3 100 | 3 100 | 1 |
| `bascula_2025-07.csv` | 2 | 2 850 | *(en blanco)* | 2 |
| `bascula_2025-08.csv` | 2 | 5 600 | 5 600 | 2 |
| **Total** | **6** | **15 750** | **12 900** | **6** |

### D7. Respuestas de análisis
* **¿Por qué el control de enero–abril nunca vio el error?:** Porque está condicionado por segmentadores estrictos para enero–abril de 2026, mientras que la falta de datos ocurrió en julio de 2025, quedando fuera de su contexto de evaluación.
* **Diferencia entre los `null`:** En la clase anterior, el `null` en `finca_id` representaba una fila residual de total que debía eliminarse; hoy el `null` en `kg` correspondía a registros legítimos de cosechas reales cuyo peso se perdió debido a una discrepancia de nombre de columna.

---

## Parte E · El arreglo en el archivo de ejemplo

### E1 & E2. Paso en Transformar archivo de ejemplo
* **Pasos aplicados iniciales:** `Origen`, `Encabezados promovidos`.
* **Fórmula ingresada tras presionar `fx`:**
```powerquery
= Table.RenameColumns( #"Encabezados promovidos" , {{"kg_neto", "kg"}} , MissingField.Ignore )
```

### E3 & E4. Validación final del modelo
* **Calidad de columna de `kg` en `h_cosecha`:** **100 % válido (0 % vacío)** sobre las **25 filas**.
* **Tarjeta `[Cosechas repetidas]`:** **0**.
* **Tarjeta `[Cosechas sin kilos]`:** **(en blanco)**.
* **Control enero–abril 2026:** **30 550 / 24 440 / 125,00 % / 9**.

#### Tabla final de los dos años (Referencia para `clase30-anios.png`)
| anio | Kilos | Cosechas |
| :--- | :--- | :--- |
| 2025 | 47 000 | 16 |
| 2026 | 30 550 | 9 |
| **Total** | **77 550** | **25** |

* **Detalle de julio de 2025:** **2 850 kg / 2 cosechas**.

### E5. Cuadre final Mayo–Agosto 2025
Con la transformación en el archivo de ejemplo, `bascula_2025-07.csv` refleja en Power BI exactamente sus **2 850 kg**, completando el consolidado de **15 750 kg** y **6 cosechas** en el bloque analizado.

### E6. Validación de respuestas de A4
Se acertaron las tres respuestas iniciales: 15 archivos en la carpeta, 25 filas finales de cosecha y 47 000 kg para el año 2025.

---

## Parte F · La fórmula del paso y preguntas de cierre

### F1. Línea literal en el Editor Avanzado
```powerquery
#"Columnas de tabla expandidas1" = Table.ExpandTableColumn(#"Filas filtradas", "Transformar archivo", {"cosecha_id", "fecha", "finca_id", "kg"}, {"cosecha_id", "fecha", "finca_id", "kg"})
```
* **¿De dónde saca la lista de columnas que expande?:** La lista de columnas queda codificada de forma fija (*hardcoded*) en el paso, generada a partir de los encabezados encontrados en el **Archivo de ejemplo** al momento de ejecutar la combinación inicial.

### F2. Pregunta sobre archivo de ejemplo alternativo
Si el archivo de ejemplo hubiera sido `bascula_2025-07.csv`, la función habría buscado el encabezado `kg_neto` en todos los archivos. Como los otros 14 archivos contienen `kg` y no `kg_neto`, un total de **23 cosechas** habrían quedado con sus kilos en `null`.

### F3. Preguntas de cierre
1. **¿Qué hace Combinar archivos con una carpeta, dicho en archivos y filas?:** Lee cada archivo de la carpeta individualmente, le aplica la función de transformación auxiliar y anexa verticalmente todas sus filas en una única tabla continua.
2. **Regla de detección del día:** *«Un cambio en el nombre de una columna entre archivos es, para Power Query, una columna nueva vacía y una columna existente perdida»*.
3. **¿Por qué el control vio una y la otra no?:** La copia de abril infló directamente el período filtrado por los segmentadores de la tabla de control (2026), mientras que la anomalía de julio se encontraba en 2025, fuera del filtro temporal activo.
4. **¿Por qué renombrar va en el archivo de ejemplo y no en `h_cosecha`?:** Porque la función auxiliar se evalúa sobre cada archivo individual antes de unirlos; si se aplicara directamente en `h_cosecha`, la columna `kg_neto` ya habría sido descartada durante el paso de expansión.
5. **Comportamiento si la báscula escribe `kilos` en noviembre:** Los registros se importarán con sus filas y fechas pero con `kg` en `null` (al no coincidir con el nombre `kg`). La medida `[Cosechas sin kilos]` dejará de estar vacía e indicará el número exacto de cosechas ingresadas sin peso.