# Ejercicio práctico 27 · Las pesadas de la báscula

**Duración: 2 horas · Individual · Solo Power BI Desktop · Entrega: un archivo `.md` y dos capturas**

---

## Qué vas a lograr hoy

1. Armar el modelo de siempre y fijar la **configuración regional** del archivo antes de cargar nada.
2. Traer un CSV nuevo por **Power Query**, con **Transformar datos**, y leer sus **Pasos aplicados**.
3. Caer en el arreglo obvio: **Quitar errores**, anexar y cargar. Ver que todo sale «válido» y que los números no cuadran.
4. Encontrar **cuáles fechas se leyeron al revés** sin quejarse, y la cosecha de julio que llegó a febrero.
5. Poner la regla donde va: **en el paso**, con **Usar configuración regional**.
6. Reconciliar el archivo contra el correo con **cuatro cifras**.

**El número que prueba que el tablero quedó bien: la báscula son 10 pesadas y 16 700 kg, no 7 y 13 200.** El que prueba que entendiste la clase es que sepas por qué `Quitar errores` no arregló nada.

---

## Antes de empezar

De nuevo solo Power BI: hoy tampoco se prende Oracle, ni Docker, ni el driver.

| | |
|---|---|
| Oracle, Docker, OCMT, `GRANT` | **no** |
| Power BI Desktop | **sí**, y es lo único |

### Lo que se baja hoy

**Hoy sí se baja algo:** [`datos/csv_clase27/`](../../datos/csv_clase27/). Son los seis CSV de la clase 19, **idénticos**, más uno nuevo: **`cosechas_bascula.csv`**, las diez pesadas de mayo a agosto de 2026.

Copia la carpeta completa a `C:\agrodb27\` y trabaja sobre esa copia.

> **Hoy se empieza con un `.pbix` vacío**, como en la clase 24 y la 26.

### El correo de operaciones

> *«Van las **10 pesadas** de mayo a agosto, **16 700 kg**. Del 7 de mayo al 24 de agosto. Súmenlas al tablero.»*

Guárdalo. Al final del día lo vas a comparar contra el tablero.

---

## Cómo se entrega

En `entregas/apellido-nombre/`, por *pull request*, **tres archivos**:

| Archivo | Qué lleva |
|---|---|
| `Ejercicio27_Apellido_Nombre.md` | cada medida en un bloque de código **con su resultado anotado debajo**, las tablas que se piden y las respuestas a las preguntas |
| `clase27-calidad.png` | **Power Query** con `cosechas_bascula` abierta y la **calidad de columna** de `fecha` en **70 % válido / 30 % error** |
| `clase27-bascula.png` | la **página de las cuatro tarjetas**, bien leída: **10 / 16 700 / 7 de mayo / 24 de agosto** |

> El `.pbix` no se entrega: el repositorio lo ignora a propósito.

---

## Parte A · El modelo (20 min)

### A0. La configuración regional, antes de cargar nada

**Archivo → Opciones y configuración → Opciones → Archivo actual → Configuración regional**: en **Configuración regional para importación** elige **Español (México)** → **Aceptar**.

Así todos leemos las fechas igual: **día/mes/año**, como cualquier laptop configurada en español de Latinoamérica. **A0a.** Anota qué configuración tenía antes de cambiarla.

### A1. Cargar

1. En la misma ventana de opciones, **Carga de datos** (en **Archivo actual**): apaga **«Detectar automáticamente nuevas relaciones después de cargar los datos»**. Como en la clase 24.
2. **Inicio → Obtener datos → Texto o CSV**, uno por uno, con **Cargar**: `h_cosecha`, `h_meta`, `dim_tiempo`, `dim_finca` y `dim_cultivo`. **`seguridad.csv` no se carga hoy, y `cosechas_bascula.csv` todavía no.**

### A2. Las cinco relaciones

En la **vista de modelo**, arrastra cada columna sobre la otra:

| # | De | A |
|---|---|---|
| 1 | `h_cosecha[finca_id]` | `dim_finca[finca_id]` |
| 2 | `h_cosecha[cultivo_id]` | `dim_cultivo[cultivo_id]` |
| 3 | `h_cosecha[fecha]` | `dim_tiempo[fecha]` |
| 4 | `h_meta[finca_id]` | `dim_finca[finca_id]` |
| 5 | `h_meta[fecha_mes]` | `dim_tiempo[fecha]` |

### A3. El calendario y las medidas

1. `dim_tiempo` → **Marcar como tabla de fechas** → `fecha`.
2. `nombre_mes` → **Ordenar por columna** → `mes`.
3. Con `h_cosecha` seleccionada en el panel **Datos**, **Nueva medida**, una por una:

```
Kilos = SUM( h_cosecha[kg] )
```

```
Meta = SUM( h_meta[kg_meta] )
```

```
Cumplimiento = DIVIDE( [Kilos] , [Meta] )
```

```
Cosechas = COUNTROWS( h_cosecha )
```

### A4. La tabla de control

Segmentadores `dim_tiempo[anio]` = **2026** y `dim_tiempo[mes]` de **1 a 4**. Tabla con `dim_finca[finca]`, `[Kilos]`, `[Meta]`, `[Cumplimiento]` y `[Cosechas]`:

| Finca | `[Kilos]` | `[Meta]` | `[Cumplimiento]` | `[Cosechas]` |
|---|---|---|---|---|
| Agricola La Union | 2 100 | 5 000 | 42,00 % | 2 |
| Finca El Guayabo | 14 250 | 9 440 | 150,95 % | 3 |
| Hacienda Santa Rosa | 14 200 | 10 000 | 142,00 % | 4 |
| **Total** | **30 550** | **24 440** | **125,00 %** | **9** |

Si el total no dice **125,00 %**, no sigas: revisa las relaciones 3 y 5.

---

## Parte B · El archivo de la báscula (15 min)

**B1.** **Inicio → Obtener datos → Texto o CSV** → `cosechas_bascula.csv` → **Transformar datos** (no **Cargar**). Se abre el **Editor de Power Query**.

**B2.** A la derecha, en **Pasos aplicados**, anota los pasos que creó Power Query solo.

**B3.** Mira el ícono a la izquierda del nombre de la columna `fecha`. Tiene que ser el **calendario** (Fecha). Si es **ABC** (texto), haz clic en el ícono → **Fecha** y anótalo.

**B4.** **Vista → Calidad de columna**. Anota lo que dice debajo de `fecha`: **Válido**, **Error** y **Vacío**. Tiene que salir **70 % / 30 % / 0 %**.

> **Toma aquí la captura `clase27-calidad.png`**: Power Query con la calidad de columna de `fecha` a la vista.

**B5.** Haz clic en el espacio en blanco junto a una celda que diga `Error` (no en la palabra). Copia **literal** el mensaje que sale abajo.

**B6.** Abre `cosechas_bascula.csv` en el Bloc de notas. ¿Qué `cosecha_id` tienen las tres filas con error, y qué tienen en común sus fechas?

---

## Parte C · El arreglo obvio (20 min)

**C1.** Con `cosechas_bascula` seleccionada: **Inicio → Quitar filas → Quitar errores**. Anota la calidad de columna de `fecha` ahora.

**C2.** Selecciona la consulta `h_cosecha` (panel de la izquierda) → **Inicio → Anexar consultas** → **Dos tablas** → tabla para anexar: `cosechas_bascula` → **Aceptar**.

**C3.** Clic derecho en `cosechas_bascula` → desmarca **Habilitar carga**. Lo que se carga es `h_cosecha`, que ya la trae adentro.

**C4.** **Inicio → Cerrar y aplicar.**

**C5.** Pon el segmentador de `mes` de **5 a 8**. Tabla nueva con `dim_tiempo[nombre_mes]`, `[Kilos]` y `[Cosechas]`. Copia la tabla. Tiene que salir Mayo **2 500**, Junio **4 100**, Julio **3 400**, total **10 000** y **4** cosechas. ¿Qué mes falta?

**C6.** La tabla por finca de A4, con `mes` de 5 a 8. Copia la tabla. Tiene que salir El Guayabo en **17,75 %** y el total en **64,68 %**.

**C7.** Regresa el segmentador de `mes` a **1–4**. ¿Sigue la tabla de A4 en **30 550** y **125,00 %**? Anota lo que dice ahora y qué finca cambió. Tiene que salir **30 950** y **126,64 %**, con **10** cosechas.

**C8.** Con `cosechas_bascula.csv` abierto: ¿**qué cosecha** es la que apareció en enero–abril? ¿Qué fecha trae el archivo, qué día es en realidad y en qué día la puso Power Query?

---

## Parte D · La regla en el paso (30 min) — es la parte que más vale

**D1.** Copia esta tabla en tu `.md` y llena la última columna **con el CSV abierto**, leyendo cada fecha como **día/mes/año**:

| `cosecha_id` | Texto en el CSV | Es (mes/día) | Se leyó (día/mes) |
|---|---|---|---|
| 26 | `05/07/2026` | 7 de mayo | |
| 27 | `05/14/2026` | 14 de mayo | |
| 28 | `05/20/2026` | 20 de mayo | |
| 29 | `06/06/2026` | 6 de junio | |
| 30 | `06/09/2026` | 9 de junio | |
| 31 | `07/02/2026` | 2 de julio | |
| 32 | `07/10/2026` | 10 de julio | |
| 33 | `08/05/2026` | 5 de agosto | |
| 34 | `08/06/2026` | 6 de agosto | |
| 35 | `08/24/2026` | 24 de agosto | |

¿Cuántas quedaron **al revés** sin marcar error? ¿Cuál salió bien, y por qué?

**D2.** **Transformar datos** → consulta `cosechas_bascula` → en **Pasos aplicados**, borra (con la **X**) primero **Errores quitados** y luego **Tipo cambiado**. Si Power Query avisa que otros pasos dependen de ese, acepta y anótalo.

**D3.** Clic derecho en el encabezado de `fecha` → **Cambiar tipo → Usar configuración regional…** → Tipo de datos **Fecha**, configuración regional **Inglés (Estados Unidos)** → **Aceptar**. La calidad de columna tiene que decir **100 % válido**, ahora con las diez filas.

**D4.** Selecciona `cosecha_id`, `finca_id`, `cultivo_id` y `kg` (con **Ctrl**) → **Transformar → Tipo de datos → Número entero**. **Cerrar y aplicar.**

**D5.** Con `mes` de **5 a 8**, la tabla por mes. Copia la tabla. Tiene que salir:

| `nombre_mes` | `[Kilos]` | `[Cosechas]` |
|---|---|---|
| Mayo | 5 800 | 3 |
| Junio | 4 400 | 2 |
| Julio | 1 700 | 2 |
| Agosto | 4 800 | 3 |
| **Total** | **16 700** | **10** |

**D6.** La tabla por finca con `mes` de 5 a 8. Tiene que salir La Union **66,67 %**, El Guayabo **85,80 %**, Santa Rosa **137,50 %** y el total **108,02 %**, con Meta **15 460**.

**D7.** Regresa `mes` a **1–4**: la tabla de A4 tiene que volver a **30 550 / 125,00 % / 9**. Anota que volvió.

**D8.** El Guayabo pasó de **17,75 %** a **85,80 %**. Explícalo **con números de `cosecha_id`**: cuáles de sus pesadas se había llevado `Quitar errores`, y cuáles se habían ido fuera de mayo–agosto.

---

## Parte E · La prueba de las cuatro cifras (25 min)

**E1.** **Página nueva**, **sin segmentadores**. Crea las cuatro medidas, con `h_cosecha` seleccionada:

```
Pesadas bascula = CALCULATE( [Cosechas] , h_cosecha[cosecha_id] >= 26 )
```

```
Kilos bascula = CALCULATE( [Kilos] , h_cosecha[cosecha_id] >= 26 )
```

```
Primera pesada = CALCULATE( MIN( h_cosecha[fecha] ) , h_cosecha[cosecha_id] >= 26 )
```

```
Ultima pesada = CALCULATE( MAX( h_cosecha[fecha] ) , h_cosecha[cosecha_id] >= 26 )
```

**E2.** Una **tarjeta** por medida. Tienen que decir **10**, **16 700**, **7 de mayo de 2026** y **24 de agosto de 2026** (el formato de la fecha depende de tu Windows: lo que importa es el día).

> **Toma aquí la captura `clase27-bascula.png`**: las cuatro tarjetas.

**E3.** Sin volver a romper nada: con tu tabla de D1, ¿qué habrían dicho las **cuatro tarjetas** con la versión de la parte C? Tiene que salir **7**, **13 200**, **7 de febrero** y **7 de octubre**. Explica de dónde sale cada una.

**E4.** Un compañero propone algo más fácil: cambiar la configuración regional **del archivo** a Inglés (Estados Unidos) y dejar el **Tipo cambiado** automático. Con este CSV, ¿funciona? En dos líneas: ¿qué pasa el día que llegue el CSV de una finca que escribe `05/07/2026` como **5 de julio**?

**E5.** En una línea: ¿por qué la prueba del **número viejo** (enero–abril en 30 550) atrapó la cosecha 31, y **no** habría atrapado la 26?

---

## Parte F · Preguntas de cierre (10 min)

1. En una línea: ¿qué hace **`Quitar errores`** con una fila, y qué **no** hace?
2. En una línea: ¿dónde vive la regla que dice que `05/07/2026` es el 7 de mayo: en el CSV, en la columna o en el paso?
3. En una línea: escribe la **regla de detección** del día, tomando como base la de la clase 26: *«si dos fechas cuentan lo mismo en todos los meses, una de las dos no se está usando»*.
4. En dos líneas: un banco recibe un archivo de pagos de una sucursal en Miami. La calidad de columna dice **100 % válido** y **ninguna** fecha tiene día mayor que 12. ¿Qué revisarías, y por qué no basta con la calidad de columna?
5. En una línea: la clase 12 terminó «bien» con ocho filas rechazadas. ¿En qué se parece a `Quitar errores`?

---

## Si algo falla

| Síntoma | Qué pasó | Qué haces |
|---|---|---|
| La báscula entra **sin ningún `Error`** | no fijaste la configuración regional en A0, y tu Windows está en inglés | haz A0 y vuelve a crear la consulta |
| `fecha` de la báscula con el ícono **ABC** | Power Query la dejó como texto | B3: clic en el ícono → **Fecha** |
| No encuentras **Configuración regional para importación** | el nombre cambia entre versiones | búscala en **Archivo actual → Configuración regional**, anota cómo se llama en la tuya, y sigue |
| **Agosto** no aparece en C5 | es lo que tiene que pasar en la parte C | sigue a la parte D |
| En D5 sigue sin salir agosto | no borraste **Errores quitados** | vuelve a D2 |
| En D3 la calidad dice **100 %** pero con **7** filas | borraste **Tipo cambiado** y dejaste **Errores quitados** | borra también **Errores quitados** |
| **Todo sale doble** | anexaste dos veces | en `h_cosecha`, **Pasos aplicados**: deja un solo **Consulta anexada** |
| Aparece la tabla `cosechas_bascula` en el modelo | no desmarcaste **Habilitar carga** | desmárcala en Power Query; no mueve los números |
| Error al anexar, o `fecha` de `h_cosecha` queda como texto | una de las dos consultas tiene `fecha` como texto | revisa el ícono de `fecha` en las dos |
| Los decimales o los miles salen distintos | configuración regional de tu Windows | **no es un error**, anótalo y sigue |

> ### La regla de los 20 minutos sigue vigente
> Veinte minutos atorado en lo mismo: lo escribes en tu archivo empezando con `DUDA`, o abres un *issue*, y sigues con lo siguiente. **Atorarse no baja la nota. Quedarse callado sí.**

---

## Plan B · Si Power BI Desktop no abre en tu máquina

1. Ponte con un compañero: el modelo se arma en una sola máquina.
2. **Tú escribes todas las medidas** en tu propio archivo `.md`, con sus resultados, y anotas con quién trabajaste.
3. **La tabla de D1 y las respuestas de B6, C8, D8, E3 y E5 las haces tú**, con tus propias palabras y con `cosechas_bascula.csv` abierto: se contestan leyendo el CSV.
4. Las capturas son las mismas para los dos, y los dos lo dicen en un comentario.

**Con el Plan B completo se llega a 100 de 100.** Lo que se califica es qué fecha se leyó y por qué, no de quién era la laptop.

---

## Rúbrica (100 puntos)

| Criterio | Pts |
|---|---|
| Parte A: la configuración regional, el modelo con sus cinco relaciones, las medidas, y la tabla de control en 125,00 % | 10 |
| Parte B: los pasos aplicados, la captura de la calidad de columna, el mensaje del error, y B6 | 10 |
| Parte C: las tablas con 10 000 y 64,68 %, el 30 950 de enero–abril, y la cosecha de C8 | 20 |
| **Parte D: la tabla de D1, la regla en el paso, 16 700 y 108,02 %, enero–abril de vuelta en 30 550, y D8** | **30** |
| Parte E: las cuatro tarjetas con 10 / 16 700, la captura, y E3, E4 y E5 | 20 |
| Parte F: las cinco preguntas con criterio | 10 |

Los criterios suman **100** exactos.

> **Lo que más se califica hoy no es el clic de Usar configuración regional**, que se copia del enunciado. Es la tabla de D1: que hayas visto que las siete fechas «válidas» se leyeron con la misma regla que rompió las otras tres, y que sepas explicar con `cosecha_id` **de dónde salió** el 17,75 % de El Guayabo.
