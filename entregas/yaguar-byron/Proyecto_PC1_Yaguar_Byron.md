# Proyecto integrador · El cierre de septiembre

## PC1 · Lunes 5 · El inventario

**Alumno:** Yaguar Rios, Byron Joel
**Herramienta:** solo Power BI Desktop · Carpeta de trabajo: `C:\\agrodbproyecto\\`
**Periodo del cierre:** 2026, de enero a septiembre (`dim\_tiempo\[anio]` = 2026, `dim\_tiempo\[mes]` de 1 a 9)

> \*\*Convención de este archivo:\*\* los recuadros ✅ son las anclas publicadas en el enunciado. Debajo de cada una va lo que salió en mi pantalla. Las preguntas ciegas (L1 a L3) se contestan con lo que muestran los archivos, sin cifras publicadas.

\---

## 1.1 · Inventario de fuentes

Cada archivo abierto con el **Bloc de notas** (no con Excel), incluidos todos los de `bascula\\`.

|Fuente|Filas|Columnas|Su llave|A qué tabla del modelo va|
|-|-|-|-|-|
|`dim\_finca.csv`|4|3 (`finca\_id`, `finca`, `provincia`)|`finca\_id`|`dim\_finca`|
|`dim\_cultivo.csv`|6|4 (`cultivo\_id`, `cultivo`, `variedad`, `tipo`)|`cultivo\_id`|`dim\_cultivo`|
|`dim\_tiempo.csv`|730 (un día por fila, del 2025-01-01 al 2026-12-31)|6 (`fecha`, `anio`, `mes`, `nombre\_mes`, `trimestre`, `anio\_mes`)|`fecha`|`dim\_tiempo` (calendario)|
|`seguridad.csv`|10|2 (`correo`, `finca\_id`)|`correo` + `finca\_id` (el correo se repite)|`seguridad` (relación con `dim\_finca`; base del rol)|
|`bascula\\` (19 archivos)|47 en total (43 cosechas distintas, más las 4 de un archivo repetido)|7 (`cosecha\_id`, `finca\_id`, `cultivo\_id`, `fecha`, `fecha\_entrega`, `calidad`, `kg`); en un archivo la última se llama `kg\_neto`|`cosecha\_id`|`h\_cosecha`|
|`bascula\_digital\_2026.csv`|10 (`cosecha\_id` 44 a 53)|7 (`cosecha\_id`, `finca\_id`, `cultivo\_id`, `fecha`, `fecha\_entrega`, `calidad`, `kg`)|`cosecha\_id`|`h\_cosecha`|
|`precios.csv`|10|3 (`cultivo\_id`, `calidad`, `precio\_kg`)|`cultivo\_id` + `calidad`|`h\_cosecha` (`precio\_kg`)|
|`metas\_planeacion.csv`|5 líneas de datos (4 fincas + una fila «Total»)|15 (`finca\_id`, `finca`, 12 meses de 2026 en columnas y `Total`)|`finca\_id` (la fila Total la trae vacía)|`h\_meta`, después de despivotar los meses|

Detalle de la carpeta `bascula\\` (archivo → filas de datos):

|Archivo|Filas de datos|Observación|
|-|-|-|
|`bascula\_2025-01.csv`|1|cosecha 1|
|`bascula\_2025-02.csv`|2|cosechas 2 y 3|
|`bascula\_2025-03.csv`|3|cosechas 4 a 6|
|`bascula\_2025-04.csv`|3|cosechas 7 a 9|
|`bascula\_2025-05.csv`|2|cosechas 10 y 11|
|`bascula\_2025-06.csv`|2|cosechas 12 y 13|
|`bascula\_2025-07.csv`|2|cosechas 14 y 15|
|`bascula\_2025-08.csv`|2|cosechas 16 y 17. **El encabezado dice `kg\_neto`, no `kg`**|
|`bascula\_2025-09.csv`|2|cosechas 18 y 19|
|`bascula\_2025-10.csv`|2|cosechas 20 y 21|
|`bascula\_2025-11.csv`|1|cosecha 22|
|`bascula\_2025-12.csv`|2|cosechas 23 y 24|
|`bascula\_2026-01.csv`|2|cosechas 25 y 26 (la 26 es la primera de Hacienda Los Ceibos)|
|`bascula\_2026-02.csv`|3|cosechas 27 a 29|
|`bascula\_2026-03.csv`|4|cosechas 30 a 33|
|`bascula\_2026-03 - copia.csv`|4|**idéntico byte por byte a `bascula\_2026-03.csv`** (230 bytes cada uno): las mismas cosechas 30 a 33|
|`bascula\_2026-04.csv`|4|cosechas 34 a 37|
|`bascula\_2026-05.csv`|3|cosechas 38 a 40|
|`bascula\_2026-06.csv`|3|cosechas 41 a 43|
|**Total**|**47**|43 sin la copia; 19 archivos = 18 meses (enero de 2025 a junio de 2026) + la copia|

\---

## 1.2 · «Lo que veo raro» (al menos cinco)

Antes de cargar nada. Para cada una: archivo y línea, qué número va a mover, y hacia dónde.

|#|Qué vi|Archivo y línea|Número que va a mover|Hacia dónde|
|-|-|-|-|-|
|1|Las fechas de la báscula digital vienen en **mes/día/año** (por ejemplo `07/15/2026`, `09/24/2026`), pero el archivo se carga en Español (México), que lee día/mes/año|`bascula\_digital\_2026.csv`, líneas 2 a 11 (las de día mayor que 12, líneas 3, 4, 7, 10 y 11, no se pueden leer: `Error`/`null`; las demás, como `07/06/2026` en la línea 2, se leen como otro mes: 7 de junio en vez de 6 de julio)|`\[Kilos]` por mes, `\[Cosechas]` y `\[Kilos]` de septiembre, kilos de julio a septiembre|Menos en julio, agosto y septiembre; más en meses anteriores; y cosechas que se quedan sin fecha|
|2|Tres cosechas de la báscula digital se **entregan en octubre**, después del cierre|`bascula\_digital\_2026.csv`, líneas 8, 10 y 11 (`fecha\_entrega` `10/07/2026`, `10/06/2026` y `10/01/2026`; cosechas 50, 52 y 53)|`\[Kilos entregados]` al 30 de septiembre|Menos que `\[Kilos]` (hasta 9 250 kg menos, si las fechas se leen bien)|
|3|La hoja de metas viene **ancha** (un mes por columna), con una columna `Total` y una fila `Total` con `finca\_id` vacío|`metas\_planeacion.csv`, línea 1 (encabezados), columna 15 (`Total`) y línea 6 (`,Total,7200,...,110000`)|`\[Meta]` y `\[Cumplimiento]`|Sin despivotar no hay `h\_meta`; si se deja el total entra como una finca más y la meta se duplica (110 000 → 220 000) y el cumplimiento baja a la mitad|
|4|La hoja de metas incluye de **octubre a diciembre**, que el cierre al 30 de septiembre no debería contar|`metas\_planeacion.csv`, columnas `2026-10-01`, `2026-11-01` y `2026-12-01`|`\[Cumplimiento]` de toda la empresa|Menos, si se compara contra el año completo (110 000) en vez de enero a septiembre|
|5|El cultivo 6 (**Café**) existe en `dim\_cultivo`, pero no tiene precio ni cosechas en ningún archivo|`dim\_cultivo.csv`, línea 7; `precios.csv` (solo `cultivo\_id` 1 a 5); `bascula\\` y `bascula\_digital\_2026.csv` (solo usan los cultivos 1 a 5)|`\[Promedio por cultivo]` y la tabla por cultivo|Menos: se dividiría entre 6 cultivos en vez de entre los 5 que sí cosecharon, y Café saldría como una fila vacía|
|6|Un archivo de la carpeta es una **copia** de otro|`bascula\\bascula\_2026-03 - copia.csv`, líneas 2 a 5 (cosechas 30 a 33), idéntico a `bascula\_2026-03.csv`|`\[Cosechas]`, `\[Cosechas repetidas]` y `\[Kilos]` de marzo de 2026|Más: 4 cosechas de más y 12 000 kg de más|
|7|Un archivo trae la columna de kilos con **otro nombre**|`bascula\\bascula\_2025-08.csv`, línea 1 (`kg\_neto` en lugar de `kg`; cosechas 16 y 17)|`\[Kilos]` de agosto de 2025 y `\[Cosechas sin kilos]`|Menos: las dos cosechas de agosto de 2025 quedarían con `kg` vacío al combinar|
|8|**Hacienda Los Ceibos** no tiene ninguna cosecha en 2025: su primera cosecha es la 26 (enero de 2026)|`bascula\\bascula\_2026-01.csv`, línea 3 (`finca\_id` 4); en 2025 no aparece `finca\_id` 4|`\[Variacion]` contra el año anterior|Más: parte del crecimiento viene de una finca nueva, no de las mismas fincas|

\---

## 1.3 · El `.pbix` del proyecto

Configuración regional del archivo: Español (México). Detección automática de relaciones: apagada. Cargadas solo `dim\_finca`, `dim\_cultivo`, `dim\_tiempo` y `seguridad`; calendario marcado como tabla de fechas sobre `dim\_tiempo\[fecha]`; `nombre\_mes` ordenada por `mes`; relación de `seguridad` con `dim\_finca`.

Resultado anotado:

> ### ✅ Anclas del lunes
> | Qué | Debe decir |
> |---|---|
> | Filas de `dim\_finca` / `dim\_cultivo` / `dim\_tiempo` / `seguridad` | \*\*4 / 6 / 730 / 10\*\* |
> | Archivos en `C:\\agrodbproyecto\\bascula\\` | \*\*19\*\* |

Lo que salió en mi pantalla:

|Qué|Salió|
|-|-|
|Filas de `dim\_finca`|4|
|Filas de `dim\_cultivo`|6|
|Filas de `dim\_tiempo`|730 (del 01/01/2025 al 31/12/2026, una fila por día)|
|Filas de `seguridad`|10|
|Archivos en `bascula\\`|19|

Las cuatro cifras coinciden con las anclas (4 / 6 / 730 / 10) y con el contenido de los CSV.

Comprobaciones hechas en el modelo:

|Comprobación|Resultado|
|-|-|
|Detección automática de relaciones|Apagada; en la vista de modelo solo hay la relación que creé yo|
|`dim\_tiempo` marcada como tabla de fechas|Sí, activada sobre la columna `fecha`|
|`anio\_mes`|Tipo **Texto** (`2025-01`, `2025-02`…)|
|Fechas en `dim\_tiempo`|La fila 2 dice 02/01/2025 con `mes` = 1: se leyó como 2 de enero (día/mes), no como 1 de febrero|

Relación creada (vista de modelo):

|De|A|Cardinalidad|Dirección|
|-|-|-|-|
|`seguridad\[finca\_id]`|`dim\_finca\[finca\_id]`|Varios a uno (\*:1)|Única (de `dim\_finca` hacia `seguridad`), relación activa|

\---

## Preguntas ciegas del lunes

**L1.** Según los archivos, sin cargar nada: ¿cuántas cosechas **distintas** hay en total, sumando la carpeta y la báscula digital?

Respuesta: **53 cosechas distintas**: 43 en la carpeta `bascula\\` (`cosecha\_id` del 1 al 43, una vez cada una, sin contar el archivo copia, que repite las cosechas 30 a 33) y 10 en `bascula\_digital\_2026.csv` (`cosecha\_id` del 44 al 53, sin cruce con la carpeta). Las filas son 57 si se cuentan las 4 de la copia, pero las cosechas distintas son 53.

**L2.** ¿Cuánto es la meta de **toda la empresa para todo 2026**, y en qué celda de qué archivo lo leíste?

Respuesta: **110 000 kg**, en `metas\_planeacion.csv`, en la fila «Total» (línea 6 del archivo, la que empieza con una coma vacía) y la columna `Total` (la última, la columna 15; es la celda **O6** si se abre en una hoja de cálculo). Se comprueba sumando los totales por finca: 24 000 + 26 000 + 8 000 + 52 000 = 110 000, y también sumando los 12 totales mensuales de esa misma fila.

**L3.** ¿Qué correo de `seguridad.csv` tiene más de una finca **sin tenerlas todas**, y cuáles son?

Respuesta: **`regional.sur@agrodb.test`**, con dos fincas: la **3** (Agricola La Union) y la **4** (Hacienda Los Ceibos). `direccion@agrodb.test` también tiene más de una, pero tiene las cuatro (1, 2, 3 y 4); los otros cuatro correos (`gerente.\*`) tienen una sola finca cada uno.

\---

## Bitácora del día

|Qué encontré|Cómo me di cuenta|Qué número daba mal|Cómo lo arreglé|Cómo sé que quedó|
|-|-|-|-|-|
|`anio\_mes` de `dim\_tiempo` llegó como fecha en vez de texto|En la vista de modelo la columna tenía icono de fecha; el archivo trae `2025-01`|Las etiquetas de mes saldrían como 01/01/2025 y no como `2025-01`|En Power Query cambié el tipo de `anio\_mes` a Texto|En Power Query la columna muestra ABC y los valores `2025-01` en todas las filas visibles|
|`dim\_tiempo` no estaba marcada como tabla de fechas|El icono de la tabla era el de una tabla normal|`Kilos año anterior` y `Variacion` (miércoles) no se calcularían bien|Herramientas de tabla → Marcar como tabla de fechas → columna `fecha`|El cuadro muestra «Activar» con `fecha`, y en la vista de modelo `fecha` cambió de icono|
|Riesgo con el formato de las fechas (el archivo es ISO, la báscula digital viene MM/DD/YYYY)|En el fichero de calendario, la fila 2 muestra 02/01/2025 con `mes` = 1|Si se leyera mes/día, enero se partiría en meses falsos|Archivo con regional Español (México) y tipo fecha en el paso «Tipo cambiado»|Las filas 1 a 21 que se ven en Power Query (01/01 a 21/01/2025) tienen `mes` = 1 y `nombre\_mes` = Enero|
|Relación de `seguridad` con `dim\_finca`|Hay 10 correos con `finca\_id` repetido (dirección tiene 4, regional.sur 2)|Si fuera uno a uno, o bidireccional, el filtro por finca fallaría|Relación Varios a uno (\*:1), dirección Única, creada a mano|Panel de propiedades: \*:1, Única, activa; el **1** queda en `dim\_finca`|

\---

