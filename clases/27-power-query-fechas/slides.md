---
marp: true
paginate: true
theme: default
title: "Clase 27 · La fecha al revés"
style: |
  section { font-family: system-ui, -apple-system, "Segoe UI", sans-serif; font-size: 26px; background: #fbfbfa; color: #1f2933; padding: 60px 70px; }
  section.lead { background: #16324f; color: #f4f7fa; }
  section.lead h1 { color: #ffffff; font-size: 54px; line-height: 1.1; }
  section.lead h2 { color: #7fb3d5; font-weight: 400; font-size: 30px; }
  h1 { color: #16324f; font-size: 40px; border-bottom: 3px solid #f2a104; padding-bottom: 10px; }
  h2 { color: #1c7293; font-size: 32px; }
  strong { color: #b3541e; }
  code { background: #eef2f6; padding: 1px 6px; border-radius: 4px; }
  pre { background: #16324f; border-radius: 8px; font-size: 20px; }
  pre code { background: transparent; color: #e8eef4; }
  table { font-size: 23px; }
  th { background: #16324f; color: #fff; }
  blockquote { border-left: 5px solid #f2a104; color: #4a5568; font-style: normal; }
  footer { color: #8a99a8; font-size: 16px; }
  .verde { background: #1E8449; color: #fff; padding: 2px 10px; border-radius: 4px; }
  .rojo { background: #C0392B; color: #fff; padding: 2px 10px; border-radius: 4px; }
footer: "Curso de SQL · AgroDB · Clase 27"
---

<!-- _class: lead -->

# La fecha al revés

## Power Query, la configuración regional y las filas que «Quitar errores» se llevó

Clase 27 · 25 de septiembre

---

# Lo que llega hoy

Desde mayo, las tres fincas pesan en **básculas digitales**. El sistema de las básculas exporta su propio archivo: `cosechas_bascula.csv`.

| `cosecha_id` | `finca_id` | `cultivo_id` | `fecha` | `calidad` | `kg` |
|---|---|---|---|---|---|
| 26 | 1 | 1 | **05/07/2026** | primera | 3 400 |
| 27 | 2 | 2 | **05/14/2026** | primera | 1 800 |
| 31 | 3 | 3 | **07/02/2026** | primera | 400 |

Las mismas seis columnas que `h_cosecha`. Y un correo de operaciones:

> *«Van las **10 pesadas** de mayo a agosto, **16 700 kg**. Del 7 de mayo al 24 de agosto. Súmenlas al tablero.»*

---

# Quién lee la fecha

Un CSV es **texto**. `05/07/2026` no es una fecha hasta que alguien decide qué es el `05` y qué es el `07`.

<br>

| El sistema de la báscula | Tu Power BI |
|---|---|
| inglés de Estados Unidos | español: **día/mes/año** |
| `05/07/2026` = **7 de mayo** | `05/07/2026` = **5 de julio** |

<br>

Quien decide es **Power Query**, en el paso **Tipo cambiado**, con la **configuración regional**. Para que todos veamos lo mismo, hoy la del archivo se fija en **Español (México)**.

> La fecha no viene en el archivo. **Viene en la regla con la que se lee.**

---

# El modelo de hoy

Un `.pbix` nuevo con cinco CSV de `datos/csv_clase27/`, las **cinco relaciones** dibujadas a mano, y una medida más:

```
Cosechas = COUNTROWS( h_cosecha )
```

Segmentadores `anio` = 2026 y `mes` en 1–4. La tabla por finca de siempre:

| Finca | `[Kilos]` | `[Meta]` | `[Cumplimiento]` | `[Cosechas]` |
|---|---|---|---|---|
| Agricola La Union | 2 100 | 5 000 | 42,00 % | 2 |
| Finca El Guayabo | 14 250 | 9 440 | 150,95 % | 3 |
| Hacienda Santa Rosa | 14 200 | 10 000 | 142,00 % | 4 |
| **Total** | **30 550** | **24 440** | **125,00 %** | **9** |

> Guárdense el **30 550**: el archivo nuevo es de mayo a agosto, así que **no tiene por qué moverlo**.

---

# El archivo de la báscula

**Obtener datos → Texto o CSV** → `cosechas_bascula.csv` → **Transformar datos**. Se abre **Power Query**.

<br>

| Pasos aplicados | |
|---|---|
| Origen | lee el texto |
| Encabezados promovidos | la primera fila se vuelve el nombre de las columnas |
| **Tipo cambiado** | aquí se decide qué es cada `fecha` |

<br>

En la columna `fecha`, **tres celdas dicen `Error`**. Con **Vista → Calidad de columna** encendida:

| `fecha` | Válido **70 %** | Error **30 %** | Vacío 0 % |
|---|---|---|---|

---

# El arreglo obvio

Tres errores de diez. Power Query tiene un botón para eso:

**Inicio → Quitar filas → Quitar errores.**

<br>

La calidad de columna pasa a **Válido 100 %**. Luego:

1. `h_cosecha` → **Inicio → Anexar consultas** → `cosechas_bascula`.
2. Clic derecho en `cosechas_bascula` → desmarcar **Habilitar carga**.
3. **Cerrar y aplicar.**

<br>

> Sin una sola celda en rojo. Todo **válido**.

---

# Mayo a agosto

Segmentador de `mes` en **5–8**. La tabla por mes y la tabla por finca:

| `nombre_mes` | `[Kilos]` | `[Cosechas]` |
|---|---|---|
| Mayo | 2 500 | 1 |
| Junio | 4 100 | 2 |
| Julio | 3 400 | 1 |
| **Total** | **10 000** | **4** |

| Finca | `[Kilos]` | `[Meta]` | `[Cumplimiento]` |
|---|---|---|---|
| Agricola La Union | | 1 500 | |
| Finca El Guayabo | 1 200 | 6 760 | <span class="rojo">**17,75 %**</span> |
| Hacienda Santa Rosa | 8 800 | 7 200 | 122,22 % |
| **Total** | **10 000** | **15 460** | **64,68 %** |

---

# ¿Y qué error dio?

## Ninguno. La calidad de columna dice 100 %.

<br>

- **Agosto no aparece**, y la báscula pesó hasta el 24 de agosto.
- La tabla cuenta **4 cosechas**; el correo dice **10**.
- El gerente de El Guayabo recibe un **17,75 %**, y el lunes le van a preguntar qué pasó.

<br>

> Quitamos los tres errores que se veían. Los que **no se veían** siguen ahí.

---

# El número que no debía moverse

Regresa el segmentador de `mes` a **1–4**. La tabla por finca de la lámina 4:

| Finca | `[Kilos]` | `[Meta]` | `[Cumplimiento]` | `[Cosechas]` |
|---|---|---|---|---|
| Agricola La Union | **2 500** | 5 000 | **50,00 %** | **3** |
| Finca El Guayabo | 14 250 | 9 440 | 150,95 % | 3 |
| Hacienda Santa Rosa | 14 200 | 10 000 | 142,00 % | 4 |
| **Total** | <span class="rojo">**30 950**</span> | **24 440** | <span class="rojo">**126,64 %**</span> | **10** |

<br>

Un archivo **de mayo a agosto** le agregó **400 kg a febrero**. Es la cosecha **31**: `07/02/2026`, el cacao de La Unión del **2 de julio**, leído como **7 de febrero**.

> La Unión «mejoró» en febrero, cinco meses después.

---

# La fecha al revés

Las diez, leídas con **día/mes/año**:

| Grupo | `cosecha_id` | Texto | Es | Se leyó |
|---|---|---|---|---|
| **Error** | 27 · 28 · 35 | `05/14` `05/20` `08/24` | 14 y 20 de mayo, 24 de agosto | no existe el mes 14 |
| **Igual** | 29 | `06/06` | 6 de junio | 6 de junio, por casualidad |
| **Al revés** | 26 | `05/07` | 7 de mayo | 5 de julio |
| | 30 | `06/09` | 9 de junio | 6 de septiembre |
| | 31 | `07/02` | 2 de julio | **7 de febrero** |
| | 32 | `07/10` | 10 de julio | 7 de octubre |
| | 33 | `08/05` | 5 de agosto | 8 de mayo |
| | 34 | `08/06` | 6 de agosto | 8 de junio |

> Seis fechas **al revés** y ninguna se quejó: un día menor que 13 **siempre** parece un mes.

---

# Los errores eran el aviso

`Quitar errores` **no arregla nada**: quita **filas**. Se llevó 3 cosechas y **3 500 kg** (1 800 + 600 + 1 100).

<br>

| Lo que vimos | Lo que significaba |
|---|---|
| 3 celdas con `Error` | la regla día/mes **no sirve para este archivo** |
| 7 celdas válidas | leídas **con la misma regla** que no sirve |

<br>

> Si tres fechas no se pudieron leer, **las otras siete se leyeron con la misma regla equivocada**. El error no estaba en tres filas: estaba en el paso.

---

# El arreglo: la regla en el paso

En `cosechas_bascula`, **Pasos aplicados**, borra de abajo hacia arriba **Errores quitados** y **Tipo cambiado**. Luego:

1. Clic derecho en `fecha` → **Cambiar tipo → Usar configuración regional…**
2. Tipo de datos **Fecha**, configuración regional **Inglés (Estados Unidos)**.
3. `cosecha_id`, `finca_id`, `cultivo_id` y `kg` → **Número entero**. **Cerrar y aplicar.**

| `mes` 5–8 | `[Kilos]` | `[Cosechas]` | | Finca | `[Cumplimiento]` |
|---|---|---|---|---|---|
| Mayo | 5 800 | 3 | | Agricola La Union | 66,67 % |
| Junio | 4 400 | 2 | | Finca El Guayabo | <span class="verde">**85,80 %**</span> |
| Julio | 1 700 | 2 | | Hacienda Santa Rosa | 137,50 % |
| Agosto | 4 800 | 3 | | **Total** | **108,02 %** |
| **Total** | **16 700** | **10** | | | |

Y enero–abril regresa a **30 550 / 125,00 %**.

---

# ¿Y si cambio la del archivo?

El atajo: **Archivo → Opciones → Configuración regional** del `.pbix` en Inglés (Estados Unidos). Con este archivo, hoy, **también funciona**.

<br>

| Dónde va la regla | Qué pasa el día que llega otro CSV |
|---|---|
| en el **archivo** `.pbix` | lo lee **todo** en mes/día: el de una finca que escribe `05/07` como 5 de julio se voltea **al revés al revés** |
| en el **paso** de la columna | cada archivo se lee con **su** regla, y la regla queda escrita junto a la columna que la necesita |

<br>

> Es la clase 26 con otra ropa: cambiar lo que usa **todo** para arreglar **una** cosa.

---

# La prueba de las cuatro cifras

Una **página nueva, sin segmentadores**, con cuatro tarjetas que miran solo lo de la báscula:

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

| | El correo | Al revés, sin errores | Bien leído |
|---|---|---|---|
| Pesadas | 10 | <span class="rojo">7</span> | **10** |
| Kilos | 16 700 | <span class="rojo">13 200</span> | **16 700** |
| Primera | 7 de mayo | <span class="rojo">7 de febrero</span> | **7 de mayo** |
| Última | 24 de agosto | <span class="rojo">7 de octubre</span> | **24 de agosto** |

---

# La prueba de cada archivo nuevo

| Qué revisas | Qué atrapa |
|---|---|
| **cuántas filas** llegaron, contra las que dice el origen | las que `Quitar errores` se llevó |
| **cuántos kilos**, contra el total del correo | lo mismo, en kilos |
| **primera y última fecha**, contra el periodo del archivo | la fecha al revés |
| un **número viejo** que no tenía por qué moverse | la fecha al revés que cayó en otro periodo |

<br>

> **Si al cargar datos nuevos se mueve un número viejo, los datos nuevos no dicen lo que crees.**

---

# Los errores que van a ver hoy

| Síntoma | Qué pasó | Arreglo |
|---|---|---|
| `fecha` de la báscula sin ningún `Error` | tu configuración regional es inglés | fija **Español (México)** en el archivo, como dice la parte A |
| `fecha` sale con el ícono **ABC** | Power Query la dejó como texto | clic en el ícono → **Fecha**: ahí aparecen los tres errores |
| **Agosto** no aparece | quitaste los errores | borra **Errores quitados** y **Tipo cambiado** |
| enero–abril dice **30 950** | la cosecha 31 cayó en febrero | la regla en el paso: **Inglés (Estados Unidos)** |
| aparece una tabla `cosechas_bascula` en el modelo | no desmarcaste **Habilitar carga** | desmárcalo; no afecta los números |
| todo sale **doble** | anexaste dos veces | en `h_cosecha`, deja un solo paso **Consulta anexada** |

---

<!-- _class: lead -->

# La idea del día

## Si tres fechas no se pudieron leer, las otras siete se leyeron con la misma regla equivocada.

<br>

`Quitar errores` dejó la columna **100 % válida** y se llevó **3 500 kg**; las seis fechas al revés movieron El Guayabo a **17,75 %** y un cacao de julio a **febrero**. Con la regla en el paso, **Inglés (Estados Unidos)**, mayo a agosto dice **16 700** y enero–abril vuelve a **30 550**.

<br>

**Práctica:** arma el modelo, carga la báscula, cae en `Quitar errores`, encuentra la cosecha que llegó a febrero, pon la regla en el paso y reconcilia contra el correo.

**Y la báscula tiene que decir 10 pesadas y 16 700 kg, no 7 y 13 200.**
