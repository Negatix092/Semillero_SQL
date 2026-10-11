# Proyecto integrador · El cierre de septiembre

## PC4 · Jueves 8 · El análisis y la seguridad

**Alumno:** Yaguar Rios, Byron Joel
**Herramienta:** solo Power BI Desktop · Carpeta de trabajo: `C:\\agrodbproyecto\\`
**Periodo del cierre:** 2026, de enero a septiembre (`dim\_tiempo\[anio]` = 2026, `dim\_tiempo\[mes]` de 1 a 9)

> \*\*Convención de este archivo:\*\* el recuadro ✅ es el ancla publicada en el enunciado. Debajo va lo que salió en mi pantalla. Las preguntas ciegas (J1 a J5) se contestan con lo que muestra mi pantalla, sin cifras publicadas.

\---

## 4.1 · El podio

Los tres cultivos con más kilos, de lo que se está viendo.

Cómo lo armé: una tabla con `dim\_cultivo\[cultivo]` y `\[Kilos]`, ordenada por `\[Kilos]` de mayor a menor. En el panel Filtros, sobre el campo `cultivo` y solo para ese visual, puse Tipo de filtro **«N superior»**, Superior **3**, Por valor `\[Kilos]`. Como es un filtro del visual, se calcula después de los segmentadores: el podio siempre es el de lo que se está viendo.

Qué pasa cuando filtro por `dim\_cultivo\[tipo]`:

* **Sin `tipo` marcado** (2026, meses 1 a 9): Banano 43 500, Mango 17 500 y Maíz 13 500 (suman 74 500).
* **`perenne`**: Banano 43 500, Mango 17 500 y Guayaba 12 050. Cacao (4 750) queda fuera.
* **`ciclo corto`**: solo sale Maíz con 13 500, porque es el único cultivo de ciclo corto con kilos. El podio no inventa filas: enseña tres como máximo, y aquí solo hay una.

\---

## 4.2 · El bono y el apoyo

La empresa paga bono por cada kilo arriba de la meta y da apoyo por cada kilo abajo. La regla se aplica finca por finca y el total de la empresa es la suma de lo que le toca a cada una.

```
Bono =
VAR m = \[Meta]
RETURN IF( ISBLANK( m ) , BLANK() ,
    SUMX( VALUES( dim\_finca\[finca\_id] ) , MAX( \[Kilos] - \[Meta] , 0 ) ) )

Apoyo =
VAR m = \[Meta]
RETURN IF( ISBLANK( m ) , BLANK() ,
    SUMX( VALUES( dim\_finca\[finca\_id] ) , MAX( \[Meta] - \[Kilos] , 0 ) ) )
```

Para comparar dejé dos medidas auxiliares que aplican la regla a la empresa entera:

```
Bono empresa entera =
IF( ISBLANK( \[Meta] ) , BLANK() , MAX( \[Kilos] - \[Meta] , 0 ) )

Apoyo empresa entera =
IF( ISBLANK( \[Meta] ) , BLANK() , MAX( \[Meta] - \[Kilos] , 0 ) )
```

Por qué finca por finca y no sobre el total de la empresa: `SUMX` recorre cada finca con `VALUES( dim\_finca\[finca\_id] )` y en cada una calcula su propio `MAX`. Así lo que le sobra a unas fincas no tapa lo que le falta a otra. Con el total de la empresa, los kilos de más de tres fincas se restan al faltante de La Unión y el apoyo desaparece.

\---

## 4.3 · ¿Y si subimos la meta?

Segmentador de 100 % a 150 %, de diez en diez, que mueve la meta y el cumplimiento.

```
Escenario: Modelado → Parámetro nuevo → Rango numérico
  Mínimo 100, Máximo 150, Incremento 10, Valor predeterminado 100, entero
  (crea la tabla Escenario con la columna Escenario y su segmentador)

Meta =
VAR niveles   = COUNTROWS( VALUES( Escenario\[Escenario] ) )
VAR hayFiltro = ISFILTERED( Escenario\[Escenario] )
RETURN
    IF( ISCROSSFILTERED( dim\_cultivo ) || ( hayFiltro \&\& niveles > 1 ) ,
        BLANK() ,
        SUM( h\_meta\[kg\_meta] ) \* SELECTEDVALUE( Escenario\[Escenario] , 100 ) / 100 )

Cumplimiento = DIVIDE( \[Kilos] , \[Meta] )
```

Qué hace si se marcan dos niveles a la vez: la meta no tiene un solo valor, así que `\[Meta]` devuelve vacío y con ella `\[Cumplimiento]`, `\[Bono]` y `\[Apoyo]`. Lo probé marcando 110 y 120 a la vez (el segmentador pasó de deslizador a mosaico con selección múltiple): las tarjetas de Bono, Apoyo, sus dos versiones de empresa entera y Cumplimiento dijeron «(En blanco)», y la tabla por finca quedó solo con los kilos. Sin ese cuidado el cumplimiento sería de un nivel cualquiera sin avisar. `\[Meta]` también queda vacía cuando se filtra por cultivo, porque la meta es por finca y mes, no por cultivo.

Cómo se mueve la meta del periodo (86 200 al 100 %): 110 % = 94 820 · 130 % = 112 060.

\---

## 4.4 · El semáforo y el KPI

```
Color semáforo =
VAR c = \[Cumplimiento]
RETURN IF( ISBLANK( c ) , "#9E9E9E" , IF( c >= 1 , "#2E7D32" , "#C62828" ) )

Diferencia vs meta = \[Kilos] - \[Meta]
Diferencia % = DIVIDE( \[Kilos] - \[Meta] , \[Meta] )
```

El semáforo es el color de fondo de la columna `Cumplimiento` de la tabla por finca (Formato → Color de fondo → Valor de campo → `\[Color semáforo]`): verde si la finca llega a su meta, rojo si no, gris si no hay meta que comparar. Con el cierre, Los Ceibos (108,75 %), El Guayabo (108,57 %) y Santa Rosa (106,58 %) salen en verde y La Unión (76,61 %) en rojo.

El KPI del año lo armé con una **tarjeta de varias filas** con `\[Kilos]`, `\[Meta]`, `\[Diferencia vs meta]` y `\[Diferencia %]`. El visual «KPI» de Power BI no dibujó nada sin eje de tendencia, y con un eje de meses enseñaría solo el último mes, no el acumulado. La tarjeta tiene filtros propios (`anio` = 2026 y `mes` de 1 a 9), así que dice lo mismo en cualquier página y el título es cierto.

Título del KPI (qué periodo enseña): **Cosechado vs meta · acumulado enero–septiembre 2026**

\---

## 4.5 · Cada quien lo suyo

Un solo rol que lee `seguridad`. Rol **`Fincas`**, con este filtro de tabla sobre `dim\_finca`:

```
dim\_finca\[finca\_id] IN
    SELECTCOLUMNS(
        FILTER( ALL( seguridad ) , seguridad\[correo] = USERPRINCIPALNAME() ) ,
        "f" , seguridad\[finca\_id]
    )
```

Para saber quién está mirando puse una tarjeta con la medida `Quién mira = USERPRINCIPALNAME()`.

Cómo funciona el filtro de `seguridad` hacia `h\_cosecha`: el rol no filtra `h\_cosecha` directamente. Deja en `dim\_finca` solo las fincas que `seguridad` le asigna al correo de quien mira, y `dim\_finca` filtra a `h\_cosecha` (y a `h\_meta`) por las relaciones \*:1 de dirección única. Quien no está en `seguridad` no obtiene ninguna finca, la lista queda vacía y no ve ninguna fila. Un solo rol sirve para todos porque lo que cambia es el correo, no el rol.

\---

## 4.6 · Ancla del jueves

> ### ✅ Ancla del jueves
> | Qué | Debe decir |
> |---|---|
> | \*\*Ver como\*\* `regional.sur@agrodb.test`: filas en la tabla por finca y `\[Cumplimiento]` del total | \*\*2 filas\*\* y \*\*104,44 %\*\* |

Lo que salió en mi pantalla:

|Qué|Salió|
|-|-|
|Filas en la tabla por finca (`regional.sur@agrodb.test`)|**2** (Hacienda Los Ceibos y Agrícola La Unión)|
|`\[Cumplimiento]` del total|**104,44 %** (48 250 / 46 200)|

Con `regional.sur` también salió: Bono 3 500, Apoyo 1 450, Bono empresa entera 2 050 y el podio con Banano 43 500 y Cacao 4 750.

\---

## Preguntas ciegas del jueves

**J1.** Con `tipo` = **perenne** marcado: ¿qué tres cultivos están en el podio, en qué lugar cada uno, y cuántos kilos suman?

Respuesta: en el periodo del cierre, con `perenne` marcado: **1.º Banano (43 500 kg), 2.º Mango (17 500 kg) y 3.º Guayaba (12 050 kg)**. Suman **73 050 kg**. Cacao (4 750 kg) es perenne pero queda en cuarto lugar, fuera del podio.

**J2.** El **bono** total y el **apoyo** total de la empresa, en kilos. Y lo que diría cada uno si la regla se aplicara a la empresa entera en vez de finca por finca.

Respuesta: finca por finca, **bono 6 550 kg** (El Guayabo 1 800 + Los Ceibos 3 500 + Santa Rosa 1 250) y **apoyo 1 450 kg** (La Unión). Con la regla sobre la empresa entera daría **bono 5 100 kg y apoyo 0**. El total de la empresa sobrepasa su meta por 5 100 kg, y esos kilos le tapan a La Unión su faltante de 1 450 kg, que es justo lo que la regla por finca quiere que se vea.

**J3.** Con la meta al **110 %**: `\[Cumplimiento]` de la empresa, y cuántas fincas cumplen.

Respuesta: `\[Cumplimiento]` de la empresa **96,29 %** (91 300 kg contra una meta de 94 820). **Ninguna finca cumple (0 de 4)**: El Guayabo 98,70 %, Los Ceibos 98,86 %, Santa Rosa 96,89 % y La Unión 69,65 %. El bono es 0 en todas y el apoyo suma 3 520 kg.

**J4.** Lo que enseña el KPI del año: valor, objetivo y la diferencia en porcentaje.

Respuesta: valor **91 300 kg**, objetivo **86 200 kg** y diferencia **+5 100 kg, es decir +5,92 %** (cumplimiento de 105,92 %).

**J5.** **Ver como** `gerente.launion@agrodb.test`: kilos, meta, cumplimiento y color. Y **ver como** `practicante@agrodb.test`.

Respuesta: `gerente.launion@agrodb.test` ve una sola finca, Agrícola La Unión: **4 750 kg** de kilos, **6 200 kg** de meta, **76,61 %** de cumplimiento y color **rojo** (también −1 450 kg, −23,39 % en el KPI). `practicante@agrodb.test` no aparece en `seguridad`: **no ve nada**, la tabla queda vacía y todas las tarjetas dicen «(En blanco)».

\---

## Bitácora del día

|Qué encontré|Cómo me di cuenta|Qué número daba mal|Cómo lo arreglé|Cómo sé que quedó|
|-|-|-|-|-|
|Con dos niveles de escenario marcados, `\[Bono]` y `\[Apoyo]` no quedaban en blanco aunque la meta sí|Marqué 110 y 120 a la vez y `\[Meta]` salió vacía, pero Bono marcó 91 300 y Apoyo 0|Bono 91 300 kg (todos los kilos contados como «sobre la meta»), porque `MAX( kilos − vacío , 0 )` trata el vacío como cero|Agregué `IF( ISBLANK( \[Meta] ) , BLANK() , … )` a `\[Bono]`, `\[Apoyo]` y a las dos medidas de empresa entera|Con 110 y 120 marcados las cuatro tarjetas y Cumplimiento dicen «(En blanco)»; con 100 sigue 6 550 / 1 450 / 5 100 / 0|
|El bono y el apoyo sobre la empresa entera esconden el faltante de una finca|Medida de comparación: empresa entera dio bono 5 100 y apoyo 0, pero La Unión está a 76,61 %|Apoyo 0 y bono 5 100 en vez de 1 450 y 6 550|`SUMX( VALUES( dim\_finca\[finca\_id] ) , MAX( … , 0 ) )`: cada finca se calcula con su propia meta|Bonos por finca 1 800 + 3 500 + 1 250 = 6 550; apoyo de La Unión 6 200 − 4 750 = 1 450|
|Segmentadores `anio` y `mes` vacíos: el tablero mezclaba años|Captura con Cumplimiento 124,18 %, Bono 35 100 y Los Ceibos en rojo (83,65 %)|Kilos de 2025 y 2026 (136 600) contra una meta que solo es de 2026 (110 000)|Marqué 2026 y los meses 1 a 9 antes de leer o capturar|Cumplimiento 105,92 %, Bono 6 550, Apoyo 1 450 y Los Ceibos en verde (108,75 %)|
|El visual «KPI» salió vacío con y sin filtros|Aunque lo agrandé, solo se veía el título|No salía ningún valor del KPI|Cambié a tarjeta de varias filas con `\[Kilos]`, `\[Meta]`, `\[Diferencia vs meta]` y `\[Diferencia %]`, con filtros propios `anio` = 2026 y `mes` 1 a 9|91 300 − 86 200 = 5 100; 5 100 / 86 200 = 5,92 %, igual a 105,92 % − 100 %; sale igual en cualquier página|
|El segmentador de escenario era un deslizador, no deja marcar dos niveles|No podía probar el caso de dos niveles a la vez|No había forma de comprobar que `\[Meta]` queda vacía|Cambié el estilo a Mosaico con selección múltiple|Con 110 y 120 marcados, Meta y Cumplimiento dicen «(En blanco)»|

\---

