# ecobici-powerbi
Análisis de viajes de Ecobici CDMX en Power BI

# Ecobici CDMX: ¿dónde y cuándo faltan bicicletas?

Análisis de los viajes de Ecobici en Power BI para identificar en qué
estaciones y horarios faltan o sobran bicicletas.

**Estado:** en construcción.

## Datos
Fuente: Datos abiertos de Ecobici (ecobici.cdmx.gob.mx/datos-abiertos)
Fuente: Diccionario de datos y cicloestaciones Ecobici (https://www.datos.cdmx.gob.mx/dataset/cicloestaciones-ecobici-nuevo-sistema)

- Periodo: enero a marzo de 2026
- Volumen: 4,707,285 viajes originales; 4,705,183 después de la limpieza

- Los archivos mensuales parecen organizarse por fecha de arribo, por lo que
  faltan los viajes que iniciaron el 31 de marzo y terminaron el 1 de abril.

## Limpieza de datos
Se eliminaron 2,102 registros (0.04% del total) con estas reglas:

| Regla | Registros eliminados |
| Duración menor a 1 minuto o mayor a 180 minutos | 1,737 |
| Estación de retiro o de arribo vacía | 0 |
| Edad fuera del rango de 16 a 90 años | 289 |
| Viajes iniciados el 31 de diciembre de 2025 | 76 |

Los viajes de menos de 1 minuto suelen ser bicicletas devueltas de inmediato,
y los de más de 180 minutos, bicicletas no ancladas correctamente.

## Limitaciones
- 40 viajes corresponden a estaciones que no aparecen en el catálogo
  de cicloestaciones y se muestran sin nombre.
- Los archivos mensuales parecen organizarse por fecha de arribo, por lo que
  faltan los viajes que iniciaron el 31 de marzo y terminaron el 1 de abril.

## Herramientas
Power BI Desktop, Power Query, DAX
