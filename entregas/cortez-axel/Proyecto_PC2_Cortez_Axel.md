# Proyecto Integrador AgroDB — PC2

**Alumno:** Axel Cortez  
**Fecha:** 8 de octubre de 2026  
**Entorno:** Power BI Desktop / Power Query / DAX  
**Etapa:** PC2 — Integración y depuración de datos de cosechas  

---

## 1. Objetivo

Integrar los registros de cosechas provenientes de los archivos de báscula tradicional y digital, realizar las transformaciones necesarias en Power Query y construir medidas DAX que permitan analizar los kilogramos, las cosechas y los ingresos.

---

## 2. Integración de datos

Se trabajó con las siguientes fuentes:

- Archivos CSV mensuales de la carpeta `bascula`.
- Archivo `bascula_digital_2026.csv`.
- Archivo `precios.csv`.

Se consolidaron los registros en la tabla de hechos `h_cosecha`, cuya estructura final requerida contiene las siguientes columnas:

`cosecha_id`, `finca_id`, `cultivo_id`, `fecha`, `fecha_entrega`, `calidad`, `kg`, `precio_kg`.

---

## 3. Transformaciones y correcciones

### 3.1. Normalización de kilogramos

Se identificó que algunos archivos utilizaban la columna `kg_neto` en lugar de `kg`, lo que provocaba valores nulos al combinar los registros.  
Se implementó una normalización para recuperar los kilogramos desde la columna disponible y unificar los valores en `kg`.

### 3.2. Corrección de fechas

Durante la integración se identificaron cinco errores de conversión en la columna `fecha`.  
Uno de los valores problemáticos fue `07/15/2026`, correspondiente al formato estadounidense `MM/DD/YYYY`.  
Se corrigió la interpretación de las fechas utilizando la configuración regional **Inglés (Estados Unidos)** en Power Query.

### 3.3. Validación de los datos

Después de las correcciones, se obtuvieron los siguientes resultados sin filtros:

| Indicador | Resultado |
|---|---|
| Cosechas | 53 |
| Cosechas repetidas | 0 |
| Kilogramos totales | 136.600 kg |

Los resultados coinciden con los valores de control establecidos para el PC2.

---

## 4. Medidas DAX

### Kilos

```dax
Kilos = SUM(h_cosecha[kg])
```

### Cosechas

```dax
Cosechas = COUNTROWS(h_cosecha)
```

### Cosechas repetidas

```dax
Cosechas repetidas =
COUNTROWS(h_cosecha) -
DISTINCTCOUNT(h_cosecha[cosecha_id])
```

### Ingresos

```dax
Ingresos =
SUMX(
    h_cosecha,
    h_cosecha[kg] * h_cosecha[precio_kg]
)
```

---

## 5. Respuestas a las preguntas del PC2

### M1. ¿Cuántos kilogramos se registraron en agosto de 2025?

**Respuesta:** **3.450 kg**, correspondientes a 2 cosechas.

### M2. ¿Cuántas cosechas y kilogramos se registraron en septiembre de 2026?

**Respuesta:**

- **Cosechas:** 4
- **Kilogramos:** 10.500 kg

### M3. ¿Cuántos kilogramos registró cada finca entre enero y septiembre de 2026?

| **Finca** | **Kilogramos** |
|---|---:|
| Hacienda Los Ceibos | 43.500 |
| Agrícola La Unión | 4.750 |
| Finca El Guayabo | 22.800 |
| Hacienda Santa Rosa | 20.250 |
| **Total** | **91.300** |

### M4. ¿Cuáles fueron los ingresos entre enero y septiembre de 2026?

**Respuesta:** **$4.716.650,00**.

El resultado se obtuvo mediante la medida `[Ingresos]`, filtrando el año 2026 y los meses del 1 al 9.

### M5. ¿Cuántas filas y qué columnas presentaron valores vacíos o errores en la primera carga?

Durante el proceso de transformación se identificaron:

- **Valores nulos en la columna de kilogramos:** Debido a diferencias entre los nombres `kg` y `kg_neto` en los archivos de origen.
- **Cinco errores de conversión en la columna fecha:** Ocasionados por diferencias en el formato de las fechas (formato `MM/DD/YYYY`).

Ambos problemas fueron corregidos en Power Query.

*Observación:* No se dispone de un conteo verificable de todas las filas con valores vacíos o errores en la primera carga. Los cinco errores de fecha corresponden a los identificados durante la transformación.

---

## 6. Evidencias

### Evidencia 1. Resultados de la tabla de hechos

**Archivo:** `proyecto-pc2-hechos.png`

Incluye:

- Tarjeta de cosechas: 53.
- Tarjeta de cosechas repetidas: 0.
- Tarjeta de kilogramos: 136.600.
- Matriz por año y mes con las medidas `[Kilos]` y `[Cosechas]`.

### Evidencia 2. Transformaciones en Power Query

**Archivo:** `proyecto-pc2-pasos.png`

Muestra la consulta `h_cosecha`, sus ocho columnas finales (`cosecha_id`, `finca_id`, `cultivo_id`, `fecha`, `fecha_entrega`, `calidad`, `kg`, `precio_kg`) y los pasos aplicados durante la integración y depuración de datos.

---

## 7. Conclusión

En el PC2 se consolidaron los registros de cosechas de diferentes fuentes, se corrigieron inconsistencias en los kilogramos y formatos de fecha, y se implementaron medidas DAX para el análisis de la información.

La tabla consolidada permitió alcanzar los valores de control de 53 cosechas, 0 registros repetidos y 136.600 kg, proporcionando una base sólida para continuar con el análisis de metas y cumplimiento en el PC3.
