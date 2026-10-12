# Ejercicio 28 · AgroDB

## PARTE A - EL MODELO

**A5.** Para encontrar un solo precio de manera unívoca en la tabla `precios.csv`, hacen falta dos columnas: el cultivo (`cultivo_id`) y su calidad (`calidad`).

---

## PARTE B - LA COMBINACIÓN OBVIA

**B1.** En el panel de Power Query, la consulta `h_cosecha` tiene inicialmente **25 filas**.

**B2.** Mensaje literal de Power Query tras la combinación: `La selección coincide con 25 de las 25 filas de la primera tabla.`

### B4 · Ingresos y Precio por kilo
```dax
Ingresos = SUMX( h_cosecha , h_cosecha[kg] * h_cosecha[precio_kg] )
Precio por kilo = DIVIDE( [Ingresos] , [Kilos] )
```

## PARTE C - EL NÚMERO QUE NO DEBÍA MOVERSE

**C1. Tabla de control (con la combinación incorrecta):**

| Finca | `[Kilos]` | `[Meta]` | `[Cumplimiento]` | `[Cosechas]` |
| --- | --- | --- | --- | --- |
| Agricola La Union | 4 200 | 5 000 | 84,00 % | 4 |
| Finca El Guayabo | 20 500 | 9 440 | 217,16 % | 5 |
| Hacienda Santa Rosa | 26 600 | 10 000 | 266,00 % | 8 |
| **Total** | **51 300** | **24 440** | **209,90 %** | **17** |
| *(Las `[Filas repetidas]` marcarían 8).* |  |  |  |  |

**C2.** Ahora `h_cosecha` tiene **46 filas** en Power Query.

**C3.** La distribución de columnas de `cosecha_id` dice: **25 distintos, 4 únicos**.

**C4.** Al filtrar `cosecha_id = 2`, salen dos filas (ambas con calidad "segunda" en la cosecha, pero cruzadas con los precios de "primera" a 0.50 y "segunda" a 0.30). El precio que le toca legítimamente es el de **0.30** (segunda).

**C5.** Las 4 cosechas que siguen siendo únicas son las número **7, 15, 19 y 24**. Lo que tienen en común es que todas son de **Maíz** (`cultivo_id` 5). No les pasó porque el Maíz es el único cultivo en `precios.csv` que tiene una sola fila registrada (solo calidad 'primera'), por lo que no había posibilidad de duplicarse en el cruce.

**C6.** El "25 de 25" cuenta **cuántas filas de la tabla original encontraron al menos una coincidencia** en la tabla destino. Sin embargo, **no** cuenta si alguna de esas filas originales encontró más de una coincidencia y se multiplicó.

**C7.** Casi no se movió el precio promedio (0,55) porque al combinarse mal, tanto el numerador (`[Ingresos]`) como el denominador (`[Kilos]`) se inflaron en la misma proporción al duplicarse las filas.

---

## PARTE D - EL SEGUNDO ARREGLO OBVIO

**D2. Las tres pruebas (tras Quitar duplicados):**

| Prueba | Tiene que decir |
| --- | --- |
| Filas de `h_cosecha` en Power Query | 25 |
| Tabla de control | 30 550 / 125,00 % / 9 |
| `[Filas repetidas]` | 0 |

**D3. Tabla de ingresos (con Quitar duplicados):**

| Finca | `[Ingresos]` |
| --- | --- |
| Agricola La Union | 5 250 |
| Finca El Guayabo | 5 610 |
| Hacienda Santa Rosa | 7 250 |
| **Total** | **18 110** |

**D4. Cálculo a mano del ingreso correcto:**

| `cosecha_id` | Cultivo | Calidad | Kilos | Precio que toca | Ingreso |
| --- | --- | --- | --- | --- | --- |
| 1 | Mango | primera | 4 200 | 0.50 | 2 100 |
| 2 | Mango | segunda | 3 100 | 0.30 | 930 |
| 3 | Mango | primera | 5 400 | 0.50 | 2 700 |
| 4 | Guayaba | primera | 1 500 | 0.60 | 900 |
| 5 | Guayaba | primera | 2 600 | 0.60 | 1 560 |
| 6 | Guayaba | segunda | 1 850 | 0.40 | 740 |
| 7 | Maiz | primera | 9 800 | 0.30 | 2 940 |
| 8 | Cacao | primera | 1 200 | 2.50 | 3 000 |
| 9 | Cacao | primera | 900 | 2.50 | 2 250 |
|  |  |  | **30 550** |  | **17 120** |

**D5.** Las dos cosechas que explican la diferencia de 990 dólares (18 110 - 17 120) son la **2** y la **6**:

* Cosecha 2 (Mango segunda): 3 100 kg cobrados a 0.50 en lugar de 0.30 (3 100 × 0.20 = **+620**).
* Cosecha 6 (Guayaba segunda): 1 850 kg cobrados a 0.60 en lugar de 0.40 (1 850 × 0.20 = **+370**).
Total de inflación silenciosa: 620 + 370 = 990.



**D6.** `Quitar duplicados` simplemente borró la segunda fila que encontró para cada `cosecha_id`. El precio que quedó lo escogió el azar del motor de Power Query (generalmente retiene la primera fila escaneada, que en este caso traía los precios de "primera" para todo).

---

## PARTE E - LA LLAVE COMPLETA

**E1.** Nombres de los pasos borrados (de abajo hacia arriba): `Duplicados quitados`, `precios expandido` y `Consultas combinadas`.

**E2.** Mensaje de Power Query al combinar por las dos columnas (cultivo y calidad): `La selección coincide con 25 de las 25 filas de la primera tabla.`

**E3.** Al expandir solo el precio, `h_cosecha` tiene ahora **25 filas** exactas. La distribución de columnas dice: **25 distintos, 25 únicos**.

**E6.** Tabla por cultivo. El **único** cultivo que dio exactamente lo mismo con las dos combinaciones (la rota con Quitar Duplicados y la correcta) es el **Maíz**. Esto ocurrió porque era el único que tenía un solo registro de precio, por lo que jamás sufrió la duplicación ni la limpieza ciega de precios incorrectos.

**E7.** Sí, acerté. La llave primaria de la tabla `precios` es compuesta: **`cultivo_id` + `calidad**`.

---

## PARTE F - QUIÉN AVISA, Y PREGUNTAS DE CIERRE

**F1.** Error literal de LOOKUPVALUE: `Se proporcionó una tabla de varios valores donde se esperaba un solo valor.`

**F2.** Con la columna `Precio buscado` (con la llave completa), la medida `Diferencia de precio` arroja **0** de manera exacta, demostrando que el cruce está limpio y correcto.

**F3.** Al arrastrar `h_cosecha[cultivo_id]` sobre `precios[cultivo_id]` en la Vista de Modelo, Power BI propone una cardinalidad de **Varios a varios (*:*)** y muestra un aviso alertando que *"Esta relación tiene una cardinalidad de varios a varios. Solo debe usarse si se espera que ninguna de las columnas contenga valores únicos..."*.

**F4. Preguntas de cierre:**

1. **Diferencia entre Anexar y Combinar:** Anexar apila tablas pegando **filas** nuevas hacia abajo (como hacer un UNION en SQL); Combinar busca coincidencias cruzadas pegando **columnas** nuevas hacia la derecha (como hacer un JOIN en SQL).


2. **Regla de detección:** "Si tras un cruce (merge) de datos, una medida de conteo de filas o volumen base (`[Kilos]`, `[Cosechas]`) se infla silenciosamente, la combinación multiplicó los registros por falta de granularidad en la llave de cruce".


3. **Quitar errores vs Quitar duplicados:** Se parecen en que ambas barren bajo la alfombra los síntomas de un problema subyacente borrando datos. Quitar duplicados es más peligroso porque sus efectos no dejan huecos evidentes ni errores visibles; te entrega números aparentemente perfectos pero financieramente falsos.


4. **¿Qué tabla hizo de fan-out hoy?** Hoy la tabla que actuó como el lado "varios" oculto y generó la explosión (fan-out) fue la tabla de **precios**, porque al combinar solo por cultivo, entregó múltiples precios posibles para una misma cosecha.


5. **¿Qué cambia?** Cambiaría que la llave de cruce en Power Query necesitaría ahora tres selecciones (`cultivo_id` + `calidad` + `mes`). La prueba vital sería comprobar que la tabla de hechos mantenga su recuento original de filas (25) tras el cruce para descartar duplicidades silenciosas.



```

```