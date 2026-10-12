# Proyecto integrador AgroDB · PC4 — El análisis y la seguridad

**Estudiante:** Axel Cortez  
**Entorno:** Power BI Desktop · Power Query · DAX  
**Período del cierre:** enero–septiembre de 2026 (`dim_tiempo[anio] = 2026`; `dim_tiempo[mes] = 1…9`)  
**Etapa:** 4 · Jueves · Análisis y seguridad

---

## 1. Ancla del jueves

| Comprobación | Resultado obtenido | Ancla esperada | Validación |
|---|---:|---:|---|
| **Ver como** `regional.sur@agrodb.test`: fincas visibles | **2** | **2** | Cuadra |
| **Ver como** `regional.sur@agrodb.test`: `[Cumplimiento]` total | **104,44 %** | **104,44 %** | Cuadra |

El rol `AccesoFincas` restringe al usuario regional a **Agrícola La Unión** y **Hacienda Los Ceibos**. En el período del cierre, sus valores conjuntos son **48.250 kg cosechados** y **46.200 kg de meta**. La validación se realizó con **Modelado → Ver como** y la identidad indicada.

---

## 2. Preguntas ciegas del jueves

### J1. Podio de cultivos con `tipo = perenne`

Se configuró un gráfico de barras agrupadas con `dim_cultivo[cultivo]`, `[Kilos]` y un filtro visual **N principales = 3**, evaluado por `[Kilos]`. El segmentador utiliza `dim_cultivo[tipo]`, de modo que el podio se recalcula con la selección.

| Lugar | Cultivo | Kilos |
|---:|---|---:|
| 1 | Banano | 43.500 |
| 2 | Mango | 17.500 |
| 3 | Guayaba | 12.050 |
| **Total** | **Tres cultivos perennes** | **73.050** |

### J2. Bono y apoyo por finca, frente al cálculo global

La regla se calcula **finca por finca**: el bono es el excedente positivo sobre la meta y el apoyo es el déficit positivo frente a ella.

| Finca | Kilos | Meta | Bono (kg) | Apoyo (kg) |
|---|---:|---:|---:|---:|
| Agrícola La Unión | 4.750 | 6.200 | 0 | 1.450 |
| Finca El Guayabo | 22.800 | 21.000 | 1.800 | 0 |
| Hacienda Los Ceibos | 43.500 | 40.000 | 3.500 | 0 |
| Hacienda Santa Rosa | 20.250 | 19.000 | 1.250 | 0 |
| **Total** | **91.300** | **86.200** | **6.550** | **1.450** |

Si se aplicara la regla a la **empresa completa** en lugar de finca por finca, el excedente neto sería `91.300 − 86.200 = 5.100 kg`: **bono global de 5.100 kg** y **apoyo global de 0 kg**. La diferencia existe porque, al agregar primero, los excedentes compensan el déficit de La Unión.

### J3. Escenario de meta al 110 %

| Indicador | Resultado |
|---|---:|
| Meta original del cierre | 86.200 kg |
| Meta ajustada al 110 % | **94.820 kg** |
| Kilos cosechados | 91.300 kg |
| `[Cumplimiento ajustado]` | **96,29 %** |
| `[Fincas que cumplen]` | **0** |

Ninguna finca alcanza su propia meta incrementada al 110 %. El segmentador admite valores del 100 % al 150 % en intervalos de 10 puntos. Sin selección, las medidas muestran la meta original; con **dos niveles seleccionados simultáneamente**, devuelven `BLANK()` para no asignar un escenario arbitrario.

### J4. KPI acumulado del año al cierre

| Indicador | Resultado |
|---|---:|
| Valor: `[Kilos acumulados]` | **91.300 kg** |
| Objetivo: `[Meta acumulada]` | **86.200 kg** |
| Diferencia absoluta | **+5.100 kg** |
| Diferencia porcentual sobre el objetivo | **+5,92 %** |

El KPI tiene eje de tendencia `dim_tiempo[fecha]` y título que identifica el período: **«KPI de cosecha vs. meta acumulada — enero a septiembre 2026»**. Se utilizaron medidas acumuladas para que el último punto represente el cierre y no únicamente septiembre.

### J5. Pruebas de seguridad por usuario

**Usuario:** `gerente.launion@agrodb.test`

| Finca visible | Kilos | Meta | Cumplimiento | Semáforo |
|---|---:|---:|---:|---|
| Agrícola La Unión | **4.750** | **6.200** | **76,61 %** | **Rojo** |

**Usuario:** `practicante@agrodb.test`

No se muestran fincas ni valores empresariales en las pruebas realizadas con **Ver como**. Esto confirma el comportamiento de **denegar por defecto** a un correo sin asignaciones en `seguridad`.

**Comprobación adicional:** `regional.sur@agrodb.test` visualiza **2 fincas**, **48.250 kg**, **46.200 kg de meta** y **104,44 %** de cumplimiento.

---

## 3. Medidas y configuración implementadas

### 3.1 Bono y apoyo

```dax
Bono =
SUMX(
    VALUES(dim_finca[finca_id]),
    MAX(0, [Kilos] - [Meta])
)

Apoyo =
SUMX(
    VALUES(dim_finca[finca_id]),
    MAX(0, [Meta] - [Kilos])
)
```

### 3.2 Escenario de meta

Tabla desconectada (sin relaciones con el modelo):

```dax
Escenario Meta = GENERATESERIES(100, 150, 10)
```

```dax
Meta ajustada =
VAR Porcentaje = SELECTEDVALUE('Escenario Meta'[Value])
RETURN
    IF(
        NOT ISFILTERED('Escenario Meta'[Value]),
        [Meta],
        IF(
            HASONEVALUE('Escenario Meta'[Value]),
            [Meta] * DIVIDE(Porcentaje, 100),
            BLANK()
        )
    )

Cumplimiento ajustado =
DIVIDE([Kilos], [Meta ajustada])

Fincas que cumplen =
COALESCE(
    COUNTROWS(
        FILTER(
            VALUES(dim_finca[finca_id]),
            [Kilos] >= [Meta ajustada]
        )
    ),
    0
)
```

La función `COALESCE` permite mostrar **0** cuando ninguna finca cumple, en lugar de una tarjeta vacía.

### 3.3 Semáforo

```dax
Color Semaforo =
SWITCH(
    TRUE(),
    ISBLANK([Meta]), BLANK(),
    [Cumplimiento] >= 1, "#22A06B",
    "#E5484D"
)
```

Se aplicó **formato condicional por valor de campo** a la columna de cumplimiento de la tabla por finca. El semáforo compara con la meta **real**, no con el escenario hipotético.

### 3.4 KPI acumulado

```dax
Kilos acumulados =
VAR FechaCorte = MAX(dim_tiempo[fecha])
RETURN
CALCULATE(
    [Kilos],
    FILTER(
        ALLSELECTED(dim_tiempo),
        dim_tiempo[fecha] <= FechaCorte
    )
)

Meta acumulada =
VAR FechaCorte = MAX(dim_tiempo[fecha])
RETURN
CALCULATE(
    [Meta],
    FILTER(
        ALLSELECTED(dim_tiempo),
        dim_tiempo[fecha] <= FechaCorte
    )
)
```

### 3.5 Rol dinámico de seguridad

**Único rol:** `AccesoFincas`  
**Tabla donde se aplica la regla:** `dim_finca`

```dax
VAR CorreoActual = LOWER(TRIM(USERPRINCIPALNAME()))
VAR FincaActual = dim_finca[finca_id]
RETURN
    COUNTROWS(
        FILTER(
            ALL(seguridad),
            LOWER(TRIM(seguridad[correo])) = CorreoActual
                && seguridad[finca_id] = FincaActual
        )
    ) > 0
```

El rol contrasta el correo de la sesión con las asignaciones de `seguridad` y conserva únicamente las fincas autorizadas. Si no existe coincidencia, no se autoriza ninguna finca. Se verificó que las relaciones relevantes utilizaran **dirección de filtro única (`Single`)**, evitando rutas ambiguas al evaluar las metas bajo RLS.

---

## 4. Bitácora del jueves

| Qué encontré | Cómo me di cuenta | Qué número daba mal | Cómo lo arreglé | Cómo sé que quedó |
|---|---|---|---|---|
| El primer gráfico del podio estaba configurado como barras apiladas al 100 % y el segmentador filtraba por cultivo individual. | El eje mostraba 0 %–100 % y el segmentador listaba Banano, Cacao, etc., en vez de los valores de `tipo`. | No se podían leer kilos reales ni obtener correctamente el podio por tipo. | Cambié a barras agrupadas, usé `dim_cultivo[tipo]` y configuré Top N = 3 por `[Kilos]`. | Con `perenne` aparecen Banano 43.500, Mango 17.500 y Guayaba 12.050 kg; suman 73.050 kg. |
| El recuento de fincas que cumplen al 110 % aparecía vacío. | La tarjeta `[Fincas que cumplen]` quedaba en blanco aunque la meta ajustada y el cumplimiento ajustado sí tenían valores. | Se veía `BLANK()` en vez de **0 fincas**. | Incorporé `COALESCE(COUNTROWS(...), 0)` a la medida. | La tarjeta mostró **0** con escenario 110 %, y `[Cumplimiento ajustado]` mostró **96,29 %**. |
| La seguridad regional mostraba fincas pero fallaban los visuales con metas. | Con `Ver como` se mostraban las dos fincas y 48.250 kg, pero Power BI devolvía el error: `Join paths are expected to form a tree`, señalando dos caminos entre `h_meta` y `dim_finca`. | `[Meta]` y `[Cumplimiento]` no se podían visualizar bajo RLS. | Revisé las relaciones y corregí una que seguía en dirección `Both`, dejándola en `Single`; inicialmente el cambio no se había aplicado. | Tras guardar la corrección, el usuario regional mostró **2 fincas**, **46.200 kg de meta** y **104,44 % de cumplimiento**, exactamente el ancla del PC4. |
| Fue necesario separar un problema de filtrado de página de uno de seguridad. | Probé el rol con una regla fija `dim_finca[finca_id] = 1` en una página sin segmentadores: solo se veía Hacienda Santa Rosa. Después, con el rol dinámico, se mostraron dos fincas. | La página original aparecía vacía, sin identificar inicialmente la causa. | Realicé pruebas aisladas de RLS y de las medidas `[Kilos]` y `[Meta]`; consulté el detalle del error de Power BI. | La prueba aislada permitió detectar la ruta ambigua; corregida la relación, la página PC4 mostró los valores del usuario regional. |
| El acceso debía quedar restringido también para usuarios sin asignación. | Probé `Ver como` con `gerente.launion@agrodb.test` y `practicante@agrodb.test`. | Un acceso no autorizado habría expuesto fincas o cifras. | Mantuve un solo rol dinámico basado en la tabla `seguridad` y comprobé cada identidad. | El gerente ve solo La Unión (4.750 kg, 6.200 kg, 76,61 %, rojo); el practicante no ve datos empresariales. |

---

## 5. Evidencias del PC4

Archivos requeridos por el enunciado, en `entregas/cortez-axel/`:

1. `Proyecto_PC4_Cortez_Axel.md` — este documento.
2. `proyecto-pc4-gerente.png` — **Ver como** `gerente.launion@agrodb.test`, con finca, kilos, meta, cumplimiento y semáforo.
3. `proyecto-pc4-regional.png` — **Ver como** `regional.sur@agrodb.test`, con dos fincas y **104,44 %**.
4. `proyecto-pc4-practicante.png` — **Ver como** `practicante@agrodb.test`, sin datos empresariales visibles.

> Las capturas deben mostrar el usuario de prueba en la barra superior de Power BI. El archivo `.pbix` se conserva localmente y no se adjunta al PR, según las instrucciones del proyecto.
