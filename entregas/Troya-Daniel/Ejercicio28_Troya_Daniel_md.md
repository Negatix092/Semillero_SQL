# Ejercicio práctico 28 · El precio de cada cosecha

## Parte A · El modelo

### A3. Medidas del modelo

```dax
Kilos = SUM( h_cosecha[kg] )
```
*Resultado (Ene–Abr 2026):* 30 550

```dax
Meta = SUM( h_meta[kg_meta] )
```
*Resultado (Ene–Abr 2026):* 24 440

```dax
Cumplimiento = DIVIDE( [Kilos] , [Meta] )
```
*Resultado (Ene–Abr 2026):* 125,00 %

```dax
Cosechas = COUNTROWS( h_cosecha )
```
*Resultado (Ene–Abr 2026):* 9

```dax
Filas repetidas = COUNTROWS( h_cosecha ) - DISTINCTCOUNT( h_cosecha[cosecha_id] )
```
*Resultado (Ene–Abr 2026):* 0

---

### A4. Tabla de control inicial (2026, Meses 1 a 4)

| Finca | [Kilos] | [Meta] | [Cumplimiento] | [Cosechas] |
| :--- | :--- | :--- | :--- | :--- |
| Agricola La Union | 2 100 | 5 000 | 42,00 % | 2 |
| Finca El Guayabo | 14 250 | 9 440 | 150,95 % | 3 |
| Hacienda Santa Rosa | 14 200 | 10 000 | 142,00 % | 4 |
| **Total** | **30 550** | **24 440** | **125,00 %** | **9** |

*Tarjeta [Filas repetidas]:* 0

---

### A5. Pregunta sobre precios.csv
Hacen falta dos columnas: `cultivo_id` y `calidad`. Con solo el cultivo no se puede identificar un precio único porque cada cultivo tiene precios distintos según su calidad.

---

## Parte B · La combinación obvia

### B2. Texto copiado de Power Query al combinar por `cultivo_id`
> "La selección coincide con 25 de 25 filas de la primera tabla."

---

### B4. Medidas de ingresos

```dax
Ingresos = SUMX( h_cosecha , h_cosecha[kg] * h_cosecha[precio_kg] )
```

```dax
Precio por kilo = DIVIDE( [Ingresos] , [Kilos] )
```

---

### B5. Tabla de ingresos por finca (Combinación errónea solo por `cultivo_id`)

| Finca | [Ingresos] | [Precio por kilo] |
| :--- | :--- | :--- |
| Agricola La Union | 9 030,00 | 0,55 |
| Finca El Guayabo | 7 390,00 | 0,55 |
| Hacienda Santa Rosa | 11 660,00 | 0,55 |
| **Total** | **28 080,00** | **0,55** |

---

### B6. Pregunta contra el correo de comercial
No habría hecho sospechar nada, porque el promedio del kilo dio 0,55 y comercial había dicho textualmente que el kilo salía en promedio a «cincuenta y tantos centavos».

---

## Parte C · El número que no debía moverse

### C1. Tabla de control con la combinación errónea

| Finca | [Kilos] | [Meta] | [Cumplimiento] | [Cosechas] |
| :--- | :--- | :--- | :--- | :--- |
| Agricola La Union | 4 200 | 5 000 | 84,00 % | 4 |
| Finca El Guayabo | 23 850 | 9 440 | 252,65 % | 6 |
| Hacienda Santa Rosa | 23 250 | 10 000 | 232,50 % | 7 |
| **Total** | **51 300** | **24 440** | **209,90 %** | **17** |

*Tarjeta [Filas repetidas]:* 8

---

### C2. Filas de `h_cosecha` en Power Query
46 filas.

---

### C3. Distribución de columnas de `cosecha_id`
25 distintos, 4 únicos.

---

### C4. Filas de `cosecha_id = 2` en Power Query

| cosecha_id | cultivo_id | calidad | kg | precio_kg |
| :--- | :--- | :--- | :--- | :--- |
| 2 | 2 | segunda | 3 100 | 0.50 |
| 2 | 2 | segunda | 3 100 | 0.30 |

El precio que le toca a la cosecha 2 es **0.30**, correspondiente a Mango de calidad segunda.

---

### C5. Cosechas que siguen siendo únicas
Son las cosechas **7, 12, 15 y 19**. Lo que tienen en común es que son cosechas de Maíz (`cultivo_id = 5`), cultivo que en `precios.csv` tiene una sola fila registrada (solo calidad primera), por lo que no generó duplicados al combinar.

---

### C6. ¿Qué cuenta y qué no cuenta el número de B2?
Cuenta cuántas filas de la tabla izquierda encontraron al menos una coincidencia en la derecha, pero no cuenta cuántas coincidencias múltiples se generaron por cada fila.

---

### C7. Comportamiento de Precio por kilo frente a la inflación de ingresos
Casi no se movió porque al duplicarse las filas de las cosechas en `h_cosecha`, tanto los Kilos como los Ingresos se inflaron prácticamente en la misma proporción, manteniendo el cociente cercano a 0,55.

---

## Parte D · El segundo arreglo obvio

### D2. Las tres pruebas tras Quitar duplicados

| Prueba | Tiene que decir | Resultado obtenido |
| :--- | :--- | :--- |
| Filas de `h_cosecha` en Power Query | 25 | 25 |
| Tabla de control | 30 550 / 125,00 % / 9 | 30 550 / 125,00 % / 9 |
| [Filas repetidas] | 0 | 0 |

---

### D3. Tabla de ingresos con Quitar duplicados

| Finca | [Ingresos] |
| :--- | :--- |
| Agricola La Union | 5 250,00 |
| Finca El Guayabo | 5 610,00 |
| Hacienda Santa Rosa | 7 250,00 |
| **Total** | **18 110,00** |

---

### D4. Cálculo manual de ingresos (Enero–Abril 2026)

| cosecha_id | Cultivo | Calidad | Kilos | Precio que toca | Ingreso |
| :---: | :--- | :--- | :---: | :---: | :---: |
| 1 | Mango | primera | 4 200 | 0,50 | 2 100 |
| 2 | Mango | segunda | 3 100 | 0,30 | 930 |
| 3 | Mango | primera | 5 400 | 0,50 | 2 700 |
| 4 | Guayaba | primera | 1 500 | 0,40 | 600 |
| 5 | Guayaba | primera | 2 600 | 0,40 | 1 040 |
| 6 | Guayaba | segunda | 1 850 | 0,20 | 370 |
| 7 | Maiz | primera | 9 800 | 0,30 | 2 940 |
| 8 | Cacao | primera | 1 200 | 2,50 | 3 000 |
| 9 | Cacao | primera | 900 | 2,50 | 2 250 |
| **Total** | | | **30 550** | | **17 120** |

---

### D5. Cosechas que explican la diferencia de 990 dólares
Las cosechas que explican la diferencia son la **2** y la **6** (ambas de calidad segunda):
* **Cosecha 2 (Mango segunda, 3 100 kg):** Quitar duplicados le asignó precio de primera (0,50) en lugar de segunda (0,30). Diferencia: $3 100 \times (0,50 - 0,30) = 620$.
* **Cosecha 6 (Guayaba segunda, 1 850 kg):** Quitar duplicados le asignó precio de primera (0,40) en lugar de segunda (0,20). Diferencia: $1 850 \times (0,40 - 0,20) = 370$.
* **Suma total de error:** $620 + 370 = 990$.

---

### D6. ¿Qué hizo Quitar duplicados?
Quitar duplicados conservó arbitrariamente la primera fila que encontró en memoria según el orden de la combinación, dejando el precio escogido al azar del orden de lectura y descartando el precio real de calidad segunda.

---

## Parte E · La llave completa

### E1. Nombres de pasos borrados
* `Duplicados quitados`
* `Tipo cambiado1` / `Se expandió precios`
* `Consultas combinadas`

---

### E2. Texto copiado de Power Query al combinar con llave compuesta (`cultivo_id` y `calidad`)
> "La selección coincide con 25 de 25 filas de la primera tabla."

---

### E3. Validación en Power Query
* **Filas:** 25 filas.
* **Distribución de `cosecha_id`:** 25 distintos, 25 únicos.

---

### E4. Tabla de control
* **Valores:** 30 550 / 125,00 % / 9
* **[Filas repetidas]:** 0

---

### E5. Tabla de ingresos por finca (Llave completa)

| Finca | [Ingresos] | [Precio por kilo] |
| :--- | :--- | :--- |
| Agricola La Union | 5 250,00 | 2,50 |
| Finca El Guayabo | 5 240,00 | 0,37 |
| Hacienda Santa Rosa | 6 630,00 | 0,47 |
| **Total** | **17 120,00** | **0,56** |

*Tarjeta [Filas repetidas]:* 0

---

### E6. Tabla por cultivo e ingresos

| Cultivo | [Ingresos] |
| :--- | :--- |
| Cacao | 5 250,00 |
| Guayaba | 3 200,00 |
| Maiz | 2 940,00 |
| Mango | 5 730,00 |
| **Total** | **17 120,00** |

*Comparación con la Parte B:* El único cultivo que dio el mismo resultado en ambas combinaciones fue **Maíz** ($2 940$), porque solo tiene una calidad registrada en `precios.csv`, por lo que nunca se duplicó ni tomó un precio incorrecto.

---

### E7. Validación de A5 y llave de precios
Sí se acertó. La llave primaria de `precios` es una llave compuesta por **`cultivo_id` + `calidad`**.

---

## Parte F · Quién avisa, y preguntas de cierre

### F1. Error al intentar LOOKUPVALUE con llave incompleta
```dax
Precio incompleto = LOOKUPVALUE( precios[precio_kg] , precios[cultivo_id] , h_cosecha[cultivo_id] )
```
*Texto literal del error:*
> "Se proporcionó una tabla de varios valores donde se esperaba un solo valor."

---

### F2. LOOKUPVALUE con llave completa

```dax
Precio buscado = LOOKUPVALUE( precios[precio_kg] ,
    precios[cultivo_id] , h_cosecha[cultivo_id] ,
    precios[calidad] , h_cosecha[calidad] )
```

```dax
Diferencia de precio = SUMX( h_cosecha , ABS( h_cosecha[precio_kg] - h_cosecha[Precio buscado] ) )
```
*Resultado:* 0

---

### F3. Intento de relación en vista de modelo
* **Cardinalidad propuesta por Power BI:** Varios a varios (*:*).
* **Aviso mostrado:**
> "Esta relación tiene una cardinalidad de varios a varios. Solo debe usarse si se espera que ninguna de las columnas contenga valores únicos..."

---

### F4. Preguntas de cierre

1. **¿Qué diferencia hay entre Anexar y Combinar, dicha en filas y columnas?**  
Anexar agrega filas hacia abajo aumentando la longitud de la tabla, mientras que Combinar agrega columnas hacia la derecha trayendo atributos de otra tabla mediante columnas coincidentes.

2. **Regla de detección del día:**  
Si al enriquecer una tabla de hechos con una dimensión sus métricas agregadas base aumentan, la dimensión no es única por la llave utilizada y está duplicando los hechos por fan-out.

3. **Quitar errores vs. Quitar duplicados:**  
Ambos eliminan filas silenciosamente sin advertir en el reporte; sin embargo, Quitar duplicados es más peligroso porque oculta la distorsión estadística al forzar el conteo correcto de filas mientras corrompe de forma irreversible la asignación de atributos y montos.

4. **Tabla con varias filas por llave (fan-out):**  
La tabla `precios` (donde había varias filas por cada `cultivo_id`).

5. **Precios por cultivo, calidad y mes:**  
En la combinación se debe incluir una tercera columna (`cultivo_id`, `calidad` y `mes`), y la primera prueba indispensable sería comprobar que la tabla de hechos mantenga exactamente sus 25 filas originales con 25 IDs únicos tras expandir.