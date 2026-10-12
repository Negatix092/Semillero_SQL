# Punto de Control 1 · El inventario
**Estudiante:** Daniel Troya  
**Fecha:** Lunes 5 de octubre de 2026  

---

## 1. Anclas del lunes

| Qué | Esperado | En pantalla | Estado |
|---|---|---|---|
| Filas de `dim_finca` | **4** | 4 | Cuadra |
| Filas de `dim_cultivo` | **6** | 6 | Cuadra |
| Filas de `dim_tiempo` | **730** | 730 | Cuadra |
| Filas de `seguridad` | **10** | 10 | Cuadra |
| Archivos en carpeta `bascula\` | **19** | 19 | Cuadra |

---

## 2. Preguntas ciegas

- **L1. Según los archivos, sin cargar nada: ¿cuántas cosechas distintas hay en total, sumando la carpeta y la báscula digital?**  
  Hay **53 cosechas distintas** (de un total de 57 registros en bruto considerando duplicados).

- **L2. ¿Cuánto es la meta de toda la empresa para todo 2026, y en qué celda de qué archivo lo leíste?**  
  La meta total es **110 000 kg**. Se encuentra en el archivo `metas_planeacion.csv`, en la intersección de la fila de total y la columna de total (celda **O6** / fila 6, columna 15).

- **L3. ¿Qué correo de `seguridad.csv` tiene más de una finca sin tenerlas todas, y cuáles son?**  
  El correo es **`regional.sur@agrodb.test`**, que tiene asignadas **2 fincas**: la finca **3** (*Agrícola La Unión*) y la finca **4** (*Hacienda Los Ceibos*).

---

## 3. Inventario de fuentes

| Fuente | Filas | Columnas | Su llave | A qué tabla del modelo va |
|---|---|---|---|---|
| `dim_finca.csv` | 4 | 3 | `finca_id` | `dim_finca` |
| `dim_cultivo.csv` | 6 | 3 | `cultivo_id` | `dim_cultivo` |
| `dim_tiempo.csv` | 730 | 5 | `fecha` | `dim_tiempo` |
| `seguridad.csv` | 10 | 2 | *(Compuesta: correo + finca_id)* | `seguridad` |
| `bascula\` (19 archivos mensuales) | 49 | 7 | `cosecha_id` | `h_cosecha` |
| `bascula_digital_2026.csv` | 8 | 7 | `cosecha_id` | `h_cosecha` |
| `precios.csv` | 12 | 3 | *(Compuesta: cultivo_id + calidad)* | Para combinar en `h_cosecha` |
| `metas_planeacion.csv` | 4 (fincas) | 15 | `finca_id` | `h_meta` (tras dinamizar columnas) |

---

## 4. «Lo que veo raro»

1. **Archivo duplicado en báscula:** En la carpeta `bascula\` existe el archivo `bascula_2026-03 - copia.csv` además de `bascula_2026-03.csv`. Esto duplica las 4 cosechas de marzo de 2026 (+4 registros, inflando los kilos de ese mes si se carga la carpeta completa sin deduplicar).
2. **Ubicación de `bascula_digital_2026.csv`:** El archivo de báscula digital vino dentro de la misma carpeta `Bascula\`. Si se usa el conector de carpeta sin filtrar nombres de archivo, intentará combinarse dos veces o colisionar con los archivos mensuales.
3. **Estructura matricial de `metas_planeacion.csv`:** El archivo viene con los meses como columnas (`2026-01-01`, `2026-02-01`, etc.) y trae filas y columnas de totales agregados (fila 6 y columna O). Si no se filtran los totales y no se anula la dinamización de columnas, no se podrá relacionar con el calendario ni modelar como tabla de hechos.
4. **Registros con desfase de entrega:** Existen cosechas donde `fecha_entrega` cae en un mes distinto a `fecha` (corte). Si se calcula el cierre por la relación activa de corte, el monto entregado/cobrado de septiembre va a moverse a octubre.
5. **Precios por combinación de cultivo y calidad:** El archivo `precios.csv` no tiene llave simple sino compuesta (`cultivo_id` + `calidad`). Al hacer el merge en Power Query para traer `precio_kg`, si no se hace el cruce por ambas columnas a la vez, se duplicarán filas o saldrán valores nulos/erróneos en el ingreso.

---

## 5. Bitácora del día

| Qué encontré | Cómo me di cuenta | Qué número daba mal | Cómo lo arreglé | Cómo sé que quedó |
|---|---|---|---|---|
| Archivo duplicado en la carpeta de báscula (`bascula_2026-03 - copia.csv`) | Inspección de la carpeta por explorador/terminal y conteo de filas (57 filas totales vs 53 registros únicos). | Sumaba 4 cosechas extra en marzo de 2026. | Se identifica para excluir o deduplicar por `cosecha_id` al armar `h_cosecha` en Power Query. | La lista única de IDs da exactamente 53. |
| Totales incluidos en `metas_planeacion.csv` | Apertura del archivo en Bloc de notas; la última fila contiene `,Total,7200...` y la última columna es `Total`. | Duplicaría al doble la meta anual (220 000 en vez de 110 000) si se suman todas las columnas. | Filtrar la fila de total y quitar la columna `Total` antes de anular dinamización. | La suma de metas de las 4 fincas da exactamente 110 000. |