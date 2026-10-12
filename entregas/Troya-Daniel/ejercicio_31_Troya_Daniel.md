# Ejercicio práctico 31 · Las pesadas de la báscula

* **Estudiante:** Daniel Troya

* **Entregables adjuntos:** `clase31-llaves.png`, `clase31-carga.png`

## Parte A · El modelo sin cosechas

### Medida DAX: Meta

```
Meta = SUM( h_meta[kg_meta] )

```

*Resultado en tabla de control (Enero - Abril 2026):*

* La Union: 5 000

* El Guayabo: 9 440

* Santa Rosa: 10 000

* **Total:** 24 440

### Respuestas A4 (Inspección en Bloc de notas de pesadas.csv)

* **¿Cuántas filas de datos tiene?:** Tiene **54 filas** de datos (excluyendo el encabezado).

* **¿Cuántas cosechas distintas?:** Tiene **25 cosechas distintas** (`cosecha_id` del 1 al 25).

* **¿En cuántos camiones se pesó la cosecha 7?:** Se pesó en **3 camiones**.

## Parte B · Carga inicial y Agrupar por básico

### Medidas DAX

```
Kilos = SUM( h_cosecha[kg] )

```

*Resultado Total:* 30 550

```
Cosechas = COUNTROWS( h_cosecha )

```

*Resultado Total:* 19

```
Cumplimiento = DIVIDE( [Kilos] , [Meta] )

```

*Resultado Total:* 125,00 %

```
Cosechas repetidas = [Cosechas] - DISTINCTCOUNT( h_cosecha[cosecha_id] )

```

*Resultado Total:* 10

### Tabla de control B3 (Enero - Abril 2026)

| finca | Meta | Kilos | Cosechas | Cumplimiento | Cosechas repetidas | 
| ----- | ----- | ----- | ----- | ----- | ----- | 
| La Union | 5 000 | 2 100 | 2 | 42,00 % | 1 | 
| El Guayabo | 9 440 | 14 250 | 7 | 150,95 % | 4 | 
| Santa Rosa | 10 000 | 14 200 | 10 | 142,00 % | 5 | 
| **Total** | **24 440** | **30 550** | **19** | **125,00 %** | **10** | 

* **Respuesta B3 (¿Qué está contando `[Cosechas]`?):**
  Está contando filas de pesadas individuales de báscula (camiones pesados), no eventos de cosecha consolidados.

* **Respuesta B4 (Agrupar por Básico con `cosecha_id`):**

  * **Filas y columnas resultantes:** 25 filas y 2 columnas (`cosecha_id` y `Recuento`).

  * **Columna nueva y operación:** Se llama `Recuento` y la operación fue `Recuento de filas`.

  * **Columnas desaparecidas:** Se perdieron `pesada_id`, `finca_id`, `cultivo_id`, `fecha`, `calidad`, `camion` y `kg`.

## Parte C · Las llaves

### Tabla de control C2 (con `camion` en las llaves - 36 filas en Power Query)

*Evidencia visual guardada en `clase31-llaves.png`.*

| finca | Meta | Kilos | Cosechas | Cumplimiento | Cosechas repetidas | 
| ----- | ----- | ----- | ----- | ----- | ----- | 
| La Union | 5 000 | 2 100 | 2 | 42,00 % | 1 | 
| El Guayabo | 9 440 | 14 250 | 5 | 150,95 % | 2 | 
| Santa Rosa | 10 000 | 14 200 | 7 | 142,00 % | 2 | 
| **Total** | **24 440** | **30 550** | **14** | **125,00 %** | **5** | 

### Respuesta C3 (Cosecha 1: pesadas vs. filas agrupadas)

* Filas de cosecha 1 en Power Query agrupado: 2 filas.

* Filas de cosecha 1 en pesadas.csv original: 3 filas.

* **¿Por qué tres camiones dieron dos filas?:**
  Porque un mismo camión realizó dos viajes para la misma cosecha, por lo que al agrupar por placa de camión esas dos pesadas colapsaron en un solo registro.

### Tabla de control C4 (Sin `camion` en llaves - 25 filas en Power Query)

| finca | Meta | Kilos | Cosechas | Cumplimiento | Cosechas repetidas | 
| ----- | ----- | ----- | ----- | ----- | ----- | 
| La Union | 5 000 | 2 100 | 1 | 42,00 % | 0 | 
| El Guayabo | 9 440 | 14 250 | 3 | 150,95 % | 0 | 
| Santa Rosa | 10 000 | 14 200 | 5 | 142,00 % | 0 | 
| **Total** | **24 440** | **30 550** | **9** | **125,00 %** | **0** | 

### Respuesta C5

En las llaves solo pueden ir los atributos que definen de forma única la entidad destino (como `cosecha_id`, fecha, finca, cultivo y calidad), y no atributos transaccionales con cardinalidad uno a muchos como `camion`. Si se hubiera incluido `pesada_id`, no habría habido consolidación alguna y habrían salido las 54 filas originales porque `pesada_id` es la clave primaria única de cada viaje.

## Parte D · Cuánto lleva un camión

### Medida DAX: Carga promedio

```
Carga promedio = AVERAGE( h_cosecha[carga_promedio] )

```

### Tabla de control D1

| finca | Meta | Kilos | Cosechas | Carga promedio | 
| ----- | ----- | ----- | ----- | ----- | 
| La Union | 5 000 | 2 100 | 1 | 1 050,00 | 
| El Guayabo | 9 440 | 14 250 | 3 | 1 866,67 | 
| Santa Rosa | 10 000 | 14 200 | 5 | 1 450,00 | 
| **Total** | **24 440** | **30 550** | **9** | **1 500,00** | 

### D2 & D3 · Datos de El Guayabo (Enero–Abril 2026)

| cosecha_id | Camiones | Kilos | Kilos entre camiones | 
| ----- | ----- | ----- | ----- | 
| 5 | 2 | 4 500 | 2 250,00 | 
| 6 | 2 | 3 850 | 1 925,00 | 
| 7 | 3 | 5 900 | 1 966,67 | 
| **El Guayabo** | **7** | **14 250** | **2 035,71** | 

### Respuesta D4 (¿Qué cosecha jala cada promedio?)

En El Guayabo, la cosecha 7 tiene más camiones (3 de los 7) y un promedio de 1 966,67 kg; la media simple de promedios `(2 250 + 1 925 + 1 966,67) / 3 = 1 866,67` le da el mismo peso de un tercio a cada cosecha e infravalora el volumen real. En Santa Rosa ocurre lo opuesto: cosechas con menos viajes pero promedios individuales elevados sobrestiman el promedio aritmético simple frente a la media ponderada global.

### Medida DAX: Carga promedio 2

```
Carga promedio 2 = DIVIDE( [Kilos] , [Cosechas] )

```

*Resultados por finca:*

* La Union: 2 100,00

* El Guayabo: 4 750,00

* Santa Rosa: 2 840,00

* **Total:** 3 394,44

### Respuesta D5 (¿Qué contesta en realidad `[Carga promedio 2]`?)

Contesta el promedio de kilos por evento de cosecha, no la capacidad transportada por viaje de camión.

### Respuesta D6

* `[Cosechas]` dio 19 en B3 y 9 ahora porque antes de agrupar la tabla contenía 19 pesadas individuales en ese período, mientras que tras la agrupación en Power Query contiene 9 cosechas consolidadas.

* La Unión dio 1 050,00 en ambas medidas de camión porque en ese rango solo tuvo 1 cosecha que se repartió exactamente en 2 viajes iguales de 1 050 kg (`2 100 / 2 = 1 050,00`), evitando cualquier distorsión por ponderaciones dispares.

## Parte E · El arreglo: la cuenta que guardaste

### Medidas DAX

```
Camiones = SUM( h_cosecha[pesadas] )

```

*Resultado Total:* 19

```
Carga por camión = DIVIDE( [Kilos] , [Camiones] )

```

*Resultado Total:* 1 607,89

### Tabla de control E2

*Evidencia visual guardada en `clase31-carga.png`.*

| finca | Carga promedio | Carga promedio 2 | Camiones | Carga por camión | 
| ----- | ----- | ----- | ----- | ----- | 
| La Union | 1 050,00 | 2 100,00 | 2 | 1 050,00 | 
| El Guayabo | 1 866,67 | 4 750,00 | 7 | 2 035,71 | 
| Santa Rosa | 1 450,00 | 2 840,00 | 10 | 1 420,00 | 
| **Total** | **1 500,00** | **3 394,44** | **19** | **1 607,89** | 

### Tarjetas sin segmentadores (Total general del modelo E3)

* `[Kilos]`: **77 550**

* `[Cosechas]`: **25**

* `[Camiones]`: **54**

* `[Carga por camión]`: **1 436,11**

* *¿Cuadran con A4?:* Sí, cuadran de manera exacta con las 25 cosechas únicas y las 54 pesadas registradas en el archivo CSV original.

### Respuesta E4

Si no se hubiese guardado la columna `pesadas` en la agregación, no habría forma de calcular el total de camiones en DAX sin regresar a Power Query o cargar la tabla de pesadas desagregada.

## Parte F · La fórmula del paso y preguntas de cierre

### F1. Línea literal de Power Query

```
#"Filas agrupadas" = Table.Group(#"Tipo cambiado", {"cosecha_id", "finca_id", "cultivo_id", "fecha", "calidad"}, {{"kg", each List.Sum([kg]), type nullable number}, {"pesadas", each Table.RowCount(_), Int64.Type}, {"carga_promedio", each List.Average([kg]), type nullable number}})

```

* **Llaves de agrupación:**
  `{"cosecha_id", "finca_id", "cultivo_id", "fecha", "calidad"}`

* **Las tres agregaciones:**

  1. `{"kg", each List.Sum([kg]), type nullable number}`

  2. `{"pesadas", each Table.RowCount(_), Int64.Type}`

  3. `{"carga_promedio", each List.Average([kg]), type nullable number}`

### Respuesta F2 (Camiones distintos por mes)

No es posible obtenerlos con la tabla actual porque al agrupar se eliminó la identificación individual de cada camión. Guardar un recuento de valores únicos por cosecha y luego sumarlo produciría doble contabilización de los camiones que operaron en más de una cosecha dentro del mismo mes.

### F3. Preguntas de cierre

1. **¿Qué hace Agrupar por, dicho en filas y columnas?:** Reduce el número de filas combinando aquellas que comparten los mismos valores en las llaves y redefine las columnas dejando únicamente las llaves elegidas junto con los cálculos agregados.

2. **Regla de detección del día:** Un promedio no puede calcularse promediando promedios previos cuando los subgrupos tienen distinto tamaño de muestra.

3. **¿Por qué el promedio de los promedios no lo atrapó ninguna prueba?:** Porque DAX ejecuta una operación matemática y sintáctica totalmente válida sobre una columna numérica real, produciendo un valor numérico coherente a simple vista pero estadísticamente distorsionado al carecer de ponderación.

4. **¿Por qué `[Camiones]` es un `SUM` y no un `COUNTROWS`?:** Porque cada fila de la tabla representa ahora una cosecha completa, por lo que el total de camiones corresponde a la suma de los valores precalculados en la columna `pesadas`.

5. **`AVERAGE( h_cosecha[kg] )` en B3:** Habría arrojado exactamente 1 607,89 kg. En ese punto sí era matemáticamente correcto porque el nivel de granularidad original de la tabla era de un camión pesado por fila.