# Ejercicio 30 · La carpeta de la báscula

- **Alumno:** Axel Cortez
- **Entorno:** Power BI Desktop / Power Query / DAX

---

# Parte A · El modelo sin cosechas

## A3. Medida de Meta

```dax
Meta = SUM( h_meta[kg_meta] )
```

- **Resultado (2026, Meses 1–4):** 24 440.

## A4. Respuestas iniciales

### ¿Cuántos archivos hay en `bascula/`?

Hay **15 archivos CSV**.

### ¿Cuántas filas tiene que tener `h_cosecha` según el correo?

Debe tener **25 filas**, correspondientes a las cosechas reales.

### ¿Cuántos kilos hay en 2025?

El total esperado para 2025 es de **47 000 kg**.

---

# Parte B · Combinar la carpeta

## B2. Primer archivo de ejemplo

El primer archivo utilizado como ejemplo fue:

```text
bascula_2025-01.csv
```

Las columnas disponibles en la vista previa fueron:

- `cosecha_id`
- `finca_id`
- `cultivo_id`
- `fecha`
- `calidad`
- `kg`

## B3. Objetos creados en Power Query

Al realizar la combinación de archivos, Power Query creó los siguientes objetos.

### Grupo de ayuda

- `Transformar archivo de ejemplo de bascula`
- `Archivo de ejemplo`
- `Parámetro1`
- `Transformar archivo`

### Consulta principal

La consulta principal fue:

```text
bascula
```

Su estado inicial después de la combinación fue:

- **31 filas**
- **6 columnas**
- `Source.Name` como primera columna

## B5. Medidas DAX de Cosecha

### Medida Kilos

```dax
Kilos = SUM( h_cosecha[kg] )
```

- **Resultado (Meses 1–4, 2026 con copia):** 50 300.

### Medida Cosechas

```dax
Cosechas = COUNTROWS( h_cosecha )
```

- **Resultado (Meses 1–4, 2026 con copia):** 15.

### Medida Cumplimiento

```dax
Cumplimiento = DIVIDE( [Kilos], [Meta] )
```

- **Resultado (Meses 1–4, 2026 con copia):** 205,81 %.

---

# Parte C · La copia de abril

## C1. Tabla de control distorsionada

**Periodo:** Meses 1–4, 2026.

| Finca | Kilos | Cosechas | Meta | Cumplimiento |
|---|---:|---:|---:|---:|
| Santa Rosa | 18 800 | 6 | 10 000 | 188,00 % |
| El Guayabo | 28 500 | 6 | 9 440 | 301,91 % |
| La Union | 3 000 | 3 | 5 000 | 60,00 % |
| **Total** | **50 300** | **15** | **24 440** | **205,81 %** |

## C2. Tabla por `Source.Name`

| Source.Name | Kilos | Cosechas |
|---|---:|---:|
| `bascula_2026-03.csv` | 10 800 | 3 |
| `bascula_2026-04.csv` | 19 750 | 6 |
| `bascula_2026-04 - copia.csv` | 19 750 | 6 |

## C3. Intento con Quitar duplicados

Al seleccionar todas las columnas y aplicar **Quitar duplicados**, no se elimina ninguna fila.

Por lo tanto, permanecen las **31 filas**.

Esto ocurre porque la columna `Source.Name` contiene valores diferentes:

```text
bascula_2026-04.csv
bascula_2026-04 - copia.csv
```

Esta diferencia provoca que cada fila sea considerada única a nivel de registro completo, aunque los datos de las cosechas sean iguales.

## C4. Medida de auditoría de duplicados

```dax
Cosechas repetidas =
[Cosechas] -
DISTINCTCOUNT( h_cosecha[cosecha_id] )
```

### Resultados

- **Con archivo copia:** 6
- **Tras filtrar `"copia"`:** 0

## C5 y C6. Diagnóstico del filtro de texto

Al aplicar el filtro de texto:

```text
No contiene "copia"
```

sobre `Source.Name`, quedan **25 filas**.

La tabla de control regresa a los valores esperados:

- **Kilos:** 30 550
- **Meta:** 24 440
- **Cumplimiento:** 125,00 %
- **Cosechas:** 9
- **[Cosechas repetidas]:** 0

### Pregunta C6

Si posteriormente llega un archivo llamado:

```text
bascula_2026-05 (2).csv
```

el filtro no lo eliminará porque busca literalmente la subcadena:

```text
"copia"
```

Sin embargo, la medida `[Cosechas repetidas]` lo detectaría inmediatamente al devolver un valor mayor a cero.

---

# Parte D · El mes sin kilos

## D1 y D2. Tabla de los dos años

| Año | Kilos | Cosechas |
|---:|---:|---:|
| 2025 | 44 150 | 16 |
| 2026 | 30 550 | 9 |
| **Total** | **74 700** | **25** |

### Mes afectado

**Julio de 2025 (`2025-07`)** registra:

- **2 cosechas**
- `[Kilos]` vacío / en cero

## D3 y D4. Diagnóstico de calidad y encabezados

### Calidad de columna en `kg`

- **Válido:** 92 %
- **Error:** 0 %
- **Vacío:** 8 %

### Primera línea de `bascula_2025-06.csv`

```text
cosecha_id,finca_id,cultivo_id,fecha,kg
```

### Primera línea de `bascula_2025-07.csv`

```text
cosecha_id,finca_id,cultivo_id,fecha,kg_neto
```

### ¿Por qué no dio error?

Power Query expandió las columnas con base en el archivo de ejemplo, buscando específicamente una columna denominada:

```text
kg
```

Al evaluar el archivo correspondiente a julio, no encontró esa columna porque el encabezado era:

```text
kg_neto
```

En lugar de generar un error de tipo de dato, Power Query rellenó esos registros con valores `null`.

## D5. Medida de control de nulos

```dax
Cosechas sin kilos =
COUNTROWS(
    FILTER(
        h_cosecha,
        ISBLANK( h_cosecha[kg] )
    )
)
```

### Resultados

- **Antes de corregir:** 2
- **Tras el arreglo:** (En blanco)

## D6. Conciliación manual de archivos

**Periodo:** Mayo–Agosto 2025.

| Archivo | Filas | Kilos en el archivo | [Kilos] en Power BI | [Cosechas] en Power BI |
|---|---:|---:|---:|---:|
| `bascula_2025-05.csv` | 1 | 4 200 | 4 200 | 1 |
| `bascula_2025-06.csv` | 1 | 3 100 | 3 100 | 1 |
| `bascula_2025-07.csv` | 2 | 2 850 | 0 | 2 |
| `bascula_2025-08.csv` | 2 | 5 600 | 5 600 | 2 |
| **Total** | **6** | **15 750** | **12 900** | **6** |

## D7. Conclusiones técnicas

El control de enero–abril nunca detectó el fallo de julio porque filtra estrictamente los meses **1 a 4 del año 2026**.

En el ejercicio 29, el `null` en `finca_id` provenía de una fila de total agregado redundante que duplicaba datos dentro del modelo.

En este ejercicio, en cambio, el `null` en `kg` correspondía a registros legítimos de producción cuya columna fue interpretada como vacía debido a un cambio de encabezado en el archivo de origen.

---

# Parte E · El arreglo en el archivo de ejemplo

## E2. Transformación aplicada en `Transformar archivo de ejemplo`

```powerquery
= Table.RenameColumns(
    #"Encabezados promovidos",
    {{"kg_neto", "kg"}},
    MissingField.Ignore
)
```

Esta transformación permite normalizar el encabezado `kg_neto` convirtiéndolo en `kg`.

El uso de:

```powerquery
MissingField.Ignore
```

evita que se genere un error cuando el archivo ya contiene directamente la columna `kg`.

## E4. Conciliación final de los dos años

| Año | Kilos | Cosechas |
|---:|---:|---:|
| 2025 | 47 000 | 16 |
| 2026 | 30 550 | 9 |
| **Total** | **77 550** | **25** |

### Tarjetas de control

- **[Cosechas repetidas]:** 0
- **[Cosechas sin kilos]:** (En blanco)

### Tabla de control

**Periodo:** Meses 1–4, 2026.

```text
Kilos / Meta / Cumplimiento / Cosechas
30 550 / 24 440 / 125,00 % / 9
```

## E5. Verificación de meses corregidos

Después de aplicar la corrección, **julio de 2025** muestra correctamente:

- **2 850 kg**
- **2 cosechas**

De esta manera, el periodo comprendido entre mayo y agosto de 2025 totaliza correctamente:

```text
15 750 kg
```

## E6. Cotejo con A4

Se confirmaron las tres respuestas iniciales:

- **15 archivos** en la carpeta.
- **25 filas** correspondientes a cosechas reales.
- **47 000 kg** para el año 2025.

---

# Parte F · Preguntas de cierre

## F1. Línea literal de expansión en el Editor Avanzado

```powerquery
= Table.ExpandTableColumn(
    #"Filas filtradas",
    "Transformar archivo",
    Table.ColumnNames(
        #"Transformar archivo"(#"Archivo de ejemplo")
    )
)
```

Esta expresión extrae los nombres de las columnas evaluando dinámicamente el resultado de la función sobre el archivo de ejemplo.

## F2. Si el archivo de ejemplo hubiese sido julio

Si:

```text
bascula_2025-07.csv
```

hubiese sido utilizado como archivo de ejemplo, la columna esperada habría sido:

```text
kg_neto
```

Como los otros **14 archivos** contienen la columna `kg`, un total de **23 cosechas** habrían quedado con los kilos en `null`.

---

## F3. Respuestas conceptuales

### Combinar archivos

**Combinar archivos** permite leer todos los archivos contenidos en una carpeta y apilarlos verticalmente bajo la estructura definida por el archivo de ejemplo.

Conceptualmente, el comportamiento es equivalente a:

```text
UNION ALL
```

siempre que los archivos tengan una estructura compatible.

### Regla de detección del día

> **«Si un archivo de la carpeta cambia el nombre de una columna, Power Query no avisa con error: la rellena con nulos.»**

### Diferencia entre los fallos

La **copia de abril** duplicó datos dentro del periodo filtrado de enero–abril de 2026.

Esto rompió directamente los indicadores visibles en la tabla de control.

En cambio, la pérdida de kilos de **julio de 2025** ocurrió dentro del histórico, fuera del periodo utilizado por la tabla de control.

Por este motivo, el problema solamente se evidenció al:

- Auditar la calidad de la columna.
- Revisar los valores `null`.
- Evaluar el consolidado anual.
- Utilizar la medida `[Cosechas sin kilos]`.

### Ubicación del paso de renombrado

El paso de renombrado debe ubicarse dentro de la función de transformación del archivo de ejemplo.

De esta manera, la transformación:

```text
kg_neto → kg
```

se ejecuta sobre cada archivo individual **antes de que todos los archivos sean combinados**.

### Comportamiento ante una columna `kilos`

Si en noviembre llega un archivo cuya columna se denomina:

```text
kilos
```

en lugar de:

```text
kg
```

el encabezado no estará contemplado por el renombrado actual.

Por lo tanto, sus valores se cargarán como `null` en la columna `kg`.

La medida:

```dax
Cosechas sin kilos =
COUNTROWS(
    FILTER(
        h_cosecha,
        ISBLANK( h_cosecha[kg] )
    )
)
```

permitiría detectar inmediatamente el problema al devolver un valor mayor a cero.