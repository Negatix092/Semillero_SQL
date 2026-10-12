# Ejercicio 31 · Las pesadas de la báscula

- **Alumno:** Axel Cortez
- **Entorno:** Power BI Desktop / Power Query / DAX

---

# Parte A · El modelo sin cosechas

## A3. Medida de Meta

```dax
Meta = SUM( h_meta[kg_meta] )
```

- **Resultado (2026, Meses 1–4):** 24 440.

## A4. Respuestas iniciales

### ¿Cuántas filas de datos tiene `pesadas.csv`?

El archivo contiene **54 filas de datos**, correspondientes a pesadas individuales.

### ¿Cuántas cosechas distintas existen?

Existen **25 cosechas únicas**.

### ¿En cuántos camiones se pesó la cosecha 7?

La cosecha 7 se pesó en **3 camiones**.

---

# Parte B · Tal cual, y Agrupar por básico

## B2. Medidas DAX iniciales

### Medida Kilos

```dax
Kilos = SUM( h_cosecha[kg] )
```

- **Resultado (2026, Meses 1–4):** 30 550.

### Medida Cosechas

```dax
Cosechas = COUNTROWS( h_cosecha )
```

- **Resultado (2026, Meses 1–4):** 19.

### Medida Cumplimiento

```dax
Cumplimiento = DIVIDE( [Kilos], [Meta] )
```

- **Resultado (2026, Meses 1–4):** 125,00 %.

### Medida Cosechas repetidas

```dax
Cosechas repetidas =
[Cosechas] -
DISTINCTCOUNT( h_cosecha[cosecha_id] )
```

- **Resultado (2026, Meses 1–4):** 10.

## B3. Diagnóstico de filas

La medida `[Cosechas]` está contando el número de filas físicas de la tabla `pesadas.csv`.

Esto equivale a la cantidad de **camiones/pesadas** y no al número de cosechas reales.

## B4. Resultados de Agrupar por básico

Después de aplicar **Agrupar por** utilizando únicamente `cosecha_id`:

- **Filas:** 25
- **Columnas:** 2
- **Columnas resultantes:** `cosecha_id` y `Recuento`
- **Operación:** Recuento de filas

Desaparecieron todas las dimensiones indispensables:

- `finca_id`
- `cultivo_id`
- `fecha`
- `calidad`
- `kg`

Esto destruyó las relaciones necesarias para el funcionamiento del modelo.

---

# Parte C · Las llaves

## C1 y C2. Agrupación con `camion` en las llaves

La agrupación produjo **36 filas**.

### Tabla de control

| Finca | Kilos | Cosechas | Meta | Cumplimiento | Cosechas repetidas |
|---|---:|---:|---:|---:|---:|
| Santa Rosa | 14 200 | 6 | 10 000 | 142,00 % | 2 |
| El Guayabo | 14 250 | 5 | 9 440 | 150,95 % | 2 |
| La Union | 2 100 | 3 | 5 000 | 42,00 % | 1 |
| **Total** | **30 550** | **14** | **24 440** | **125,00 %** | **5** |

## C3. Inspección de Cosecha 1

En `pesadas.csv`, la cosecha 1 tuvo:

- **3 pesadas**
- **2 camiones distintos**

La distribución fue:

- `CAM-01`: 2 viajes
- `CAM-02`: 1 viaje

Al incluir `camion` dentro de las llaves de agrupación, las dos pesadas correspondientes al mismo camión se sumaron en una sola fila.

Por lo tanto:

```text
3 pesadas → 2 filas agrupadas
```

## C4. Corrección de llaves

Se retiró `camion` de las llaves y se agregaron las siguientes operaciones:

- `kg` → **Suma**
- `pesadas` → **Recuento de filas**
- `carga_promedio` → **Promedio**

La tabla quedó finalmente en **25 filas**.

La tabla de control regresó a:

```text
Kilos / Meta / Cumplimiento / Cosechas
30 550 / 24 440 / 125,00 % / 9
```

Además:

- **[Cosechas repetidas]:** 0

## C5. Criterio de llaves

Solo pueden incluirse como llaves las columnas que sean atributos propios y únicos del evento agrupado:

- `cosecha_id`
- `finca_id`
- `cultivo_id`
- `fecha`
- `calidad`

Si se hubiese incluido `pesada_id`, no se habría agrupado nada y habrían permanecido las **54 filas originales**.

---

# Parte D · Cuánto lleva un camión

## D1. Medida de Carga Promedio

Esta primera medida calcula un **promedio simple de promedios**.

```dax
Carga promedio =
AVERAGE( h_cosecha[carga_promedio] )
```

### Resultados

| Finca | Carga promedio |
|---|---:|
| La Union | 1 050,00 |
| El Guayabo | 1 866,67 |
| Santa Rosa | 1 450,00 |
| **Total** | **1 500,00** |

## D2 y D3. Conciliación manual de El Guayabo

**Periodo:** Meses 1–4, 2026.

| cosecha_id | Camiones | Kilos | Kilos entre camiones |
|---:|---:|---:|---:|
| 5 | 1 | 3 100 | 3 100,00 |
| 6 | 3 | 5 550 | 1 850,00 |
| 7 | 3 | 5 600 | 1 866,67 |
| **El Guayabo** | **7** | **14 250** | **2 035,71** |

## D4. ¿Por qué difiere `[Carga promedio]`?

En **El Guayabo**, la cosecha 5 llevó **3 100 kg en un solo camión**, mientras que las cosechas 6 y 7 utilizaron **3 camiones cada una**, con promedios menores.

Al calcular un promedio simple:

```text
(3 100 + 1 850 + 1 866,67) / 3
```

la cosecha que utilizó un solo camión tiene el mismo peso estadístico que cada cosecha que utilizó tres camiones.

Esto provoca que se subestime la carga real ponderada por camión.

En **Santa Rosa** ocurre el comportamiento inverso debido a la presencia de camiones con cargas más pequeñas.

## D5. Medida de Carga Promedio 2

```dax
Carga promedio 2 =
DIVIDE( [Kilos], [Cosechas] )
```

### Resultados

| Finca | Carga promedio 2 |
|---|---:|
| El Guayabo | 4 750,00 |
| Santa Rosa | 3 550,00 |
| **Total** | **3 394,44** |

### Significado

Esta medida **no calcula la carga de un camión**.

En realidad, calcula el peso promedio producido por cada cosecha completa:

```text
Kilos totales / Número de cosechas
```

## D6. Análisis de resultados

### ¿Por qué `[Cosechas]` cambió de 19 a 9?

Antes de agrupar, `[Cosechas]` contaba las pesadas físicas individuales cargadas en la tabla.

Después de agrupar, cada fila representa un evento real de cosecha consolidado.

Por lo tanto:

```text
Antes de agrupar: 19 filas/pesadas
Después de agrupar: 9 cosechas
```

### ¿Por qué La Unión dio 1 050,00 en ambas medidas?

Cada una de las dos cosechas de **La Unión** se transportó utilizando exactamente **1 camión**.

Por ello:

```text
2 100 kg / 2 cosechas = 1 050 kg
```

y:

```text
2 100 kg / 2 camiones = 1 050 kg
```

En este caso particular, el promedio por cosecha y el promedio por camión coinciden.

---

# Parte E · El arreglo: la cuenta que guardaste

## E1. Medidas ponderadas de logística

### Medida Camiones

```dax
Camiones =
SUM( h_cosecha[pesadas] )
```

### Medida Carga por camión

```dax
Carga por camión =
DIVIDE( [Kilos], [Camiones] )
```

## E2. Tabla de control final comparativa

| Finca | Carga promedio | Carga promedio 2 | Camiones | Carga por camión |
|---|---:|---:|---:|---:|
| Santa Rosa | 1 450,00 | 3 550,00 | 10 | 1 420,00 |
| El Guayabo | 1 866,67 | 4 750,00 | 7 | 2 035,71 |
| La Union | 1 050,00 | 1 050,00 | 2 | 1 050,00 |
| **Total** | **1 500,00** | **3 394,44** | **19** | **1 607,89** |

## E3. Totales globales históricos

Considerando ambos años:

- **Kilos:** 77 550
- **Cosechas:** 25
- **Camiones:** 54
- **Carga por camión:** 1 436,11

## E4. Alternativa sin guardar `pesadas`

Si no se hubiera conservado la columna `pesadas` durante la transformación en Power Query, no habría forma de recuperar directamente la cantidad de camiones mediante DAX.

Para obtener nuevamente esa información sería necesario:

- Volver al origen, o
- Realizar combinaciones adicionales.

Esto ocurre porque el detalle individual del camión fue eliminado durante la agregación.

---

# Parte F · Preguntas de cierre

## F1. Línea literal en el Editor Avanzado

```powerquery
= Table.Group(
    #"Tipo cambiado",
    {"cosecha_id", "finca_id", "cultivo_id", "fecha", "calidad"},
    {
        {"kg", each List.Sum([kg]), type nullable number},
        {"pesadas", each Table.RowCount(_), type number},
        {"carga_promedio", each List.Average([kg]), type nullable number}
    }
)
```

### Llaves utilizadas

- `cosecha_id`
- `finca_id`
- `cultivo_id`
- `fecha`
- `calidad`

### Agregaciones utilizadas

- `kg` → `List.Sum`
- `pesadas` → `Table.RowCount`
- `carga_promedio` → `List.Average`

## F2. Pregunta de logística sobre camiones únicos

No se puede determinar cuántos **camiones distintos** trabajaron en el mes utilizando únicamente la tabla agrupada.

Esto ocurre porque un mismo camión puede haber participado en múltiples cosechas distintas.

Por lo tanto, sumar un recuento de camiones distintos calculado individualmente por cosecha podría provocar un **doble conteo de vehículos** a nivel mensual.

---

# F3. Respuestas conceptuales

## Agrupar por

**Agrupar por** reduce filas combinando registros que comparten valores idénticos en las columnas utilizadas como llaves.

Las demás columnas se colapsan mediante operaciones de agregación, como:

- Suma
- Promedio
- Recuento
- Mínimo
- Máximo

## Regla de detección del día

> **«Si una llave de agrupación varía dentro del evento que quieres resumir, Agrupar por no agrega columnas: parte la cosecha en pedazos.»**

## Falla silenciosa del promedio

La medida `[Cosechas repetidas]` compara cardinalidades directas contra llaves primarias, permitiendo detectar problemas estructurales en las filas.

En cambio, un **promedio de promedios** puede generar un resultado numérico matemáticamente válido, pero estadísticamente no ponderado.

Por ello, el cálculo puede ser incorrecto para el objetivo del análisis sin producir ninguna alarma estructural.

## ¿Por qué se utiliza `SUM` en `[Camiones]`?

Cada fila de la tabla agrupada representa una cosecha completa y almacena en la columna `pesadas` la cantidad total de pesadas/camiones correspondientes a esa cosecha.

Por lo tanto, para obtener el total general se deben sumar esos valores precalculados:

```dax
Camiones =
SUM( h_cosecha[pesadas] )
```

## Comportamiento previo en B3

Antes de realizar la agrupación, el cálculo habría devuelto:

```text
1 607,89
```

y habría sido matemáticamente correcto.

Esto ocurre porque en B3 cada fila representaba exactamente una **pesada individual de un camión**.

En ese nivel de granularidad, calcular el promedio simple de los kilos de cada fila equivalía directamente a obtener la **media global de carga por pesada/camión**.