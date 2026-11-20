# Medidas DAX

Las 9 medidas principales del reporte y 5 de apoyo, guardadas en la tabla `Medidas` del modelo.
Los valores de validación corresponden al periodo completo
(enero a marzo de 2026), sin filtros.

## Resumen

| Medida | Qué responde | Formato |
| --- | --- | --- |
| Total Viajes | ¿Cuántos viajes hubo? | Entero |
| Viajes por Dia | ¿Cuántos viajes hay en un día típico? | Entero |
| Duracion Promedio (min) | ¿Cuánto dura un viaje? | 1 decimal |
| Salidas | ¿Cuántos viajes salieron de la estación? | Entero |
| Llegadas | ¿Cuántos viajes llegaron a la estación? | Entero |
| Balance Neto | ¿La estación gana o pierde bicicletas? | Entero |
| Viajes Mes Anterior | ¿Cuántos viajes hubo el mes previo? | Entero |
| Variacion % Mensual | ¿Creció o cayó el uso? | Porcentaje |
| % Viajes Entre Semana | ¿Qué parte del uso es de lunes a viernes? | Porcentaje |

## Total Viajes

Cuenta los renglones de `Viajes` que pasan los filtros activos. Es la base
de casi todas las demás medidas.

```dax
Total Viajes = COUNTROWS ( Viajes )
```

Validación: 4,704,622, igual al número de filas cargadas.

## Viajes por Dia

Calcula el total de cada día del calendario y promedia esos totales.

```dax
Viajes por Dia = AVERAGEX ( VALUES ( Calendario[Fecha] ), [Total Viajes] )
```

Validación: 52,274, que equivale a 4,704,622 viajes entre 90 días.

## Duracion Promedio (min)

Promedio de la duración de los viajes, en minutos.

```dax
Duracion Promedio (min) = AVERAGE ( Viajes[Duracion_Min] )
```

Validación: 15.16 minutos en el periodo completo.

## Salidas

Viajes que salieron de la estación. Es `Total Viajes` con otro nombre, porque
la relación activa entre `Estaciones` y `Viajes` es la de estación de retiro.

```dax
Salidas = [Total Viajes]
```

## Llegadas

Viajes que llegaron a la estación. `USERELATIONSHIP` activa, solo dentro de
esta medida, la relación inactiva por estación de arribo.

```dax
Llegadas =
CALCULATE (
    [Total Viajes],
    USERELATIONSHIP ( Estaciones[Estacion_ID], Viajes[Ciclo_EstacionArribo] )
)
```

Validación: el total es 4,704,622, igual al de `Salidas`, porque cada viaje
tiene una salida y una llegada.

## Balance Neto

Diferencia entre llegadas y salidas de una estación. Es la medida central del
reporte.

- Negativo: la estación pierde bicicletas (salen más de las que llegan).
- Positivo: la estación acumula bicicletas.

```dax
Balance Neto = [Llegadas] - [Salidas]
```

Validación: el total del sistema es 0.

Mide el flujo de viajes, no cuántas bicicletas hay en la estación.

## Viajes Mes Anterior

Viajes del mes previo al que está en contexto. `DATEADD` recorre las fechas
activas un mes hacia atrás y `CALCULATE` recalcula el total con esas fechas.

```dax
Viajes Mes Anterior =
CALCULATE ( [Total Viajes], DATEADD ( Calendario[Fecha], -1, MONTH ) )
```

Validación: en febrero muestra 1,524,681, el total de enero. En enero queda en
blanco porque no hay datos de diciembre.

Solo tiene sentido con un mes en el contexto (en un eje, una fila o un
segmentador). Requiere que `Calendario` esté marcada como tabla de fechas.

## Variacion % Mensual

Cambio porcentual frente al mes anterior. `DIVIDE` evita el error de dividir
entre cero y devuelve blanco cuando no hay mes anterior.

```dax
Variacion % Mensual =
DIVIDE ( [Total Viajes] - [Viajes Mes Anterior], [Viajes Mes Anterior] )
```

Validación: febrero +0.2%, marzo +8.1%.

Igual que la anterior, solo tiene sentido con un mes en el contexto.

## % Viajes Entre Semana

Proporción de viajes realizados de lunes a viernes.

```dax
% Viajes Entre Semana =
DIVIDE (
    CALCULATE ( [Total Viajes], Calendario[Tipo Dia] = "Entre semana" ),
    CALCULATE ( [Total Viajes], ALL ( Calendario[Tipo Dia] ) )
)
```

El denominador usa `ALL ( Calendario[Tipo Dia] )` para ignorar el filtro de
tipo de día. Sin él, al elegir "Fin de semana" en el segmentador la medida
dividía todos los viajes entre semana entre los de fin de semana y daba 398.4%.

Validación: 79.9%.

## Medidas de apoyo

Medidas adicionales para las páginas de estaciones, de detalle y de usuarios.

### Ranking Estacion

Lugar de cada estación según sus viajes; 1 es la más usada. `ALL` quita el
filtro de la estación actual para compararla contra todas.

```dax
Ranking Estacion =
RANKX ( ALL ( Estaciones[Nombre] ), [Total Viajes] )
```

Solo tiene sentido en un visual que incluya `Estaciones[Nombre]`.

### Estaciones en Deficit

Número de estaciones con balance neto negativo en el contexto actual.

```dax
Estaciones en Deficit =
COUNTROWS (
    FILTER ( VALUES ( Estaciones[Estacion_ID] ), [Balance Neto] < 0 )
)
```

Validación: 377 en el periodo completo. La cifra incluye el renglón en blanco
de las estaciones que no están en el catálogo, por lo que son 376 estaciones
con nombre.

### % del Total

Porcentaje de los viajes del sistema que salen de cada estación.

```dax
% del Total =
DIVIDE ( [Total Viajes], CALCULATE ( [Total Viajes], ALL ( Estaciones ) ) )
```

Validación: la columna suma 100%.

### Estacion Seleccionada

Nombre de la estación filtrada. Se usa como título en la página de detalle y
en el tooltip del mapa. Si hay más de una estación en el contexto, muestra
"Varias estaciones".

```dax
Estacion Seleccionada =
SELECTEDVALUE ( Estaciones[Nombre], "Varias estaciones" )
```

### % Viajes Mujeres

Proporción de viajes realizados por mujeres.

```dax
% Viajes Mujeres =
DIVIDE (
    CALCULATE ( [Total Viajes], Viajes[Genero_Usuario] = "F" ),
    [Total Viajes]
)
```

Validación: 27.7%.

## Columna calculada

### Ruta Nombre

Ruta con los nombres de las estaciones de origen y destino, en la tabla
`Viajes`. `RELATED` trae el origen por la relación activa; el destino se busca
con `LOOKUPVALUE` porque su relación está inactiva.

```dax
Ruta Nombre =
RELATED ( Estaciones[Nombre] ) & " → " &
LOOKUPVALUE ( Estaciones[Nombre], Estaciones[Estacion_ID], Viajes[Ciclo_EstacionArribo] )
```

Validación: las 10 rutas más usadas suman 17,340 viajes.