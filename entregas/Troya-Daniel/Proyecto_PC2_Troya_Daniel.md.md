# Punto de Control 2 · Los hechos

**Estudiante:** Daniel Troya  
**Fecha:** Martes 6 de octubre de 2026  

---

## 1. Anclas del martes (sin segmentadores)

| Medida / Validación | Esperado | En pantalla | Estado |
|---|---|---|---|
| `[Cosechas]` | **53** | 53 | Cuadra |
| `[Cosechas repetidas]` | **0** | 0 | Cuadra |
| `[Kilos]` | **136 600** | 136 600 | Cuadra |

---

## 2. Preguntas ciegas

- **M1. `[Kilos]` de agosto de 2025:**  
  **3 450 kg**.

- **M2. `[Cosechas]` y `[Kilos]` de septiembre de 2026:**  
  **4 cosechas** y **10 500 kg**.

- **M3. `[Kilos]` de cada una de las cuatro fincas, en el periodo del cierre (2026, meses 1 a 9):**  
  - **Agrícola La Unión:** 4 750 kg  
  - **Finca El Guayabo:** 22 800 kg  
  - **Hacienda Los Ceibos:** 43 500 kg  
  - **Hacienda Santa Rosa:** 20 250 kg  
  *(Total del periodo: 91 300 kg)*

- **M4. `[Ingresos]` de la empresa en el periodo del cierre, con dos decimales:**  
  **$34 750,00**

- **M5. ¿Cuántas filas tenía `h_cosecha` la primera vez que la cargaste, antes de corregir nada? ¿Y cuántas columnas tenían valores vacíos o `Error`, y cuáles?**  
  - Filas iniciales: **57 filas** (53 cosechas reales más 4 filas duplicadas procedentes del archivo redundante `bascula_2026-03 - copia.csv`).  
  - Columnas con valores vacíos (`null`): **2 columnas**. La columna `kg` presentó 2 valores vacíos (4 % de la tabla) en las cosechas 16 y 17 debido a que el archivo `bascula_2025-08.csv` utilizaba el encabezado heterogéneo `kg_neto`. Asimismo, la columna `Source.Name` generada por la carga de carpeta quedó vacía en las filas anexadas de báscula digital.  
  - Columnas con `Error`: **2 columnas** (`fecha` y `fecha_entrega`), las cuales arrojaron 9 % de filas con error tras una conversión global con configuración regional `es-MX`, debido a la presencia de registros de `bascula_digital_2026.csv` formateados en estándar anglosajón (`MM/DD/YYYY`).

---

## 3. Bitácora del día

| Qué encontré | Cómo me di cuenta | Qué número daba mal | Cómo lo arreglé | Cómo sé que quedó |
|---|---|---|---|---|
| Columna `kg` con 2 valores `null` en agosto de 2025. | Al perfilar columnas en Power Query, la barra de calidad de `kg` indicó 4 % de vacíos; la matriz mostraba 0 kg en agosto de 2025. | `[Kilos]` total daba 133 150 kg en lugar del ancla esperada de 136 600 kg. | Se inspeccionó `bascula_2025-08.csv`, constatando que la columna venía rotulada como `kg_neto`. Se normalizó el nombre del campo a `kg` para su consolidación con el resto de meses. | La barra de calidad de `kg` marcó 100 % válido y la tarjeta de `[Kilos]` alcanzó exactamente 136 600 kg. |
| Valores de `precio_kg` truncados a enteros (ceros y unos). | Los ingresos calculados arrojaban cifras inconsistentes y la columna expandida mostraba valores sin decimales. | `[Ingresos]` subestimaba gravemente la facturación de cosechas con precios unitarios fraccionarios (ej. 0.28, 0.38, 0.55). | Se ajustó el tipo de dato de `precio_kg` en la consulta `precios` y en la tabla combinada a Número Decimal con separador de punto. | Los precios preservaron sus decimales y los ingresos totales de 2026 cerraron en $34 750,00. |
| Errores de conversión de tipo fecha en cosechas del segundo semestre de 2026. | Al aplicar cambio de tipo a Fecha con configuración `es-MX`, aparecieron celdas de `Error` en días superiores a 12. | Se perdían cosechas en el modelo tabular al no poder parsear días como meses (ej. 15/07/2026 o 27/07/2026). | Se aplicó la transformación de tipo con configuración regional `en-US` específicamente en la consulta `bascula_digital_2026` antes del paso de combinación/anexado. | 100 % de valores válidos y 0 % de error en `fecha` y `fecha_entrega`. |
| Cosechas duplicadas al consolidar carpeta de báscula. | La medida `[Cosechas repetidas]` arrojaba 4 y el conteo de filas ascendía a 57. | `[Cosechas]` daba 57 en lugar de las 53 requeridas. | Se excluyó del origen el archivo duplicado `bascula_2026-03 - copia.csv` y se aplicó eliminación de duplicados por `cosecha_id` en Power Query. | `[Cosechas]` calibró en 53 exactos y `[Cosechas repetidas]` quedó en 0. |