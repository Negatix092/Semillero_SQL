# Proyecto integrador · El cierre de septiembre

## PC3 · Miércoles 7 · Las metas y el modelo

**Alumno:** Yaguar Rios, Byron Joel
**Herramienta:** solo Power BI Desktop · Carpeta de trabajo: `C:\\agrodbproyecto\\`
**Periodo del cierre:** 2026, de enero a septiembre (`dim\_tiempo\[anio]` = 2026, `dim\_tiempo\[mes]` de 1 a 9)

> \*\*Convención de este archivo:\*\* los recuadros ✅ son las anclas publicadas en el enunciado. Debajo de cada una va lo que salió en mi pantalla. Las preguntas ciegas (X1 a X5) se contestan con lo que muestra mi pantalla, sin cifras publicadas.

\---

## 3.1 · Cómo armé `h\_meta`

Una fila por finca y por mes de 2026, con `finca\_id`, `fecha\_mes` y `kg\_meta`, a partir de `metas\_planeacion.csv`, en Power Query.

|#|Decisión|Qué hice (paso de Power Query)|Por qué|
|-|-|-|-|
|1|Cómo llegó el archivo|Origen → `Tipo cambiado` automático. Llegó con 15 columnas (`Column1` a `Column15`) y 6 filas, todo como texto, con los encabezados como la primera fila de datos|Hay que subir la primera fila a encabezado antes de tocar nada|
|2|Encabezados|`Encabezados promovidos` + `Tipo cambiado1`|Así las columnas se llaman `finca\_id`, `finca`, `2026-01-01` … `2026-12-01` y `Total`|
|3|Fila `Total` (sin `finca\_id`)|`Filas filtradas`: quité los vacíos de `finca\_id`. Quedan 4 filas|No es una finca: si entra, la meta de la empresa se cuenta dos veces|
|4|Columna `Total` y nombre de la finca|`Columnas quitadas`: `finca` y `Total`|`Total` no es un mes y el nombre ya está en `dim\_finca`. Quedan `finca\_id` y los 12 meses|
|5|De ancha a larga|`Otras columnas con anulación de dinamización` con solo `finca\_id` seleccionada: 4 fincas × 12 meses = 48 filas|Una fila por finca y por mes, que es lo que se relaciona con el calendario|
|6|Nombres y tipos|`Columnas con nombre cambiado` (`Atributo` → `fecha\_mes`, `Valor` → `kg\_meta`) y `Tipo cambiado2`: `finca\_id` y `kg\_meta` enteros, `fecha\_mes` fecha|La fecha viene en formato ISO y se lee igual en cualquier configuración regional|

Orden que importa: primero se filtra la fila `Total` y se quita la columna `Total`, y después se anula la dinamización. Al revés, `Total` entraría como una finca más y como un mes 13.

Relaciones de `h\_meta` (varios a uno, dirección única, activas): `h\_meta\[finca\_id]` → `dim\_finca\[finca\_id]` y `h\_meta\[fecha\_mes]` → `dim\_tiempo\[fecha]`.

\---

## 3.2 · La relación de `fecha\_entrega`

`h\_cosecha\[fecha\_entrega]` → `dim\_tiempo\[fecha]`: varios a uno, dirección única e **inactiva** (en la vista de modelo es la línea punteada). La activa sigue siendo `h\_cosecha\[fecha]`. Se usa solo dentro de `\[Kilos entregados]`, con `USERELATIONSHIP`.

\---

## 3.3 · Medidas

```
Meta =
IF(
    ISCROSSFILTERED( dim\_cultivo ) ,
    BLANK() ,
    SUM( h\_meta\[kg\_meta] )
)

Cumplimiento = DIVIDE( \[Kilos] , \[Meta] )

Kilos entregados = CALCULATE( \[Kilos] , USERELATIONSHIP( h\_cosecha\[fecha\_entrega] , dim\_tiempo\[fecha] ) )

Kilos año anterior = CALCULATE( \[Kilos] , SAMEPERIODLASTYEAR( dim\_tiempo\[fecha] ) )

Variacion = DIVIDE( \[Kilos] - \[Kilos año anterior] , \[Kilos año anterior] )

Promedio por cultivo = DIVIDE( \[Kilos] , DISTINCTCOUNT( h\_cosecha\[cultivo\_id] ) )
```

|Medida|Qué pregunta contesta|
|-|-|
|`\[Meta]`|¿Cuántos kilos se planearon, con lo que se esté viendo? No enseña nada cuando se corta por cultivo, porque la meta no se fijó por cultivo.|
|`\[Cumplimiento]`|¿Qué parte de la meta se cosechó? Si no hay meta, queda vacío.|
|`\[Kilos entregados]`|¿Cuántos kilos se entregaron (y se cobran) en el periodo, según la fecha de entrega y no la de corte?|
|`\[Kilos año anterior]`|¿Cuánto se cosechó en el mismo periodo del año pasado?|
|`\[Variacion]`|¿Cuánto cambió lo cosechado contra el mismo periodo del año pasado, en porcentaje?|
|`\[Promedio por cultivo]`|¿Cuántos kilos da en promedio cada cultivo que sí cosechó? (el Café, sin cosechas, no entra en la cuenta)|

\---

## 3.4 · Anclas del miércoles

> ### ✅ Anclas del miércoles
> | Qué | Debe decir |
> |---|---|
> | Filas de `h\_meta` | \*\*48\*\* |
> | `\[Meta]` con solo `anio` = 2026 | \*\*110 000\*\* |
> | `\[Cumplimiento]` de la empresa, periodo del cierre | \*\*105,92 %\*\* |

Lo que salió en mi pantalla:

|Qué|Salió|
|-|-|
|Filas de `h\_meta`|**48** (la barra de Power Query dice 3 columnas y 48 filas)|
|`\[Meta]` con solo `anio` = 2026 (los 12 meses marcados)|**110 000**|
|`\[Cumplimiento]` de la empresa, periodo del cierre|**105,92 %** (con `anio` = 2026 y `mes` del 1 al 9; la meta del periodo es 86 200)|

\---

## Preguntas ciegas del miércoles

**X1.** `\[Cumplimiento]` de **cada una** de las cuatro fincas.

Respuesta, con `anio` = 2026 y `mes` de 1 a 9:

|Finca|Kilos|Meta|Cumplimiento|
|-|-|-|-|
|Agricola La Union|4 750|6 200|**76,61 %**|
|Finca El Guayabo|22 800|21 000|**108,57 %**|
|Hacienda Los Ceibos|43 500|40 000|**108,75 %**|
|Hacienda Santa Rosa|20 250|19 000|**106,58 %**|
|**Total**|**91 300**|**86 200**|**105,92 %**|

Solo Agricola La Union queda por debajo de su meta.

**X2.** `\[Kilos entregados]` de la empresa en el periodo del cierre. ¿Por qué no es igual a `\[Kilos]`?

Respuesta: **82 800 kg**, contra 91 300 de `\[Kilos]`. La diferencia es de 8 500 kg y sale de cuatro cosechas, porque `\[Kilos]` agrupa por la fecha de corte y `\[Kilos entregados]` por la fecha de entrega:

* **Salen por octubre (se cortaron en septiembre y se entregaron en octubre):** la cosecha **50** (corte 10/09/2026, entrega 07/10/2026, 1 050 kg), la **52** (corte 24/09/2026, entrega 06/10/2026, 3 300 kg) y la **53** (corte 29/09/2026, entrega 01/10/2026, 4 900 kg). Suman **9 250 kg**: cuentan en `\[Kilos]` y no en `\[Kilos entregados]`.
* **Entra por enero (se cortó en 2025 y se entregó dentro del periodo):** la cosecha **24** (corte 16/12/2025, entrega 12/01/2026, 750 kg). Cuenta en `\[Kilos entregados]` y no en `\[Kilos]`.

Cuenta: 91 300 − 9 250 + 750 = **82 800**.

**X3.** `\[Variacion]` de la empresa. ¿Crecimos eso?

Respuesta: la medida da **128,25 %** (91 300 kg contra 40 000 kg del mismo periodo de 2025). **No es el número que presentaría.** Hacienda Los Ceibos se compró en enero de 2026 y no tiene cosechas en 2025: en la tabla por finca su celda de `\[Kilos año anterior]` y de `\[Variacion]` sale vacía. Sus 43 500 kg están en el numerador del total, pero no hay nada que compararles en el denominador.

El número que presentaría es la variación de las fincas que sí tienen base: **19,50 %** (47 800 kg contra 40 000 kg, sin Los Ceibos).

En tres líneas:

1. 47 800 kg es 91 300 menos los 43 500 de Los Ceibos, y 40 000 kg es lo cosechado de enero a septiembre de 2025 por las otras tres fincas (4 450 + 19 850 + 15 700).
2. Los Ceibos se excluye porque no existía en 2025: su producción es crecimiento por compra, no por mejora de las fincas que ya teníamos.
3. Por finca: Agricola La Union 6,74 %, Finca El Guayabo 14,86 % y Hacienda Santa Rosa 28,98 %. El 128,25 % solo se puede decir con la aclaración de que incluye una finca nueva.

**X4.** `\[Promedio por cultivo]` en el periodo del cierre.

Respuesta: **18 260 kg** por cultivo (91 300 kg entre los 5 cultivos que cosecharon en el periodo: Mango, Guayaba, Cacao, Banano y Maíz). El Café no tiene cosechas y no cuenta; si se dividiera entre los 6 cultivos saldría un promedio más bajo y engañoso.

**X5.** Pon `\[Meta]` en una tabla por `dim\_cultivo\[cultivo]`. ¿Qué enseña, y qué debería enseñar?

Respuesta:

* **Qué enseñaba** (con la medida `SUM( h\_meta\[kg\_meta] )` sin proteger): **86 200 en cada fila**, también en Café, que ni siquiera tiene cosechas. Es la meta de toda la empresa repetida en cada cultivo.
* **Por qué:** la meta se fijó por finca y por mes, no por cultivo. `h\_meta` no está relacionada con `dim\_cultivo`, así que el filtro del cultivo no la corta y se repite el total.
* **Qué debería enseñar:** **vacío** en cada cultivo, porque esa meta no existe. Con `\[Meta]` vacía, `\[Cumplimiento]` también queda vacío y no se inventa un porcentaje.
* **Cómo lo arreglé:** `IF( ISCROSSFILTERED( dim\_cultivo ) , BLANK() , SUM( h\_meta\[kg\_meta] ) )`. Después del arreglo la tabla por cultivo ya no muestra meta en ningún cultivo (el Café, sin kilos ni meta, desaparece de la tabla) y las tarjetas y la tabla por finca siguen igual: 86 200 y 105,92 %.

\---

## Bitácora del día

|Qué encontré|Cómo me di cuenta|Qué número daba mal|Cómo lo arreglé|Cómo sé que quedó|
|-|-|-|-|-|
|`metas\_planeacion.csv` es una hoja ancha, con una fila `Total` y una columna `Total`|Al cargarla: 15 columnas, 6 filas, `finca\_id` vacío en la última fila (17 % vacío en `Column1`) y una columna `Total` al final|`\[Meta]` de 2026 saldría duplicada (la fila `Total` cuenta como quinta finca: 220 000 en vez de 110 000) y habría un mes 13 llamado `Total`|Quité la fila vacía de `finca\_id` y las columnas `finca` y `Total` antes de anular la dinamización de los meses|`h\_meta` tiene 48 filas (4 fincas × 12 meses), `finca\_id` con 4 distintos y `fecha\_mes` con 12; `\[Meta]` con `anio` = 2026 da 110 000|
|Los encabezados llegaron como la primera fila de datos|Columnas llamadas `Column1` a `Column15` y todo como texto|No se podía anular la dinamización de los meses por su nombre|`Encabezados promovidos` y un nuevo `Tipo cambiado1`|Las columnas se llaman `finca\_id`, `finca`, `2026-01-01` … `Total`|
|`\[Meta]` repetida por cultivo|Tabla por `dim\_cultivo\[cultivo]`: 86 200 en cada fila, también en Café, que no tiene cosechas|`\[Meta]` y `\[Cumplimiento]` por cultivo, que no existen|`\[Meta]` devuelve vacío si `dim\_cultivo` está filtrando (`ISCROSSFILTERED`)|La tabla por cultivo ya no enseña meta; las tarjetas y la tabla por finca siguen en 86 200 y 105,92 %|
|Hacienda Los Ceibos no tiene 2025|En la tabla por finca, su `\[Kilos año anterior]` y su `\[Variacion]` salían vacíos, y el total decía 128,25 %|`\[Variacion]` de la empresa (128,25 %) sale inflada por una finca nueva|Variación solo de las fincas con base, con un filtro del visual que excluye Los Ceibos|47 800 kg contra 40 000 kg = 19,50 %; 47 800 = 91 300 − 43 500 y 40 000 = 4 450 + 19 850 + 15 700, igual que el total de la tabla por finca|
|`\[Kilos entregados]` no es igual a `\[Kilos]`|Tarjetas: 82 800 contra 91 300|`\[Kilos]` mide cosecha por fecha de corte y finanzas cobra por fecha de entrega|Relación inactiva de `fecha\_entrega` y `USERELATIONSHIP` dentro de `\[Kilos entregados]`|Tablas con las cosechas 50, 52 y 53 (9 250 kg, entregadas en octubre) y la 24 (750 kg, entregada en enero): 91 300 − 9 250 + 750 = 82 800|
|Tarjetas de la variación sin segmentadores|Las tarjetas de la variación comparable decían 93 mil, 45 mil y 105,52 %; la tabla por finca decía 40 000 de año anterior|Comparaba todo 2025 y 2026 desplazado un año, no el periodo del cierre|Pasé las tarjetas a la página que tiene `anio` = 2026 y `mes` de 1 a 9|`\[Kilos año anterior]` sin Los Ceibos = 40 000 y la variación = 19,50 %|
|La meta del periodo|`\[Meta]` con `anio` = 2026 da 110 000, pero con `mes` de 1 a 9 da 86 200|Comparar la cosecha de 9 meses contra la meta de 12 daría 83,00 % (91 300 de 110 000)|`\[Cumplimiento]` siempre con `anio` = 2026 y `mes` de 1 a 9|91 300 / 86 200 = 105,92 %, el ancla del enunciado. Además, al desmarcar agosto por error salió 106,03 %, otra prueba de que el periodo importa|

\---

