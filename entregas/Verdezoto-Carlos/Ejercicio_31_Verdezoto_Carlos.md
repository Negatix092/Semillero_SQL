# Ejercicio 31 · Las pesadas de la báscula

## PARTE A - EL MODELO SIN COSECHAS

**A4. Respuestas de exploración inicial en `pesadas.csv`:**
* ¿Cuántas filas de datos tiene? Tiene **54 filas** en total.
* ¿Cuántas cosechas distintas? Hay **25 cosechas** distintas (`cosecha_id`).
* ¿En cuántos camiones se pesó la cosecha 7? Se pesó en **4 camiones** (4 filas para el `cosecha_id = 7`).

---

## PARTE B - TAL CUAL, Y AGRUPAR POR BÁSICO

**B3.** La medida `[Cosechas]` está contando el total de **filas de la tabla**, que en este momento equivalen a **pesadas individuales (camiones)**, no a cosechas completas. Al haber 19 camiones en el primer cuatrimestre, el recuento arroja 19 en lugar de las 9 cosechas reales.

---

## PARTE C - LAS LLAVES

**C3.** Tres camiones dieron dos filas porque uno de los camiones (placa `LRA-1101`) realizó dos viajes (pesadas) en esa misma cosecha. Al incluir la columna `camion` como llave en el agrupamiento, Power Query colapsó los dos viajes de esa placa en una sola fila, sumando sus kilos.

**C5.** 
* **Qué puede ir en las llaves:** Cualquier columna que describa la entidad principal a la que quieres llegar (la cosecha). Esto incluye `cosecha_id`, `finca_id`, `cultivo_id`, `fecha` y `calidad`.
* **Qué NO puede ir:** Columnas con un nivel de granularidad más bajo o detallado (como `camion` o `pesada_id`), porque rompen la agrupación.
* Si hubiéramos puesto `pesada_id` en las llaves, Power Query no habría podido agrupar absolutamente nada y habrían salido las mismas **54 filas** originales, ya que cada `pesada_id` es único.

---

## PARTE D - CUÁNTO LLEVA UN CAMIÓN

### D3 · Tabla a mano (El Guayabo en Enero–Abril 2026)
| `cosecha_id` | Camiones | Kilos | Kilos entre camiones |
|---|---|---|---|
| 5 | 2 | 2 600 | 1 300 |
| 6 | 1 | 1 850 | 1 850 |
| 7 | 4 | 9 800 | 2 450 |
| **El Guayabo** | **7** | **14 250** | **2 035,71** |
**

**D4.** En El Guayabo, el promedio DAX simple da 1 866,67 porque suma (1 300 + 1 850 + 2 450) / 3, dándole el mismo peso a cada cosecha. La **cosecha 7** jala el promedio real hacia arriba porque tuvo 4 camiones muy pesados, pero el `AVERAGE` ignora cuántos camiones hubo dentro de cada fila agrupada[cite: 28].

**D5.** La medida `[Carga promedio 2]` (Kilos / Cosechas) contesta en realidad el **rendimiento o peso promedio por cosecha completa**, no la carga promedio por camión[cite: 28].

**D6.** 
* En B3 `[Cosechas]` contaba las filas originales sin agrupar (19 pesadas)[cite: 28]. Ahora cuenta las filas agrupadas (9 cosechas)[cite: 28].
* La Unión dio 1 050,00 con las dos medidas porque en esa finca **cada cosecha se transportó en exactamente 1 camión** (2 cosechas = 2 camiones)[cite: 28]. Al ser 1 a 1, no hay ponderación que distorsione el promedio matemático[cite: 28].

---

## PARTE E - EL ARREGLO: LA CUENTA QUE GUARDASTE

### E1 · Medidas
```dax
Camiones = SUM( h_cosecha[pesadas] )

```

```dax
Carga por camión = DIVIDE( [Kilos] , [Camiones] )

```

**E4.** Si no hubieras guardado el recuento de filas original (`pesadas`) durante la agrupación en Power Query, no tendrías de dónde sacar los camiones en DAX porque ese nivel de detalle se destruyó al consolidar la tabla. Tendrías que volver a Power Query a rehacer el paso o cargar la tabla de pesadas original como una tabla de hechos paralela.

---

## PARTE F - LA FÓRMULA DEL PASO, Y PREGUNTAS DE CIERRE

**F1.** Línea literal:

```powerquery
= Table.Group(#"Tipo cambiado", {"cosecha_id", "finca_id", "cultivo_id", "fecha", "calidad"}, {{"kg", each List.Sum([kg]), type nullable number}, {"pesadas", each Table.RowCount(_), type number}, {"carga_promedio", each List.Average([kg]), type nullable number}})

```

* **Llaves:** `{"cosecha_id", "finca_id", "cultivo_id", "fecha", "calidad"}`
* **Agregaciones:** `kg` (Suma), `pesadas` (Recuento de filas), `carga_promedio` (Promedio).



**F2.** Con la tabla ya agrupada a nivel de cosecha, **no se puede** saber cuántos camiones distintos trabajaron en el mes. No sirve guardar un "Recuento de filas distintas" de placas dentro del `Table.Group` porque, al sumarlos después por mes, estarías contando doble a un mismo camión si participó en varias cosechas distintas.

**F3. Preguntas de cierre:**

1. **¿Qué hace Agrupar por?** Colapsa múltiples **filas** con llaves idénticas en una sola, resumiendo o agregando matemáticamente los valores de las demás **columnas**.


2. **Regla de detección:** *"Si al cargar datos detallados, una medida de promedios o recuentos (como COUNTROWS) se infla y el modelo no agrupa por las llaves primarias correctas, las filas representan fragmentos de la transacción y no la entidad completa"* (o *"Si [Cosechas repetidas] arroja filas de más, tienes que subir el grano agrupando"*).


3. **¿Por qué el promedio de promedios no lo atrapó ninguna prueba?** Porque el modelo de datos (Power Query) ya estaba perfectamente agrupado a 1 fila por cosecha, y no había duplicidad en las relaciones. El error no era estructural, sino conceptual en la fórmula DAX al promediar porcentajes o ratios sin su denominador ponderado (`pesadas`).


4. **¿Por qué [Camiones] es un SUM?** Porque en nuestro nuevo modelo agrupado, la tabla de hechos es `h_cosecha`; si usamos `COUNTROWS`, contaríamos cosechas. Los camiones quedaron guardados como un valor numérico agregado dentro de la columna `pesadas`, por lo que hay que sumarlos fila por fila.


5. **¿Qué habría dado AVERAGE en B3?** En B3, `AVERAGE(h_cosecha[kg])` habría dado exactamente la carga promedio real por camión (los kilos totales divididos por las 54 pesadas originales). Ahí sí estaba bien matemáticamente porque, antes de agrupar, cada fila en el modelo representaba exactamente un camión/pesada.
