# Clase 27 · La fecha al revés
**Viernes 25 de septiembre**

**50 minutos de clase** y el resto de práctica. De nuevo del lado de Power BI, y hoy por primera vez **adentro de Power Query**: **hoy tampoco se prende Oracle.**

## Material

| Qué | Dónde |
|---|---|
| Diapositivas | [slides.md](slides.md) · [versión web](https://negatix092.github.io/Semillero_SQL/27-power-query-fechas.html) |
| Ejercicio práctico | [ejercicio.md](ejercicio.md) |
| Datos del día | [`datos/csv_clase27/`](../../datos/csv_clase27/): los seis CSV de la clase 19, **idénticos**, más **`cosechas_bascula.csv`**, las diez pesadas de mayo a agosto de 2026 |

> **Hoy sí se baja algo.** Copia `datos/csv_clase27/` a `C:\agrodb27\` y empieza con un **`.pbix` vacío**, como en la clase 24 y la 26.

## Qué hace falta tener listo

| | |
|---|---|
| Power BI Desktop | y nada más |
| Oracle, Docker, el OCMT | **no**, hoy tampoco |
| El `.pbix` de clases anteriores | **no**: hoy se arma uno nuevo con cinco CSV, y la báscula entra por Power Query |

## De qué se trata

Desde mayo, las fincas pesan en **básculas digitales**, y el sistema de las básculas exporta su propio CSV con las mismas seis columnas que `h_cosecha`. Operaciones lo manda con un correo: **10 pesadas, 16 700 kg, del 7 de mayo al 24 de agosto**. Hay que anexarlo al tablero.

Lo único distinto es la fecha: el sistema está en inglés de Estados Unidos y escribe **mes/día/año**. `05/07/2026` es el **7 de mayo**. Un CSV es texto, y quien decide qué es cada número es **Power Query**, en el paso **Tipo cambiado**, con la **configuración regional**. En español, `05/07/2026` es el **5 de julio**.

## El giro de hoy

Al traer el archivo, **tres fechas dicen `Error`**: `05/14`, `05/20` y `08/24`, porque no existe el mes 14. El arreglo obvio es un botón: **Quitar errores**. La calidad de columna pasa a **100 % válido**, se anexa, se carga, y ningún aviso.

Pero mayo a agosto dice **10 000 kg**, no 16 700. **Agosto no aparece**. El Guayabo sale en **17,75 %** de su meta. Y lo más raro: el número de control de enero–abril, que el archivo nuevo no tenía por qué tocar, pasa de **30 550** a **30 950**. Es la cosecha **31**, cacao del **2 de julio**, leído como **7 de febrero**.

`Quitar errores` no arregló nada: quitó **3 filas y 3 500 kg**. Y las siete «válidas» se leyeron con la misma regla equivocada: **seis al revés** y una bien por casualidad (`06/06`).

El arreglo es poner la regla **en el paso**: **Cambiar tipo → Usar configuración regional → Fecha, Inglés (Estados Unidos)**. Mayo a agosto vuelve a **16 700**, El Guayabo a **85,80 %**, el total a **108,02 %**, y enero–abril regresa a **30 550 / 125,00 %**.

> Ninguna fecha estaba mal escrita. Lo que estaba mal era **la regla con la que se leyeron las diez**.

## Lo que hay que saber al terminar

- Que un CSV es **texto**, y que la fecha la decide el paso **Tipo cambiado** con la **configuración regional**
- Leer los **Pasos aplicados** de una consulta, y borrar uno
- **Vista → Calidad de columna**, y por qué **100 % válido** no quiere decir **bien leído**
- Que **`Quitar errores` quita filas**, no errores
- **Anexar consultas**, y **Habilitar carga**
- **Usar configuración regional** en el paso de la columna, y por qué ahí y no en todo el archivo
- **Reconciliar** un archivo contra su origen: filas, kilos, primera y última fecha, y un número viejo que no debía moverse

## La idea del día

**Si tres fechas no se pudieron leer, las otras siete se leyeron con la misma regla equivocada.**

## Las cuatro cosas que son la clase

Si el día se complica y hay que recortar, estas no se recortan:

1. **Los tres `Error`** en Power Query, y la calidad de columna en 70 %.
2. **`Quitar errores`**: 100 % válido, y mayo a agosto en **10 000**, con El Guayabo en **17,75 %**.
3. **El 30 550 que se movió** a 30 950: la cosecha 31 en febrero.
4. **La regla en el paso**: 16 700, 108,02 %, y enero–abril de vuelta en 30 550.

## Los números de control

Con segmentadores en `anio` = 2026, salvo donde se dice.

| Dónde | Qué debe decir |
|---|---|
| La tabla por finca, `mes` 1–4, antes de la báscula | La Union **2 100 / 5 000 / 42,00 % / 2** · El Guayabo **14 250 / 9 440 / 150,95 % / 3** · Santa Rosa **14 200 / 10 000 / 142,00 % / 4** · total **30 550 / 24 440 / 125,00 % / 9** |
| Power Query, `fecha` de la báscula | calidad **70 % válido / 30 % error**: las cosechas 27, 28 y 35 |
| Con `Quitar errores`, `mes` 5–8, por mes | Mayo **2 500** · Junio **4 100** · Julio **3 400** · Agosto no aparece · total **10 000** en **4** cosechas |
| Lo mismo por finca | La Union sin kilos · El Guayabo **1 200 / 6 760 / 17,75 %** · Santa Rosa **8 800 / 7 200 / 122,22 %** · total **10 000 / 15 460 / 64,68 %** |
| Con `Quitar errores`, `mes` 1–4 | La Union **2 500 / 50,00 % / 3** · total **30 950 / 126,64 % / 10** |
| Lo que se llevó `Quitar errores` | **3** filas y **3 500** kg: 1 800 + 600 + 1 100 |
| Con la regla en el paso, `mes` 5–8, por mes | Mayo **5 800** · Junio **4 400** · Julio **1 700** · Agosto **4 800** · total **16 700** en **10** cosechas |
| Lo mismo por finca | La Union **66,67 %** · El Guayabo **85,80 %** · Santa Rosa **137,50 %** · total **16 700 / 15 460 / 108,02 %** |
| Con la regla en el paso, `mes` 1–4 | de vuelta en **30 550 / 24 440 / 125,00 % / 9** |
| Las cuatro tarjetas, sin segmentadores | bien leído **10 / 16 700 / 7 de mayo / 24 de agosto** · al revés **7 / 13 200 / 7 de febrero / 7 de octubre** |

## Entrega

En `entregas/apellido-nombre/`, por *pull request*, **tres archivos**:

| Archivo | Qué lleva |
|---|---|
| `Ejercicio27_Apellido_Nombre.md` | cada medida con **su resultado anotado debajo**, las tablas que se piden y las respuestas |
| `clase27-calidad.png` | Power Query con la **calidad de columna** de `fecha` en **70 % / 30 %** |
| `clase27-bascula.png` | las cuatro tarjetas bien leídas: **10 / 16 700 / 7 de mayo / 24 de agosto** |

> El `.pbix` no se entrega: el repositorio lo ignora a propósito.

## Nota sobre el material

Los números de esta clase —cómo se lee cada fecha con día/mes y con mes/día, lo que se lleva `Quitar errores`, las tablas por mes y por finca, el 30 950 y las cuatro tarjetas— están **verificados contra los CSV publicados en `datos/csv_clase27/`** con `docente/clase27_verificacion_docente.py`, que lee cada fecha de la báscula con las dos reglas y modela `Quitar errores` y el anexado.

Lo que **ningún script puede verificar** es lo que hace Power Query en tu versión: si el **Tipo cambiado** automático deja `fecha` como Fecha con tres errores o como texto, el texto literal del error, cómo se llaman los pasos y la opción de configuración regional en tu idioma. Si en tu máquina algo sale distinto, **anótalo en la entrega: eso puntúa**.
