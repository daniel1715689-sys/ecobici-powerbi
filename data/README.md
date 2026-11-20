# Datos

Los archivos originales no se incluyen en el repositorio por su tamaño
(cada mes de viajes pesa cerca de 100 MB). Para reproducir el proyecto,
descárgalos y colócalos en las carpetas indicadas.

Fuente: [Datos abiertos de Ecobici](https://ecobici.cdmx.gob.mx/datos-abiertos/)

## 1. Histórico de viajes

1. Descarga los archivos de enero, febrero y marzo de 2026.
2. Colócalos en `data/raw/viajes/` con estos nombres:

```
data/raw/viajes/2026-01.csv
data/raw/viajes/2026-02.csv
data/raw/viajes/2026-03.csv
```

Power Query combina todos los CSV de esa carpeta, así que no pongas ahí
ningún otro archivo.

## 2. Catálogo de cicloestaciones

1. Descarga el catálogo de cicloestaciones. Viene en formato `.dbf`.
2. Ábrelo en Excel (Archivo → Abrir → tipo "Todos los archivos") y guárdalo
   como libro de Excel con el nombre `cicloestaciones_ecobici.xlsx`.
3. Colócalo en `data/raw/estaciones/`.

## 3. Conectar el reporte a tus archivos

El archivo de Power BI guarda la ruta de la computadora donde se creó. Al
abrirlo en otra computadora:

1. Abre `report/ecobici.pbix`.
2. Inicio → Transformar datos → **Configuración de origen de datos**.
3. Selecciona cada origen (la carpeta de viajes y el archivo de estaciones),
   pulsa **Cambiar origen** y elige tu ruta.
4. Cierra y pulsa **Actualizar**.