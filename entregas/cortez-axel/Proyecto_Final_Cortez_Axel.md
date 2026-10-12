# Proyecto integrador AgroDB — El cierre de septiembre

**Estudiante:** Axel Cortez  
**Entorno:** Power BI Desktop · Power Query · DAX  
**Corte:** 30 de septiembre de 2026  
**Período principal:** enero–septiembre de 2026  
**Entrega:** `Proyecto_Final_Cortez_Axel.md`

> **Criterio de documentación.** Se consolidan PC1, PC2, PC3, PC4 y las pruebas finales. Las cifras comprobadas se distinguen de las verificaciones pendientes. No se atribuyen a los archivos originales números o ubicaciones que no estén documentados.

---

## 1. Resumen ejecutivo

AgroDB cerró enero–septiembre de 2026 con **91.300 kg** cosechados frente a **86.200 kg** de meta. El cumplimiento fue **105,92 %**, con excedente de **5.100 kg** e ingresos calculados de **$4.716.650,00**.

| Indicador | Resultado |
|---|---:|
| Kilos cosechados | 91.300 kg |
| Meta | 86.200 kg |
| Diferencia | +5.100 kg |
| Cumplimiento | 105,92 % |
| Ingresos | $4.716.650,00 |
| Kilos entregados | 82.800 kg |
| Variación frente a enero–septiembre 2025 | +128,25 % |
| Bono (calculado por finca) | 6.550 kg |
| Apoyo (calculado por finca) | 1.450 kg |

### 1.1 Cumplimiento por finca

| Finca | Kilos | Meta | Cumplimiento | Diagnóstico |
|---|---:|---:|---:|---|
| Agrícola La Unión | 4.750 | 6.200 | 76,61 % | Bajo la meta |
| Finca El Guayabo | 22.800 | 21.000 | 108,57 % | Supera la meta |
| Hacienda Los Ceibos | 43.500 | 40.000 | 108,75 % | Supera la meta |
| Hacienda Santa Rosa | 20.250 | 19.000 | 106,58 % | Supera la meta |
| **Empresa** | **91.300** | **86.200** | **105,92 %** | **Supera la meta** |

**Tres hallazgos para dirección.** (1) La Unión presenta déficit de **1.450 kg**; conviene investigar las causas operativas. (2) Los Ceibos lidera la producción con **43.500 kg**; conviene evaluar prácticas transferibles. (3) Banano, Mango y Guayaba suman **73.050 kg**; conviene monitorear la concentración productiva.

**Advertencia:** el cumplimiento global no significa que todas las fincas cumplieron. Además, el aumento interanual incluye a Hacienda Los Ceibos, incorporada en enero de 2026: **128,25 %** es crecimiento del total reportado, no necesariamente crecimiento orgánico comparable.

---

## 2. PC1 — Inventario de fuentes (lunes 5 de octubre)

### 2.1 Inventario previo a la carga

| Fuente | Filas | Columnas | Llave | Tabla de destino |
|---|---:|---:|---|---|
| `dim_finca.csv` | 4 | 2 | `finca_id` | `dim_finca` |
| `dim_cultivo.csv` | 6 | 3 | `cultivo_id` | `dim_cultivo` |
| `dim_tiempo.csv` | 730 | 8 | `fecha` | `dim_tiempo` |
| `seguridad.csv` | 10 | 2 | correo + finca_id | `seguridad` |
| Carpeta `bascula/` | 19 archivos, filas variables | 6 / 7 | `cosecha_id` | `h_cosecha` |
| `bascula_digital_2026.csv` | Variable | 7 | `cosecha_id` | `h_cosecha` |
| `precios.csv` | Variable | 3 | cultivo_id + calidad | `h_cosecha` |
| `metas_planeacion.csv` | 5 | 14 | finca en matriz horizontal | `h_meta` |

### 2.2 Cinco observaciones previas a la carga

| Qué observé | Fuente | Efecto previsto |
|---|---|---|
| Archivo duplicado `bascula_2026-03 - copia.csv` | Carpeta `bascula/` | Inflaría cosechas y kilos de marzo si no se excluye |
| Metas en matriz horizontal, con fila de total | `metas_planeacion.csv` | Requiere despivotar meses y excluir total para evitar duplicidad |
| Hacienda Los Ceibos aparece desde 2026 | `dim_finca.csv` y datos de cosecha | Afecta comparabilidad con 2025 |
| Diferencias entre `kg` y `kg_neto` | Archivos de báscula | Genera valores nulos o subestimación de kilos |
| Precio definido por cultivo y calidad | `precios.csv` | Una combinación incompleta puede duplicar filas e inflar ingresos |

**Pendiente de la consigna original:** registrar el **número de línea exacto** de cada observación en el CSV. PC1 identifica los archivos y efectos, pero no contiene dichas líneas.

### 2.3 Anclas PC1

`dim_finca`: **4**; `dim_cultivo`: **6**; `dim_tiempo`: **730**; `seguridad`: **10**; carpeta de báscula: **19 archivos**. Todas figuran conciliadas en el PC1.

### 2.4 Preguntas ciegas PC1

- **L1:** **53 cosechas distintas** sumando báscula y báscula digital, tras excluir el efecto de archivos duplicados.
- **L2:** **110.000 kg** de meta anual 2026, según la fila de total y columna consolidada de `metas_planeacion.csv`. **Celda exacta:** no registrada en PC1.
- **L3:** `regional.sur@agrodb.test`, con **2 fincas**. **Discrepancia documental:** PC1 dice IDs **2 y 3**; una captura posterior de `seguridad` muestra IDs **3 y 4**. Debe prevalecer la versión del CSV/modelo que se entregue; el acceso a **2 fincas** sí fue probado con RLS.

**Evidencia:** `proyecto-pc1-modelo.png`.

---

## 3. PC2 — Integración y depuración de cosechas

### 3.1 Construcción de `h_cosecha`

Se combinaron los CSV mensuales de `bascula/`, `bascula_digital_2026.csv` y `precios.csv`. La tabla resultante integra `cosecha_id`, `finca_id`, `cultivo_id`, `fecha`, `fecha_entrega`, `calidad`, `kg` y `precio_kg`.

**Controles sin filtros:** **53 cosechas**, **0 repetidas**, **136.600 kg**.

### 3.2 Preguntas ciegas PC2

- **M1:** agosto 2025: **3.450 kg**, en **2 cosechas**.
- **M2:** septiembre 2026: **4 cosechas**, **10.500 kg**.
- **M3:** cierre 2026 por finca: La Unión **4.750 kg**; El Guayabo **22.800 kg**; Los Ceibos **43.500 kg**; Santa Rosa **20.250 kg**.
- **M4:** ingresos de enero–septiembre 2026: **$4.716.650,00**.
- **M5:** primera carga: se documentaron valores `null` en `kg` por diferencias con `kg_neto` y **5 errores de conversión de fecha**. **No se dispone del conteo exacto de filas de la primera carga**; no se inventa.

### 3.3 Correcciones verificadas

**Kilos faltantes.** En Power Query, las cosechas **16 y 17** mostraban `null` en `kg`, pero **2.200** y **1.250** en `kg_normalizado`. La normalización recuperó **3.450 kg**. El control histórico final fue **136.600 kg**. La cifra previa de **133.150 kg** se utiliza como comparación documentada en el trabajo; conviene contrastarla con la evidencia si se presenta como medición original.

**Fechas digitales.** Cinco fechas generaban errores de conversión; un ejemplo es `07/15/2026`, formato `MM/DD/YYYY`. Se interpretaron con configuración regional **Inglés (Estados Unidos)**. El control posterior de septiembre mostró **4 cosechas y 10.500 kg**.

**Evidencias:** `proyecto-pc2-hechos.png` y `proyecto-pc2-pasos.png`.

---

## 4. PC3 — Metas, relaciones y medidas

### 4.1 Anclas PC3

| Control | Resultado |
|---|---:|
| Filas de `h_meta` | 48 |
| Meta anual 2026 | 110.000 kg |
| Cumplimiento enero–septiembre 2026 | 105,92 % |

Se relacionaron las tablas de hechos con las dimensiones. La relación de `dim_tiempo[fecha]` con `h_cosecha[fecha]` es activa y la de `fecha_entrega` se utiliza como relación inactiva mediante `USERELATIONSHIP`.

### 4.2 Preguntas ciegas PC3

**X1. Cumplimiento por finca:** La Unión **76,61 %**; El Guayabo **108,57 %**; Los Ceibos **108,75 %**; Santa Rosa **106,58 %**.

**X2. Kilos entregados:** **82.800 kg** frente a **91.300 kg** cosechados; diferencia **8.500 kg**. Los cortes temporales son distintos: una medida filtra por `fecha` y la otra por `fecha_entrega`. **Pendiente:** enumerar los `cosecha_id` concretos que explican la diferencia, tal como solicita la pregunta.

**X3. Variación interanual:** **128,25 %**. El mismo período de 2025 registró **40.000 kg**, frente a **91.300 kg** en 2026: **+51.300 kg**. No se debe atribuir toda la diferencia a crecimiento orgánico porque Los Ceibos se incorporó en 2026.

**X4. Promedio por cultivo con cosechas:** **18.260 kg** durante enero–septiembre de 2026, **sin filtro de tipo**. Se corrigió la medida original `DIVIDE([Kilos],[Cosechas])`, que calculaba promedio **por cosecha**, sustituyéndola por `AVERAGEX` sobre los cultivos con cosechas. El total de **91.300 kg** corresponde a **5 cultivos productivos**. Con `tipo = perenne`, la tarjeta mostró **19.450 kg**; ese valor no sustituye la respuesta general de X4.

**X5. Meta por cultivo:** las metas están definidas por finca y mes, **no por cultivo**. No se debe atribuir a cada cultivo una meta que no existe. El PC3 documenta una protección en `[Cumplimiento]` para mostrar `BLANK()` cuando `dim_cultivo[cultivo]` está en contexto; **queda pendiente verificar que `[Meta]` también se comporte como exige la pregunta**, ya que ocultar cumplimiento no oculta automáticamente meta.

### 4.3 Bitácora PC3

| Qué encontré | Cómo me di cuenta | Qué número daba mal | Cómo lo arreglé | Cómo sé que quedó |
|---|---|---|---|---|
| Segmentadores no afectaban la tarjeta en otra página | La tarjeta conservaba el valor global | 124,18 % en vez de 105,92 % | Ajusté interacciones/sincronización de segmentadores | La tarjeta mostró 105,92 %, 91.300 kg y meta 86.200 kg |
| Fechas de entrega digitales requerían región | Errores en `fecha_entrega` | Errores de conversión | Convertí con región Inglés (Estados Unidos) | `[Kilos entregados]` mostró 82.800 kg al cierre |
| Relación temporal impedía comparar año anterior | Error de selección de fechas no contiguas | Variación no calculable | Ajusté dirección de filtro a única | Enero 2026: 900 kg año anterior y 588,89 % variación |
| Promedio estaba calculado por cosecha | Fórmula `DIVIDE([Kilos],[Cosechas])` | 3.148,28 kg en vez de promedio por cultivo | Sustituí por `AVERAGEX` de cultivos con cosechas | 18.260 kg sin filtro de tipo |

**Evidencias:** `proyecto-pc3-modelo.png` y `proyecto-pc3-control.png`.

---

## 5. PC4 — Análisis y seguridad

### 5.1 Ancla PC4

Con **Ver como** `regional.sur@agrodb.test`: **2 fincas**, **104,44 %** de cumplimiento total; **48.250 kg** cosechados y **46.200 kg** de meta.

### 5.2 Preguntas ciegas PC4

**J1. Podio perenne (2026, enero–septiembre):**

| Lugar | Cultivo | Kilos |
|---:|---|---:|
| 1 | Banano | 43.500 |
| 2 | Mango | 17.500 |
| 3 | Guayaba | 12.050 |
| **Total** | | **73.050** |

**J2. Bono y apoyo por finca:**

| Finca | Bono | Apoyo |
|---|---:|---:|
| Agrícola La Unión | 0 | 1.450 |
| Finca El Guayabo | 1.800 | 0 |
| Hacienda Los Ceibos | 3.500 | 0 |
| Hacienda Santa Rosa | 1.250 | 0 |
| **Total** | **6.550** | **1.450** |

Si se calcula incorrectamente sobre la empresa agregada: **bono 5.100 kg**, **apoyo 0 kg**.

**J3. Escenario 110 %:** meta ajustada **94.820 kg**, cumplimiento ajustado **96,29 %**, **0 fincas** que cumplen. El selector funciona entre **100 % y 150 %** en incrementos de diez puntos.

**J4. KPI anual acumulado:** valor **91.300 kg**, objetivo **86.200 kg**, diferencia **+5.100 kg** y **+5,92 %**. Título del visual: *KPI de cosecha vs. meta acumulada — enero a septiembre 2026*.

**J5. RLS:** `gerente.launion@agrodb.test` ve La Unión: **4.750 kg**, meta **6.200 kg**, **76,61 %**, rojo. `practicante@agrodb.test` no ve datos empresariales. `regional.sur@agrodb.test` ve dos fincas y **104,44 %**.

### 5.3 Bitácora PC4

| Qué encontré | Cómo me di cuenta | Qué número daba mal | Cómo lo arreglé | Cómo sé que quedó |
|---|---|---|---|---|
| Podio en barras 100 % y filtro por cultivo individual | Eje 0–100 % y segmentador con nombres de cultivos | No se podían leer kilos del podio | Barras agrupadas, segmentador por `tipo`, Top N=3 | Perenne: Banano 43.500, Mango 17.500, Guayaba 12.050 |
| Fincas que cumplen aparecía en blanco | Escenario 110 % con tarjeta vacía | `BLANK()` en vez de 0 | `COALESCE(COUNTROWS(...),0)` | Tarjeta 0; cumplimiento ajustado 96,29 % |
| RLS provocaba ambigüedad de relaciones | Error `Join paths are expected to form a tree` | Meta y cumplimiento no visibles bajo RLS | Cambié relación de `Both` a `Single` | Regional: 2 fincas, meta 46.200, 104,44 % |
| Prueba de identidad no coincidía | Se usó `regional.sur` en vez del correo completo | 0 fincas en simulación | Usé `regional.sur@agrodb.test` | 2 fincas visibles |
| Usuario sin autorización debía quedar bloqueado | Prueba `Ver como` practicante | Riesgo de mostrar datos | Regla única basada en `seguridad` | Sin datos visibles |

**Evidencias:** `proyecto-pc4-gerente.png`, `proyecto-pc4-regional.png`, `proyecto-pc4-practicante.png`.

---

## 6. Auditoría del tablero final

Las tres páginas finales son **Resumen**, **Análisis** y **Auditoría**. Los filtros de año y mes se comprobaron, y la simulación RLS restringió los datos en las tres páginas.

La página Auditoría contiene tarjetas de registros, kilos, meta y diferencia; una tabla de controles de calidad; cumplimiento por finca; identidad y fincas visibles; y detalle individual.

**Prueba de enero de 2026:** **2 registros**, **6.200 kg**, **7.200 kg** de meta y **−1.000 kg** de diferencia. Las tres validaciones mostraron **0** en el contexto probado:
- Registros sin fecha.
- Kilos inválidos (nulos, cero o negativos).
- Registros sin finca.

Estos ceros corresponden al **contexto filtrado de la prueba**, no garantizan por sí solos ausencia de errores en todos los períodos.

**Capturas finales:**
- `proyecto-final-1-resumen.png`
- `proyecto-final-2-analisis.png`
- `proyecto-final-3-auditoria.png`

---

## 7. Medidas DAX y preguntas que responden

> Se transcriben fórmulas documentadas en PC1–PC4 o confirmadas en el trabajo posterior. **Las definiciones que no se han copiado directamente del PBIX se identifican para cotejo**. Para cumplir literalmente la rúbrica, revisar el panel de medidas del PBIX y agregar cualquier medida no incluida.

### 7.1 Producción e ingresos

**¿Cuántos kilos se cosecharon?**
```dax
Kilos = SUM(h_cosecha[kg])
```

**¿Cuántos registros de cosecha existen?**
```dax
Cosechas = COUNTROWS(h_cosecha)
```

**¿Cuántos identificadores de cosecha están repetidos?**
```dax
Cosechas repetidas =
COUNTROWS(h_cosecha) - DISTINCTCOUNT(h_cosecha[cosecha_id])
```

**¿Cuánto ingreso generan los kilos y precios registrados?**
```dax
Ingresos =
SUMX(
    h_cosecha,
    h_cosecha[kg] * h_cosecha[precio_kg]
)
```

### 7.2 Metas y comparación temporal

**¿Cuántos kilos se planificaron?**
```dax
Meta = SUM(h_meta[kg_meta])
```
**Advertencia X5:** confirmar comportamiento por cultivo; la meta no está definida a ese nivel.

**¿Qué porcentaje de la meta se cumplió?** (fórmula documentada en PC3)
```dax
Cumplimiento =
IF(
    ISINSCOPE(dim_cultivo[cultivo]),
    BLANK(),
    DIVIDE([Kilos], [Meta])
)
```

**¿Cuántos kilos se entregaron dentro del período, según fecha de entrega?**
```dax
Kilos entregados =
CALCULATE(
    [Kilos],
    USERELATIONSHIP(h_cosecha[fecha_entrega], dim_tiempo[fecha])
)
```

**¿Cuántos kilos se cosecharon en el mismo período del año anterior?**
```dax
Kilos año anterior =
CALCULATE(
    [Kilos],
    SAMEPERIODLASTYEAR(dim_tiempo[fecha])
)
```

**¿Cuál es la variación relativa frente al año anterior?**
```dax
Variacion =
DIVIDE([Kilos] - [Kilos año anterior], [Kilos año anterior])
```

**¿Cuál es el promedio entre los cultivos que efectivamente cosecharon?** (fórmula corregida y probada)
```dax
Promedio por cultivo =
AVERAGEX(
    FILTER(
        VALUES(dim_cultivo[cultivo_id]),
        [Cosechas] > 0
    ),
    [Kilos]
)
```

**¿Cuál es la diferencia de kilos frente a la meta?** (cotejar fórmula exacta en PBIX)
```dax
Diferencia Kilos = [Kilos] - [Meta]
```

### 7.3 Bono, apoyo y semáforo

**¿Cuánto excede cada finca su meta, sumado sin compensar déficits?**
```dax
Bono =
SUMX(
    VALUES(dim_finca[finca_id]),
    MAX(0, [Kilos] - [Meta])
)
```

**¿Cuánto falta por finca para llegar a la meta?**
```dax
Apoyo =
SUMX(
    VALUES(dim_finca[finca_id]),
    MAX(0, [Meta] - [Kilos])
)
```

**¿Qué color corresponde al cumplimiento de cada finca?**
```dax
Color Semaforo =
SWITCH(
    TRUE(),
    ISBLANK([Meta]), BLANK(),
    [Cumplimiento] >= 1, "#22A06B",
    "#E5484D"
)
```

### 7.4 Escenario de meta

**¿Qué niveles hipotéticos puede elegir la dirección?** (tabla calculada desconectada)
```dax
Escenario Meta = GENERATESERIES(100, 150, 10)
```

**¿Cuál sería la meta con el porcentaje seleccionado?**
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
```

**¿Qué porcentaje se cumple bajo la meta hipotética?**
```dax
Cumplimiento ajustado =
DIVIDE([Kilos], [Meta ajustada])
```

**¿Cuántas fincas cumplen su meta hipotética?**
```dax
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

### 7.5 KPI acumulado

**¿Cuántos kilos se acumularon hasta la fecha visible?**
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
```

**¿Cuál es la meta acumulada hasta esa fecha?**
```dax
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

### 7.6 Validación de datos

**¿Cuántos registros no tienen fecha?**
```dax
Registros Sin Fecha =
COUNTROWS(
    FILTER(h_cosecha, ISBLANK(h_cosecha[fecha]))
) + 0
```

**¿Cuántos registros tienen kilos nulos o no positivos?**
```dax
Kilos Invalidos =
COUNTROWS(
    FILTER(
        h_cosecha,
        ISBLANK(h_cosecha[kg]) || h_cosecha[kg] <= 0
    )
) + 0
```

**¿Cuántos registros no tienen finca asignada?**
```dax
Registros Sin Finca =
COUNTROWS(
    FILTER(h_cosecha, ISBLANK(h_cosecha[finca_id]))
) + 0
```

**¿Qué resultado corresponde a la validación seleccionada?**
```dax
Resultado Validacion =
SWITCH(
    SELECTEDVALUE('Validaciones Auditoria'[Validacion]),
    "Registros sin fecha", [Registros Sin Fecha],
    "Kilos invalidos", [Kilos Invalidos],
    "Registros sin finca", [Registros Sin Finca],
    BLANK()
)
```

**¿Cuál es el estado de una validación?**
```dax
Estado Validacion =
VAR Errores = [Resultado Validacion]
RETURN
    SWITCH(
        TRUE(),
        ISBLANK(Errores), "Sin datos",
        Errores = 0, "Correcto",
        Errores > 0, "Revisar",
        "Sin datos"
    )
```

**¿Qué color representa el estado?**
```dax
Color Estado Validacion =
SWITCH(
    [Estado Validacion],
    "Correcto", "#22A06B",
    "Revisar", "#E5484D",
    "Sin datos", "#808080",
    "#808080"
)
```

**¿Cuántos registros aparecen en el período filtrado?** (cotejar definición del PBIX)
```dax
Total Registros = COUNTROWS(h_cosecha)
```

### 7.7 Seguridad e identidad

**¿Qué usuario se está simulando o consultando?**
```dax
Usuario Actual = USERPRINCIPALNAME()
```

**¿Cuántas fincas son visibles en el contexto actual?**
```dax
Fincas Visibles =
DISTINCTCOUNT(dim_finca[finca_id]) + 0
```

### 7.8 Elementos adicionales por cotejar

En las capturas del panel de campos aparecen, entre otras, las medidas **`Prueba Auditoria`** y **`Prueba Seguridad`**, además de posibles medidas de diagnóstico. Si siguen en el PBIX final, agregar su fórmula y propósito o retirarlas del modelo si eran pruebas temporales y ya no se necesitan. Revisar también si existen medidas adicionales ocultas o de ranking.

---

## 8. Seguridad RLS: rol único `AccesoFincas`

**Tabla de aplicación:** `dim_finca`.  
**Pregunta:** ¿Qué fincas puede ver el correo conectado según las asignaciones de `seguridad`?

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

El rol solo habilita fincas cuyo par **correo + finca_id** existe en `seguridad`. La validación se realizó con **Modelado → Ver como**, no únicamente con una tarjeta de usuario. Se corrigió una relación bidireccional que generaba ambigüedad en la evaluación de metas bajo RLS.

| Usuario probado | Resultado |
|---|---|
| `regional.sur@agrodb.test` | 2 fincas, 48.250 kg, meta 46.200 kg, 104,44 % |
| `gerente.launion@agrodb.test` | Solo La Unión: 4.750 kg, meta 6.200 kg, 76,61 % |
| `practicante@agrodb.test` | Sin datos autorizados |

---

## 9. Bitácora consolidada de los cuatro puntos de control

La tabla siguiente reúne los casos documentados. Las pruebas PC1 previas a la carga están en §2.2; las correcciones de PC2–PC4 están detalladas en §3.3, §4.3 y §5.3.

| Qué encontré | Cómo me di cuenta | Qué número daba mal | Cómo lo arreglé | Cómo sé que quedó |
|---|---|---|---|---|
| Diferencia `kg`/`kg_neto` | Cosechas 16 y 17 con `kg = null` y `kg_normalizado` de 2.200/1.250 | Faltaban 3.450 kg | Normalicé el peso en Power Query | Total histórico 136.600 kg |
| Formato estadounidense en fechas digitales | Cinco errores de conversión; ejemplo `07/15/2026` | Registros no se ubicaban correctamente | Conversión con región Inglés (Estados Unidos) | Septiembre: 4 cosechas, 10.500 kg |
| Filtros no afectaban tarjeta de cumplimiento | La tarjeta mantenía el total global | 124,18 % vs. 105,92 % | Ajusté segmentadores e interacciones | 105,92 % al cierre |
| Relación temporal ambigua | Error de fechas no contiguas | Variación no calculable | Filtro de relación en dirección única | Enero 2026: 588,89 % |
| Podio no mostraba kilos reales | Barras 100 % y filtro por cultivo | Eje porcentual, podio incorrecto | Barras agrupadas, filtro por tipo y Top N=3 | 73.050 kg en tres perennes |
| Escenario 110 % mostraba tarjeta vacía | `[Fincas que cumplen]` en blanco | BLANK() vs. 0 | COALESCE | 0 fincas; 96,29 % |
| RLS generaba rutas de unión ambiguas | `Join paths are expected to form a tree` | Meta/cumplimiento no visibles | Relación Both → Single | Regional 104,44 % |
| Prueba de RLS usó identificador incompleto | `regional.sur` mostró 0 | 0 fincas en prueba | Correo completo del registro | 2 fincas visibles |
| Promedio confundía cosechas y cultivos | `[Kilos]/[Cosechas]` | 3.148,28 kg por cosecha | AVERAGEX sobre cultivos con cosechas | 18.260 kg por cultivo |
| Auditoría devolvía BLANK() en validaciones | Tarjetas vacías aunque había registros | BLANK() en lugar de 0 | COUNTROWS(FILTER(...)) + 0 | Tres controles en 0 |

---

## 10. Entrega y evidencias

**Presentación ejecutiva:** `Proyecto_Final_Cortez_Axel.pptx` (máximo 10 láminas y 20 minutos con preguntas). Se recomienda cubrir cierre, tablero Resumen, tres hallazgos con recomendaciones, advertencia, dos filas de bitácora y Auditoría.

**Capturas finales obligatorias:**
- `proyecto-final-1-resumen.png`
- `proyecto-final-2-analisis.png`
- `proyecto-final-3-auditoria.png`

**Capturas de controles anteriores:**
- PC1: `proyecto-pc1-modelo.png`
- PC2: `proyecto-pc2-hechos.png`, `proyecto-pc2-pasos.png`
- PC3: `proyecto-pc3-modelo.png`, `proyecto-pc3-control.png`
- PC4: `proyecto-pc4-gerente.png`, `proyecto-pc4-regional.png`, `proyecto-pc4-practicante.png`

**Ruta solicitada:** `entregas/cortez-axel/`, mediante pull request. El `.pbix` se conserva para la demostración y **no se sube**.

### Comprobaciones aún pendientes

1. **X2:** identificar cosechas concretas que explican la diferencia entre kilos cosechados y entregados.
2. **X5:** verificar la medida `[Meta]` por cultivo y dejar en blanco metas no asignadas a cultivos.
3. **PC1:** registrar líneas exactas de las cinco observaciones y celda exacta de la meta anual si se requiere para revisión.
4. **M5:** recuperar el conteo exacto de filas en la primera carga, si existe evidencia.
5. **Medidas:** contrastar con el PBIX las fórmulas indicadas como «cotejar» y cualquier medida adicional no documentada.
6. **L3:** resolver la diferencia entre los IDs de finca consignados en PC1 y los observados en la tabla `seguridad` del modelo final.

---

## 11. Conclusión

El tablero AgroDB Integrator integra cosechas, precios, metas, calendario y permisos por finca para presentar el cierre agrícola de septiembre de 2026. Sus tres páginas permiten explicar resultados empresariales, detectar diferencias por finca y cultivo, verificar calidad de datos y restringir el acceso mediante RLS. La bitácora documenta las correcciones realizadas y los controles utilizados para comprobarlas.
