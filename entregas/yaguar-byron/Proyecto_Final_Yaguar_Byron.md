# Proyecto integrador · El cierre de septiembre
## Viernes 9 · La entrega final

**Alumno:** Yaguar Rios, Byron Joel
**Herramienta:** solo Power BI Desktop · un solo archivo `proyecto_cierre.pbix`
**Periodo del cierre:** 2026, de enero a septiembre (`dim_tiempo[anio]` = 2026, `dim_tiempo[mes]` de 1 a 9)

---

# Parte 1 · La bitácora completa (lunes a viernes)

Una sola tabla, con las cinco columnas. Cada fila lleva entre corchetes el día en que la encontré.

| Qué encontré | Cómo me di cuenta | Qué número daba mal | Cómo lo arreglé | Cómo sé que quedó |
|---|---|---|---|---|
| **[Lun]** `anio_mes` de `dim_tiempo` llegó como fecha en vez de texto | En la vista de modelo la columna tenía icono de fecha; el archivo trae `2025-01` | Las etiquetas de mes saldrían como 01/01/2025 y no como `2025-01` | En Power Query cambié el tipo de `anio_mes` a Texto | En Power Query la columna muestra ABC y los valores `2025-01` en todas las filas visibles |
| **[Lun]** `dim_tiempo` no estaba marcada como tabla de fechas | El icono de la tabla era el de una tabla normal | `Kilos año anterior` y `Variacion` (miércoles) no se calcularían bien | Herramientas de tabla → Marcar como tabla de fechas → columna `fecha` | El cuadro muestra «Activar» con `fecha`, y en la vista de modelo `fecha` cambió de icono |
| **[Lun]** Riesgo con el formato de las fechas (el calendario es ISO, la báscula digital viene MM/DD/YYYY) | En el fichero de calendario, la fila 2 muestra 02/01/2025 con `mes` = 1 | Si se leyera mes/día, enero se partiría en meses falsos | Archivo con regional Español (México) y tipo fecha en el paso «Tipo cambiado» | Las filas 1 a 21 que se ven en Power Query (01/01 a 21/01/2025) tienen `mes` = 1 y `nombre_mes` = Enero |
| **[Lun]** Relación de `seguridad` con `dim_finca` | Hay 10 correos con `finca_id` repetido (dirección tiene 4, regional.sur 2) | Si fuera uno a uno, o bidireccional, el filtro por finca fallaría | Relación Varios a uno (*:1), dirección Única, creada a mano | Panel de propiedades: *:1, Única, activa; el **1** queda en `dim_finca` |
| **[Mar]** `bascula_2025-08.csv` trae `kg_neto` en vez de `kg` | En la primera carga, la calidad de columna de `kg` decía 96 % válido y 4 % vacío (2 filas de 47) | Los kilos de agosto de 2025 saldrían vacíos (faltarían las cosechas 16 y 17) y el total quedaría corto | Dentro de la función `Transformar archivo`: `Table.RenameColumns` de `kg_neto` a `kg` con `MissingField.Ignore` | `kg` pasó a 100 % válido y 0 % vacío, la fila 16 dice 2200 y la matriz da 3 450 kg en agosto de 2025 |
| **[Mar]** `bascula_2026-03 - copia.csv`: copia idéntica de marzo 2026 | `cosecha_id` tenía 43 distintos pero solo 39 únicos en 47 filas | `[Cosechas repetidas]` habría dado 4 y `[Kilos]` se inflaría en 12 000 | Filtré `Source.Name` distinto de la copia | 43 filas con 43 distintos y 43 únicos; después de anexar la digital, `[Cosechas repetidas]` = 0 |
| **[Mar]** `bascula_digital_2026.csv`: fechas en MM/DD/YYYY | Llegaron como texto (icono ABC) con valores como `07/15/2026`, que no existen en día/mes | Con un cambio de tipo normal habría Error en las filas con día mayor a 12 y las demás se leerían con el mes cambiado (06/07 como 7 de junio) | Cambié el tipo con *Usar configuración regional* → Fecha → Inglés (Estados Unidos) en `fecha` y `fecha_entrega` | 0 % Error y 0 % Vacío; fila 1: 06/07/2026 con entrega 02/08/2026; julio 2026 = 8 950 kg y agosto 2026 = 8 700 kg, que son las sumas de las cosechas 44–46 y 47–49 del archivo |
| **[Mar]** La relación `h_cosecha[fecha]` → `dim_tiempo[fecha]` se creó como uno a uno y con filtro en ambos sentidos | Panel de propiedades de la relación: «Uno a uno (1:1)» y «Ambas» | Segmentadores y filtros del calendario se comportarían mal (la tabla de hechos filtraría al calendario) | Cardinalidad Varios a uno (*:1) y dirección Única | El panel dice *:1 y Único; la vista de modelo muestra las tres relaciones con el 1 en la dimensión |
| **[Mar]** El precio depende del cultivo y de la calidad | `precios.csv` tiene 10 filas, dos por cultivo, y no hay precio del cultivo 6 | Combinar solo por `cultivo_id` repetiría filas y movería los kilos | Combiné por `cultivo_id` y `calidad`, externa izquierda, y expandí solo `precio_kg` | `h_cosecha` quedó con 53 filas antes y después, `precio_kg` con 0 % vacío; `[Ingresos]` del periodo = 47 166.50 |
| **[Mar]** Las tarjetas mostraban `136.600 mil` y `47.17 mil` | Lectura de la tarjeta: «mil» con decimales, que se puede leer como 136,6 mil o como 136 600 | Los kilos y los ingresos se leerían con otra escala | Unidades de visualización «Ninguna» y separador de miles; 2 decimales en Ingresos | La tarjeta de Ingresos dice 47,166.50 y la de Kilos 91,300 con el periodo (136 600 sin segmentadores) |
| **[Mié]** `metas_planeacion.csv` es una hoja ancha, con una fila `Total` y una columna `Total` | Al cargarla: 15 columnas, 6 filas, `finca_id` vacío en la última fila (17 % vacío en `Column1`) y una columna `Total` al final | `[Meta]` de 2026 saldría duplicada (la fila `Total` cuenta como quinta finca: 220 000 en vez de 110 000) y habría un mes 13 llamado `Total` | Quité la fila vacía de `finca_id` y las columnas `finca` y `Total` antes de anular la dinamización de los meses | `h_meta` tiene 48 filas (4 fincas × 12 meses); `[Meta]` con `anio` = 2026 da 110 000 |
| **[Mié]** Los encabezados de `metas_planeacion.csv` llegaron como la primera fila de datos | Columnas llamadas `Column1` a `Column15` y todo como texto | No se podía anular la dinamización de los meses por su nombre | `Encabezados promovidos` y un nuevo `Tipo cambiado1` | Las columnas se llaman `finca_id`, `finca`, `2026-01-01` … `Total` |
| **[Mié]** `[Meta]` repetida por cultivo | Tabla por `dim_cultivo[cultivo]`: 86 200 en cada fila, también en Café, que no tiene cosechas | `[Meta]` y `[Cumplimiento]` por cultivo, que no existen | `[Meta]` devuelve vacío si `dim_cultivo` está filtrando (`ISCROSSFILTERED`) | La tabla por cultivo ya no enseña meta; las tarjetas y la tabla por finca siguen en 86 200 y 105,92 % |
| **[Mié]** Hacienda Los Ceibos no tiene 2025 | En la tabla por finca, su `[Kilos año anterior]` y su `[Variacion]` salían vacíos, y el total decía 128,25 % | `[Variacion]` de la empresa (128,25 %) sale inflada por una finca nueva | Variación solo de las fincas con base (medida `[Variacion comparable]`, sin Los Ceibos) | 47 800 kg contra 40 000 kg = 19,50 %; 47 800 = 91 300 − 43 500 y 40 000 = 4 450 + 19 850 + 15 700, igual que el total de la tabla por finca |
| **[Mié]** `[Kilos entregados]` no es igual a `[Kilos]` | Tarjetas: 82 800 contra 91 300 | `[Kilos]` mide cosecha por fecha de corte y finanzas cobra por fecha de entrega | Relación inactiva de `fecha_entrega` y `USERELATIONSHIP` dentro de `[Kilos entregados]` | Tablas con las cosechas 50, 52 y 53 (9 250 kg, entregadas en octubre) y la 24 (750 kg, entregada en enero): 91 300 − 9 250 + 750 = 82 800 |
| **[Mié]** Tarjetas de la variación sin segmentadores | Las tarjetas decían 93 mil, 45 mil y 105,52 %; la tabla por finca decía 40 000 de año anterior | Comparaba todo 2025 y 2026 desplazado un año, no el periodo del cierre | Pasé las tarjetas a la página que tiene `anio` = 2026 y `mes` de 1 a 9 | `[Kilos año anterior]` sin Los Ceibos = 40 000 y la variación = 19,50 % |
| **[Mié]** La meta del periodo | `[Meta]` con `anio` = 2026 da 110 000, pero con `mes` de 1 a 9 da 86 200 | Comparar la cosecha de 9 meses contra la meta de 12 daría 83,00 % (91 300 de 110 000) | `[Cumplimiento]` siempre con `anio` = 2026 y `mes` de 1 a 9 | 91 300 / 86 200 = 105,92 %, el ancla. Al desmarcar agosto por error salió 106,03 %, otra prueba de que el periodo importa |
| **[Jue]** Con dos niveles de escenario marcados, `[Bono]` y `[Apoyo]` no quedaban en blanco aunque la meta sí | Marqué 110 y 120 a la vez y `[Meta]` salió vacía, pero Bono marcó 91 300 y Apoyo 0 | Bono 91 300 kg (todos los kilos contados como «sobre la meta»), porque `MAX( kilos − vacío , 0 )` trata el vacío como cero | Agregué `IF( ISBLANK( [Meta] ) , BLANK() , … )` a `[Bono]`, `[Apoyo]` y a las dos medidas de empresa entera | Con 110 y 120 marcados las cuatro tarjetas y Cumplimiento dicen «(En blanco)»; con 100 sigue 6 550 / 1 450 / 5 100 / 0 |
| **[Jue]** El bono y el apoyo sobre la empresa entera esconden el faltante de una finca | Medida de comparación: empresa entera dio bono 5 100 y apoyo 0, pero La Unión está a 76,61 % | Apoyo 0 y bono 5 100 en vez de 1 450 y 6 550 | `SUMX( VALUES( dim_finca[finca_id] ) , MAX( … , 0 ) )`: cada finca con su propia meta | Bonos por finca 1 800 + 3 500 + 1 250 = 6 550; apoyo de La Unión 6 200 − 4 750 = 1 450 |
| **[Jue]** Segmentadores `anio` y `mes` vacíos: el tablero mezclaba años | Captura con Cumplimiento 124,18 %, Bono 35 100 y Los Ceibos en rojo (83,65 %) | Kilos de 2025 y 2026 (136 600) contra una meta que solo es de 2026 (110 000) | Marqué 2026 y los meses 1 a 9 antes de leer o capturar | Cumplimiento 105,92 %, Bono 6 550, Apoyo 1 450 y Los Ceibos en verde (108,75 %) |
| **[Jue]** El visual «KPI» salió vacío con y sin filtros | Aunque lo agrandé, solo se veía el título | No salía ningún valor del KPI | Cambié a tarjeta de varias filas con `[Kilos]`, `[Meta]`, `[Diferencia vs meta]` y `[Diferencia %]`, con filtros propios `anio` = 2026 y `mes` 1 a 9 | 91 300 − 86 200 = 5 100; 5 100 / 86 200 = 5,92 %, igual a 105,92 % − 100 %; sale igual en cualquier página |
| **[Jue]** El segmentador de escenario era un deslizador, no deja marcar dos niveles | No podía probar el caso de dos niveles a la vez | No había forma de comprobar que `[Meta]` queda vacía | Cambié el estilo a Mosaico con selección múltiple | Con 110 y 120 marcados, Meta y Cumplimiento dicen «(En blanco)» |
| **[Vie]** Los ingresos del cierre incluyen dinero que se cobra en octubre | Gráfico de kilos cosechados contra entregados por mes: septiembre cosechó 10 500 kg y entregó 1 250 kg; las cosechas 50, 52 y 53 se entregan el 7, el 6 y el 1 de octubre | `[Ingresos]` = 47 166,50 se lee como «cobrado en el periodo», pero 5 462,00 son de cosechas aún no entregadas | No lo cambié: el tablero enseña `[Kilos]` y `[Kilos entregados]` lado a lado y el hallazgo va a la presentación | 1 050 × 2,80 + 3 300 × 0,20 + 4 900 × 0,38 = 2 940 + 660 + 1 862 = 5 462,00, el 11,6 % de 47 166,50 |
| **[Vie]** Las pruebas «deben dar 0» de la página Auditoría enseñaban un vacío y no un cero | La tarjeta de varias filas mostró 0 solo en `[Cosechas repetidas]` y `--` en las otras seis | Seis pruebas sin valor: un vacío no demuestra que no haya filas con problema, solo que la medida no devolvió nada (`COUNTROWS` de una tabla sin filas da vacío) | Agregué `+ 0` al final de cada una de las seis medidas | Las siete tarjetas dicen 0; la tabla de pruebas contra otras fuentes dice «Sí cuadra» en las ocho filas |

---

# Parte 2 · Las medidas y el rol

Cada bloque lleva debajo una línea con la pregunta que contesta.

## Los hechos

```
Kilos = SUM( h_cosecha[kg] )
```
Pregunta: ¿cuántos kilos se cosecharon, con lo que se esté viendo?

```
Cosechas = COUNTROWS( h_cosecha )
```
Pregunta: ¿cuántas cosechas hay, con lo que se esté viendo?

```
Cosechas repetidas = COUNTROWS( h_cosecha ) - DISTINCTCOUNT( h_cosecha[cosecha_id] )
```
Pregunta: ¿alguna cosecha entró dos veces? (debe dar 0)

```
Ingresos = SUMX( h_cosecha , h_cosecha[kg] * h_cosecha[precio_kg] )
```
Pregunta: ¿cuánto vale lo cosechado, kilo por kilo, con el precio de su cultivo y su calidad?

## Las metas y el escenario

```
Escenario = GENERATESERIES( 100 , 150 , 10 )
(creado con Modelado → Parámetro nuevo → Rango numérico, entero, de 100 a 150, incremento 10, predeterminado 100)

Meta =
VAR niveles   = COUNTROWS( VALUES( Escenario[Escenario] ) )
VAR hayFiltro = ISFILTERED( Escenario[Escenario] )
RETURN
    IF( ISCROSSFILTERED( dim_cultivo ) || ( hayFiltro && niveles > 1 ) ,
        BLANK() ,
        SUM( h_meta[kg_meta] ) * SELECTEDVALUE( Escenario[Escenario] , 100 ) / 100 )
```
Pregunta: ¿cuántos kilos se planearon, al nivel de escenario elegido? Queda vacía si se corta por cultivo o si hay dos niveles marcados, porque entonces no hay una meta que enseñar.

```
Cumplimiento = DIVIDE( [Kilos] , [Meta] )
```
Pregunta: ¿qué parte de la meta se cosechó?

## El tiempo

```
Kilos entregados = CALCULATE( [Kilos] , USERELATIONSHIP( h_cosecha[fecha_entrega] , dim_tiempo[fecha] ) )
```
Pregunta: ¿cuántos kilos se entregaron (y se cobran) en el periodo, según la fecha de entrega y no la de corte?

```
Kilos año anterior = CALCULATE( [Kilos] , SAMEPERIODLASTYEAR( dim_tiempo[fecha] ) )
```
Pregunta: ¿cuánto se cosechó en el mismo periodo del año pasado?

```
Variacion = DIVIDE( [Kilos] - [Kilos año anterior] , [Kilos año anterior] )
```
Pregunta: ¿cuánto cambió lo cosechado contra el mismo periodo del año pasado, en porcentaje?

```
Variacion comparable = CALCULATE( [Variacion] , dim_finca[finca] <> "Hacienda Los Ceibos" )
```
Pregunta: ¿cuánto crecimos de verdad, sin contar la finca que se compró en enero y no tiene 2025?

```
Promedio por cultivo = DIVIDE( [Kilos] , DISTINCTCOUNT( h_cosecha[cultivo_id] ) )
```
Pregunta: ¿cuántos kilos da en promedio cada cultivo que sí cosechó?

## El análisis

```
Bono =
VAR m = [Meta]
RETURN IF( ISBLANK( m ) , BLANK() ,
    SUMX( VALUES( dim_finca[finca_id] ) , MAX( [Kilos] - [Meta] , 0 ) ) )
```
Pregunta: ¿cuántos kilos por encima de la meta hay, sumando finca por finca?

```
Apoyo =
VAR m = [Meta]
RETURN IF( ISBLANK( m ) , BLANK() ,
    SUMX( VALUES( dim_finca[finca_id] ) , MAX( [Meta] - [Kilos] , 0 ) ) )
```
Pregunta: ¿cuántos kilos le faltan a las fincas que no llegaron a su meta, sumando finca por finca?

```
Bono empresa entera =
IF( ISBLANK( [Meta] ) , BLANK() , MAX( [Kilos] - [Meta] , 0 ) )

Apoyo empresa entera =
IF( ISBLANK( [Meta] ) , BLANK() , MAX( [Meta] - [Kilos] , 0 ) )
```
Pregunta: ¿qué diría la regla si se aplicara al total de la empresa en vez de a cada finca? (sirve para comparar y ver cuánto tapa el total)

```
Color semáforo =
VAR c = [Cumplimiento]
RETURN IF( ISBLANK( c ) , "#9E9E9E" , IF( c >= 1 , "#2E7D32" , "#C62828" ) )
```
Pregunta: ¿esta finca cumple su meta? Verde si cumple, rojo si no, gris si no hay meta que comparar.

```
Diferencia vs meta = [Kilos] - [Meta]
Diferencia % = DIVIDE( [Kilos] - [Meta] , [Meta] )
```
Pregunta: ¿cuánto cosechado sobra o falta contra la meta del año acumulada al cierre, en kilos y en porcentaje?

```
Quién mira = USERPRINCIPALNAME()
```
Pregunta: ¿con qué correo está viendo el tablero quien lo abre?

```
Titulo cierre =
"Cosechamos " & FORMAT( [Kilos] , "#,0" ) & " kg contra una meta de " &
FORMAT( [Meta] , "#,0" ) & " kg (" & FORMAT( [Cumplimiento] , "0.00%" ) & ")"
```
Pregunta: ¿cuál es el cierre en una línea, siempre con el periodo que se esté viendo?

## La auditoría

```
Cosechas sin precio = COUNTBLANK( h_cosecha[precio_kg] ) + 0
Cosechas sin kg = COUNTBLANK( h_cosecha[kg] ) + 0
Cosechas sin finca = COUNTROWS( FILTER( h_cosecha , ISBLANK( RELATED( dim_finca[finca_id] ) ) ) ) + 0
Cosechas sin cultivo = COUNTROWS( FILTER( h_cosecha , ISBLANK( RELATED( dim_cultivo[cultivo_id] ) ) ) ) + 0
Cosechas sin fecha en calendario = COUNTROWS( FILTER( h_cosecha , ISBLANK( RELATED( dim_tiempo[fecha] ) ) ) ) + 0
Fincas sin meta = COUNTROWS( FILTER( ALL( dim_finca ) , ISBLANK( [Meta] ) ) ) + 0
```
El `+ 0` es para que la prueba enseñe un 0 y no un vacío (`--`) cuando no hay ninguna fila con problema.

Pregunta (las seis): ¿cuántas cosechas quedaron sin precio, sin kilos, sin finca, sin cultivo o sin día en el calendario, y cuántas fincas quedaron sin meta? (todas deben dar 0)

Tabla `Pruebas` (Inicio → Introducir datos): una fila por prueba, con las columnas `Prueba`, `Esperado` y `Fuente del esperado`. Los esperados son 53, 136 600, 110 000, 48, 86 200, 40 000, 91 300 y 47 166,5. Cada uno sale de una fuente que no es el modelo (los archivos originales y la hoja de planeación).

```
Resultado prueba =
SWITCH( SELECTEDVALUE( Pruebas[Prueba] ) ,
    "Cosechas distintas" , [Cosechas] ,
    "Kilos de todo el archivo" , [Kilos] ,
    "Meta 2026 completa" , CALCULATE( [Meta] , dim_tiempo[anio] = 2026 ) ,
    "Filas de h_meta" , COUNTROWS( h_meta ) ,
    "Meta ene-sep 2026" , CALCULATE( [Meta] , dim_tiempo[anio] = 2026 , dim_tiempo[mes] <= 9 ) ,
    "Kilos ene-sep 2025" , CALCULATE( [Kilos] , dim_tiempo[anio] = 2025 , dim_tiempo[mes] <= 9 ) ,
    "Kilos ene-sep 2026" , CALCULATE( [Kilos] , dim_tiempo[anio] = 2026 , dim_tiempo[mes] <= 9 ) ,
    "Ingresos ene-sep 2026" , ROUND( CALCULATE( [Ingresos] , dim_tiempo[anio] = 2026 , dim_tiempo[mes] <= 9 ) , 2 ) )
```
Pregunta: ¿qué da el modelo en cada prueba de la tabla `Pruebas`?

```
¿Cuadra? = IF( [Resultado prueba] = SELECTEDVALUE( Pruebas[Esperado] ) , "Sí cuadra" , "NO cuadra" )
```
Pregunta: ¿lo que da el modelo es igual al número que sacó otra fuente?

## El rol

Rol **`Fincas`**, filtro de tabla sobre `dim_finca`:

```
dim_finca[finca_id] IN
    SELECTCOLUMNS(
        FILTER( ALL( seguridad ) , seguridad[correo] = USERPRINCIPALNAME() ) ,
        "f" , seguridad[finca_id]
    )
```
Pregunta: ¿qué fincas puede ver quien abre el tablero? Las que `seguridad` le asigna a su correo; quien no está en la tabla no ve ninguna.

---

## RESUMEN DE ENTREGAS

**Archivos:**
- `Proyecto_Final_Yaguar_Byron.pptx` — la presentación (máximo 10 láminas), subida antes de la sesión
- `Proyecto_Final_Yaguar_Byron.md` — este archivo: la bitácora completa, las medidas y el rol

**Capturas:**
- `proyecto-final-1-resumen.png` — página Resumen
- `proyecto-final-2-analisis.png` — página Análisis
- `proyecto-final-3-auditoria.png` — página Auditoría
