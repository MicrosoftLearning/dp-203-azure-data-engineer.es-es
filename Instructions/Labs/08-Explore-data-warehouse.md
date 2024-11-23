---
lab:
  title: Exploración de un almacenamiento de datos relacional
  ilt-use: Suggested demo
---

# Exploración de un almacenamiento de datos relacional

Azure Synapse Analytics se basa en un conjunto de funcionalidades escalable para admitir el almacenamiento de datos empresariales; incluidos el análisis de datos basados en archivos en un lago de datos, así como almacenes de datos relacionales a gran escala y las canalizaciones de transferencia y transformación de datos usadas para cargarlos. En este laboratorio, explorarás cómo usar un grupo de SQL dedicado en Azure Synapse Analytics para almacenar y consultar datos en un almacenamiento de datos relacional.

Este laboratorio tardará aproximadamente **45** minutos en completarse.

## Antes de empezar

Necesitará una [suscripción de Azure](https://azure.microsoft.com/free) en la que tenga acceso de nivel administrativo.

## Aprovisionar un área de trabajo de Azure Synapse Analytics

Un *área de trabajo* de Azure Synapse Analytics proporciona un punto central para administrar datos y entornos de ejecución de procesamiento de datos. Puedes aprovisionar un área de trabajo mediante la interfaz interactiva en Azure Portal, o bien, puedes implementar un área de trabajo y recursos dentro de él mediante un script o una plantilla. En la mayoría de los escenarios de producción, es mejor automatizar el aprovisionamiento con scripts para poder incorporar la implementación de recursos en un proceso de desarrollo y operaciones repetibles (*DevOps*).

En este ejercicio, usarás una combinación de un script de PowerShell y una plantilla de ARM para aprovisionar Azure Synapse Analytics.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) en `https://portal.azure.com`.
2. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** y crear almacenamiento si se solicita. Cloud Shell proporciona una interfaz de línea de comandos en un panel situado en la parte inferior de Azure Portal, como se muestra a continuación:

    ![Azure Portal con un panel de Cloud Shell](./images/cloud-shell.png)

    > **Nota**: si creaste anteriormente un Cloud Shell que usa un entorno de *Bash*, usa el menú desplegable situado en la parte superior izquierda del panel de Cloud Shell para cambiarlo a ***PowerShell***.

3. Ten en cuenta que puedes cambiar el tamaño de Cloud Shell arrastrando la barra de separación en la parte superior del panel, o usando los iconos **&#8212;** , **&#9723;** y **X** en la parte superior derecha para minimizar, maximizar y cerrar el panel. Para obtener más información sobre el uso de Azure Cloud Shell, consulta la [documentación de Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. En el panel de PowerShell, introduce los siguientes comandos para clonar este repositorio:

    ```
    rm -r dp203 -f
    git clone  https://github.com/MicrosoftLearning/Dp-203-azure-data-engineer dp203
    ```

5. Una vez clonado el repositorio, escribe los siguientes comandos para cambiar a la carpeta de este laboratorio y ejecuta el script **setup.ps1** que contiene:

    ```
    cd dp203/Allfiles/labs/08
    ./setup.ps1
    ```

6. Si se te solicita, elige la suscripción que deseas usar (esto solo ocurrirá si tienes acceso a varias suscripciones de Azure).
7. Cuando se te solicite, escribe una contraseña adecuada que se va a establecer para el grupo de SQL de Azure Synapse.

    > **Nota**: Asegúrate de recordar esta contraseña.

8. Espera a que se complete el script: normalmente tarda unos 15 minutos, pero en algunos casos puede tardar más tiempo. Mientras esperas, revisa el artículo [¿Qué es un grupo de SQL dedicado en Azure Synapse Analytics?](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is) en la documentación de Azure Synapse Analytics.

## Exploración del esquema de almacenamiento de datos

En este laboratorio, el almacenamiento de datos se hospeda en un grupo de SQL dedicado en Azure Synapse Analytics.

### Inicio del grupo de SQL dedicado

1. Una vez completado el script, en Azure Portal, ve al grupo de recursos **dp203-*xxxxxxx*** que creó y selecciona el área de trabajo de Synapse.
2. En la página **Información general** del área de trabajo de Synapse, en la tarjeta **Abrir Synapse Studio**, selecciona **Abrir** para abrir Synapse Studio en una nueva pestaña del explorador; inicia sesión si se te solicita.
3. En el lado izquierdo de Synapse Studio, usa el icono **&rsaquo;&rsaquo;** para expandir el menú; se muestran las distintas páginas de Synapse Studio que usarás para administrar recursos y realizar tareas de análisis de datos.
4. En la página **Administrar**, asegúrate de que la pestaña **Grupos de SQL** está seleccionada y después selecciona el grupo de SQL dedicado **sql*xxxxxxx*** y usa su icono **▷** para iniciarlo; confirma que deseas reanudarlo cuando se te solicite.
5. Espera a que se reanude el grupo de SQL. Esta operación puede tardar unos minutos. Usa el botón **↻ Actualizar** para comprobar su estado periódicamente. El estado se mostrará como **En línea** cuando esté listo.

### Visualización de las tablas de la base de datos

1. En Synapse Studio, selecciona la página **Datos** y asegúrate de que la pestaña **Área de trabajo** está seleccionada y que contiene una categoría de **base de datos SQL**.
2. Expande la **base de datos SQL**, el grupo **sql*xxxxxxx*** y su carpeta **Tablas** para ver las tablas de la base de datos.

    Normalmente, un almacenamiento de datos relacional se basa en un esquema que consta de tablas de *hechos* y *dimensiones*. Las tablas están optimizadas para consultas analíticas en las que las métricas numéricas de las tablas de hechos se agregan mediante atributos de las entidades representadas por las tablas de dimensiones, por ejemplo, lo que permite agregar ingresos de ventas de Internet por producto, cliente, fecha, etc.
    
3. Expande la tabla **dbo.FactInternetSales** y su carpeta **Columns** para ver las columnas de esta tabla. Ten en cuenta que muchas de las columnas son *claves* que hacen referencia a las filas de las tablas de dimensiones. Otras son valores numéricos (*medidas*) para el análisis.
    
    Las claves se usan para relacionar una tabla de hechos con una o varias tablas de dimensiones, a menudo en un esquema de *estrella*; en la que la tabla de hechos está directamente relacionada con cada tabla de dimensiones (formando una "estrella" de varias puntas con la tabla de hechos en el centro).

4. Examina las columnas de la tabla **dbo.DimPromotion** y ten en cuenta que tiene una **PromotionKey** única que identifica de forma exclusiva cada fila de la tabla. También tiene una **AlternateKey**.

    Normalmente, los datos de un almacenamiento de datos se han importado de uno o varios orígenes transaccionales. La clave *alternativa* refleja el identificador de negocio de la instancia de esta entidad en el origen, pero normalmente se genera una clave *suplente* numérica única para identificar de forma única cada fila de la tabla de dimensiones del almacenamiento de datos. Una de las ventajas de este enfoque es que permite que el almacenamiento de datos contenga varias instancias de la misma entidad en distintos momentos en el tiempo (por ejemplo, registros para el mismo cliente que reflejan su dirección en el momento en que se realizó un pedido).

5. Observa las columnas de **dbo.DimProduct**, y observa que contiene una columna **ProductSubcategoryKey**, que hace referencia a la tabla **dbo.DimProductSubcategory**, que a su vez contiene una columna **ProductCategoryKey** que hace referencia a la tabla **dbo.DimProductCategory**.

    En algunos casos, las dimensiones se normalizan parcialmente en varias tablas relacionadas para permitir distintos niveles de granularidad, como productos que se pueden agrupar en subcategorías y categorías. Esto hace que una estrella simple se extienda a un esquema de *copo de nieve*, en el que la tabla de hechos central está relacionada con una tabla de dimensiones, que se relaciona con tablas de dimensiones adicionales.

6. Consulta las columnas de la tabla **dbo.DimDate** y observa que contiene varias columnas que reflejan diferentes atributos temporales de una fecha, incluyendo el día de la semana, el día del mes, el mes, el año, el nombre del día, el nombre del mes, etc.

    Las dimensiones de tiempo de un almacenamiento de datos normalmente se implementan como una tabla de dimensiones que contiene una fila para cada una de las unidades temporales más pequeñas de granularidad (a menudo denominada *intervalo* de la dimensión) por la que deseas agregar las medidas en las tablas de hechos. En este caso, el intervalo más bajo en el que se pueden agregar medidas es una fecha individual y la tabla contiene una fila para cada fecha de la primera a la última fecha a la que se hace referencia en los datos. Los atributos de la tabla **DimDate** permiten a los analistas agregar medidas basadas en cualquier clave de fecha de la tabla de hechos, usando un conjunto coherente de atributos temporales (por ejemplo, visualización de pedidos por mes basada en la fecha del pedido). La tabla **FactInternetSales** contiene tres claves que se relacionan con la tabla **DimDate**: **OrderDateKey**, **DueDateKey** y **ShipDateKey**.

## Consulta de las tablas de almacenamiento de datos

Ahora que has explorado algunos de los aspectos más importantes del esquema de almacenamiento de datos, estás listo para consultar las tablas y recuperar algunos datos.

### Tablas de hechos y dimensiones

Los valores numéricos de un almacenamiento de datos relacional se almacenan en tablas de hechos con tablas de dimensiones relacionadas que puedes usar para agregar los datos entre varios atributos. Este diseño significa que la mayoría de las consultas de un almacenamiento de datos relacional implican la agregación y agrupación de datos ( con funciones de agregación y cláusulas GROUP BY) en tablas relacionadas (con cláusulas JOIN).

1. En la página **Datos**, selecciona el grupo **sql*xxxxxxx*** SQL y en su menú **...**, selecciona **Nuevo script SQL** > **Script vacío**.
2. Cuando se abra una nueva pestaña **Script SQL 1**, en tu panel **Propiedades**, cambia el nombre del script a **Analizar ventas por Internet** y cambia la **Configuración de resultados por consulta** para que devuelva todas las filas. Después, usa el botón **Publicar** de la barra de herramientas para guardar el script, y usa el botón **Propiedades** (que tiene un aspecto similar a **.**) en el extremo derecho de la barra de herramientas para cerrar el panel **Propiedades** y poder ver el panel de script.
3. En el script vacío, agrega el siguiente código:

    ```sql
    SELECT  d.CalendarYear AS Year,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear
    ORDER BY Year;
    ```

4. Usa el botón **▷ Ejecutar** para ejecutar el script y revisa los resultados, que deberían mostrar los totales de ventas por Internet de cada año. Esta consulta combina la tabla de hechos de ventas por Internet a una tabla de dimensiones de tiempo basada en la fecha de pedido y agrega la medida de importe de ventas en la tabla de hechos por el atributo mes natural de la tabla de dimensiones.

5. Modificación de la consulta como se indica a continuación para agregar el atributo mes desde la dimensión de tiempo y después ejecuta la consulta modificada.

    ```sql
    SELECT  d.CalendarYear AS Year,
            d.MonthNumberOfYear AS Month,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear, d.MonthNumberOfYear
    ORDER BY Year, Month;
    ```

    Tenga en cuenta que los atributos de la dimensión de tiempo permiten agregar las medidas de la tabla de hechos en varios niveles jerárquicos, en este caso, año y mes. Se trata de un patrón común en los almacenamientos de datos.

6. Modifica la consulta como se indica a continuación para quitar el mes y agregar una segunda dimensión a la agregación y después ejecútalo para ver los resultados (que muestran los totales de ventas por Internet anuales para cada región):

    ```sql
    SELECT  d.CalendarYear AS Year,
            g.EnglishCountryRegionName AS Region,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    GROUP BY d.CalendarYear, g.EnglishCountryRegionName
    ORDER BY Year, Region;
    ```

    Tenga en cuenta que geografía es una dimensión de *copo de nieve* que está relacionada con la tabla de hechos ventas por Internet a través de la dimensión del cliente. Por lo tanto, necesitas dos combinaciones en la consulta para agregar las ventas por Internet por geografía.

7. Modifica y vuelve a ejecutar la consulta para agregar otra dimensión de copo de nieve y agregar las ventas regionales anuales por categoría de producto:

    ```sql
    SELECT  d.CalendarYear AS Year,
            pc.EnglishProductCategoryName AS ProductCategory,
            g.EnglishCountryRegionName AS Region,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    JOIN DimProduct AS p ON i.ProductKey = p.ProductKey
    JOIN DimProductSubcategory AS ps ON p.ProductSubcategoryKey = ps.ProductSubcategoryKey
    JOIN DimProductCategory AS pc ON ps.ProductCategoryKey = pc.ProductCategoryKey
    GROUP BY d.CalendarYear, pc.EnglishProductCategoryName, g.EnglishCountryRegionName
    ORDER BY Year, ProductCategory, Region;
    ```

    Esta vez, la dimensión de copo de nieve para la categoría de producto requiere tres combinaciones para reflejar la relación jerárquica entre productos, subcategorías y categorías.

8. Publicación del script para guardarlo.

### Uso de funciones de categoría

Otro requisito común al analizar grandes volúmenes de datos es agrupar los datos por particiones y determinar la *clasificación* de cada entidad de la partición en función de una métrica específica.

1. En la consulta existente, agrega el siguiente código SQL para recuperar los valores de ventas de 2022 a través de particiones en función del nombre de país o región:

    ```sql
    SELECT  g.EnglishCountryRegionName AS Region,
            ROW_NUMBER() OVER(PARTITION BY g.EnglishCountryRegionName
                              ORDER BY i.SalesAmount ASC) AS RowNumber,
            i.SalesOrderNumber AS OrderNo,
            i.SalesOrderLineNumber AS LineItem,
            i.SalesAmount AS SalesAmount,
            SUM(i.SalesAmount) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionTotal,
            AVG(i.SalesAmount) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionAverage
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    WHERE d.CalendarYear = 2022
    ORDER BY Region;
    ```

2. Selecciona solo el nuevo código de consulta y usa el botón **▷ Ejecutar** para ejecutarlo. Después revisa los resultados, que deben tener un aspecto similar a la tabla siguiente:

    | Region | RowNumber | OrderNo | LineItem | SalesAmount | RegionTotal | RegionAverage |
    |--|--|--|--|--|--|--|
    |Australia|1|SO73943|2|2,2900|2172278.7900|375.8918|
    |Australia|2|SO74100|4|2,2900|2172278.7900|375.8918|
    |...|...|...|...|...|...|...|
    |Australia|5779|SO64284|1|2443.3500|2172278.7900|375.8918|
    |Canadá|1|SO66332|2|2,2900|563177.1000|157.8411|
    |Canadá|2|SO68234|2|2,2900|563177.1000|157.8411|
    |...|...|...|...|...|...|...|
    |Canadá|3568|SO70911|1|2443.3500|563177.1000|157.8411|
    |Francia|1|SO68226|3|2,2900|816259.4300|315.4016|
    |Francia|2|SO63460|2|2,2900|816259.4300|315.4016|
    |...|...|...|...|...|...|...|
    |Francia|2588|SO69100|1|2443.3500|816259.4300|315.4016|
    |Alemania|1|SO70829|3|2,2900|922368.2100|352.4525|
    |Alemania|2|SO71651|2|2,2900|922368.2100|352.4525|
    |...|...|...|...|...|...|...|
    |Alemania|2617|SO67908|1|2443.3500|922368.2100|352.4525|
    |Reino Unido|1|SO66124|3|2,2900|1051560.1000|341.7484|
    |Reino Unido|2|SO67823|3|2,2900|1051560.1000|341.7484|
    |...|...|...|...|...|...|...|
    |Reino Unido|3077|SO71568|1|2443.3500|1051560.1000|341.7484|
    |Estados Unidos|1|SO74796|2|2,2900|2905011.1600|289.0270|
    |Estados Unidos|2|SO65114|2|2,2900|2905011.1600|289.0270|
    |...|...|...|...|...|...|...|
    |Estados Unidos|10051|SO66863|1|2443.3500|2905011.1600|289.0270|

    Observa los siguientes hechos sobre estos resultados:

    - Hay una fila para cada elemento de línea de pedido de ventas.
    - Las filas se organizan en particiones basadas en la geografía donde se realizó la venta.
    - Las filas de cada partición geográfica se numeran en orden de importe de ventas (de menor a mayor).
    - Para cada fila, se incluye el importe de ventas del elemento de línea, así como el total regional y los importes promedio de ventas.

3. En las consultas existentes, agrega el código siguiente para aplicar funciones de ventana dentro de una consulta GROUP BY y clasificar las ciudades de cada región en función de su importe total de ventas:

    ```sql
    SELECT  g.EnglishCountryRegionName AS Region,
            g.City,
            SUM(i.SalesAmount) AS CityTotal,
            SUM(SUM(i.SalesAmount)) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionTotal,
            RANK() OVER(PARTITION BY g.EnglishCountryRegionName
                        ORDER BY SUM(i.SalesAmount) DESC) AS RegionalRank
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    GROUP BY g.EnglishCountryRegionName, g.City
    ORDER BY Region;
    ```

4. Selecciona solo el nuevo código de consulta y usa el botón **▷ Ejecutar** para ejecutarlo. Después, revisa los resultados y observa lo siguiente:
    - Los resultados incluyen una fila para cada ciudad, agrupada por región.
    - Las ventas totales (suma de importes de ventas individuales) se calculan para cada ciudad
    - El total de ventas regionales (la suma de la suma de los importes de ventas de cada ciudad de la región) se calcula en función de la partición regional.
    - La clasificación de cada ciudad dentro de su partición regional se calcula ordenando el importe total de ventas por ciudad en orden descendente.

5. Publica el script actualizado para guardar los cambios.

> **Sugerencia**: ROW_NUMBER y RANK son ejemplos de funciones de clasificación disponibles en Transact-SQL. Para obtener más detalles, consulta la referencia [Funciones de categoría](https://docs.microsoft.com/sql/t-sql/functions/ranking-functions-transact-sql) en la documentación del lenguaje Transact-SQL.

### Recuperación de un recuento aproximado

Al explorar grandes volúmenes de datos, las consultas pueden tardar un tiempo significativo y necesitar recursos al ejecutarse. Después, el análisis de datos no requiere valores totalmente precisos: una comparación de valores aproximados puede ser suficiente.

1. En las consultas existentes, agrega el código siguiente para recuperar el número de pedidos de ventas de cada año natural:

    ```sql
    SELECT d.CalendarYear AS CalendarYear,
        COUNT(DISTINCT i.SalesOrderNumber) AS Orders
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear
    ORDER BY CalendarYear;
    ```

2. Selecciona solo el nuevo código de consulta y usa el botón **▷ Ejecutar** para ejecutarlo. Después, revisa la salida que se devuelve:
    - En la pestaña **Resultados** de la consulta, observa los recuentos de pedidos de cada año.
    - En la pestaña **Mensajes**, observa el tiempo total de ejecución de la consulta.
3. Modifica la consulta como se indica a continuación para devolver un recuento aproximado para cada año. Luego vuelve a ejecutar la consulta.

    ```sql
    SELECT d.CalendarYear AS CalendarYear,
        APPROX_COUNT_DISTINCT(i.SalesOrderNumber) AS Orders
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear
    ORDER BY CalendarYear;
    ```

4. Revisión de la salida que se devuelve:
    - En la pestaña **Resultados** de la consulta, observa los recuentos de pedidos de cada año. Deben estar dentro del 2 % de los recuentos reales recuperados por la consulta anterior.
    - En la pestaña **Mensajes**, observa el tiempo total de ejecución de la consulta. Debe ser más corto que para la consulta anterior.

5. Publica el script para guardar los cambios.

> **Sugerencia**: consulta la documentación de la función [APPROX_COUNT_DISTINCT](https://docs.microsoft.com/sql/t-sql/functions/approx-count-distinct-transact-sql) para obtener más detalles.

## Desafío: análisis de ventas de revendedores

1. Cree un nuevo script vacío para el grupo de SQL **sql*xxxxxxx*** y guárdalo con el nombre **Analyze Reseller Sales**.
2. Crea consultas SQL en el script para buscar la siguiente información basada en la tabla de hechos **FactResellerSales** y las tablas de dimensiones a las que está relacionada:
    - La cantidad total de artículos vendidos por año fiscal y trimestre.
    - La cantidad total de artículos vendidos por año fiscal, trimestre y región del territorio de ventas asociado al empleado que realizó la venta.
    - La cantidad total de artículos vendidos por año fiscal, trimestre y región de territorio de ventas por categoría de producto.
    - El rango de cada territorio de ventas por año fiscal en función del importe total de ventas del año.
    - El número aproximado de pedidos de ventas por año en cada territorio de ventas.

    > **Sugerencia**: Compara las consultas con las del script de **solución** en la página **Desarrollar** de Synapse Studio.

3. Experimenta con consultas para explorar el resto de las tablas en el esquema del almacenamiento de datos a modo de entretenimiento.
4. Cuando hayas terminado, en la página **Administrar**, pausa el grupo de SQL dedicado **sql*xxxxxxx***.

## Eliminación de recursos de Azure

Si ha terminado de explorar Azure Synapse Analytics, debe eliminar los recursos que ha creado para evitar costos innecesarios de Azure.

1. Cierre la pestaña del explorador de Synapse Studio y vuelva a Azure Portal.
2. En Azure Portal, en la página **Inicio**, seleccione **Grupos de recursos**.
3. Selecciona el grupo de recursos **dp203-*xxxxxxx*** para tu área de trabajo de Synapse Analytics (no el grupo de recursos administrados) y verifica que contiene el área de trabajo de Synapse, la cuenta de almacenamiento y el grupo de SQL dedicado para tu área de trabajo.
4. En la parte superior de la página **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
5. Especifica el nombre del grupo de recursos **dp203-*xxxxxxx*** para confirmar que quieres eliminarlo y selecciona **Eliminar**.

    Después de unos minutos, se eliminarán el grupo de recursos de área de trabajo de Azure Synapse y el grupo de recursos de área de trabajo administrado asociado a él.
