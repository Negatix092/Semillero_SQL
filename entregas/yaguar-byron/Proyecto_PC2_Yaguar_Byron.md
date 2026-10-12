# Proyecto integrador · El cierre de septiembre

## PC2 · Martes 6 · Los hechos

**Alumno:** Yaguar Rios, Byron Joel
**Herramienta:** solo Power BI Desktop · Carpeta de trabajo: `C:\\agrodbproyecto\\`
**Periodo del cierre:** 2026, de enero a septiembre (`dim\_tiempo\[anio]` = 2026, `dim\_tiempo\[mes]` de 1 a 9)

> \*\*Convención de este archivo:\*\* los recuadros ✅ son las anclas publicadas en el enunciado. Debajo de cada una va lo que salió en mi pantalla. Las preguntas ciegas (M1 a M5) se contestan con lo que muestra mi pantalla, sin cifras publicadas.

\---

## 2.1 · Cómo armé `h\_cosecha`

Una fila por cosecha, de enero de 2025 a septiembre de 2026, con `cosecha\_id`, `finca\_id`, `cultivo\_id`, `fecha`, `fecha\_entrega`, `calidad`, `kg` y `precio\_kg`. Todo en Power Query, a partir de la carpeta `bascula\\`, `bascula\_digital\_2026.csv` y `precios.csv`.

|#|Decisión|Qué hice (paso de Power Query)|Por qué|
|-|-|-|-|
|1|Carpeta `bascula\\`|Obtener datos → Carpeta → Combinar y transformar; los 19 archivos entran con `Source.Name`. Consulta renombrada `h\_cosecha`|Un archivo por mes: se leen todos con la misma función|
|2|Archivo `bascula\_2026-03 - copia.csv`|`Filas filtradas`: `Source.Name` distinto de `bascula\_2026-03 - copia.csv`|La copia es idéntica a marzo 2026 y repite las cosechas 30 a 33. Filtré por nombre y no con «Quitar duplicados», porque ese paso conserva la primera fila y la copia queda antes del original en el orden de archivos|
|3|Columna `kg\_neto` de `bascula\_2025-08.csv`|En la función `Transformar archivo`: `Table.RenameColumns(..., {{"kg\_neto", "kg"}}, MissingField.Ignore)`|Si se corrige solo en `h\_cosecha` ya llegó vacío; dentro de la función cada archivo llega con `kg`. `MissingField.Ignore` deja pasar los 18 archivos que ya la traen|
|4|Báscula digital: unir con la carpeta|`Consulta anexada` (`Table.Combine`) de `h\_cosecha` con `bascula\_digital\_2026`; después `Columnas quitadas` para sacar `Source.Name`. La consulta `bascula\_digital\_2026` queda sin cargar|Mismas 7 columnas y llaves distintas (cosechas 44 a 53); `Source.Name` ya había servido para filtrar la copia y las 10 filas digitales no la traen (19 % vacío)|
|5|Fechas MM/DD/YYYY de la báscula digital|Tipo de `fecha` y `fecha\_entrega` con **Usar configuración regional → Fecha → Inglés (Estados Unidos)**|El archivo trae mes/día/año. Con la configuración del archivo (día/mes) las filas con día mayor a 12 darían Error y las demás se leerían con el mes y el día cambiados|
|6|Tipos de columna|`Tipo cambiado` en la consulta de la carpeta (llaves y `kg` enteros, fechas como fecha, `calidad` texto)|Las llaves son enteros y `fecha`/`fecha\_entrega` fecha; `calidad` texto; `precio\_kg` decimal|
|7|Precio por kilo (`precios.csv`): combinar por `cultivo\_id` y `calidad`|`Consultas combinadas`, externa izquierda, con `cultivo\_id` y `calidad` en el mismo orden en las dos tablas; `Se expandió precios` solo con `precio\_kg`. La consulta `precios` queda sin cargar|El precio depende del cultivo **y** de la calidad (10 combinaciones). Externa izquierda para no perder ninguna cosecha; `precio\_kg` salió con 0 % vacío y 53 filas antes y después|

Relaciones creadas: `h\_cosecha\[finca\_id]` → `dim\_finca\[finca\_id]`; `h\_cosecha\[cultivo\_id]` → `dim\_cultivo\[cultivo\_id]`; `h\_cosecha\[fecha]` → `dim\_tiempo\[fecha]` (todas varios a uno, dirección única).

\---

## 2.2 · Medidas

```
Kilos = SUM( h\_cosecha\[kg] )
Cosechas = COUNTROWS( h\_cosecha )
Cosechas repetidas = COUNTROWS( h\_cosecha ) - DISTINCTCOUNT( h\_cosecha\[cosecha\_id] )
Ingresos = SUMX( h\_cosecha , h\_cosecha\[kg] \* h\_cosecha\[precio\_kg] )
```

\---

## 2.3 · Anclas del martes (sin segmentadores)

> ### ✅ Anclas del martes
> | Qué | Debe decir |
> |---|---|
> | `\[Cosechas]` | \*\*53\*\* |
> | `\[Cosechas repetidas]` | \*\*0\*\* |
> | `\[Kilos]` | \*\*136 600\*\* |

Lo que salió en mi pantalla:

|Qué|Salió|
|-|-|
|`\[Cosechas]`|**53**|
|`\[Cosechas repetidas]`|**0**|
|`\[Kilos]`|**136 600**|

La matriz lo confirma: 2025 suma 45 300 kg en 24 cosechas, 2026 suma 91 300 kg en 29 cosechas, y el total es 136 600 kg en 53 cosechas.

\---

## Preguntas ciegas del martes

**M1.** `\[Kilos]` de **agosto de 2025**.

Respuesta: **3 450 kg**, leídos en la matriz (2025 → Agosto). Son las cosechas 16 y 17, de `bascula\_2025-08.csv`, que son las que traían `kg\_neto`.

**M2.** `\[Cosechas]` y `\[Kilos]` de **septiembre de 2026**.

Respuesta: **4 cosechas y 10 500 kg**, leídos en la matriz (2026 → Septiembre).

**M3.** `\[Kilos]` de cada una de las cuatro fincas, en el periodo del cierre.

Respuesta, con los segmentadores `anio` = 2026 y `mes` de 1 a 9:

|Finca|Kilos|
|-|-|
|Hacienda Santa Rosa|20 250|
|Finca El Guayabo|22 800|
|Agricola La Union|4 750|
|Hacienda Los Ceibos|43 500|
|**Total**|**91 300**|

El total coincide con el de la matriz para 2026 (91 300).

**M4.** `\[Ingresos]` de la empresa en el periodo del cierre, con dos decimales.

Respuesta: **47 166.50** (tarjeta con unidades de visualización en «Ninguna» y dos decimales, con 2026 y los meses 1 a 9 marcados).

**M5.** ¿Cuántas filas tenía `h\_cosecha` la primera vez que la cargué, antes de corregir nada? ¿Y cuántas columnas tenían valores vacíos o `Error`, y cuáles?

Respuesta: la primera vez que cargué `h\_cosecha` (solo la carpeta `bascula\\`, sin tocar nada) tenía **47 filas y 8 columnas**. **Una sola columna** tenía vacíos: **`kg`**, con **4 %** (2 filas: las cosechas 16 y 17 de `bascula\_2025-08.csv`, cuyo archivo trae `kg\_neto`). **Ninguna** columna tenía `Error`. En esa misma carga `cosecha\_id` tenía 43 distintos y solo 39 únicos, por las 4 cosechas repetidas de la copia de marzo.

La báscula digital la cargué aparte: 10 filas y 7 columnas, sin vacíos ni errores, pero con `fecha` y `fecha\_entrega` como **texto** en formato MM/DD/YYYY (por eso no marcó error: Power Query no las convirtió).

\---

## Bitácora del día

|Qué encontré|Cómo me di cuenta|Qué número daba mal|Cómo lo arreglé|Cómo sé que quedó|
|-|-|-|-|-|
|`bascula\_2025-08.csv` trae `kg\_neto` en vez de `kg`|En la primera carga, la calidad de columna de `kg` decía 96 % válido y 4 % vacío (2 filas de 47)|Los kilos de agosto de 2025 saldrían vacíos (faltarían las cosechas 16 y 17) y el total quedaría corto|Dentro de la función `Transformar archivo`: `Table.RenameColumns` de `kg\_neto` a `kg` con `MissingField.Ignore`|`kg` pasó a 100 % válido y 0 % vacío, la fila 16 dice 2200 y la matriz da 3 450 kg en agosto de 2025|
|`bascula\_2026-03 - copia.csv`: copia idéntica de marzo 2026|`cosecha\_id` tenía 43 distintos pero solo 39 únicos en 47 filas|`\[Cosechas repetidas]` habría dado 4 y `\[Kilos]` se inflaría en 12 000|Filtré `Source.Name` distinto de la copia|43 filas con 43 distintos y 43 únicos; después de anexar la digital, `\[Cosechas repetidas]` = 0|
|`bascula\_digital\_2026.csv`: fechas en MM/DD/YYYY|Llegaron como texto (icono ABC) con valores como `07/15/2026`, que no existen en día/mes|Con un cambio de tipo normal habría Error en las filas con día mayor a 12 y las demás se leerían con el mes cambiado (por ejemplo 06/07 como 7 de junio)|Cambié el tipo con *Usar configuración regional* → Fecha → Inglés (Estados Unidos) en `fecha` y `fecha\_entrega`|0 % Error y 0 % Vacío; fila 1: 06/07/2026 con entrega 02/08/2026; la entrega de la fila 7 es 07/10/2026; julio 2026 = 8 950 kg y agosto 2026 = 8 700 kg, que son las sumas de las cosechas 44–46 y 47–49 del archivo|
|La relación `h\_cosecha\[fecha]` → `dim\_tiempo\[fecha]` se creó como uno a uno y con filtro en ambos sentidos|Panel de propiedades de la relación: «Uno a uno (1:1)» y «Ambas»|Segmentadores y filtros del calendario se comportarían mal (la tabla de hechos filtraría al calendario)|Cardinalidad Varios a uno (\*:1) y dirección Única|El panel dice \*:1 y Único; la vista de modelo muestra las tres relaciones con el 1 en la dimensión y las flechas hacia `h\_cosecha`|
|El precio depende del cultivo y de la calidad|`precios.csv` tiene 10 filas, dos por cultivo (primera y segunda), y no hay precio del cultivo 6|Combinar solo por `cultivo\_id` repetiría filas y movería los kilos|Combiné por `cultivo\_id` y `calidad`, externa izquierda, y expandí solo `precio\_kg`|`h\_cosecha` quedó con 53 filas antes y después, `precio\_kg` con 0 % vacío y 10 valores distintos; la fila 1 (cacao, primera) cobra 2.8; `\[Ingresos]` del periodo = 47 166.50|
|Las tarjetas mostraban `136.600 mil` y `47.17 mil`|Lectura de la tarjeta: «mil» con decimales, que se puede leer como 136,6 mil o como 136 600|Los kilos y los ingresos se leerían con otra escala|Unidades de visualización «Ninguna» y separador de miles; 2 decimales en Ingresos|La tarjeta de Ingresos dice 47,166.50 y la de Kilos dice 91,300 con el periodo (136 600 sin segmentadores)|

\---

