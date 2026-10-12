# Ejercicio 27 · AgroDB

## PARTE A - EL MODELO

**A0a.** Antes de cambiarla, la configuración regional estaba en **Español (España)** [o la que tenga el sistema por defecto], lo cual interpreta las fechas como DD/MM/AAAA.

---

## PARTE B - EL ARCHIVO DE LA BÁSCULA

**B2.** Los pasos automáticos que creó Power Query fueron:
1. `Origen`
2. `Encabezados promovidos`
3. `Tipo cambiado`

**B3.** El ícono junto a la columna `fecha` es un **calendario**, indicando que Power Query detectó y aplicó el tipo Fecha en su paso automático, pero falló en algunas filas.

**B4. Calidad de columna:**
* Válido: **70 %**
* Error: **30 %**
* Vacío: **0 %**

**B5.** Mensaje de error textual: `DataFormat.Error: No pudimos analizar la entrada proporcionada como un valor Date.`

**B6.** Las tres filas con error tienen los `cosecha_id` **27, 28 y 35**. Lo que tienen en común es que sus fechas (`05/14/2026`, `05/20/2026` y `08/24/2026`) tienen números mayores a 12 en la posición central. Como Power Query intentaba leerlas como DD/MM/AAAA, falló al buscar el mes 14, 20 y 24.

---

## PARTE C - EL ARREGLO OBVIO

**C1.** Tras aplicar `Quitar errores`, la calidad de columna dice: **100 % válido / 0 % error / 0 % vacío**.

**C5.** Falta el mes de **Agosto**.

**C7.** La tabla ya no está en 30 550. Ahora el total dice **30 950** kilos, con un cumplimiento del **126,64 %** y **10** cosechas. La finca que cambió fue **Hacienda Santa Rosa** (su `[Kilos]` aumentó).

**C8.** La cosecha que apareció erróneamente en enero-abril es la **31**. En el archivo trae la fecha `07/02/2026` (que en realidad es el 2 de julio), pero Power Query la leyó al revés y la puso en el **7 de febrero**.

---

## PARTE D - LA REGLA EN EL PASO

**D1. Tabla de lecturas de fechas:**

| `cosecha_id` | Texto en el CSV | Es (mes/día) | Se leyó (día/mes) |
|---|---|---|---|
| 26 | `05/07/2026` | 7 de mayo | 5 de julio |
| 27 | `05/14/2026` | 14 de mayo | Error |
| 28 | `05/20/2026` | 20 de mayo | Error |
| 29 | `06/06/2026` | 6 de junio | 6 de junio |
| 30 | `06/09/2026` | 9 de junio | 6 de septiembre |
| 31 | `07/02/2026` | 2 de julio | 7 de febrero |
| 32 | `07/10/2026` | 10 de julio | 7 de octubre |
| 33 | `08/05/2026` | 5 de agosto | 8 de mayo |
| 34 | `08/06/2026` | 6 de agosto | 8 de junio |
| 35 | `08/24/2026` | 24 de agosto | Error |

* ¿Cuántas quedaron al revés sin marcar error? **Seis fechas** (26, 30, 31, 32, 33, 34).
* ¿Cuál salió bien? Solo la **29**, porque el día y el mes son idénticos (06/06), por lo que da igual si se lee al revés.

**D2.** Al borrar los pasos, Power Query avisó que `La eliminación de este paso puede afectar a los pasos posteriores...`.

**D7.** El total regresó exitosamente a **30 550 / 125,00 % / 9**.

**D8.** El Guayabo (finca 2) tenía asignadas las cosechas 27, 30, 32 y 34:
* `Quitar errores` eliminó completamente la **27**.
* Al leerse al revés, la **30** se fue a septiembre y la **32** se fue a octubre (fuera del rango de mayo-agosto).
* Solo le había quedado la **34** (leída como 8 de junio). Al aplicar la configuración regional correcta (Inglés EE. UU.), recuperó todas sus pesadas dentro del periodo, volviendo a 85,80 %.

---

## PARTE E - LA PRUEBA DE LAS CUATRO CIFRAS

### E1 · Las medidas
```dax
Pesadas bascula = CALCULATE( [Cosechas] , h_cosecha[cosecha_id] >= 26 )
Kilos bascula   = CALCULATE( [Kilos] , h_cosecha[cosecha_id] >= 26 )
Primera pesada  = CALCULATE( MIN( h_cosecha[fecha] ) , h_cosecha[cosecha_id] >= 26 )
Ultima pesada   = CALCULATE( MAX( h_cosecha[fecha] ) , h_cosecha[cosecha_id] >= 26 )}


```

**E3.** Con la versión rota de la Parte C habrían dicho:

* `Pesadas bascula`: **7** (porque `Quitar errores` borró 3 filas permanentemente).
* `Kilos bascula`: **13 200** (la suma de los kilos de las 7 pesadas sobrevivientes).
* `Primera pesada`: **7 de febrero** (la cosecha 31, `07/02/2026`, leída al revés).
* `Ultima pesada`: **7 de octubre** (la cosecha 32, `07/10/2026`, leída al revés).

**E4.** Si cambias la configuración global a Inglés, solucionaría este CSV, pero rompería los CSV anteriores que venían en español. El día que llegue un archivo con fechas como "5 de julio" (DD/MM), la configuración en Inglés lo leerá como el 7 de mayo (MM/DD), generando exactamente el mismo problema a la inversa.

**E5.** Atrapó la 31 porque cayó en **febrero** (adentro del filtro de meses 1 a 4), alterando el total. No habría atrapado la 26 porque Power Query la mandó al 5 de **julio**, mes que el segmentador estaba filtrando y ocultando.

---

## PARTE F - PREGUNTAS DE CIERRE

1. **¿Qué hace Quitar errores?** Elimina por completo la fila conflictiva de la tabla; **no** arregla el formato ni recupera el dato.
2. **¿Dónde vive la regla?** Vive exclusivamente en el **paso aplicado** de Power Query (`Tipo cambiado` con configuración regional).
3. **Regla de detección:** *"Si al cargar un archivo la calidad dice 100 % válido pero faltan registros o los kilos no cuadran con el correo original, Power Query eliminó filas o interpretó fechas al revés silenciosamente"*.
4. **¿Qué revisarías?** Revisaría la configuración regional del paso aplicado para asegurar que coincida con el origen (MM/DD/AAAA en EE. UU.). No basta la calidad de columna porque, si ningún día supera el 12, todas las fechas se leerán "válidamente" pero al revés (ej. 10 de enero como 1 de octubre).
5. **¿En qué se parece a la clase 12?** En que ambas "soluciones" barren el problema bajo la alfombra descartando la evidencia (filas rechazadas o borradas), en lugar de corregir la causa raíz del formato.

```
