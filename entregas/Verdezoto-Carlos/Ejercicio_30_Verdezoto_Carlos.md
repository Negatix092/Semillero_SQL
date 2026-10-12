# Ejercicio 30 · La carpeta de la báscula

## PARTE A - EL MODELO SIN COSECHAS

**A4. Respuestas de exploración inicial:**
* ¿Cuántos archivos hay? Hay **15 archivos** en la carpeta `bascula/` (un archivo por mes más la copia)[cite: 27].
* ¿Cuántas filas tiene que tener `h_cosecha`? Según las clases previas y el correo, debe tener **25 cosechas**[cite: 27].
* ¿Cuántos kilos en 2025? Tienen que ser **47 000 kg**[cite: 27].

---

## PARTE B - COMBINAR LA CARPETA

**B2.** El primer archivo (usado como ejemplo) suele ser `bascula_2025-05.csv` por orden alfabético. Muestra las columnas del CSV (`cosecha_id`, `finca_id`, `cultivo_id`, `fecha`, `kg`)[cite: 27].

**B3.** En el panel de consultas apareció la carpeta `Transformar archivo de ejemplo` con el archivo de muestra, el parámetro, y la función transformadora. En `bascula` dice **31 filas** y la primera columna se llama **`Source.Name`**[cite: 27].

### Medidas base
```dax
Meta = SUM( h_meta[kg_meta] )

```

```dax
Kilos = SUM( h_cosecha[kg] )

```

```dax
Cosechas = COUNTROWS( h_cosecha )

```

```dax
Cumplimiento = DIVIDE( [Kilos] , [Meta] )

```

---

## PARTE C - LA COPIA DE ABRIL

**C3.** Al hacer `Quitar duplicados` con `Ctrl + A`, quedan las mismas **31 filas**. Esto ocurre porque la columna `Source.Name` es distinta para las filas duplicadas (`bascula_2026-04 - copia.csv` vs `bascula_2026-04.csv`), lo que hace que la fila completa no sea un duplicado exacto para Power Query.

### Medida de control de duplicados

```dax
Cosechas repetidas = [Cosechas] - DISTINCTCOUNT( h_cosecha[cosecha_id] )

```

*Resultado en tarjeta antes del filtro: **6***.
*Resultado tras filtrar "no contiene copia": **0***.

**C6.** El filtro de "no contiene `copia`" **no quitaría** el archivo `bascula_2026-05 (2).csv` porque no lleva la palabra literal "copia" en su nombre. La medida que sí lo atraparía y nos avisaría es **`[Cosechas repetidas]`**, ya que contaría filas adicionales con el mismo `cosecha_id`.

---

## PARTE D - EL MES SIN KILOS

**D2.** El mes de 2025 que tiene cosechas pero no tiene kilos es **Julio** (2025-07).

**D4.** En el archivo de julio, el encabezado de la columna cambió a **`kg_neto`** (en lugar de `kg`). Power Query no dio ningún `Error` porque al no encontrar la columna `kg` en ese archivo en particular, simplemente la rellenó con valores `null` (vacíos) al anexarla con los demás.

### Medida de control de nulos

```dax
Cosechas sin kilos = COUNTROWS( FILTER( h_cosecha , ISBLANK( h_cosecha[kg] ) ) )

```

*Resultado con los `null` presentes: **2***.

**D6. Cálculo a mano de las cosechas (Mayo - Agosto 2025):**

| Archivo | Filas | Kilos en el archivo | `[Kilos]` en Power BI | `[Cosechas]` en Power BI |
| --- | --- | --- | --- | --- |
| `bascula_2025-05.csv` | 1 | 4 200 | 4 200 | 1 |
| `bascula_2025-06.csv` | 1 | 3 100 | 3 100 | 1 |
| `bascula_2025-07.csv` | 2 | 2 850 | (En blanco) | 2 |
| `bascula_2025-08.csv` | 2 | 5 600 | 5 600 | 2 |
| **Total** | **6** | **15 750** | **12 900** | **6** |
| *(Los 12 900 de Power BI se deben a que faltan los 2 850 de julio).* |  |  |  |  |

**D7.**

* ¿Por qué el control de enero-abril nunca lo vio? Porque el archivo roto era de **julio de 2025**, el cual quedaba completamente excluido por los segmentadores de enero a abril de 2026 del lienzo.


* ¿Qué era el null cada vez? El martes (clase anterior) el `null` era la ausencia de una llave foránea (`finca_id`); hoy el `null` es la ausencia del valor numérico del hecho (`kg`), lo que hace que no se sume nada en los cálculos aritméticos.



---

## PARTE E - EL ARREGLO EN EL ARCHIVO DE EJEMPLO

### Paso corregido en el archivo de ejemplo

```powerquery
= Table.RenameColumns( #"Encabezados promovidos" , {{"kg_neto", "kg"}} , MissingField.Ignore )

```

**E6.** Sí, se acertaron las respuestas de A4: la tabla final arroja los 47 000 kilos en 2025, un total de 25 cosechas y el control volvió a sus números estables y sin errores.

---

## PARTE F - LA FÓRMULA DEL PASO, Y PREGUNTAS DE CIERRE

**F1.** `Table.ExpandTableColumn` saca la lista de columnas quemándolas en duro ("hardcoded") directamente en la fórmula, basándose exactamente en las columnas que tenía el **archivo de ejemplo** en el momento de crear la combinación.

**F2.** Si el archivo de ejemplo hubiera sido el de julio (`bascula_2025-07.csv`), Power Query habría asumido que la columna correcta era `kg_neto` y la habría buscado en los otros 14 archivos. Como esos archivos tenían `kg`, **23 cosechas** (las de los otros meses) habrían salido sin kilos (como `null`).

**F3. Preguntas de cierre:**

1. **¿Qué hace Combinar archivos?** Apila las **filas** de todos los **archivos** de la carpeta hacia abajo (como un gran *Anexar*), unificándolos según la estructura de columnas del archivo de ejemplo.


2. **Regla de detección:** *"Si al combinar una carpeta aparecen filas con valores en blanco sin arrojar un Error de tipo, a ese archivo en particular le cambiaron el nombre del encabezado"* (o *"un archivo que pierde una columna entera se rellena con nulos silenciosamente"*).


3. **¿Por qué atrapó una y la otra no?** La copia de abril (50 300 kg) afectaba a abril de 2026, cayendo de lleno dentro del filtro del objeto visual del control. Julio (2025) estaba fuera de la ventana de tiempo vigilada.


4. **¿Por qué en Transformar archivo de ejemplo?** Porque el renombramiento debe ocurrir archivo por archivo *antes* de que Power Query los expanda y mezcle. Si se hace en `h_cosecha`, la columna "vieja" ya se perdió en la expansión.


5. **¿Qué pasa al Actualizar?** El nuevo archivo con `kilos` entrará, pero sus kilos quedarán como `null` en la columna unificada `kg`. La medida **`[Cosechas sin kilos]`** nos lo avisará inmediatamente al dejar de estar vacía.