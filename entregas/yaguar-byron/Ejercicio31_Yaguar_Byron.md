# Ejercicio 31 · Las pesadas de la báscula

**Alumno:** Yaguar Rios, Byron Joel
**Herramienta:** solo Power BI Desktop · Carpeta de trabajo: `C:\\agrodb31\\` (4 archivos: `dim\_tiempo.csv`, `dim\_finca.csv`, `h\_meta.csv`, `pesadas.csv`)

> \*\*Convención de este archivo:\*\* cada medida va en un bloque de código con su resultado anotado debajo. Los recuadros ✅ son los puntos de control del enunciado. Debajo de cada uno va lo que me salió a mí.

**El número que prueba que el tablero quedó bien:** 125,00 % con 30 550 en 9 cosechas en la tabla de control, y la carga por camión **1 607,89** con **19** camiones.

\---

## PARTE A · El modelo sin cosechas

**A0.** Configuración regional del archivo actual: Español (México).

Resultado anotado: Archivo → Opciones y configuración → Opciones → ARCHIVO ACTUAL → Configuración regional: **Configuración regional para la importación = Español (México)** (la configuración regional de cadena de formato queda en «Automático»).

**A1.** Se cargaron `dim\_tiempo`, `dim\_finca` y `h\_meta` (sin `pesadas.csv`), con «Detectar automáticamente nuevas relaciones» apagado.

Resultado anotado: archivo nuevo `power\_bi\_31.pbix`, sin restos de la clase 30 (sin `dim\_cultivo` ni `h\_cosecha` viejo). El modelo arranca con tres tablas: `dim\_tiempo` (`anio`, `anio\_mes`, `fecha`, `mes`, `nombre\_mes`, `trimestre`), `dim\_finca` (`finca`, `finca\_id`, `provincia`) y `h\_meta` (`fecha\_mes`, `finca\_id`, `kg\_meta`, `meta\_id`). Detección automática de relaciones apagada: no se creó ninguna sola.

**A2.** Relaciones de las metas:

|#|De|A|¿Creada?|
|-|-|-|-|
|1|`h\_meta\[finca\_id]`|`dim\_finca\[finca\_id]`|Sí: Varios a uno (\*:1), filtro en una dirección|
|2|`h\_meta\[fecha\_mes]`|`dim\_tiempo\[fecha]`|Sí: Varios a uno (\*:1), filtro en una dirección|

**A3.** Tabla de fechas marcada sobre `dim\_tiempo\[fecha]`; `nombre\_mes` ordenado por `mes`.

```dax
Meta = SUM( h\_meta\[kg\_meta] )
```

> ### ✅ Punto de control
> Segmentadores `anio` = 2026 y `mes` de 1 a 4. Tabla `dim\_finca\[finca]` + `\[Meta]`:
>
> | finca | `\[Meta]` |
> |---|---|
> | La Union | \*\*5 000\*\* |
> | El Guayabo | \*\*9 440\*\* |
> | Santa Rosa | \*\*10 000\*\* |
> | \*\*Total\*\* | \*\*24 440\*\* |

Resultado anotado: `dim\_tiempo\[fecha]` marcada como tabla de fechas y `nombre\_mes` ordenada por `mes` (vista de Modelo). Con `anio` = 2026 y el segmentador de `mes` en 1, 2, 3 y 4:

|finca|`\[Meta]`|
|-|-|
|Agricola La Union|5 000|
|Finca El Guayabo|9 440|
|Hacienda Santa Rosa|10 000|
|**Total**|**24 440**|

Nota: la primera vez la tabla dio 29 000 (5 500 / 11 700 / 11 800) porque el segmentador de `nombre\_mes` tenía un mes de más seleccionado (los 4 botones visibles estaban bien, pero el segmentador tenía más meses a la derecha). Al usar el segmentador de `dim\_tiempo\[mes]` solo con 1 a 4 el total volvió a **24 440**, igual que el punto de control.

**A4.** Con `pesadas.csv` abierto en el Bloc de notas (se vuelve a estas respuestas en E3):

* ¿Cuántas filas de datos tiene?
* ¿Cuántas cosechas distintas?
* ¿En cuántos camiones se pesó la cosecha 7?

Respuesta: Leído de `pesadas.csv` abierto completo (la última fila es la 55, y la última `pesada\_id` es la 54):

* **54 filas de datos** (filas 2 a 55, sin el encabezado; `pesada\_id` va de 1 a 54).
* **25 cosechas distintas** (`cosecha\_id` del 1 al 25).
* **Cosecha 7: 4 camiones** (4 pesadas, filas 15 a 18 del archivo (`pesada\_id` 14 a 17): `GYE-2201` 2 500 kg, `GYE-2202` 2 500 kg, `GYE-2203` 2 400 kg y otra vez `GYE-2201` 2 400 kg). Son 3 placas distintas, porque `GYE-2201` viajó dos veces; el ejercicio llama «camión» a cada pesada, o sea a cada fila.

Cosecha 7: 4 pesadas = 9 800 kg (2 500 + 2 500 + 2 400 + 2 400).

\---

## PARTE B · Tal cual, y Agrupar por básico

**B1.** `pesadas.csv` cargado y renombrado a **`h\_cosecha`**. `fecha` = Fecha, `kg` = Número entero.

Resultado anotado: la consulta se llama `h\_cosecha` y sus Pasos aplicados son **Origen**, **Encabezados promovidos** y **Tipo de columna cambiado**. Tipos: `pesada\_id`, `cosecha\_id`, `finca\_id`, `cultivo\_id` y `kg` como número entero; `fecha` como fecha; `calidad` y `camion` como texto. Calidad de columna: las 8 columnas con **100 % válido, 0 % error, 0 % vacío**. Valores distintos: `pesada\_id` 54, `cosecha\_id` 25, `finca\_id` 3, `cultivo\_id` 4, `fecha` 24, `calidad` 2, `camion` 6, `kg` 19. Las fechas se leen en día/mes/año (por ejemplo 9/4/2025 = 9 de abril).

**B2.** Relaciones de las cosechas:

|#|De|A|¿Creada?|
|-|-|-|-|
|3|`h\_cosecha\[finca\_id]`|`dim\_finca\[finca\_id]`|Sí: Varios a uno (\*:1), filtro en una dirección|
|4|`h\_cosecha\[fecha]`|`dim\_tiempo\[fecha]`|Sí: Varios a uno (\*:1), filtro en una dirección|

```dax
Kilos = SUM( h\_cosecha\[kg] )
Cosechas = COUNTROWS( h\_cosecha )
Cumplimiento = DIVIDE( \[Kilos] , \[Meta] )
Cosechas repetidas = \[Cosechas] - DISTINCTCOUNT( h\_cosecha\[cosecha\_id] )
```

**B3.** Las cuatro medidas en la tabla de A3.

> ### ✅ Punto de control
> Total: `\[Kilos]` \*\*30 550\*\*, `\[Meta]` \*\*24 440\*\*, `\[Cumplimiento]` \*\*125,00 %\*\*, `\[Cosechas]` \*\*19\*\*, `\[Cosechas repetidas]` \*\*10\*\*.

Tabla anotada:

|finca|`\[Meta]`|`\[Kilos]`|`\[Cumplimiento]`|`\[Cosechas]`|`\[Cosechas repetidas]`|
|-|-|-|-|-|-|
|La Union|5 000|2 100|42 % (0,42)|2|0|
|El Guayabo|9 440|14 250|151 % (1,51)|7|4|
|Santa Rosa|10 000|14 200|142 % (1,42)|10|6|
|**Total**|24 440|30 550|125,00 %|19|10|

Con `anio` = 2026 y `mes` de 1 a 4. Coincide con el punto de control. El modelo se rehízo desde cero en un `.pbix` limpio (Español (México), detección de relaciones apagada, tabla de fechas marcada sobre `dim\_tiempo\[fecha]`); la tabla funciona con las cuatro relaciones, que es lo que confirma que las relaciones 3 y 4 están bien. Nota: la columna `\[Cumplimiento]` tiene que llevar formato de porcentaje con dos decimales para que se vea **125,00 %** en la captura.

En una línea: ¿qué está contando `\[Cosechas]`?

Respuesta: Está contando **filas de `h\_cosecha`, o sea pesadas (camiones), no cosechas**: da 19 porque en enero–abril de 2026 hay 19 filas, pero solo 9 cosechas distintas; los 10 de `\[Cosechas repetidas]` son esa diferencia (19 − 9) y salen de que una misma cosecha tiene varias filas, una por camión.

**B4.** Agrupar por → **Básico** → `cosecha\_id` → lo que propone (luego borrado el paso).

||Resultado|
|-|-|
|Filas (abajo a la izquierda)|**25** (una por cosecha, de 54 filas originales)|
|Columnas|**2** (`cosecha\_id` y `Recuento`)|
|Nombre de la columna nueva|**`Recuento`**|
|Operación de la columna nueva|**Recuento de filas** (`each Table.RowCount(\_)`, tipo `Int64.Type`)|
|Columnas que desaparecieron|`pesada\_id`, `finca\_id`, `cultivo\_id`, `fecha`, `calidad`, `camion` y **`kg`**|

Línea del paso, tal como quedó en la barra de fórmulas:

```
= Table.Group(#"Tipo cambiado", {"cosecha\_id"}, {{"Recuento", each Table.RowCount(\_), Int64.Type}})
```

Lo que dejó **Básico**: solo la llave `cosecha\_id` y una columna nueva con el recuento de filas de cada grupo (por ejemplo cosecha 1 = 3, cosecha 2 = 2, cosecha 3 = 4, cosecha 7 = 4). Todo lo demás desapareció, incluidos `finca\_id`, `fecha` (con las que se hacen las relaciones 3 y 4) y `kg` (los kilos). Por eso Básico no sirve aquí: se pierde lo que `\[Kilos]` suma. El paso se borró después con la X de Pasos aplicados.

\---

## PARTE C · Las llaves

**C1.** Agrupar por → **Avanzado**. Llaves: `cosecha\_id`, `finca\_id`, `cultivo\_id`, `fecha`, `calidad`, `camion`. Agregación: `kg` = Suma de `kg`.

> ### ✅ Punto de control
> Filas que quedan: \*\*36\*\*.

Resultado anotado: **36 filas, 7 columnas** (las seis llaves más `kg`, que ahora es la suma). La línea literal de `Table.Group` de este paso se copia del Editor avanzado en F1. En la barra de fórmulas empieza con `= Table.Group(#"Tipo cambiado", {"cosecha\_id", "finca\_id", "cultivo\_id", "fecha", "calidad", "camion"}, {{"kg", ...`.

Al abrir la ventana de **Agrupar por → Avanzado** aparecía además una agregación `Recuento` / Recuento de filas (la que deja por defecto); se eliminó para dejar solo `kg` = Suma de `kg`. Con 54 filas de origen, agrupar por las seis llaves (incluida `camion`) bajó a 36 filas: se juntaron las pesadas de un mismo camión dentro de la misma cosecha.

**C2.** Tabla de control.

> ### ✅ Punto de control
> Total \*\*30 550 / 125,00 %\*\*, `\[Cosechas]` \*\*14\*\*, `\[Cosechas repetidas]` \*\*5\*\*.
>
> \*\*Captura `clase31-llaves.png`\*\*: la tabla de control con `\[Cosechas]` en 14 y `\[Cosechas repetidas]` en 5 (con `camion` en las llaves, antes de arreglarlas).

Resultado anotado, con `anio` = 2026 y `mes` de 1 a 4:

|finca|`\[Meta]`|`\[Kilos]`|`\[Cumplimiento]`|`\[Cosechas]`|`\[Cosechas repetidas]`|
|-|-|-|-|-|-|
|Agricola La Union|5 000|2 100|42,00 %|2|0|
|Finca El Guayabo|9 440|14 250|150,95 %|5|2|
|Hacienda Santa Rosa|10 000|14 200|142,00 %|7|3|
|**Total**|**24 440**|**30 550**|**125,00 %**|**14**|**5**|

Los kilos no cambiaron (30 550 y 125,00 %): sumar los kilos de las filas de un camión da lo mismo. Lo que sí cambió es el conteo: `\[Cosechas]` bajó de 19 a **14** y `\[Cosechas repetidas]` de 10 a **5**, porque ahora hay una fila por cosecha y camión, pero una cosecha con varios camiones sigue repitiéndose. Es la prueba de que `camion` en las llaves es una llave de más: `\[Cosechas repetidas]` debería ser 0.

**C3.** Filtro `cosecha\_id` = 1 en Power Query (luego borrado):

|Camión|kg|
|-|-|
|`LRA-1101`|**2 700**|
|`LRA-1102`|**1 500**|

Las tres filas de la cosecha 1 en `pesadas.csv`:

|`pesada\_id`|`camion`|`kg`|
|-|-|-|
|1|`LRA-1101`|1 500|
|2|`LRA-1102`|1 500|
|3|`LRA-1101`|1 200|

En una línea: ¿por qué tres camiones dieron dos filas?

Respuesta: Porque `camion` identifica la placa, no el viaje: en el archivo las pesadas 1 y 3 son del mismo camión, `LRA-1101` (1 500 + 1 200), y al agrupar por `camion` entre las llaves Power Query las junta en una sola fila de **2 700**; `LRA-1102` queda solo con **1 500**. Es decir, tres pesadas se volvieron dos filas, y como esa cosecha sigue teniendo más de una fila, `\[Cosechas repetidas]` no llega a 0.

**C4.** Engrane de **Filas agrupadas**: se quita `camion` de las llaves y quedan tres agregaciones.

|Nuevo nombre de columna|Operación|Columna|
|-|-|-|
|`kg`|Suma|`kg`|
|`pesadas`|Recuento de filas||
|`carga\_promedio`|Promedio|`kg`|

> ### ✅ Punto de control
> Filas que quedan: \*\*25\*\*. Control: \*\*30 550 / 24 440 / 125,00 % / 9\*\*, `\[Cosechas repetidas]` = \*\*0\*\*.

Resultado anotado: **8 columnas, 25 filas**: cinco llaves (`cosecha\_id`, `finca\_id`, `cultivo\_id`, `fecha`, `calidad`) más `kg` (suma), `pesadas` (recuento de filas) y `carga\_promedio` (promedio de `kg`). Ejemplos de la vista previa: cosecha 1 = 4 200 kg, 3 pesadas, carga promedio 1 400; cosecha 7 = 9 800 kg, 4 pesadas, 2 450.

Tabla de control (`anio` = 2026, `mes` de 1 a 4):

|finca|`\[Meta]`|`\[Kilos]`|`\[Cumplimiento]`|`\[Cosechas]`|`\[Cosechas repetidas]`|
|-|-|-|-|-|-|
|Agricola La Union|5 000|2 100|42,00 %|2|0|
|Finca El Guayabo|9 440|14 250|150,95 %|3|0|
|Hacienda Santa Rosa|10 000|14 200|142,00 %|4|0|
|**Total**|**24 440**|**30 550**|**125,00 %**|**9**|**0**|

Los kilos y el cumplimiento no se movieron (30 550 / 125,00 %), y ahora `\[Cosechas]` = **9** y `\[Cosechas repetidas]` = **0**: una fila por cosecha.

**C5.** En dos líneas: qué columna puede ir en las llaves y cuál no. ¿Habría pasado lo mismo con `pesada\_id` en las llaves? ¿Cuántas filas habrían salido?

Respuesta: En las llaves van solo las columnas que valen lo mismo en todas las pesadas de una cosecha (`cosecha\_id`, `finca\_id`, `cultivo\_id`, `fecha`, `calidad`); `camion` no puede ir porque cambia dentro de una misma cosecha, y `kg` tampoco, porque es lo que se suma. Con `pesada\_id` en las llaves no se habría juntado nada, porque es único en cada fila: habrían salido las **54 filas** originales, `\[Cosechas]` seguiría en 19 y `\[Cosechas repetidas]` en 10.

\---

## PARTE D · Cuánto lleva un camión

**D1.**

```dax
Carga promedio = AVERAGE( h\_cosecha\[carga\_promedio] )
```

> ### ✅ Punto de control
> Formato: Número decimal, 2 decimales.
>
> | finca | `\[Carga promedio]` |
> |---|---|
> | La Union | \*\*1 050,00\*\* |
> | El Guayabo | \*\*1 866,67\*\* |
> | Santa Rosa | \*\*1 450,00\*\* |
> | \*\*Total\*\* | \*\*1 500,00\*\* |

Resultado anotado, con `anio` = 2026 y `mes` de 1 a 4 (formato Número decimal, 2 decimales):

|finca|`\[Carga promedio]`|
|-|-|
|Agricola La Union|1 050,00|
|Finca El Guayabo|1 866,67|
|Hacienda Santa Rosa|1 450,00|
|**Total**|**1 500,00**|

Coincide con el punto de control. La medida promedia la columna `carga\_promedio` que se guardó al agrupar: es un promedio de promedios (el promedio de los promedios de cada cosecha), no el promedio de los camiones.

**D2.** Tabla nueva (mismos segmentadores): `cosecha\_id`, `pesadas`, `kg`, `carga\_promedio` (cada una en **No resumir**). Filas de El Guayabo:

|`cosecha\_id`|`pesadas`|`kg`|`carga\_promedio`|
|-|-|-|-|
|5|2|2 600|1 300,00|
|6|1|1 850|1 850,00|
|7|4|9 800|2 450,00|

La tabla completa (con `finca` al principio) trae las 9 cosechas de enero–abril de 2026: Hacienda Santa Rosa (cosechas 1 a 4), Finca El Guayabo (5 a 7) y Agricola La Union (8 y 9). El promedio de las tres cargas promedio de El Guayabo es (1 300 + 1 850 + 2 450) / 3 = **1 866,67**, que es lo que dijo `\[Carga promedio]`.

**D3.** Cuenta a mano con `pesadas.csv` (El Guayabo, enero–abril 2026):

|`cosecha\_id`|Camiones|Kilos|Kilos entre camiones|
|-|-|-|-|
|5|2|2 600|1 300,00|
|6|1|1 850|1 850,00|
|7|4|9 800|2 450,00|
|**El Guayabo**|**7**|**14 250**|**2 035,71**|

> ### ✅ Punto de control
> \*\*14 250 / 7 = 2 035,71\*\*, donde `\[Carga promedio]` dice \*\*1 866,67\*\*.

Resultado anotado: contado a mano en `pesadas.csv` (filas 12 a 18 del archivo, `pesada\_id` 11 a 17): cosecha 5 = 2 camiones (`GYE-2201` 1 300 y `GYE-2201` 1 300 = 2 600 kg); cosecha 6 = 1 camión (`GYE-2202`, 1 850 kg); cosecha 7 = 4 camiones (`GYE-2201` 2 500, `GYE-2202` 2 500, `GYE-2203` 2 400 y `GYE-2201` 2 400 = 9 800 kg). En total **7 camiones y 14 250 kg**, y 14 250 / 7 = **2 035,71 kg por camión**. La medida `\[Carga promedio]` dijo 1 866,67, o sea 168,95 kg menos de lo real.

**D4.** En dos líneas: en Santa Rosa `\[Carga promedio]` se pasa (**1 450,00** contra **1 420,00**) y en El Guayabo se queda corta. ¿Qué cosecha jala cada promedio, y por qué?

Respuesta: `\[Carga promedio]` le da a cada cosecha el mismo peso (una cosecha = un voto), sin importar cuántos camiones tuvo. En Santa Rosa sube a 1 450 porque la cosecha 4 (1 camión, 1 500) y la 2 (2 camiones, 1 550) pesan igual que la 3 (4 camiones, 1 350), cuando el real, 14 200 / 10, es **1 420**. En El Guayabo baja a 1 866,67 porque la cosecha 7 (**4 de los 7 camiones**, 2 450 kg cada uno) cuenta solo un tercio, y las 5 y 6, con menos camiones y menos carga, la arrastran hacia abajo; el real es 14 250 / 7 = **2 035,71**.

**D5.**

```dax
Carga promedio 2 = DIVIDE( \[Kilos] , \[Cosechas] )
```

> ### ✅ Punto de control
> El Guayabo \*\*4 750,00\*\*, Santa Rosa \*\*3 550,00\*\*, total \*\*3 394,44\*\*.

Resultado anotado, en la tabla de control (`anio` = 2026, `mes` de 1 a 4):

|finca|`\[Carga promedio]`|`\[Carga promedio 2]`|
|-|-|-|
|Agricola La Union|1 050,00|1 050,00|
|Finca El Guayabo|1 866,67|**4 750,00**|
|Hacienda Santa Rosa|1 450,00|**3 550,00**|
|**Total**|**1 500,00**|**3 394,44**|

Coincide con el punto de control: El Guayabo = 14 250 / 3 cosechas = 4 750; Santa Rosa = 14 200 / 4 = 3 550; total = 30 550 / 9 = 3 394,44.

En una línea: ¿qué contesta en realidad esta medida?

Respuesta: Contesta **cuántos kilos lleva en promedio una cosecha**, no un camión: divide los kilos entre las cosechas (9), y una cosecha puede tener varios camiones, así que sale mucho más alta que la carga real de un camión (3 394,44 contra 1 607,89).

**D6.**

1. `\[Cosechas]` dijo **19** en B3 y dice **9** ahora, con la misma fórmula. ¿Qué cambió?

Respuesta: Cambió la tabla, no la fórmula. En B3 `h\_cosecha` tenía una fila por pesada (camión), y `COUNTROWS` contaba **19 filas**; después de agrupar con las cinco llaves, cada fila es una cosecha, y la misma fórmula cuenta **9 cosechas**. `\[Cosechas]` siempre cuenta filas: lo que significa depende de qué es una fila.

2. ¿Por qué La Unión dio **1 050,00** con las dos medidas?

Respuesta: Porque en La Unión cada una de sus 2 cosechas (la 8 y la 9) tuvo **un solo camión**: ahí cosecha y camión son lo mismo. El promedio de los promedios es (1 200 + 900) / 2 = 1 050; `\[Kilos]` entre `\[Cosechas]` es 2 100 / 2 = 1 050; y la carga real, 2 100 kg entre 2 camiones, también da 1 050. Las tres medidas coinciden cuando todas las cosechas tienen el mismo número de camiones, y se separan en cuanto una cosecha tiene más camiones que otra.

\---

## PARTE E · El arreglo: la cuenta que guardaste

**E1.**

```dax
Camiones = SUM( h\_cosecha\[pesadas] )
Carga por camión = DIVIDE( \[Kilos] , \[Camiones] )
```

Resultado anotado: las dos medidas se crearon sobre `h\_cosecha`. `\[Camiones]` suma la columna `pesadas` que se guardó al agrupar (cada cosecha dice cuántas filas, o sea camiones, tuvo) y `\[Carga por camión]` divide los kilos entre esos camiones, es decir, una carga real por camión y no un promedio de promedios.

**E2.** Tabla de control con las dos medidas nuevas.

> ### ✅ Punto de control
> `\[Camiones]` \*\*2 / 7 / 10 / 19\*\*. `\[Carga por camión]`: La Union \*\*1 050,00\*\*, El Guayabo \*\*2 035,71\*\*, Santa Rosa \*\*1 420,00\*\*, total \*\*1 607,89\*\*.
>
> \*\*Captura `clase31-carga.png`\*\*: tabla por finca con `\[Carga promedio]`, `\[Carga promedio 2]`, `\[Camiones]` y `\[Carga por camión]`, total en 1 607,89.

Tabla anotada:

|finca|`\[Carga promedio]`|`\[Carga promedio 2]`|`\[Camiones]`|`\[Carga por camión]`|
|-|-|-|-|-|
|La Union|1 050,00|1 050,00|2|**1 050,00**|
|El Guayabo|1 866,67|4 750,00|7|**2 035,71**|
|Santa Rosa|1 450,00|3 550,00|10|**1 420,00**|
|**Total**|1 500,00|3 394,44|**19**|**1 607,89**|

Con `anio` = 2026 y `mes` de 1 a 4; coincide con el punto de control.

Comparada con la tabla de D3: El Guayabo da **2 035,71** (14 250 / 7 camiones), exactamente la cuenta que se hizo a mano con `pesadas.csv`, mientras que `\[Carga promedio]` daba 1 866,67 y `\[Carga promedio 2]` 4 750,00. En Santa Rosa, la carga real es 14 200 / 10 = 1 420,00 contra los 1 450,00 del promedio de promedios. En La Unión las tres dan 1 050 porque cada cosecha tuvo un solo camión. Y en el total, 30 550 / 19 = **1 607,89**, contra 1 500,00 y 3 394,44.

**E3.** Página nueva, sin segmentadores, cuatro tarjetas.

> ### ✅ Punto de control
> `\[Kilos]` \*\*77 550\*\*, `\[Cosechas]` \*\*25\*\*, `\[Camiones]` \*\*54\*\*, `\[Carga por camión]` \*\*1 436,11\*\*.

Resultado anotado: página nueva, sin segmentadores, con cuatro tarjetas: `\[Cosechas]` = **25**, `\[Camiones]` = **54**, `\[Kilos]` = **77 550** (la tarjeta lo muestra como «78 mil» por las unidades de visualización automáticas) y `\[Carga por camión]` = **1 436,11** (la tarjeta lo muestra como «1,43611 mil»). Las unidades de visualización de las tarjetas se pusieron en **Ninguna** para ver el número completo.

¿Cuadran con tus respuestas de A4?

Respuesta: Sí. `\[Camiones]` = **54** es el mismo número de filas de datos que se contó en `pesadas.csv` en A4 (54 pesadas, `pesada\_id` del 1 al 54), y `\[Cosechas]` = **25** son las 25 cosechas distintas de A4. Los 77 550 kg entre 54 camiones dan 1 436,11 kg por camión. Todo cuadra con lo contado antes de empezar.

**E4.** En una línea: si en C4 no hubieras guardado `pesadas`, ¿de dónde sacarías los camiones sin volver a Power Query?

Respuesta: De ninguna parte: la tabla agrupada ya no trae una fila por camión, y desde DAX no se puede recuperar lo que Agrupar por borró. La única salida sería cargar una **segunda tabla con `pesadas.csv` sin agrupar** y contar sus filas con `COUNTROWS`, o sea, volver a Power Query de todos modos; por eso se guarda la cuenta (`pesadas`) en el mismo paso en que se agrupa.

\---

## PARTE F · La fórmula del paso, y preguntas de cierre

**F1.** Línea literal de `Table.Group` en el Editor avanzado de `h\_cosecha`, con las llaves y las tres agregaciones señaladas.

Respuesta: Línea literal del Editor avanzado de `h\_cosecha`:

```
#"Filas agrupadas" = Table.Group(#"Tipo cambiado", {"cosecha\_id", "finca\_id", "cultivo\_id", "fecha", "calidad"}, {{"kg", each List.Sum(\[kg]), type nullable number}, {"pesadas", each Table.RowCount(\_), Int64.Type}, {"carga\_promedio", each List.Average(\[kg]), type nullable number}})
```

* **Llaves** (la lista entre llaves `{...}` del segundo argumento): `"cosecha\_id"`, `"finca\_id"`, `"cultivo\_id"`, `"fecha"` y `"calidad"`. No están `camion`, `pesada\_id` ni `kg`.
* **Agregaciones** (la segunda lista): `kg` = `each List.Sum(\[kg])` (suma de los kilos de las pesadas), `pesadas` = `each Table.RowCount(\_)` (cuántas filas, o sea camiones, tuvo la cosecha) y `carga\_promedio` = `each List.Average(\[kg])` (promedio de `kg` de esas pesadas).

**F2.** En dos líneas: logística pide además **cuántos camiones distintos** trabajaron cada mes. Con la `h\_cosecha` agrupada de hoy, ¿se puede? ¿Por qué no sirve guardar un Recuento de filas distintas de `camion` por cosecha y sumarlo?

Respuesta: Con la `h\_cosecha` agrupada de hoy no se puede, porque la tabla ya no trae la columna `camion`: Agrupar por la borró. Y guardar un recuento de distintos por cosecha tampoco sirve, porque una misma placa trabaja en varias cosechas (por ejemplo `LRA-1101` en las cosechas 1, 2 y 3) y al sumar los recuentos de cada cosecha se contaría varias veces el mismo camión; un recuento de distintos no se puede sumar.

**F3.** Preguntas de cierre:

1. En una línea: ¿qué hace Agrupar por, dicho en filas y columnas?

Respuesta: Junta todas las filas que repiten los mismos valores de las llaves en una sola fila, y deja solo las columnas de las llaves más las columnas nuevas de las agregaciones: aquí **54 filas pasaron a 25** (una por cosecha) y las columnas que no estaban en ninguna de las dos listas (`pesada\_id`, `camion`) desaparecieron.

2. En una línea: regla de detección del día, tomando como base la de la 30 («una carpeta se combina con las columnas del primer archivo»):

Respuesta: Una agrupación se queda solo con lo que le pongas, en las llaves o en las agregaciones, y todo lo demás lo borra; por eso, **un promedio guardado nunca se vuelve a promediar: se guarda también la cuenta (`pesadas`) y se divide al final** (kilos entre camiones), igual que en la 30 la columna del primer archivo definía lo que sobrevivía.

3. En dos líneas: la llave de más la atrapó `\[Cosechas repetidas]`; el promedio de los promedios no lo atrapó ninguna prueba que ya tuviéramos. ¿Por qué?

Respuesta: La llave de más (`camion`) produjo filas repetidas por cosecha, y `\[Cosechas repetidas]` justo compara el número de filas contra las cosechas distintas (14 contra 9, o sea 5). El promedio de promedios no repite, ni pierde, ni deja vacío nada: da un número perfectamente posible (1 500,00), los kilos y el 125,00 % siguen cuadrando y `\[Cosechas repetidas]` da 0, así que ninguna prueba tenía contra qué compararlo; solo se descubrió contando a mano los camiones de El Guayabo (D3).

4. En una línea: ¿por qué `\[Camiones]` es un `SUM` y no un `COUNTROWS`?

Respuesta: Porque cada fila de `h\_cosecha` ya es una cosecha que trae guardado en `pesadas` cuántos camiones tuvo: `COUNTROWS` contaría cosechas (9 en el control), y para contar camiones hay que sumar esa cuenta (19).

5. En dos líneas: antes de agrupar, en B3, ¿qué habría dado `AVERAGE( h\_cosecha\[kg] )`? ¿Por qué ahí sí estaba bien?

Respuesta: En el control de B3 (2026, enero a abril) habría dado 30 550 / 19 = **1 607,89**, el mismo valor que después dio `\[Carga por camión]`, y para todo el archivo 77 550 / 54 = 1 436,11. Ahí sí estaba bien porque cada fila era un camión y cada camión pesaba lo mismo en el promedio; no se estaba promediando algo ya promediado.

\---

