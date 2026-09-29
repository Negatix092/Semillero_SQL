# Ejercicio 28 · El precio de cada cosecha

- **Alumno:** Cortez Cardozo Axel Josue
- **Ejercicio / proyecto:** Ejercicio 28 · Combinar consultas, granularidad de llaves, fan-out y LOOKUPVALUE
- **Archivo:** `entregas/cortez-axel/Ejercicio28_Cortez_Axel.md`

---

# Parte A · El modelo

## A0. Configuración regional

Se estableció la configuración regional en **Español (México)** para procesar correctamente el separador decimal con punto en `precios.csv` (por ejemplo, `0.50`), evitando que se interprete como el entero 50.

## A3. Medidas del modelo

### Medida Kilos

```dax
Kilos = SUM( h_cosecha[kg] )
```

- **Resultado:** 30 550 kg en la ventana de control inicial.

### Medida Meta

```dax
Meta = SUM( h_meta[kg_meta] )
```

- **Resultado:** 24 440 kg presupuestados.

### Medida Cumplimiento

```dax
Cumplimiento = DIVIDE( [Kilos] , [Meta] )
```

- **Resultado:** 125,00 %.

### Medida Cosechas

```dax
Cosechas = COUNTROWS( h_cosecha )
```

- **Resultado:** 9 cosechas en el rango de enero a abril.

### Medida Filas repetidas

```dax
Filas repetidas =
COUNTROWS( h_cosecha ) -
DISTINCTCOUNT( h_cosecha[cosecha_id] )
```

- **Resultado:** 0.

## A4. La tabla de control

**Segmentadores:** `anio = 2026`, `mes = 1 a 4`.

| Finca | [Kilos] | [Meta] | [Cumplimiento] | [Cosechas] |
|---|---:|---:|---:|---:|
| Agricola La Union | 2 100 | 5 000 | 42,00 % | 2 |
| Finca El Guayabo | 14 250 | 9 440 | 150,95 % | 3 |
| Hacienda Santa Rosa | 14 200 | 10 000 | 142,00 % | 4 |
| **Total** | **30 550** | **24 440** | **125,00 %** | **9** |

- **Tarjeta `[Filas repetidas]`:** 0

## A5. Columnas necesarias para encontrar un solo precio

Se requieren `cultivo_id` y `calidad`, puesto que el precio unitario varía según si el corte es de primera o de segunda calidad para un mismo cultivo.

---

# Parte B · La combinación obvia

## B1. Filas iniciales en `h_cosecha`

La consulta registra inicialmente **25 filas**.

## B2. Mensaje literal al combinar solo por `cultivo_id`

```text
La selección coincide con 25 de 25 filas de la primera tabla.
```

## B4. Medidas de ingresos

### Medida Ingresos

```dax
Ingresos =
SUMX(
    h_cosecha,
    h_cosecha[kg] * h_cosecha[precio_kg]
)
```

- **Resultado preliminar:** 28 080.

### Medida Precio por kilo

```dax
Precio por kilo =
DIVIDE(
    [Ingresos],
    [Kilos]
)
```

- **Resultado preliminar:** 0,55.

## B5. Tabla de ingresos por finca

**Combinación simple por cultivo.**

**Segmentadores:** `anio = 2026`, `mes = 1 a 4`.

| Finca | [Ingresos] | [Precio por kilo] |
|---|---:|---:|
| Agricola La Union | 9 030 | 0,55 |
| Finca El Guayabo | 7 390 | 0,55 |
| Hacienda Santa Rosa | 11 660 | 0,55 |
| **Total** | **28 080** | **0,55** |

## B6. Sospecha contra el correo comercial

No habría levantado sospechas inmediatas de forma aislada, dado que el precio ponderado arrojó **0,55**, cuadrando superficialmente con la expectativa de «cincuenta y tantos centavos» manifestada por el área comercial.

---

# Parte C · El número que no debía moverse

## C1. Impacto en la tabla de control

| Finca | [Kilos] | [Meta] | [Cumplimiento] | [Cosechas] |
|---|---:|---:|---:|---:|
| Agricola La Union | 4 200 | 5 000 | 84,00 % | 4 |
| Finca El Guayabo | 24 050 | 9 440 | 254,77 % | 5 |
| Hacienda Santa Rosa | 23 050 | 10 000 | 230,50 % | 8 |
| **Total** | **51 300** | **24 440** | **209,90 %** | **17** |

- **Tarjeta `[Filas repetidas]`:** 8

## C2. Filas de `h_cosecha` tras la combinación

La consulta se incrementó a **46 filas**.

## C3. Distribución de columnas de `cosecha_id`

- **Distintos:** 25
- **Únicos:** 4
- **Captura:** `clase28-distribucion.png`

## C4. Análisis de la cosecha duplicada (`cosecha_id = 2`)

Al filtrar `cosecha_id = 2` se observaron dos filas:

- **Fila 1:** `calidad = segunda`, `precio_kg = 0.50`.
- **Fila 2:** `calidad = segunda`, `precio_kg = 0.30`.

El precio legítimo correspondiente era **0.30**, pero al realizar el cruce sin discriminar calidad, Power Query multiplicó la fila incorporando ambos precios de la lista.

## C5. Las cuatro cosechas únicas

Las cosechas **7, 14, 21 y 25** se mantuvieron únicas porque corresponden a **Maíz**, cultivo que en `precios.csv` contiene un único registro (`calidad = primera`), evitando el fan-out.

## C6. Alcance del mensaje «25 de 25»

El mensaje indica que todas las filas de la tabla izquierda hallaron al menos una correspondencia en la tabla derecha.

Sin embargo, no indica si cada fila coincidió con una única fila o si se multiplicó debido a coincidencias múltiples en la tabla de destino.

## C7. Estabilidad del precio por kilo

El precio por kilo casi no se alteró (**0,55 frente a 0,56**) debido a que la duplicación cartesiana incrementó proporcionalmente tanto los kilos computados como el importe económico, atenuando la variación dentro del cociente.

---

# Parte D · El segundo arreglo obvio

## D2. Las tres pruebas tras Quitar duplicados

| Prueba | Resultado obtenido |
|---|---:|
| Filas de `h_cosecha` en Power Query | 25 |
| Tabla de control | 30 550 / 125,00 % / 9 |
| `[Filas repetidas]` | 0 |

## D3. Tabla de ingresos tras Quitar duplicados

| Finca | [Ingresos] |
|---|---:|
| Agricola La Union | 5 250 |
| Finca El Guayabo | 5 610 |
| Hacienda Santa Rosa | 7 250 |
| **Total** | **18 110** |

## D4. Cálculo manual del ingreso por cosecha

| cosecha_id | Cultivo | Calidad | Kilos | Precio que toca | Ingreso |
|---:|---|---|---:|---:|---:|
| 1 | Mango | primera | 4 200 | 0,50 | 2 100 |
| 2 | Mango | segunda | 3 100 | 0,30 | 930 |
| 3 | Mango | primera | 5 400 | 0,50 | 2 700 |
| 4 | Guayaba | primera | 1 500 | 0,40 | 600 |
| 5 | Guayaba | primera | 2 600 | 0,40 | 1 040 |
| 6 | Guayaba | segunda | 1 850 | 0,20 | 370 |
| 7 | Maiz | primera | 9 800 | 0,30 | 2 940 |
| 8 | Cacao | primera | 1 200 | 2,50 | 3 000 |
| 9 | Cacao | primera | 900 | 2,50 | 2 250 |
| **Total** |  |  | **30 550** |  | **17 120** |

## D5. Cosechas que explican la diferencia de 990 dólares

La discrepancia entre **18 110** y **17 120** proviene de las dos cosechas realizadas con calidad de segunda.

### Cosecha 2 — Mango

```text
3 100 kg × (0,50 - 0,30) = 620 USD de más
```

### Cosecha 6 — Guayaba

```text
1 850 kg × (0,40 - 0,20) = 370 USD de más
```

### Diferencia total

```text
620 + 370 = 990 USD
```

Por lo tanto, la diferencia total es de **990 USD**.

## D6. Acción de Quitar duplicados

**Quitar duplicados** conserva arbitrariamente el primer registro que encuentra en el búfer de memoria y suprime los subsecuentes, dejando la asignación de precios sujeta al orden incidental de arribo tras el `join`.

---

# Parte E · La llave completa

## E1. Pasos aplicados eliminados en Power Query

Se eliminaron los siguientes pasos:

1. **Filas duplicadas quitadas**
2. **Se expandió precios**
3. **Consultas combinadas**

## E2. Mensaje al combinar por la llave compuesta

```text
La selección coincide con 25 de 25 filas de la primera tabla.
```

## E3. Filas y distribución en `cosecha_id`

- **Filas totales:** 25
- **Distribución:** 25 distintos, 25 únicos

## E4. Comprobación de control

- **Kilos:** 30 550
- **Meta:** 24 440
- **Cumplimiento:** 125,00 %
- **Cosechas:** 9
- **Filas repetidas:** 0

## E5. Tabla de ingresos por finca definitiva

**Segmentadores:** `anio = 2026`, `mes = 1 a 4`.

| Finca | [Ingresos] | [Precio por kilo] |
|---|---:|---:|
| Agricola La Union | 5 250 | 2,50 |
| Finca El Guayabo | 5 240 | 0,37 |
| Hacienda Santa Rosa | 6 630 | 0,47 |
| **Total** | **17 120** | **0,56** |

- **Tarjeta `[Filas repetidas]`:** 0

> **Captura:** `clase28-ingresos.png`

## E6. Comparación por cultivo

| Cultivo | Ingresos (Parte B) | Ingresos (Parte E) |
|---|---:|---:|
| Mango | 9 030 | 5 730 |
| Guayaba | 3 710 | 3 200 |
| Cacao | 10 500 | 5 250 |
| Maiz | 2 940 | 2 940 |

**Cultivo idéntico:** **Maíz**, ya que carece de registros con calidad de segunda en la lista maestra y no sufrió multiplicaciones ni sobreprecios en ningún caso.

## E7. Llave primaria de `precios`

La llave unívoca está constituida por la tupla compuesta:

```text
cultivo_id + calidad
```

---

# Parte F · Quién avisa, y preguntas de cierre

## F1. Error arrojado por `LOOKUPVALUE` con llave incompleta

```text
Se proporcionó una tabla con varios valores donde se esperaba un solo valor.
```

## F2. Medida de validación contra búsqueda completa

```dax
Diferencia de precio =
SUMX(
    h_cosecha,
    ABS(
        h_cosecha[precio_kg] -
        h_cosecha[Precio buscado]
    )
)
```

- **Resultado:** 0.

## F3. Cardinalidad propuesta en la vista de modelo

Al intentar relacionar:

```text
h_cosecha[cultivo_id]
```

con:

```text
precios[cultivo_id]
```

Power BI propuso una relación **Varios a varios (`*:*`)**, advirtiendo que ninguna de las columnas contenía valores unívocos.

## F4. Respuestas a preguntas de cierre

### Anexar vs. Combinar

**Anexar** apila tablas añadiendo filas hacia abajo, mientras que **Combinar** fusiona tablas agregando columnas hacia los costados basándose en claves.

### Regla de detección del día

> **«Si al combinar consultas se inflan los totales preexistentes de la tabla de hechos, la combinación multiplicó registros por carecer de una llave con la granularidad adecuada.»**

### Quitar duplicados vs. Quitar errores

Ambos ocultan anomalías eliminando datos.

Sin embargo, **Quitar duplicados** es más crítico porque simula normalidad al restaurar las métricas de volumen, encubriendo asignaciones financieras erróneas.

### Tabla causante de fan-out

La tabla `precios` desempeñó el rol de **tabla repetidora (1 a varios)**, duplicando filas en la tabla de hechos.

### Precios por cultivo, calidad y mes

Si los precios se definieran por **cultivo, calidad y mes**, se requeriría combinar utilizando una clave triple:

```text
cultivo_id + calidad + mes
```

Antes de realizar la combinación, debería validarse que no existan duplicados en esa terna dentro de la tabla de precios.