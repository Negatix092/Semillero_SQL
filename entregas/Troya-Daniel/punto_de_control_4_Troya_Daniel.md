# Punto de Control 4 · El análisis y la seguridad

**Estudiante:** Daniel Troya  
**Fecha:** Jueves 8 de octubre de 2026  

---

## 1. Anclas del jueves

| Validación / Vista simulada | Condición de filtro | Esperado | En pantalla | Estado |
| :--- | :--- | :---: | :---: | :---: |
| Vista general (sin RLS) | Periodo cierre (`anio = 2026`, meses `1 a 9`) | 4 fincas / 105,92 % | 4 fincas / 105,92 % | Cuadra |
| `regional.sur@agrodb.test` | Periodo cierre (`anio = 2026`, meses `1 a 9`) | **2 filas / 104,44 %** | **2 filas / 104,44 %** | **Cuadra** |
| `gerente.launion@agrodb.test` | Periodo cierre (`anio = 2026`, meses `1 a 9`) | 1 fila / 76,61 % | 1 fila / 76,61 % | Cuadra |
| `practicante@agrodb.test` | Periodo cierre (`anio = 2026`, meses `1 a 9`) | 0 filas / BLANK | 0 filas / BLANK | Cuadra |

---

## 2. Preguntas ciegas

* **J1. Con `tipo = perenne` marcado: ¿qué tres cultivos están en el podio, en qué lugar cada uno, y cuántos kilos suman?**
  * **1.er lugar:** Banano (Hacienda Los Ceibos / Hacienda Santa Rosa) con **31 500 kg**.
  * **2.º lugar:** Palma Africana con **26 500 kg**.
  * **3.er lugar:** Cacao con **20 250 kg**.
  * **Total acumulado del podio:** **78 250 kg**.
  * *Comportamiento dinámico:* La medida DAX evaluada con `ALLSELECTED(dim_cultivo[cultivo])` recalculó la jerarquía de 1 a 3 preservando el filtro de contexto de cultivos perennes sin romper la cardinalidad ante segmentaciones cruzadas.

* **J2. El bono total y el apoyo total de la empresa, en kilos. Y lo que diría cada uno si la regla se aplicara a la empresa entera en vez de finca por finca:**
  * **Cálculo real (Finca por finca - DAX iterativo `SUMX`):**
    * **Bono total empresa:** **6 550 kg**  
      *(Finca El Guayabo: $+1\,800\text{ kg}$; Hacienda Los Ceibos: $+3\,500\text{ kg}$; Hacienda Santa Rosa: $+1\,250\text{ kg}$).*
    * **Apoyo total empresa:** **1 450 kg**  
      *(Agrícola La Unión: déficit de $1\,450\text{ kg}$ respecto a su meta de $6\,200\text{ kg}$).*
  * **Cálculo consolidado global (Empresa entera sin granularidad):**
    * **Bono global:** **5 100 kg** ($91\,300\text{ kg cosecha} - 86\,200\text{ kg meta}$).
    * **Apoyo global:** **0 kg** (al superar la meta agregada en $5\,100\text{ kg}$, el déficit particular de Agrícola La Unión queda absorbido/compensado artificialmente).

* **J3. Con la meta al 110 %: `[Cumplimiento]` de la empresa, y cuántas fincas cumplen:**
  * **Meta global ajustada:** $86\,200 \times 1,10 = \mathbf{94\,820\text{ kg}}$.
  * **`[Cumplimiento Ajustado]` de la empresa:** **96,29 %** ($91\,300 / 94\,820$).
  * **Fincas que cumplen la nueva meta ($\ge 100\,\%$):** **0 fincas** (ninguna finca alcanza el 110 % de su presupuesto base individual: Agrícola La Unión 69,65 %, Hacienda Santa Rosa 96,89 %, Finca El Guayabo 98,70 % y Hacienda Los Ceibos 98,86 %).

* **J4. Lo que enseña el KPI del año: valor, objetivo y la diferencia en porcentaje:**
  * **Valor actual (Kilos mes 9 - Septiembre):** **10,50 mil kg** ($10\,500\text{ kg}$).
  * **Objetivo (Meta mes 9 - Septiembre):** **8,60 mil kg** ($8\,600\text{ kg}$).
  * **Diferencia porcentual:** **+22,09 %** (indicador visual en estado verde por superar el umbral de entrega mensual).
  * *Título visual configurado:* `"Cierre Enero - Septiembre 2026"`.

* **J5. Ver como `gerente.launion@agrodb.test`: kilos, meta, cumplimiento y color. Y ver como `practicante@agrodb.test`:**
  * **`gerente.launion@agrodb.test`:**
    * Kilos: **4 750 kg**
    * Meta: **6 200 kg**
    * Cumplimiento: **76,61 %**
    * Semáforo / Color condicional: **Rojo** (por encontrarse por debajo del umbral del 100 %).
  * **`practicante@agrodb.test`:**
    * Al no existir el correo dentro del registro de asignaciones territoriales en la tabla `seguridad`, la política de seguridad RLS restringe por completo el contexto de evaluación:
    * Filas en tabla por finca: **0 filas**.
    * Medidas de tarjeta y tablas: **En blanco (`BLANK`)**.

---

## 3. Fórmulas DAX implementadas

### 4.1 Podio dinámico Top 3
```dax
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

### 4.2 Bono y Apoyo iterativo por entidad territorial
```dax
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
Apoyo = 
SUMX(
    VALUES( dim_finca[finca_id] ),
    VAR _k = [Kilos]
    VAR _m = [Meta]
    RETURN
    IF( _k < _m, _m - _k, 0 )
)
```

### 4.3 Parámetro de sensibilidad de Meta
```dax
Meta Ajustada = 
VAR _factor = SELECTEDVALUE( 'Factor Meta'[Factor Meta], 1.0 )
RETURN
[Meta] * _factor
```

```dax
Cumplimiento Ajustado = 
IF(
    ISBLANK( [Meta Ajustada] ) || [Meta Ajustada] = 0,
    BLANK(),
    DIVIDE( [Kilos], [Meta Ajustada] )
)
```

### 4.5 Seguridad a Nivel de Fila (RLS)
*Aplicada sobre la tabla `seguridad`:*
```dax
[correo] = USERPRINCIPALNAME()
```

---

## 4. Bitácora técnica del día

| Qué encontré | Cómo me di cuenta | Qué número daba mal | Cómo lo arreglé | Cómo sé que quedó |
| :--- | :--- | :--- | :--- | :--- |
| Parámetro What-If bloqueaba la entrada de rangos con decimales ($1.0$ a $1.5$). | El cuadro emergente de parámetros arrojaba error de validación en rojo al intentar colocar $1.5$ o saltos de $0.1$. | No permitía crear el segmentador de incremento decimal. | Se cambió el tipo de dato subyacente en la interfaz a Número Decimal con escala $1$ a $1,5$ (o escala porcentual entera $100$ a $150$ con paso $10$). | Se creó correctamente el segmentador y la medida responde sin interpolaciones indebidas. |
| El visual de KPI mostraba `Objetivo: (En blanco) (+Infinity %)`. | En el canvas de reporte, el KPI arrojaba infinito y no leía la meta acumulada del mes. | El cálculo comparativo de tendencia de cierre no graficaba el target. | Se retiró `dim_tiempo[fecha]` (diaria) del eje de tendencia y se asignó `dim_tiempo[mes]`, alineando la granularidad temporal donde existe `[Meta]` (día 1 de mes). | El KPI adoptó de inmediato el valor de septiembre ($10,50\text{ mil}$ vs $8,60\text{ mil}$) marcando $+22,09\,\%$. |
| El filtro de RLS no limitaba los datos al simular usuario en Power BI Desktop. | Al usar *Ver como* con `regional.sur@agrodb.test`, se continuaban visualizando las 4 fincas y el total del 105,92 %. | No se restringía la seguridad por rol corporativo. | Se habilitó la dirección de filtro cruzado en **Ambas** en la relación entre `seguridad` y `dim_finca`, y se marcó la casilla del rol de seguridad simultáneamente con "Otro usuario". | La vista filtró de inmediato a exactamente 2 fincas (*Agrícola La Unión* y *Hacienda Los Ceibos*) con cumplimiento total de **104,44 %**. |