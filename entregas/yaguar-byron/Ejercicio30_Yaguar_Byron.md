# Ejercicio 30 · La carpeta de la báscula
## Byron Yaguar Rios

---

## PARTE A · El modelo sin cosechas

### A0 · Configuración regional
`Archivo → Opciones y configuración → Opciones → Archivo actual → Configuración regional` → **Español (México)** → Aceptar.

### A1 · Cargar
1. **Archivo actual → Carga de datos**: apaga "Detectar automáticamente nuevas relaciones después de cargar los datos".
2. Con **Cargar**, uno por uno: `dim_tiempo`, `dim_finca`, `h_meta`. **La carpeta `bascula` todavía no.**

### A2 · Relaciones de las metas
| # | De | A |
|---|---|---|
| 1 | `h_meta[finca_id]` | `dim_finca[finca_id]` |
| 2 | `h_meta[fecha_mes]` | `dim_tiempo[fecha]` |

### A3 · Calendario y meta
1. `dim_tiempo` → Marcar como tabla de fechas → `fecha`. `nombre_mes` → Ordenar por columna → `mes`.

```dax
Meta = SUM( h_meta[kg_meta] )
```

Segmentadores: `anio` = 2026, `mes` = 1 a 4. Tabla `dim_finca[finca]` + `[Meta]`:

> ### ✅ Punto de control
> | Finca | `[Meta]` |
> |---|---|
> | Agricola La Union | **5 000** |
> | Finca El Guayabo | **9 440** |
> | Hacienda Santa Rosa | **10 000** |
> | **Total** | **24 440** |

Resultado anotado: con `anio` = 2026 y `mes` 1 a 4, la tabla da Agricola La Union **5 000**, Finca El Guayabo **9 440**, Hacienda Santa Rosa **10 000**, total **24 440**. Coincide con el punto de control.

**A4.** Abre `C:\agrodb30\bascula\` (vista Detalles). Antes de seguir:
- ¿Cuántos archivos hay?
- ¿Cuántas filas debe tener `h_cosecha`, según el correo?
- ¿Cuántos kilos en 2025?

Respuesta: **15 archivos**: los 12 meses de 2025 (`bascula_2025-01.csv` a `bascula_2025-12.csv`), más `bascula_2026-03.csv`, `bascula_2026-04.csv` y `bascula_2026-04 - copia.csv`. `h_cosecha` debe tener **25 filas** (25 cosechas: "las mismas de siempre", un archivo por mes). Si Power BI trae más de 25, es porque está leyendo algo repetido (la copia de abril). En **2025** deben sumar **47 000 kg**, como dijo la clase 16.

---

## PARTE B · Combinar la carpeta

**B2.** Archivo de ejemplo = Primer archivo. ¿Cuál es el primero y qué columnas enseña la vista previa?

Respuesta: El primero es `bascula_2025-01.csv` (el primero de la carpeta por orden de nombre; la vista previa lo confirma con la fecha `2025-01-24`). La ventana "Combinar archivos" muestra seis columnas: `cosecha_id`, `finca_id`, `cultivo_id`, `fecha`, `calidad` y `kg`. Con "Archivo de ejemplo: Primer archivo", origen UTF-8, delimitador Coma y detección del tipo "Basado en las primeras 200 filas". Todas las columnas salen todavía como texto (ABC) y la vista previa enseña una fila: cosecha 10, finca 3, cultivo 3, 2025-01-24, primera, 2000.

**B3.** Todo lo que apareció en el panel de consultas (`bascula` + carpeta de ayuda):

Respuesta: Al combinar la carpeta aparecieron la consulta **`bascula`** (la combinación, que después renombré a `h_cosecha`) y la carpeta **"Transformar archivo de bascula"**, que trae **cuatro cosas de ayuda**:
1. **`Archivo de ejemplo`**: el archivo binario de muestra (el primero de la carpeta).
2. **`Parámetro`**: el parámetro que recibe el archivo de ejemplo.
3. **`Transformar archivo`**: la función (fx) que se aplica a cada archivo de la carpeta.
4. **`Transformar Archivo de ejemplo`**: la consulta con los pasos aplicados al archivo de ejemplo. Es donde se arregla un problema para que valga en todos los archivos (la parte E).

Las tres primeras están dentro de la subcarpeta "Consultas auxiliares".

En `bascula`: filas / columnas (abajo a la izquierda) y nombre de la **primera** columna:

Respuesta: **31 filas** y **7 columnas** (abajo a la izquierda: "Columnas: 7, Filas: 31"). La primera columna es **`Source.Name`**, que Power Query agrega sola con el nombre del archivo de donde sale cada fila; después vienen las 6 del archivo (`cosecha_id`, `finca_id`, `cultivo_id`, `fecha`, `calidad`, `kg`). Pasos aplicados de la consulta: Origen, Archivos ocultos filtrados, Personalizado agregado, Invocar función personalizada, Columnas con nombre cambiado, Se han quitado otras columnas, Columna de tabla expandida y Tipo de columna cambiado.

> ### ✅ Punto de control (del troubleshooting)
> `bascula`: **31 filas** (con `Source.Name`)

**B4.** `fecha` = Fecha, `kg` = Número entero. Renombrar `bascula` → **`h_cosecha`**. Cerrar y aplicar.

**B5.** Relaciones y medidas:

| # | De | A |
|---|---|---|
| 3 | `h_cosecha[finca_id]` | `dim_finca[finca_id]` |
| 4 | `h_cosecha[fecha]` | `dim_tiempo[fecha]` |

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

## PARTE C · La copia de abril

**C1.** Tabla de control con `[Meta]`, `[Kilos]`, `[Cumplimiento]`, `[Cosechas]`:

> ### ✅ Punto de control
> | Finca | `[Kilos]` | `[Meta]` | `[Cumplimiento]` | `[Cosechas]` |
> |---|---|---|---|---|
> | Agricola La Union | 3 000 | 5 000 | **60,00 %** | |
> | Finca El Guayabo | 28 500 | 9 440 | **301,91 %** | |
> | Hacienda Santa Rosa | 18 800 | 10 000 | **188,00 %** | |
> | **Total** | **50 300** | **24 440** | **205,81 %** | **15** |

Tu tabla (con `anio` = 2026 y `mes` 1 a 4):

| Finca | `[Kilos]` | `[Meta]` | `[Cumplimiento]` | `[Cosechas]` |
|---|---|---|---|---|
| Agricola La Union | 3 000 | 5 000 | 60,00 % | 3 |
| Finca El Guayabo | 28 500 | 9 440 | 301,91 % | 6 |
| Hacienda Santa Rosa | 18 800 | 10 000 | 188,00 % | 6 |
| **Total** | **50 300** | **24 440** | **205,81 %** | **15** |

Coincide con el punto de control. Los kilos pasaron de 30 550 (los reales) a 50 300 porque la copia de abril se está sumando otra vez: 19 750 de más.

**C2.** Tabla `h_cosecha[Source.Name]`, `[Kilos]`, `[Cosechas]`:

> ### ✅ Punto de control
> | `Source.Name` | `[Kilos]` | `[Cosechas]` |
> |---|---|---|
> | `bascula_2026-03.csv` | 10 800 | 3 |
> | archivo de abril (1) | 19 750 | 6 |
> | archivo de abril (copia) | 19 750 | 6 |

Tu tabla (con `anio` = 2026 y `mes` 1 a 4):

| `Source.Name` | `[Kilos]` | `[Cosechas]` |
|---|---|---|
| `bascula_2026-03.csv` | 10 800 | 3 |
| `bascula_2026-04 - copia.csv` | 19 750 | 6 |
| `bascula_2026-04.csv` | 19 750 | 6 |
| **Total** | **50 300** | **15** |

Coincide con el punto de control: los dos archivos de abril traen exactamente las mismas 6 cosechas con 19 750 kg, y por eso el total se infla.

> **Captura `clase30-archivos.png`**: la tabla por `Source.Name` con la copia de abril a la vista, ANTES de filtrarla.

**C3.** Quitar duplicados (Ctrl+A → clic derecho en encabezado). ¿Cuántas filas quedan? En una línea: ¿por qué? Luego **borra ese paso**.

Respuesta: Quedan **31 filas**: no se quitó ninguna. Quitar duplicados (`Table.Distinct`) compara las 7 columnas, incluida `Source.Name`, y la copia de abril solo se distingue por eso: sus 6 filas son idénticas a las de `bascula_2026-04.csv` en `cosecha_id`, `finca_id`, `cultivo_id`, `fecha`, `calidad` y `kg`, pero con otro `Source.Name` (`bascula_2026-04 - copia.csv`), así que para Power Query ninguna fila es repetida. El paso lo borré (Pasos aplicados → X en "Duplicados quitados").

**C4.**
```dax
Cosechas repetidas = [Cosechas] - DISTINCTCOUNT( h_cosecha[cosecha_id] )
```
> ### ✅ Punto de control
> `[Cosechas repetidas]`: **6**

Resultado anotado: la tarjeta con `[Cosechas repetidas]` dio **6**. Son las 6 cosechas de abril que están dos veces (con los segmentadores de enero a abril de 2026: `[Cosechas]` = 15 contra 9 `cosecha_id` distintos). A diferencia de Quitar duplicados, esta medida cuenta por `cosecha_id` y por eso sí atrapa la copia.

**C5.** Filtro de `Source.Name` → No contiene → `copia`.

> ### ✅ Punto de control
> Filas: **25** — Tabla de control: **30 550 / 24 440 / 125,00 % / 9** — `[Cosechas repetidas]`: **0**

Resultado anotado: el filtro quedó como `Table.SelectRows(#"Tipo de columna cambiado", each not Text.Contains([Source.Name], "copia"))` y `h_cosecha` quedó en **25 filas** (barra de abajo: "Columnas: 7, Filas: 25"). Tras Cerrar y aplicar: la tabla por `Source.Name` da `bascula_2026-03.csv` 10 800 / 3 y `bascula_2026-04.csv` 19 750 / 6 (total 30 550 / 9); la tabla de control da La Union 2 100 / 5 000 / 42,00 % / 2, El Guayabo 14 250 / 9 440 / 150,95 % / 3, Santa Rosa 14 200 / 10 000 / 142,00 % / 4, total **30 550 / 24 440 / 125,00 % / 9**; y `[Cosechas repetidas]` = **0**.

**C6.** En dos líneas: el filtro de C5 quita la copia de hoy. El mes que viene alguien deja `bascula_2026-05 (2).csv`. ¿Lo quita? ¿Cuál de tus medidas lo atraparía?

Respuesta: No lo quita: el filtro solo descarta los archivos cuyo nombre contiene la palabra "copia", y `bascula_2026-05 (2).csv` no la tiene, así que entraría y duplicaría las cosechas de mayo otra vez. Lo atraparía `[Cosechas repetidas]`, que dejaría de estar en 0 porque cuenta por `cosecha_id` y no por el nombre del archivo.

---

## PARTE D · El mes sin kilos (la parte que más vale)

**D1.** Página nueva, SIN segmentadores. Tabla `dim_tiempo[anio]`, `[Kilos]`, `[Cosechas]`:

> ### ✅ Punto de control
> | Año | `[Kilos]` | `[Cosechas]` |
> |---|---|---|
> | 2025 | **44 150** | **16** |
> | 2026 | **30 550** | **9** |
> | **Total** | **74 700** | **25** |

Tu tabla, comparada con A4:

| Año | `[Kilos]` | `[Cosechas]` |
|---|---|---|
| 2025 | 44 150 | 16 |
| 2026 | 30 550 | 9 |
| **Total** | **74 700** | **25** |

(En la captura también aparece `[Meta]` como columna extra: 2026 = 47 000 y 2025 vacía; no se pedía, no cambia nada.)

Coincide con el punto de control. Comparada con A4: las **25 cosechas** sí cuadran (16 de 2025 + 9 de 2026), pero en 2025 esperaba **47 000 kg** y Power BI da **44 150**: faltan **2 850 kg**. Las cosechas están contadas, pero a una parte de ellas les falta el peso.

**D2.** Tabla `dim_tiempo[anio_mes]`, `[Kilos]`, `[Cosechas]`. Meses de 2025. ¿Qué mes tiene cosechas y no tiene kilos?

Respuesta: **Julio de 2025**: aparece con **2 cosechas** y la columna `[Kilos]` **vacía**. Los otros meses de 2025 traen kilos: enero 2 000 / 1, febrero 3 800 / 1, marzo 8 200 / 2, abril 9 500 / 2, mayo 4 200 / 2, junio 3 100 / 1, agosto 5 600 / 1, septiembre 1 800 / 1, octubre 3 150 / 1, noviembre 1 600 / 1 y diciembre 1 200 / 1. Julio explica los 2 850 kg que faltan frente a los 47 000 de A4.

**D3.** Calidad de columna de `kg`: válido / error / vacío.

> ### ✅ Punto de control
> `kg`: **8 %** vacío

Filtro de `kg` → solo `(null)`: filas que quedan y su `Source.Name`. Luego borra ese paso.

Respuesta: Calidad de columna de `kg` (con las 25 filas): **92 % válido, 0 % error, 8 % vacío**. Las filas con `kg` = `null` son **2**, y las dos vienen del mismo archivo: **`bascula_2025-07.csv`** (`cosecha_id` 19, finca 2, 8/7/2025, y `cosecha_id` 20, finca 3, 25/7/2025; ambas de calidad "primera"). Es decir, los dos pesos de julio de 2025 llegaron vacíos.

**D4.** Primera línea literal del archivo de D3 y de `bascula_2025-06.csv`. En dos líneas: ¿qué cambió, y por qué Power Query no dio ningún `Error`?

Respuesta:

Primera línea de `bascula_2025-07.csv` (el archivo de D3):
```
cosecha_id,finca_id,cultivo_id,fecha,calidad,kg_neto
```
Primera línea de `bascula_2025-06.csv`:
```
cosecha_id,finca_id,cultivo_id,fecha,calidad,kg
```
Qué cambió: en el archivo de julio la última columna se llama **`kg_neto`** en vez de **`kg`**; el resto de la línea es igual. Sus dos filas sí traen peso (19: 1 850 kg y 20: 1 000 kg, o sea 2 850 en total).

Por qué no hubo `Error`: Power Query combina los archivos buscando las columnas por **nombre**, tomando como molde el archivo de ejemplo (el primero, que usa `kg`). En el archivo de julio no existe ninguna columna llamada `kg`, así que esa columna queda en `null` y `kg_neto` se ignora. Nada falló al convertir: simplemente no encontró dato, y un `null` no es un `Error`, por eso la calidad de columna dio 0 % error y 8 % vacío.

**D5.** Segundo arreglo obvio: filtro de `kg` → desmarca `(null)`. Cerrar y aplicar.

> ### ✅ Punto de control
> | Prueba | Tiene que decir |
> |---|---|
> | Calidad de columna de `kg` | 100 % válido |
> | Control enero–abril | 30 550 / 125,00 % |
> | Tabla de D1, 2025 | 44 150 / **14** |
> | Tabla de D1, total | 74 700 / **23** |

Resultados anotados, con el filtro de `(null)` puesto (`h_cosecha` en 23 filas):
- Calidad de `kg`: **100 % válido**.
- Tabla de D1: 2025 = **44 150 / 14**; 2026 = 30 550 / 9; total = **74 700 / 23**.
- Control de enero–abril (`anio` = 2026, `mes` 1 a 4): **30 550 / 24 440 / 125,00 % / 9**, igual que antes. En la captura sigue así con `[Cosechas repetidas]` = 0, y el filtro de `kg` no lo mueve porque ninguna fila de 2026 tiene `kg` vacío.
- Los kilos no cambiaron (74 700) y las cosechas bajaron de 25 a 23: el filtro borró 2 cosechas reales (las de julio) y no recuperó ni un kilo.

```dax
Cosechas sin kilos = COUNTROWS( FILTER( h_cosecha , ISBLANK( h_cosecha[kg] ) ) )
```
Tarjeta con el filtro puesto: vacía (`--`). El propio filtro ya había borrado las 2 filas con `kg` vacío, así que la medida no encuentra nada que contar. Es lo que hace engañoso este arreglo: el síntoma desaparece, pero las 2 cosechas de julio ya no existen en el modelo.
Tarjeta tras borrar el filtro de `(null)` (tiene que decir **2**): dice **2**. Con el filtro quitado, `[Cosechas repetidas]` sigue en 0 y la tabla por `Source.Name` vuelve a **74 700 / 25**.

**D6.** Con los archivos abiertos, a mano, kilos de mayo a agosto 2025:

> ### ✅ Punto de control
> | Archivo | Filas | Kilos en el archivo | `[Kilos]` en Power BI | `[Cosechas]` en Power BI |
> |---|---|---|---|---|
> | `bascula_2025-05.csv` | | 4 200 | | |
> | `bascula_2025-06.csv` | | 3 100 | | |
> | `bascula_2025-07.csv` | | 2 850 | 0 / vacío | |
> | `bascula_2025-08.csv` | | 5 600 | | |
> | **Total** | **6** | **15 750** | **12 900** | **6** |

Tu tabla (leída de los archivos abiertos y de la tabla por mes de D2, con el filtro de `(null)` ya borrado):

| Archivo | Filas | Kilos en el archivo | `[Kilos]` en Power BI | `[Cosechas]` en Power BI |
|---|---|---|---|---|
| `bascula_2025-05.csv` | 2 (cosechas 16 y 17: 2 750 + 1 450) | 4 200 | 4 200 | 2 |
| `bascula_2025-06.csv` | 1 (cosecha 18) | 3 100 | 3 100 | 1 |
| `bascula_2025-07.csv` | 2 (cosechas 19 y 20: 1 850 + 1 000) | 2 850 | **vacío** | 2 |
| `bascula_2025-08.csv` | 1 (cosecha 21) | 5 600 | 5 600 | 1 |
| **Total** | **6** | **15 750** | **12 900** | **6** |

Julio es el único mes donde Power BI no coincide con el archivo: el archivo dice 2 850 kg y Power BI dice vacío, o sea 2 850 kg menos (15 750 − 12 900), aunque sus 2 cosechas sí se cuentan. Los otros tres meses cuadran.

**D7.** En una línea: el control enero–abril dijo 125,00 % con julio roto, antes y después de D5. ¿Por qué nunca lo vio? En otra: el martes quitar los `null` de `finca_id` fue el arreglo, hoy quitar los de `kg` no. ¿Qué era el `null` cada vez?

Respuesta: El control de enero–abril nunca lo vio porque solo mira 2026 (enero a abril) y el problema está en julio de 2025, fuera de ese rango: sus 30 550 y su 125,00 % salen iguales con o sin el arreglo, así que un control que no toca la parte dañada no avisa de nada. El martes, el `null` de `finca_id` era la fila "Total" de la hoja, un dato sobrante que duplicaba las metas, y quitarlo era el arreglo correcto. Hoy el `null` de `kg` es un dato real que no llegó (el peso sí existe en el archivo, pero bajo el nombre `kg_neto`), así que quitarlo borra 2 cosechas reales y no recupera ningún kilo.

---

## PARTE E · El arreglo, en el archivo de ejemplo

**E1.** Filtro de `(null)` de D5 borrado: `[Cosechas sin kilos]` = **2**, tabla de D1 = **74 700 / 25**.

Resultado anotado: ya registrado en D5: con el filtro borrado, `[Cosechas sin kilos]` = **2** y la tabla vuelve a **74 700 / 25** (los kilos siguen sin las 2 cosechas de julio hasta aplicar el arreglo real).

**E2.** Transformar archivo de ejemplo. Pasos aplicados originales:

Respuesta: Solo dos: **Origen** y **Encabezados promovidos** (`= Table.PromoteHeaders(Origen, [PromoteAllScalars = true])`). El archivo de ejemplo es el de enero (`bascula_2025-01.csv`), que usa la columna `kg`; por eso el molde queda con `cosecha_id, finca_id, cultivo_id, fecha, calidad, kg`. El archivo de julio trae `kg_neto`, y como la expansión se hace por nombre, esa columna no encuentra su pareja y llega como `null`.

Paso nuevo (fx):
```
= Table.RenameColumns( #"Encabezados promovidos" , {{"kg_neto", "kg"}} , MissingField.Ignore )
```

Nota de lo que pasó: al añadir el paso solo en la consulta `Transformar Archivo de ejemplo`, `h_cosecha` siguió mostrando los dos `null` de julio (94 % válido, 6 % vacío) aunque se actualizó la vista previa. El Editor avanzado de la función `Transformar archivo` mostraba solo `Origen` y `Encabezados promovidos`: la función no había copiado el paso. Se añadió ahí también (`Personalizado = Table.RenameColumns(...)`, con el `in` interno devolviendo `Personalizado`) y entonces `h_cosecha` se corrigió. Es la función la que se ejecuta una vez por archivo, así que el arreglo debe vivir en ella.

**E3.** Calidad de columna de `kg` en `h_cosecha`:

> ### ✅ Punto de control
> `kg`: **0 %** vacío — Filas: **25**

Resultado anotado: `kg` **100 % válido, 0 % error, 0 % vacío**; barra de estado: **Columnas: 7, Filas: 25**. Las dos filas de julio de 2025 (`bascula_2025-07.csv`, cosechas 19 y 20) ahora traen **1 850** y **1 000** (suma 2 850) en lugar de `null`.

**E4.** Tabla de los dos años:

> ### ✅ Punto de control
> | Año | `[Kilos]` | `[Cosechas]` |
> |---|---|---|
> | 2025 | **47 000** | **16** |
> | 2026 | **30 550** | **9** |
> | **Total** | **77 550** | **25** |
>
> Julio 2025: **2 850 / 2**. `[Cosechas repetidas]` **0**. `[Cosechas sin kilos]` **vacía**. Control: **30 550 / 24 440 / 125,00 % / 9**.
>
> **Captura `clase30-anios.png`**: tabla de los dos años + tarjetas.

Resultado anotado, tras el arreglo en la función y **Cerrar y aplicar**:

| Año | `[Kilos]` | `[Cosechas]` |
|---|---|---|
| 2025 | **47 000** | **16** |
| 2026 | **30 550** | **9** |
| **Total** | **77 550** | **25** |

- Julio de 2025 en la tabla por `anio_mes`: **2 850 kg / 2 cosechas** (antes salía vacío / 2). El resto de meses no cambió y el total de la tabla es 77 550 / 25.
- `[Cosechas repetidas]` = **0**. `[Cosechas sin kilos]` = **vacía** (`--`): esta vez porque de verdad no hay ninguna cosecha sin kilos, no porque un filtro las haya borrado.
- Tabla por finca (sin segmentadores): Agrícola La Unión 10 900 / 7, Finca El Guayabo 31 450 / 9, Hacienda Santa Rosa 35 200 / 9; total 77 550 kg y 25 cosechas.
- Página de control, `anio` = 2026 y `mes` de enero a abril: **30 550 kg / 24 440 de meta / 125,00 % / 9 cosechas** (por finca: 2 100 / 5 000, 14 250 / 9 440, 14 200 / 10 000), igual que antes del arreglo. La tabla por `Source.Name` de esa página solo muestra `bascula_2026-03.csv` (10 800 / 3) y `bascula_2026-04.csv` (19 750 / 6).
- En la captura, la tabla de los dos años lleva además la columna `[Meta]` si se deja puesta (2026 = 47 000, 2025 vacía); el checkpoint solo pide `anio`, `[Kilos]` y `[Cosechas]`.

**E5.** Completa la última columna de la tabla de D6 con lo que dice Power BI ahora (los 4 meses deben cuadrar con el archivo):

Respuesta:

| Archivo | Filas | Kilos en el archivo | `[Kilos]` en Power BI | `[Cosechas]` en Power BI |
|---|---|---|---|---|
| `bascula_2025-05.csv` | 2 | 4 200 | 4 200 | 2 |
| `bascula_2025-06.csv` | 1 | 3 100 | 3 100 | 1 |
| `bascula_2025-07.csv` | 2 | 2 850 | **2 850** (antes vacío) | 2 |
| `bascula_2025-08.csv` | 1 | 5 600 | 5 600 | 1 |
| **Total** | **6** | **15 750** | **15 750** (antes 12 900) | **6** |

Ahora los cuatro meses cuadran con su archivo. La diferencia de 2 850 kg que había en D6 desapareció, y las cosechas siguen siendo 6 (nunca se perdió ninguna fila: lo que faltaba era el peso, no la cosecha).

**E6.** Vuelve a A4. ¿Acertaste las tres?

Respuesta: Sí, las tres, y solo se confirman cuando todo el trabajo está hecho: **15 archivos** (12 de 2025 + `2026-03` + `2026-04` + la copia de abril, que luego se filtra), **25 filas** en `h_cosecha` (25 cosechas tras filtrar la copia) y **47 000 kg en 2025** (Power BI da 47 000 / 16 cosechas). Lo que A4 ya dejaba claro es lo que fallaba mientras las cifras no cuadraban: con la copia llegaba a 31 filas y 50 300, y con julio sin `kg` llegaba a 44 150: A4 era la vara de medir.

---

## PARTE F · La fórmula del paso, y preguntas de cierre

**F1.** Editor avanzado de `h_cosecha`: línea literal de `Table.ExpandTableColumn`. ¿De dónde saca la lista de columnas que expande?

Respuesta: Línea literal del Editor avanzado de `h_cosecha`:

```
#"Columna de tabla expandida" = Table.ExpandTableColumn(#"Se han quitado otras columnas.", "Transformar archivo", Table.ColumnNames(#"Transformar archivo"(#"Archivo de ejemplo")))
```

La lista de columnas no está escrita a mano: la calcula `Table.ColumnNames( #"Transformar archivo"( #"Archivo de ejemplo" ) )`, es decir, los nombres de columna que devuelve la función `Transformar archivo` aplicada al **archivo de ejemplo** (el de enero). Por eso el molde lo define ese archivo: lo que no coincide por nombre con esa lista (como `kg_neto` de julio) queda como `null`, y por eso el renombrado debe estar dentro de la función para que cada archivo llegue ya con la columna `kg`.

**F2.** En dos líneas: ¿qué habría pasado si en B2 el archivo de ejemplo hubiera sido `bascula_2025-07.csv`? ¿Cuántas cosechas habrían salido sin kilos?

Respuesta: Con julio como archivo de ejemplo, el molde habría quedado con la columna `kg_neto` en vez de `kg`, y la expansión, que va por nombre, habría buscado `kg_neto` en todos los demás archivos; como ahí la columna se llama `kg`, les habría llegado `null`. Habrían salido sin kilos **23 de las 25 cosechas** (todas menos las 2 de julio), y la columna se habría llamado `kg_neto`, así que las medidas sobre `h_cosecha[kg]` tampoco habrían funcionado.

**F3.** Preguntas de cierre:

1. En una línea: ¿qué hace Combinar archivos con una carpeta, dicho en archivos y filas?

Respuesta: Toma los 15 archivos de la carpeta, les aplica a cada uno los mismos pasos (el molde del archivo de ejemplo) y apila todas sus filas en una sola tabla, añadiendo `Source.Name` para saber de qué archivo viene cada fila: aquí 31 filas con la copia, 25 sin ella.

2. Regla de detección del día (base de la 29: "un total escrito en la hoja es, para Power Query, una finca más"):

Respuesta: Antes de confiar en una carga, compara contra una cifra externa que conozcas (aquí A4: 15 archivos, 25 filas, 47 000 kg en 2025) y mira por columna lo que le falta o le sobra a cada archivo: la Calidad de columna (`kg` con 8 % vacío) y `[Cosechas sin kilos]` delatan lo que un total global no muestra. Lo que sobra (la copia, el total de la 29) aumenta cifras; lo que falta (el `kg_neto` de julio) las baja sin dar error.

3. En dos líneas: la copia de abril dio 50 300; julio dio 74 700. ¿Por qué una la atrapó el control enero–abril y la otra no?

Respuesta: La copia era de abril de 2026, que sí está dentro del control enero–abril: sus kilos y cosechas repetidas inflaban el total y la meta dejaba de cuadrar (50 300 y más cosechas). Julio era de 2025, fuera de ese rango, así que el control seguía en 30 550 / 125,00 % / 9 con o sin el daño; solo lo mostraban la tabla de 2025, la Calidad de columna y `[Cosechas sin kilos]`.

4. En una línea: ¿por qué el paso de renombrar va en Transformar archivo de ejemplo y no en `h_cosecha`?

Respuesta: Porque `kg_neto` hay que renombrarlo dentro de cada archivo, antes de apilar: en `h_cosecha` la expansión ya descartó esa columna y solo quedaría el `null`. El paso va en el ejemplo (y, como vimos, también hay que comprobar que llegue a la función `Transformar archivo`, que es la que se ejecuta archivo por archivo).

5. En dos líneas: la báscula de repuesto vuelve en noviembre y esta vez escribe `kilos`. Con tu consulta de hoy, ¿qué pasa al Actualizar? ¿Cuál de tus medidas lo avisa?

Respuesta: La báscula de repuesto escribiría `kilos`, que el renombrado de hoy no conoce (solo cambia `kg_neto`); esas cosechas llegarían con `kg` en `null`, sin error y sin cambiar el número de filas. La medida que lo avisa es **`[Cosechas sin kilos]`**: dejaría de estar vacía y mostraría cuántas cosechas llegaron sin peso. Habría que añadir `{"kilos", "kg"}` a la lista del paso `Table.RenameColumns`.

---