# Checkpoint 2: Modelo de datos y medidas DAX

*Autora:* Gimena Carbone  
*Herramienta:* Power BI Desktop  
*Archivo:* Carbone_Gimena_Checkpoint2.pbix

## Objetivo

Construir un modelo de datos con relaciones activas, una tabla calendario y cinco medidas DAX para analizar las ventas, el acumulado anual y la variación respecto del año anterior.

## 1. Archivo de partida

Se trabajó sobre el archivo Pipeline_ETL_Carbone_Gimena.pbix del módulo anterior, que contenía estas tablas:

- Fact_Ventas
- Dim_Clientes
- Dim_Productos
- Dim_Categoria

## 2. Relaciones entre tablas

Se revisó la relación existente entre productos y ventas y se incorporaron las relaciones de clientes, categorías y fechas.

| Tabla de origen | Columna | Tabla de destino | Columna |
| --- | --- | --- | --- |
| Dim_Clientes | id_cliente | Fact_Ventas | id_cliente |
| Dim_Productos | id_producto | Fact_Ventas | id_producto |
| Dim_Categoria | id_categoria | Dim_Productos | id_categoria |
| Dim_Fechas | Date | Fact_Ventas | fecha_venta |

Las relaciones se configuraron con:

- Cardinalidad uno a varios (1:*).
- Dirección de filtro cruzado única.
- Relación activa.

## 3. Tabla calendario

Se creó la tabla Dim_Fechas utilizando las fechas mínima y máxima de las ventas:

dax
Dim_Fechas = CALENDAR(
    MIN(Fact_Ventas[fecha_venta]),
    MAX(Fact_Ventas[fecha_venta])
)


Se agregaron las siguientes columnas calculadas, cada una por separado:

### Año

dax
Año = YEAR(Dim_Fechas[Date])


### Mes Número

dax
Mes Número = MONTH(Dim_Fechas[Date])


### Mes Nombre

dax
Mes Nombre = FORMAT(Dim_Fechas[Date], "MMMM")


### Trimestre

dax
Trimestre = "T" & QUARTER(Dim_Fechas[Date])


### Semana

dax
Semana = WEEKNUM(Dim_Fechas[Date])


Se marcó Dim_Fechas como tabla de fechas, seleccionando la columna Date.

También se ordenó Mes Nombre por Mes Número para mostrar los meses de enero a diciembre.

## 4. Tabla de medidas

Se creó la tabla _Medidas para organizar las cinco medidas DAX.

Después de crear la primera medida, se eliminó la columna vacía Columna1, dejando únicamente las medidas.

### Total Ventas

Suma los importes de la columna total_venta.

dax
Total Ventas = SUM(Fact_Ventas[total_venta])


### Ventas Online

Calcula las ventas del canal Online utilizando CALCULATE con un filtro.

dax
Ventas Online = CALCULATE(
    [Total Ventas],
    Fact_Ventas[canal] = "Online"
)


### Ventas YTD

Calcula las ventas acumuladas desde el comienzo del año hasta la fecha evaluada.

dax
Ventas YTD = TOTALYTD(
    [Total Ventas],
    Dim_Fechas[Date]
)


### Ventas LY

Calcula las ventas del mismo período del año anterior.

dax
Ventas LY = CALCULATE(
    [Total Ventas],
    SAMEPERIODLASTYEAR(Dim_Fechas[Date])
)


### % Crecimiento Anual

Calcula la variación de las ventas respecto del mismo período del año anterior.

Se utilizaron variables con VAR y la función DIVIDE. La medida se configuró con formato de porcentaje.

dax
% Crecimiento Anual =
VAR VentasActual = [Total Ventas]
VAR VentasAnterior = [Ventas LY]
RETURN
    DIVIDE(
        VentasActual - VentasAnterior,
        VentasAnterior
    )


## 5. Matriz de validación

Se creó la página Validación y se incorporó una matriz con esta configuración:

| Apartado | Campos |
| --- | --- |
| Filas | Mes Nombre |
| Columnas | Año |
| Valores | Total Ventas, Ventas YTD, Ventas LY y % Crecimiento Anual |

Los meses se ordenaron cronológicamente y los resultados quedaron separados por año.

## 6. Comprobación de resultados

Se verificaron los siguientes resultados en la matriz:

| Comprobación | Resultado |
| --- | --- |
| Enero de 2023 | Total Ventas y Ventas YTD coinciden en 2.967,50. |
| Febrero de 2023 | Ventas YTD es 4.984,50, correspondiente a enero más febrero. |
| Enero de 2024 | Total Ventas y Ventas YTD coinciden en 3.018,00. |
| Febrero de 2024 | Ventas YTD es 3.967,00, correspondiente a enero más febrero. |
| Ventas LY de enero de 2024 | Muestra 2.967,50, correspondiente a enero
