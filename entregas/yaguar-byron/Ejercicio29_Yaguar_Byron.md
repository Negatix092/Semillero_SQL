# Ejercicio 29 · Las metas a lo largo
## Byron Yaguar Rios

---

## PARTE A · El modelo sin metas

### A0 · Configuración regional antes de cargar

`Archivo → Opciones y configuración → Opciones → Archivo actual → Configuración regional` → **Español (México)** → Aceptar.

### A1 · Cargar

1. En **Archivo actual → Carga de datos**, apaga **"Detectar automáticamente nuevas relaciones después de cargar los datos"**.
2. Cargar uno por uno (con **Cargar**, no Transformar): `h_cosecha`, `dim_tiempo`, `dim_finca`, `dim_cultivo`. `metas_planeacion.csv` todavía no.

### A2 · Las tres relaciones de las cosechas

| # | De | A |
|---|---|---|
| 1 | `h_cosecha[finca_id]` | `dim_finca[finca_id]` |
| 2 | `h_cosecha[cultivo_id]` | `dim_cultivo[cultivo_id]` |
| 3 | `h_cosecha[fecha]` | `dim_tiempo[fecha]` |

### A3 · El calendario y las medidas de cosecha

1. `dim_tiempo` → **Marcar como tabla de fechas** → `fecha`.
2. `nombre_mes` → **Ordenar por columna** → `mes`.

```dax
Kilos = SUM( h_cosecha[kg] )
```

```dax
Cosechas = COUNTROWS( h_cosecha )
```

### A4 · La tabla de control, sin metas todavía

Segmentadores: `dim_tiempo[anio]` = 2026, `dim_tiempo[mes]` = 1 a 4.

> ### ✅ Punto de control
> | Finca | `[Kilos]` | `[Cosechas]` |
> |---|---|---|
> | Agricola La Union | **2 100** | **2** |
> | Finca El Guayabo | **14 250** | **3** |
> | Hacienda Santa Rosa | **14 200** | **4** |
> | **Total** | **30 550** | **9** |

**A5.** Abre `metas_planeacion.csv` en el Bloc de notas. Antes de seguir: ¿cuántas filas debe tener `h_meta` para guardar una meta por finca y por mes? ¿Cuánto debe sumar el año, según el correo?

Respuesta: **36 filas** (3 fincas reales × 12 meses). El año completo debe sumar **47 000** kg de meta — la hoja trae una cuarta fila "Total" de más (la del "finca_id" vacío) que no cuenta como finca real y hay que excluirla.

---

## PARTE B · Anular dinamización

**B1.** **Obtener datos → Texto o CSV** → `metas_planeacion.csv` → **Transformar datos** (no Cargar). ¿Cuántas filas y columnas dice abajo a la izquierda?

Respuesta: **5 filas, 15 columnas** (3 fincas reales + la fila Total = 4 filas de datos, más el encabezado; columnas: `finca_id`, `finca`, los 12 meses y `Total`).

**B2.** Clic en `finca_id`, Ctrl + clic en `finca` → clic derecho → **Anular dinamización de otras columnas**.

> ### ✅ Punto de control
> Filas después de anular dinamización: **52**

Fórmula literal del paso (cópiala de la barra de fórmulas):

Respuesta:
```
#"Dinamización anulada de otras columnas" = Table.UnpivotOtherColumns(#"Tipo de columna cambiado", {"finca_id", "finca"}, "Atributo", "Valor")
```

**B3.** Renombrar columnas: `Atributo` → `fecha_mes`, `Valor` → `kg_meta`.

**B4.** Clic en el ícono de tipo de `fecha_mes` → Fecha.

> ### ✅ Punto de control
> Celdas con `Error`: **4**

Antes de tocarlas, en Pasos aplicados haz clic en el paso anterior al cambio de tipo y anota qué decía `fecha_mes` en esas 4 filas, con su `finca` y `kg_meta`:

Respuesta:
| `finca_id` | `finca` | `fecha_mes` | `kg_meta` |
|---|---|---|---|
| 1 | Hacienda Santa Rosa | "Total" (texto) | 20 000 |
| 2 | Finca El Guayabo | "Total" (texto) | 19 000 |
| 3 | Agricola La Union | "Total" (texto) | 8 000 |
| (vacío) | Total | "Total" (texto) | 47 000 |

Son las 4 filas que venían de la columna `Total` del CSV (el total anual de cada finca): al anular dinamización, esa columna se volvió una fila más por cada finca, con la palabra "Total" como valor de `fecha_mes` — imposible de convertir a Fecha.

**B5.** Borra el paso del cambio de tipo. Abre el filtro de `fecha_mes`, desmarca **Total**, Aceptar. Ahora sí: `fecha_mes` → Fecha, `kg_meta` → Número entero.

> ### ✅ Punto de control
> Filas: **48** — Errores: **0**

**B6.** Renombra la consulta `metas_planeacion` → **`h_meta`**. **Cerrar y aplicar**.

**B7.** Las dos relaciones de las metas:

| # | De | A |
|---|---|---|
| 4 | `h_meta[finca_id]` | `dim_finca[finca_id]` |
| 5 | `h_meta[fecha_mes]` | `dim_tiempo[fecha]` |

```dax
Meta = SUM( h_meta[kg_meta] )
```

```dax
Cumplimiento = DIVIDE( [Kilos] , [Meta] )
```

```dax
Filas de meta = COUNTROWS( h_meta )
```

---

## PARTE C · La finca que se llamaba Total

**C1.** Agrega `[Meta]`, `[Cumplimiento]` y `[Filas de meta]` a la tabla de control.

> ### ✅ Punto de control
> | Finca | `[Kilos]` | `[Meta]` | `[Cumplimiento]` | `[Cosechas]` |
> |---|---|---|---|---|
> | Agricola La Union | 2 100 | | **42,00 %** | 2 |
> | Finca El Guayabo | 14 250 | | **150,95 %** | 3 |
> | Hacienda Santa Rosa | 14 200 | | **142,00 %** | 4 |
> | (En blanco) | | **24 440** | | |
> | **Total** | **30 550** | **48 880** | **62,50 %** | — |
>
> `[Filas de meta]` total: **16**

**C2.** **Transformar datos** → `h_meta` → **Vista → Calidad de columna**. Anota `finca_id`: válido / error / vacío.

> ### ✅ Punto de control
> `finca_id`: **25 %** vacío
>
> **Captura `clase29-calidad.png`**: Power Query con `h_meta` abierta y la calidad de columna de `finca_id` a la vista.

**C3.** Filtro de `finca_id` → deja marcado solo **(null)**. ¿Cuántas filas quedan y qué dice `finca`? Compara los 12 `kg_meta` contra la última fila del CSV. Luego borra ese paso de filtro (todavía no es el arreglo).

Respuesta: Quedan 12 filas. En todas, `finca` = "Total". Los 12 valores de `kg_meta` (4040, 5200, 6900, 8300, 4560, 4000, 3200, 3700, 2500, 2100, 1500, 1000) son exactamente los de la fila "Total" del CSV: no es una finca real, es la suma mensual de las otras tres fincas, y por eso su `finca_id` nunca tuvo un valor.

**C4.** En dos líneas: la columna Total dio `Error` en B4. La fila Total no dio ninguno. ¿Por qué?

Respuesta: La columna Total (B4) falló porque `Table.TransformColumnTypes` intentó convertir la palabra "Total" a Fecha, y eso no es un valor de fecha válido, así que Power Query marca cada celda como `Error`. La fila Total no dio error porque en esa fila `fecha_mes` sí trae una fecha real; lo único que falta ahí es `finca_id`, que queda `null` (vacío) porque esa fila nunca tuvo un id de finca. Vacío y Error son cosas distintas en Power Query: uno truena la conversión, el otro simplemente no tiene dato.

**C5.** Quita el segmentador de mes (deja `anio` = 2026).

> ### ✅ Punto de control
> Total: **30 550 / 94 000 / 32,50 %**

¿Por qué 94 000, si el correo dice 47 000?

Respuesta: Porque la fila "Total" (la de `finca_id` vacío) sigue en la tabla y sus 12 `kg_meta` YA son la suma mensual de las tres fincas. Al sumarlos junto con los `kg_meta` reales de las fincas, todo se duplica: 47 000 (fincas reales) + 47 000 (fila Total) = 94 000.

**C6.** B5 quitó la columna Total por su nombre, antes del cambio de tipo, no con "Quitar errores". Si hubieras usado "Quitar errores", ¿habría dado lo mismo hoy? ¿Y si además una celda real hubiera traído un error?

Respuesta: No habría dado lo mismo. La fila Total nunca marcó `Error` (C4) — solo la columna Total lo marcaba, y eso ya se había filtrado antes por nombre. "Quitar errores" solo borra filas donde SÍ hay un `Error` visible en ese momento; la fila Total, con `fecha_mes` válida y `finca_id` simplemente vacío, no calificaría, así que el problema de duplicación seguiría intacto. Y si además una celda real (de una finca de verdad) hubiera traído un error, "Quitar errores" la habría borrado por completo sin avisar, perdiendo una cosecha o meta real para siempre — por eso es peligroso: no distingue entre "esto es basura" y "esto es un dato real con un problema que hay que arreglar".

---

## PARTE D · El segundo arreglo obvio (la parte que más vale)

**D1.** Regresa el segmentador de mes a 1-4. Tabla de control → panel Filtros → **Filtros en este objeto visual** → `finca` → marca todo menos `(En blanco)`.

**D2.** Las tres pruebas:

> ### ✅ Punto de control
> | Prueba | Resultado |
> |---|---|
> | La fila `(En blanco)` | ya no sale |
> | Total de la tabla de control | **30 550 / 24 440 / 125,00 %** |
> | `[Filas de meta]` en el total de esa tabla | **12** |

**D3.** Tarjeta con `[Meta]` en la misma página.

> ### ✅ Punto de control
> `[Meta]`: **48 880**

**D4.**

```dax
Meta sin finca = CALCULATE( [Meta] , ISBLANK( h_meta[finca_id] ) )
```

> ### ✅ Punto de control
> `[Meta sin finca]`: **24 440**

**D5.** Página nueva, mismos segmentadores (anio 2026, mes 1-4). Tabla con `nombre_mes`, `[Kilos]`, `[Meta]`, `[Cumplimiento]`.

> ### ✅ Punto de control
> | Mes | `[Kilos]` | `[Meta]` | `[Cumplimiento]` |
> |---|---|---|---|
> | Enero | | **8 080** | |
> | Febrero | | **10 400** | |
> | Marzo | 10 800 | **13 800** | **78,26 %** |
> | Abril | 19 750 | **16 600** | **118,98 %** |

¿Aparece alguna fila `(En blanco)`? ¿Por qué no?

Respuesta: No aparece. Aquí la tabla agrupa por `nombre_mes`, no por `finca`. La fila con `finca_id` vacío tiene un `fecha_mes` válido, así que encaja dentro del mes que le toca en vez de generar una categoría "en blanco" aparte — se mezcla con las demás filas de ese mes y duplica el total sin dejar ninguna pista visual.

**D6.** Con `metas_planeacion.csv` abierto, calcula a mano la meta de la empresa en cada mes sumando solo las tres fincas:

> ### ✅ Punto de control
> | Mes | Santa Rosa | El Guayabo | La Union | Meta del mes | Kilos | Cumplimiento |
> |---|---|---|---|---|---|---|
> | Enero | 1 500 | 1 440 | 1 100 | **4 040** | — | — |
> | Febrero | 2 000 | 2 000 | 1 200 | **5 200** | — | — |
> | Marzo | 3 000 | 2 500 | 1 400 | **6 900** | 10 800 | **156,52 %** |
> | Abril | 3 500 | 3 500 | 1 300 | **8 300** | 19 750 | **237,95 %** |
> | **Total** | **10 000** | **9 440** | **5 000** | **24 440** | **30 550** | **125,00 %** |

**D7.** En una línea: ¿qué arregló el filtro de D1, y qué no arregló? En otra: ¿por qué la prueba del número viejo pasó?

Respuesta: El filtro de D1 solo arregló la vista de ESA tabla (la agrupada por finca): quitó la fila `(En blanco)` y dejó el total en 125,00 %. No arregló el modelo — `h_meta` sigue teniendo la fila Total con `finca_id` vacío, así que cualquier otro visual sin ese mismo filtro (como la tabla por mes de D5) sigue duplicando. La prueba del número viejo "pasó" porque 94 000/47 000 ya no aparecían en ESA tabla puntual, pero fue una prueba engañosa: el problema seguía intacto en cualquier otro rincón del reporte.

---

## PARTE E · El arreglo, en Power Query

**E1.** Quita el filtro de `finca` del objeto visual.

**E2.** **Transformar datos** → `h_meta` → filtro de `finca_id` → desmarca **(null)** → Aceptar. **Cerrar y aplicar**.

> ### ✅ Punto de control
> Filas: **36** — `finca_id` vacío: **0 %**

**E3.** Tabla de control, sin filtros en el objeto visual:

> ### ✅ Punto de control
> | | `[Kilos]` | `[Meta]` | `[Cumplimiento]` | `[Cosechas]` |
> |---|---|---|---|---|
> | **Total** | **30 550** | **24 440** | **125,00 %** | **9** |
>
> Sin fila `(En blanco)`. Tarjetas: `[Meta]` = **24 440**, `[Meta sin finca]` = **vacía**.
>
> **Captura `clase29-metas.png`**: la tabla de control en 125,00 % y las dos tarjetas.

**E4.** Tabla por mes de D5:

> ### ✅ Punto de control
> | Mes | `[Meta]` | `[Cumplimiento]` |
> |---|---|---|
> | Enero | **4 040** | |
> | Febrero | **5 200** | |
> | Marzo | **6 900** | **156,52 %** |
> | Abril | **8 300** | **237,95 %** |

**E5.** Quita el segmentador de mes (deja `anio` = 2026). Tabla por finca:

> ### ✅ Punto de control
> | Finca | `[Kilos]` | `[Meta]` | `[Cumplimiento]` | `[Filas de meta]` |
> |---|---|---|---|---|
> | Hacienda Santa Rosa | 14 200 | **20 000** | **71,00 %** | |
> | Finca El Guayabo | 14 250 | **19 000** | **75,00 %** | |
> | Agricola La Union | 2 100 | **8 000** | **26,25 %** | |
> | **Total** | **30 550** | **47 000** | **65,00 %** | **36** |

Compara cada `[Meta]` contra la columna Total de la hoja.

**E6.** Vuelve a tus respuestas de A5. ¿Acertaste las dos?

Respuesta: Sí, las dos: `h_meta` limpia terminó con 36 filas (3 fincas × 12 meses) y el total del año dio exactamente 47 000, igual a lo que decía el correo. Coincide con la tabla de E5.

---

## PARTE F · La fórmula del paso, y preguntas de cierre

**F1.** **Transformar datos** → `h_meta` → **Editor avanzado**. Copia literal la línea de `Table.UnpivotOtherColumns`. ¿Qué columnas nombra: las que se quedan o las que se anulan?

Respuesta:
```
#"Dinamización anulada de otras columnas" = Table.UnpivotOtherColumns(#"Tipo de columna cambiado", {"finca_id", "finca"}, "Atributo", "Valor")
```
Nombra las columnas que SE QUEDAN igual (`finca_id`, `finca`). Todo lo que no está en esa lista — los 13 nombres de mes/Total — se anula ("otras columnas" = las demás, las que no se nombran).

**F2.** El año que viene, planeación agrega a la misma hoja las columnas `2027-01-01` a `2027-12-01`. ¿Qué pasa con tu consulta al Actualizar? ¿Y qué habría pasado con "Anular solo la dinamización de las columnas seleccionadas"?

Respuesta: Con "Anular dinamización de otras columnas" (la que usaste), al actualizar las 12 columnas nuevas de 2027 se anulan automáticamente también, porque la fórmula solo nombra `finca_id`/`finca` como fijas — todo lo demás se pivotea, sin tocar la fórmula. Con "Anular solo la dinamización de las columnas seleccionadas", la fórmula guarda una lista fija de las columnas de 2026 que sí anular; las 12 columnas nuevas de 2027 quedarían fuera, sin anular, y aparecerían como columnas extra sin usar — habría que editar el paso a mano cada año.

**F3.** Preguntas de cierre:

1. ¿Qué hace Anular dinamización, en filas y columnas?

Respuesta: Toma columnas anchas (una por mes) y las convierte en filas. Multiplica el número de filas (cada fila original se reparte en tantas filas como columnas anuladas) y reduce el número de columnas a las fijas (`finca_id`, `finca`) más dos nuevas (`Atributo`/`Valor`).

2. Regla de detección del día (tomando como base la de la clase 28: "una combinación que cambia el número de filas no agregó columnas: agregó cosechas"):

Respuesta: Un anular dinamización que cambia el número de filas no es un error ni duplicó registros por accidente: es el diseño esperado — cada columna ancha se convierte en una fila propia, así que multiplicar filas es la transformación haciendo su trabajo, no una señal de que algo salió mal.

3. El filtro del objeto visual (hoy) y "Quitar duplicados" (clase 28): ¿en qué se parecen, y cuál deja el error dentro del modelo?

Respuesta: Se parecen en que los dos "limpian" solo lo que se ve en ese momento, sin preguntarse por qué aparecieron esas filas de más en la fuente. La diferencia importante: el filtro del objeto visual deja el error VIVO dentro del modelo — `h_meta` sigue con la fila en blanco, solo se esconde en esa tabla puntual, y cualquier otro visual sin el mismo filtro la vuelve a mostrar (como pasó en D5). "Quitar duplicados" en Power Query sí cambia los datos reales que entran al modelo, para bien o para mal.

4. La clase 21 tuvo una fila de total que no era la suma de las filas. Hoy la hoja traía una fila de total que sí lo era. ¿Por qué hoy eso fue el problema?

Respuesta: Porque al ser exactamente la suma correcta, el valor duplicado no se ve raro ni genera ningún error visible — se suma junto con los datos reales y dobla el total sin ninguna señal de alarma. Si no coincidiera (como en clase 21), sería fácil detectarlo comparando contra el resto; aquí, precisamente porque coincide, el error se esconde a plena vista.

5. Planeación agrega una columna `Observaciones` con texto al final de la hoja. Con "otras columnas", ¿dónde aparece, y qué te avisaría?

Respuesta: Como `Observaciones` no está en la lista fija (`finca_id`, `finca`), Power Query la mete en "otras columnas" y la anula junto con los meses: aparecería como filas extra con ese texto en `fecha_mes` (columna Atributo) y el contenido de la observación en `kg_meta` (columna Valor). El aviso sería el mismo que con la columna Total: `Error` al intentar convertir ese texto a Fecha, o al convertir texto a número entero en `kg_meta`.
