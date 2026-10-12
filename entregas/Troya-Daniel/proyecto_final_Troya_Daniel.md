# Proyecto Final · Cierre Integral de Cosechas y Metas Agrícolas

**Estudiante:** Daniel Troya  
**Fecha:** Viernes 9 de octubre de 2026  
**Repositorio / Rama:** `entregas/troya-daniel/`  

---

## 1. Resumen Ejecutivo del Cierre (Enero - Septiembre 2026)

* **Cosecha Real acumulada:** **91 300,00 kg** (91,30 mil kg).
* **Meta Presupuestada acumulada:** **86 200,00 kg** (86,20 mil kg).
* **Cumplimiento Global de la Empresa:** **105,92 %** (+5,92 % por encima del objetivo planificado).
* **Ingresos Brutos Generados:** **$ 34 750,00 USD**.
* **Kilos Entregados en Destino:** **82 950,00 kg** (83 mil kg en visual).
* **Variación Interanual vs. Mismo Periodo 2025:** **+128,25 %** (comparado contra los 40 000 kg registrados a septiembre 2025).

---

## 2. Medidas DAX y Reglas de Seguridad

A continuación se presentan todas las medidas implementadas en el modelo tabular, indicando explícitamente qué requerimiento o pregunta ciega resuelve cada una:

### 2.1 Medidas Base de Auditoría y Hechos (Lunes / Martes)

```dax
-- Resuelve: Conteo de transacciones totales y validación del cierre 2026 (Ancla Martes = 29)
Cosechas = 
COUNTROWS( h_cosecha )
```

```dax
-- Resuelve: Auditoría de duplicados en la ingesta (Debe dar 0)
Cosechas repetidas = 
COUNTROWS( h_cosecha ) - DISTINCTCOUNT( h_cosecha[cosecha_id] )
```

```dax
-- Resuelve: Volumen cosechado en kilogramos netos
Kilos = 
SUM( h_cosecha[kg] )
```

```dax
-- Resuelve: Facturación bruta calculada a partir de la relación snowflake con precios
Ingresos = 
SUMX(
    h_cosecha,
    VAR _finca = RELATED( dim_finca[finca] )
    VAR _cultivo_id = h_cosecha[cultivo_id]
    VAR _calidad = h_cosecha[calidad]
    VAR _precio = 
        LOOKUPVALUE(
            precios[precio_kg],
            precios[cultivo_id], _cultivo_id,
            precios[calidad], _calidad
        )
    RETURN
    h_cosecha[kg] * _precio
)
```

---

### 2.2 Medidas de Planificación, Desempeño y Tiempo (Miércoles)

```dax
-- Resuelve: Planificación acumulada (Ancla Miércoles = 86 200 kg a Sep, 110 000 kg anual)
Meta = 
SUM( h_meta[kg_meta] )
```

```dax
-- Resuelve: Pregunta X1 y Ancla Miércoles (105,92 %). Protege contra contextos donde no existe meta
Cumplimiento = 
IF(
    ISBLANK( [Meta] ) || [Meta] = 0,
    BLANK(),
    DIVIDE( [Kilos], [Meta] )
)
```

```dax
-- Resuelve: Pregunta X2. Kilos efectivamente recibidos activando la relación secundaria
Kilos entregados = 
CALCULATE(
    [Kilos],
    USERELATIONSHIP( h_cosecha[fecha_entrega], dim_tiempo[fecha] )
)
```

```dax
-- Resuelve: Base para el cálculo de variación interanual
Kilos año anterior = 
CALCULATE(
    [Kilos],
    SAMEPERIODLASTYEAR( dim_tiempo[fecha] )
)
```

```dax
-- Resuelve: Pregunta X3. Crecimiento porcentual contra 2025 (+128,25 %)
Variacion = 
DIVIDE( [Kilos] - [Kilos año anterior], [Kilos año anterior] )
```

```dax
-- Resuelve: Pregunta X4. Promedio de kilos evaluando únicamente cultivos cosechados (18,26 mil kg)
Promedio por cultivo = 
AVERAGEX(
    KEEPFILTERS( VALUES( dim_cultivo[cultivo_id] ) ),
    [Kilos]
)
```

---

### 2.3 Medidas Analíticas Avanzadas y Simulación (Jueves)

```dax
-- Resuelve: Requerimiento 4.1 y Pregunta J1. Ranking dinámico de los 3 mejores cultivos visibles
Ranking Cultivo = 
IF(
    ISBLANK( [Kilos] ),
    BLANK(),
    RANKX(
        ALLSELECTED( dim_cultivo[cultivo] ),
        [Kilos],
        ,
        DESC,
        Dense
    )
)
```

```dax
-- Resuelve: Requerimiento 4.2 y Pregunta J2. Bono acumulado evaluado finca por finca (6 550 kg)
Bono = 
SUMX(
    VALUES( dim_finca[finca_id] ),
    VAR _k = [Kilos]
    VAR _m = [Meta]
    RETURN
    IF( _k > _m, _k - _m, 0 )
)
```

```dax
-- Resuelve: Requerimiento 4.2 y Pregunta J2. Apoyo compensatorio evaluado finca por finca (1 450 kg)
Apoyo = 
SUMX(
    VALUES( dim_finca[finca_id] ),
    VAR _k = [Kilos]
    VAR _m = [Meta]
    RETURN
    IF( _k < _m, _m - _k, 0 )
)
```

```dax
-- Resuelve: Requerimiento 4.3 y Pregunta J3. Meta recalculada según parámetro What-If (100% a 150%)
Meta Ajustada = 
VAR _factor = SELECTEDVALUE( 'Factor Meta'[Factor Meta], 1.0 )
RETURN
[Meta] * _factor
```

```dax
-- Resuelve: Requerimiento 4.3 y Pregunta J3. Cumplimiento con meta al 110% (96,29 %)
Cumplimiento Ajustado = 
IF(
    ISBLANK( [Meta Ajustada] ) || [Meta Ajustada] = 0,
    BLANK(),
    DIVIDE( [Kilos], [Meta Ajustada] )
)
```

```dax
-- Resuelve: Tarjeta de auditoría de seguridad en informe
Usuario Actual = 
USERPRINCIPALNAME()
```

---

### 2.4 Seguridad a Nivel de Fila (RLS)

* **Nombre del Rol:** `Seguridad_Fincas`
* **Tabla filtrada:** `seguridad`
* **Expresión DAX:**
  ```dax
  [correo] = USERPRINCIPALNAME()
  ```
* **Condición del Modelo:** Relación bidireccional (*Ambos*) entre `seguridad` y `dim_finca` con la opción *"Aplicar filtro de seguridad en ambas direcciones"* activada.

---

## 3. Bitácora Completa Consolidada (Lunes a Jueves)

| Qué encontré | Cómo me di cuenta | Qué número daba mal | Cómo lo arreglé | Cómo sé que quedó |
| :--- | :--- | :--- | :--- | :--- |
| **Columna espuria `Source.Name` y tipos como texto.** | Al revisar el esquema importado en Power Query; `kg` venía como texto y generaba errores de suma. | No permitía agregar numéricamente los kilos cosechados. | Se eliminaron metadatos innecesarios y se tipificó `kg` a Decimal y fechas a formato Date. | La suma total de kilos del histórico arrojó exactamente 136 600 kg. |
| **Cosechas con fecha futura e IDs duplicados.** | Auditoría de integridad mediante `[Cosechas repetidas]`. | La tarjeta de duplicados marcaba valores $> 0$ y fechas en noviembre 2026. | Se filtraron registros erróneos posteriores al corte de septiembre y duplicados por doble pesaje. | `[Cosechas repetidas]` quedó en 0 y el conteo de cosechas 2026 dio exactamente 29. |
| **Archivo `metas_planeacion.csv` en matriz ancha con totales redundantes.** | Inspección en Power Query: los meses estaban como columnas horizontales y existía fila/columna de "Total". | Duplicaba los kilos de meta y no permitía relacionar la fecha mensual con `dim_tiempo`. | Se filtró la fila `Total`, se quitaron las columnas redundantes y se aplicó *Anular dinamización de otras columnas* sobre `finca_id`. | La tabla `h_meta` quedó en 48 filas exactas (4 fincas $\times$ 12 meses) con `kg_meta` numérico. |
| **Pérdida de filtro temporal en `h_meta`.** | Al armar la matriz con meses 1 a 9, la columna Meta sumaba 110 000 kg en lugar de 86 200 kg. | `[Cumplimiento]` marcaba 83,00 % en lugar del 105,92 % esperado. | Se creó y activó la relación `1:*` entre `dim_tiempo[fecha]` y `h_meta[fecha_mes]`. | `[Meta]` bajó a 86 200 kg y el cumplimiento se situó en el valor ancla de 105,92 %. |
| **Desactivación involuntaria de la relación activa de `h_cosecha`.** | Al activar la relación de metas, la matriz mensual y la tabla por finca fijaron 136 600 kg en cada celda. | `[Cumplimiento]` saltó erróneamente a 158,47 % ($136\,600 / 86\,200$). | Se reactivó la relación directa `dim_tiempo[fecha] -> h_cosecha[fecha]`, dejando como inactiva la de entrega. | `[Kilos]` regresó a 91 300 kg y `[Cumplimiento]` retornó a 105,92 %. |
| **Regla de formato condicional leía porcentaje relativo en lugar de umbral absoluto.** | Al simular al gerente de La Unión ($76,61\,\%$), la celda aparecía en verde a pesar de estar incumpliendo la meta. | Semáforo visual en verde para una finca con déficit productivo. | Se cambiaron las unidades de las reglas de formato de "Porcentaje" a "Número" ($< 1$ rojo, $\ge 1$ verde). | La celda de Agrícola La Unión ($0,7661$) cambió inmediatamente a color rojo. |
| **Fuga de seguridad en simulación RLS en Power BI Desktop.** | Al usar *Ver como* con `regional.sur@agrodb.test`, se seguían viendo las 4 fincas y cumplimiento global. | El usuario regional visualizaba entidades fuera de su jurisdicción territorial. | Se habilitó la dirección de filtro cruzado en **Ambas** entre `seguridad` y `dim_finca`, y se marcó el checkbox del rol junto a "Otro usuario". | La vista se restringió a 2 fincas (*Agrícola La Unión* y *Hacienda Los Ceibos*) con cumplimiento exacto de 104,44 %. |

---

## 4. Guía Estructurada de Diapositivas (PowerPoint · Presentación a Dirección)

**Archivo:** `Proyecto_Final_Troya_Daniel.pptx` (Máximo 10 láminas, 1 número grande por lámina).

* **Lámina 1: El Cierre en una Línea**
  * *Número grande:* **105,92 %**
  * *Contenido:* Cierre acumulado Enero - Septiembre 2026: **91 300 kg** cosechados frente a una meta de **86 200 kg**.
* **Lámina 2: El Tablero Operativo**
  * *Contenido:* Captura de la página **Resumen** (`proyecto-final-1-resumen.png`).
* **Lámina 3: Hallazgo 1 · Asimetría Territorial (Déficit Oculto)**
  * *Número grande:* **-1 450 kg (76,61 %)**
  * *Explicación y Recomendación:* Aunque la empresa superó la meta global, **Agrícola La Unión** es la única finca con déficit crónico. Se recomienda auditoría de suelos y asistencia técnica inmediata en fertirriego.
* **Lámina 4: Hallazgo 2 · Desfase Logístico en Entregas**
  * *Número grande:* **8 350 kg**
  * *Explicación y Recomendación:* Existe una brecha de $8\,350\text{ kg}$ entre lo cosechado en campo ($91\,300\text{ kg}$) y lo recibido en centros de acopio ($82\,950\text{ kg}$) durante el periodo. Se debe auditar el tiempo de tránsito y capacidad de transporte en frío.
* **Lámina 5: Hallazgo 3 · Concentración Productiva en Perennes**
  * *Número grande:* **78 250 kg (85,7 %)**
  * *Explicación y Recomendación:* El podio (Banano, Palma Africana y Cacao) concentra casi el $86\,\%$ de la producción real de la empresa. Se recomienda evaluar rotación con cultivos transitorios para balancear el flujo estacional de caja.
* **Lámina 6: La Advertencia**
  * *Número grande:* **$ 5 100 \text{ vs } 6 550 \text{ kg}**
  * *Explicación:* **Cálculo de Bonos:** Si la dirección evalúa el bono a nivel empresa global pagaría solo $5\,100\text{ kg}$ absorbiendo indebidamente las pérdidas. La regla contractual exige evaluar finca por finca, lo que representa un desembolso real de **6 550 kg** en bonificaciones más **1 450 kg** en apoyos.
* **Lámina 7: Bitácora · Integridad de Ingesta**
  * *Número grande:* **48 Filas**
  * *Explicación:* Corrección del archivo matricial de metas en Power Query eliminando totales estáticos y dinamizando meses.
* **Lámina 8: Bitácora · Gobernanza Relacional y RLS**
  * *Número grande:* **104,44 %**
  * *Explicación:* Calibración de la propagación bidireccional en seguridad por fila para aislar el acceso de gerencias territoriales.
* **Lámina 9: Por Qué Creerle al Tablero (Auditoría)**
  * *Número grande:* **0 Cosechas Repetidas**
  * *Explicación:* Captura de la página **Auditoría** (`proyecto-final-3-auditoria.png`). Muestra reconciliación cruzada de balances, ausencia de duplicados e integridad referencial sin claves huérfanas.