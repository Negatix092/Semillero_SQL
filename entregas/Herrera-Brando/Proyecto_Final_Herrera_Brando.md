Proyecto Final – Cierre Septiembre 2026
Estudiante: Brando Herrera

Resumen ejecutivo
AgroDB cerró el período enero–septiembre 2026 con:

Kilos cosechados: 91.300
Meta acumulada: 86.200
Cumplimiento: 105,92 %
Ingresos: 47,17 mil
La empresa cumplió su meta acumulada del período y cerró con desempeño positivo frente a la planificación.

Hallazgo 1 – Producción líder
Hacienda Los Ceibos fue la finca con mayor producción durante el cierre.

43.500 kg
Representa el mayor aporte individual a los resultados de la empresa.

Recomendación
Mantener las prácticas operativas implementadas en la finca y monitorear los factores que impulsaron el rendimiento.

Hallazgo 2 – Cumplimiento por finca
La única finca que no alcanzó la meta en el período fue:

Agrícola La Unión
Cumplimiento: 76,61 %
Recomendación
Analizar causas operativas, disponibilidad productiva y planificación para el último trimestre.

Hallazgo 3 – Cultivos líderes
Podio de producción:

Puesto	Cultivo	Kilos
1	Banano	43.500
2	Mango	33.400
3	Maíz	25.200
Recomendación
Priorizar seguimiento comercial y de producción en los cultivos líderes debido a su impacto en el resultado global.

Advertencia para dirección
Los indicadores:

Kilos cosechados
y

Kilos entregados
no representan el mismo momento de negocio.

Kilos cosechados = 91.300
Kilos entregados = 137.000
La diferencia existe porque las fechas de corte y las fechas de entrega pertenecen a momentos diferentes del proceso productivo y financiero.

Indicadores principales
Kilos
Kilos =
SUM(h_cosecha[kg])
Meta
Meta =
SUM(h_meta[kg_meta])
Cumplimiento
Cumplimiento =
DIVIDE([Kilos],[Meta])
Ingresos
Ingresos =
SUMX(
    h_cosecha,
    h_cosecha[kg] * h_cosecha[precio_kg]
)
Kilos entregados
Kilos entregados =
CALCULATE(
    [Kilos],
    USERELATIONSHIP(
        h_cosecha[fecha_entrega],
        dim_tiempo[fecha]
    )
)
Kilos año anterior
Kilos año anterior =
CALCULATE(
    [Kilos],
    SAMEPERIODLASTYEAR(
        dim_tiempo[fecha]
    )
)
Variación
Variacion =
DIVIDE(
    [Kilos] - [Kilos año anterior],
    [Kilos año anterior]
)
Promedio por cultivo
Promedio por cultivo =
DIVIDE(
    [Kilos],
    DISTINCTCOUNT(h_cosecha[cultivo_id])
)
Análisis del cierre
Bono
Aplicado finca por finca:

35.100 kg
Apoyo
Aplicado finca por finca:

8.500 kg
Escenario 110 %
Meta ajustada = 121.000
Cumplimiento ajustado = 112,89 %
Seguridad dinámica (RLS)
Rol implementado mediante:

[correo] = USERPRINCIPALNAME()
Tabla utilizada:

seguridad
Validación realizada:

regional.sur@agrodb.test
Resultado:

2 fincas visibles
Cumplimiento = 104,44 %
Auditoría y calidad del modelo
Validaciones
Cosechas = 53
Cosechas repetidas = 0
Meta anual 2026 = 110.000
Filas h_meta = 48
Bitácora consolidada
Qué encontré	Cómo me di cuenta	Qué número afectaba	Cómo lo arreglé	Cómo validé
Archivo duplicado bascula_2026-03 - copia.csv	Revisión de Source.Name	Kilos y cosechas aumentaban	Archivo excluido	Cosechas repetidas = 0
Encabezado kg_neto	Calidad de columna	Kilos incompletos	Renombrado automático a kg	100 % válido
Unión incorrecta con precios	h_cosecha llegó a 106 filas	Kilos e ingresos duplicados	Merge por cultivo_id + calidad	h_cosecha = 53 filas
Fechas 2026 con errores	Registros fuera del calendario	Relación con dim_tiempo fallaba	Configuración regional correcta	Fechas válidas
RLS sin propagación	regional.sur veía todas las fincas	Seguridad incorrecta	Configuración de propagación de seguridad	2 filas visibles
Evidencias
Resumen
proyecto-final-1-resumen.png
Incluye:

Kilos
Meta
Cumplimiento
Ingresos
Tabla por finca
Kilos vs Meta
Análisis
proyecto-final-2-analisis.png
Incluye:

Podio
Comparación 2026 vs 2025
Bono
Apoyo
Escenario
Kilos entregados
Auditoría
proyecto-final-3-auditoria.png
Incluye:

53 cosechas
0 repetidas
110.000 kg meta anual
48 filas h_meta
Validación RLS
¿Por qué creerle a este tablero?
✅ Sin cosechas duplicadas
✅ Metas conciliadas con Planeación
✅ Seguridad por usuario validada
✅ Indicadores auditados
✅ Relaciones verificadas
✅ Trazabilidad documentada mediante bitácora
Conclusión
AgroDB alcanzó un cumplimiento de 105,92 % durante el cierre de enero a septiembre de 2026, superando la meta acumulada del período. El modelo incorpora controles de calidad, seguridad dinámica por usuario y auditorías de consistencia que respaldan la confiabilidad de los resultados presentados.