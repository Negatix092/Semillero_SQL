# Proyecto Integrador · Punto de Control 1 (PC1)

**Alumno:** Axel Cortez  
**Fecha:** 5 de octubre de 2026  
**Entorno:** Power BI Desktop / Modelado dimensional  

---

## 1.1 Tabla de Fuentes

| Fuente | Filas | Columnas | Su llave | A qué tabla del modelo va |
|---|---|---|---|---|
| `dim_finca.csv` | 4 | 2 | `finca_id` | `dim_finca` |
| `dim_cultivo.csv` | 6 | 3 | `cultivo_id` | `dim_cultivo` |
| `dim_tiempo.csv` | 730 | 8 | `fecha` | `dim_tiempo` |
| `seguridad.csv` | 10 | 2 | *(Compuesta: correo + finca_id)* | `seguridad` |
| `bascula\` (19 archivos) | Variable | 6 / 7 | `cosecha_id` | `h_cosecha` |
| `bascula_digital_2026.csv` | Variable | 7 | `cosecha_id` | `h_cosecha` |
| `precios.csv` | Variable | 3 | *(Compuesta: cultivo_id + calidad)* | `h_cosecha` |
| `metas_planeacion.csv` | 5 | 14 | `finca` (matriz horizontal) | `h_meta` |

---

## 1.2 Lo que veo raro (Inspección previa)

1. **Archivo duplicado en báscula (`bascula_2026-03 - copia.csv`):** En la carpeta `bascula\` existe un archivo duplicado que aumentará artificialmente los kilos y cosechas de marzo de 2026 si no se filtra por nombre de origen.
2. **Estructura horizontal en `metas_planeacion.csv`:** Las metas vienen presentadas en formato de matriz de reporte (fincas en filas y meses de enero a diciembre en columnas), con una fila final de total; requerirá anulación de dinamización de columnas (unpivot) y exclusión de la fila de total agregado para evitar duplicidad.
3. **Nueva finca incorporada (`Hacienda Los Ceibos`):** Aparece con `finca_id = 4` a partir de 2026; si alguna consulta histórica asume solo 3 fincas, subestimará los datos globales.
4. **Variación en nombres de encabezados de báscula:** Posible discrepancia entre columnas de peso (como `kg` frente a `kg_neto`), lo que generaría valores nulos al combinar si no se estandariza en la función de transformación.
5. **Granularidad en `precios.csv`:** El precio unitario está determinado por la combinación de `cultivo_id` y `calidad`; una combinación simple por una sola columna provocaría un producto cartesiano (fan-out) inflando los ingresos.

---

## Anclas del Lunes

| Medida / Objeto | Esperado | Obtenido | Estado |
|---|---|---|---|
| Filas de `dim_finca` | 4 | 4 | ✅ Conciliado |
| Filas de `dim_cultivo` | 6 | 6 | ✅ Conciliado |
| Filas de `dim_tiempo` | 730 | 730 | ✅ Conciliado |
| Filas de `seguridad` | 10 | 10 | ✅ Conciliado |
| Archivos en `C:\agrodbproyecto\bascula\` | 19 | 19 | ✅ Conciliado |

---

## Preguntas Ciegas del Lunes

- **L1. Total de cosechas distintas (sumando carpeta y báscula digital):**  
  Hay **53 cosechas distintas** en total (descartando duplicados de archivos copia).
- **L2. Meta de toda la empresa para todo 2026:**  
  La meta total anual es de **110 000 kg**, leída en la celda de intersección de la fila de total y la columna de consolidado anual dentro de `metas_planeacion.csv`.
- **L3. Correo con más de una finca sin tenerlas todas:**  
  El usuario **`regional.sur@agrodb.test`**, que tiene asignadas 2 fincas (`finca_id` 2 y 3: El Guayabo y Santa Rosa).

---

## Captura del Modelo
![Modelo Dimensional Inicial](proyecto-pc1-modelo.png)