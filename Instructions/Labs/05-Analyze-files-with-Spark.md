---
lab:
  title: Análisis de datos en un lago de datos con Spark
  ilt-use: Suggested demo
---
# Análisis de datos en un lago de datos con Spark

Apache Spark es un motor de código abierto para el procesamiento de datos distribuido y se usa ampliamente para explorar, procesar y analizar grandes volúmenes de datos en el almacenamiento de lago de datos. Spark está disponible como opción de procesamiento en muchos productos de plataforma de datos, como Azure HDInsight, Azure Databricks y Azure Synapse Analytics en la plataforma en la nube Microsoft Azure. Una de las ventajas de Spark es la compatibilidad con una amplia variedad de lenguajes de programación, como Java, Scala, Python y SQL, lo que lo convierte en una solución muy flexible para cargas de trabajo de procesamiento de datos, incluida la limpieza y manipulación de datos, el análisis estadístico y el aprendizaje automático, y el análisis y la visualización de datos.

Este laboratorio tardará aproximadamente **45** minutos en completarse.

## Antes de empezar

Necesitará una [suscripción de Azure](https://azure.microsoft.com/free) en la que tenga acceso de nivel administrativo.

## Aprovisionar un área de trabajo de Azure Synapse Analytics

Necesitarás un área de trabajo de Azure Synapse Analytics con acceso a Data Lake Storage y un grupo de Apache Spark que puedes usar para consultar y procesar archivos en el lago de datos.

En este ejercicio usarás una combinación de un script de PowerShell y una plantilla de ARM para aprovisionar un área de trabajo de Azure Synapse Analytics.

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
    cd dp203/Allfiles/labs/05
    ./setup.ps1
    ```

6. Si se te solicita, elige la suscripción que deseas usar (esto solo ocurrirá si tienes acceso a varias suscripciones de Azure).
7. Cuando se te solicite, escribe una contraseña adecuada que se va a establecer para el grupo de SQL de Azure Synapse.

    > **Nota**: Asegúrate de recordar esta contraseña.

8. Espera a que se complete el script: normalmente tarda unos 10 minutos, pero en algunos casos puede tardar más. Mientras esperas, consulta el artículo [Apache Spark en Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics/spark/apache-spark-overview) en la documentación de Azure Synapse Analytics.

## Consulta de datos en archivos

El script aprovisiona un área de trabajo de Azure Synapse Analytics y una cuenta de Azure Storage para hospedar el lago de datos y luego carga algunos archivos de datos en el lago de datos.

### Visualización de archivos en el lago de datos

1. Una vez completado el script, en Azure Portal, ve al grupo de recursos **dp203-*xxxxxxx*** que creó y selecciona el área de trabajo de Synapse.
2. En la página **Información general** de tu área de trabajo de Synapse, en la tarjeta **Abrir Synapse Studio**, selecciona **Abrir** para abrir Synapse Studio en una nueva pestaña del explorador e inicia sesión si se te solicita.
3. En el lado izquierdo de Synapse Studio, usa el icono **&rsaquo;&rsaquo;** para expandir el menú. Esta acción mostrará las diferentes páginas de Synapse Studio que usarás para administrar recursos y realizar tareas de análisis de datos.
4. En la página **Administrar**, selecciona la pestaña **Grupos de Apache Spark** y observa que un grupo de Spark con un nombre similar a **spark*xxxxxxx*** se ha aprovisionado en el área de trabajo. Más adelante usarás este grupo de Spark para cargar y analizar datos de archivos en el almacenamiento de lago de datos para el área de trabajo.
5. En la página **Datos**, consulta la pestaña **Vinculado** y comprueba que el área de trabajo incluye un vínculo a la cuenta de almacenamiento de Azure Data Lake Storage Gen2, que debe tener un nombre similar a **synapse*xxxxxxx* (Primary - datalake*xxxxxxx*)**.
6. Expande tu cuenta de almacenamiento y comprueba que contiene un contenedor del sistema de archivos denominado **files**.
7. Selecciona el contenedor **files** y ten en cuenta que contiene carpetas denominadas **sales** y **synapse**. Azure Synapse usa la carpeta **synapse** y la carpeta **sales** contiene los archivos de datos que vas a consultar.
8. Abre la carpeta **sales** y la carpeta **orders** que contiene, y observa que la carpeta **orders** contiene archivos .csv que corresponden a tres años de datos de ventas.
9. Haz clic con el botón derecho en cualquiera de los archivos y selecciona **Vista previa** para ver los datos que contiene. Ten en cuenta que los archivos no contienen una fila de encabezado, por lo que puedes anular la selección de la opción para mostrar los encabezados de columna.

### Usar Spark para explorar datos

1. Selecciona cualquiera de los archivos de la carpeta **orders** y, después, en la lista **Nuevo cuaderno** de la barra de herramientas, selecciona **Cargar en DataFrame**. Un objeto dataframe es una estructura de Spark que representa un conjunto de datos tabular.
2. En la nueva pestaña **Cuaderno 1** que se abre, en la lista **Asociar a**, selecciona el grupo de Spark (**spark*xxxxxxx***). Después, usa el botón **▷ Ejecutar todo** para ejecutar todas las celdas del cuaderno (actualmente solo hay una).

    Dado que esta es la primera vez que has ejecutado código de Spark en esta sesión, se debe iniciar el grupo de Spark. Esto significa que la primera ejecución de la sesión puede tardar unos minutos. Las ejecuciones posteriores serán más rápidas.

3. Mientras esperas a que se inicialice la sesión de Spark, revisa el código que se generó; que tiene un aspecto similar al siguiente:

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/sales/orders/2019.csv', format='csv'
    ## If header exists uncomment line below
    ##, header=True
    )
    display(df.limit(10))
    ```

4. Cuando el código haya terminado de ejecutarse, revisa la salida debajo de la celda del cuaderno. Muestra las diez primeras filas del archivo seleccionado, con nombres de columna automáticos en el formulario **_c0**, **_c1**, **_c2**, etc.
5. Modifica el código para que la función **spark.read.load** lea los datos de <u>todos</u> los archivos CSV de la carpeta y la función de **visualización** muestre las primeras 100 filas. El código debe tener este aspecto (con *datalakexxxxxxx* que coincida con el nombre del almacén de Data Lake):

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/sales/orders/*.csv', format='csv'
    )
    display(df.limit(100))
    ```

6. Usa el botón **▷** situado a la izquierda de la celda de código para ejecutar solo esa celda y revisar los resultados.

    La trama de datos ahora incluye datos de todos los archivos, pero los nombres de columna no son útiles. Spark usa un enfoque de "esquema en lectura" para intentar determinar los tipos de datos adecuados para las columnas en función de los datos que contienen y si una fila de encabezado está presente en un archivo de texto, se puede usar para identificar los nombres de columna (especificando un parámetro **header=True** en la función **load**). Como alternativa, puedes definir un esquema explícito para la trama de datos.

7. Modifica el código de la siguiente manera (reemplazando *datalakexxxxxxx*), para definir un esquema explícito para la trama de datos que incluya los nombres de columna y los tipos de datos. Vuelve a ejecutar el código en la celda.

    ```Python
    %%pyspark
    from pyspark.sql.types import *
    from pyspark.sql.functions import *

    orderSchema = StructType([
        StructField("SalesOrderNumber", StringType()),
        StructField("SalesOrderLineNumber", IntegerType()),
        StructField("OrderDate", DateType()),
        StructField("CustomerName", StringType()),
        StructField("Email", StringType()),
        StructField("Item", StringType()),
        StructField("Quantity", IntegerType()),
        StructField("UnitPrice", FloatType()),
        StructField("Tax", FloatType())
        ])

    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/sales/orders/*.csv', format='csv', schema=orderSchema)
    display(df.limit(100))
    ```

8. En los resultados, usa el botón **＋Código** para agregar una nueva celda de código al cuaderno. Entonces, en la nueva celda, agrega el código siguiente para mostrar el esquema del objeto dataframe:

    ```Python
    df.printSchema()
    ```

9. Ejecuta la nueva celda y comprueba que el esquema de dataframe coincide con el elemento **orderSchema** definido. La función **printSchema** puede ser útil cuando se usa un objeto dataframe con un esquema inferido automáticamente.

## Exploración de datos en un objeto dataframe

El objeto **dataframe** de Spark es simular a un objeto dataframe Pandas de Python e incluye una amplia variedad de funciones que puedes usar para filtrar, agrupar y analizar de cualquier otra forma los datos que contiene.

### Filtrado de un objeto DataFrame

1. Agregue una nueva celda de código al cuaderno y, luego, escriba en ella el código siguiente:

    ```Python
    customers = df['CustomerName', 'Email']
    print(customers.count())
    print(customers.distinct().count())
    display(customers.distinct())
    ```

2. Ejecute la nueva celda de código y revise los resultados. Observe los siguientes detalles:
    - Cuando se realiza una operación en un objeto DataFrame, el resultado es un nuevo DataFrame (en este caso, se crea un nuevo DataFrame **customers** seleccionando un subconjunto específico de columnas del DataFrame **df**).
    - Los objetos DataFrame proporcionan funciones como **count** y **distinct** que se pueden usar para resumir y filtrar los datos que contienen.
    - La sintaxis `dataframe['Field1', 'Field2', ...]` es una forma abreviada de definir un subconjunto de columna. También puede usar el método **select**, por lo que la primera línea del código anterior se podría escribir como `customers = df.select("CustomerName", "Email")`.

3. Modifique el código de la siguiente manera:

    ```Python
    customers = df.select("CustomerName", "Email").where(df['Item']=='Road-250 Red, 52')
    print(customers.count())
    print(customers.distinct().count())
    display(customers.distinct())
    ```

4. Ejecute el código modificado para ver los clientes que han comprado el producto *Road-250 Red, 52*. Tenga en cuenta que puede "encadenar" varias funciones para que la salida de una función se convierta en la entrada de la siguiente; en este caso, el objeto DataFrame creado por el método **select** es el objeto DataFrame de origen para el método **where** que se usa para aplicar criterios de filtrado.

### Agregación y agrupación de datos en un objeto DataFrame

1. Agregue una nueva celda de código al cuaderno y, luego, escriba en ella el código siguiente:

    ```Python
    productSales = df.select("Item", "Quantity").groupBy("Item").sum()
    display(productSales)
    ```

2. Ejecute la celda de código que agregó y observe que los resultados muestran la suma de las cantidades de pedido agrupadas por producto. El método **groupBy** agrupa las filas por *Item* y la función de agregado **sum** subsiguiente se aplica a todas las columnas numéricas restantes (en este caso, *Quantity*).

3. Agregue otra nueva celda de código al cuaderno y escriba en ella el código siguiente:

    ```Python
    yearlySales = df.select(year("OrderDate").alias("Year")).groupBy("Year").count().orderBy("Year")
    display(yearlySales)
    ```

4. Ejecute la celda de código que agregó y observe que los resultados muestran el número de pedidos de ventas por año. Ten en cuenta que el método **select** incluye una función **year** de SQL para extraer el componente del año del campo *OrderDate* y, luego se usa un método **alias** para asignar un nombre de columna al valor de año extraído. Los datos se agrupan entonces por la columna *Year* derivada y el recuento de filas de cada grupo se calcula antes de que finalmente se use el método **orderBy** para ordenar el objeto DataFrame resultante.

## Consulta de datos con Spark SQL

Como se ha visto, los métodos nativos del objeto dataframe te permiten consultar y analizar datos de un archivo de forma bastante eficaz. Sin embargo, a muchos analistas de datos les gusta más trabajar con sintaxis SQL. Spark SQL es una API de lenguaje SQL en Spark que puedes usar para ejecutar instrucciones SQL o incluso conservar datos en tablas relacionales.

### Uso de Spark SQL en código PySpark

El lenguaje predeterminado en los cuadernos de Azure Synapse Studio es PySpark, que es un entorno de ejecución de Python basado en Spark. Dentro de este entorno de ejecución, puedes usar la biblioteca **spark.sql** para insertar la sintaxis SQL de Spark en el código de Python y trabajar con construcciones SQL como tablas y vistas.

1. Agregue una nueva celda de código al cuaderno y, luego, escriba en ella el código siguiente:

    ```Python
    df.createOrReplaceTempView("salesorders")

    spark_df = spark.sql("SELECT * FROM salesorders")
    display(spark_df)
    ```

2. Ejecute la celda y revise los resultados. Observe lo siguiente:
    - El código conserva los datos en el objeto dataframe **df** como una vista temporal denominada **salesorders**. Spark SQL admite el uso de vistas temporales o tablas persistentes como orígenes para consultas SQL.
    - Después, se usa el método **spark.sql** para ejecutar una consulta SQL en la vista **salesorders**.
    - Los resultados de la consulta se almacenan en un objeto dataframe.

### Ejecución de código SQL en una celda

Aunque resulta útil poder insertar instrucciones SQL en una celda que contenga código de PySpark, los analistas de datos suelen preferir trabajar directamente en SQL.

1. Agregue una nueva celda de código al cuaderno y, luego, escriba en ella el código siguiente:

    ```sql
    %%sql
    SELECT YEAR(OrderDate) AS OrderYear,
           SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue
    FROM salesorders
    GROUP BY YEAR(OrderDate)
    ORDER BY OrderYear;
    ```

2. Ejecute la celda y revise los resultados. Observe lo siguiente:
    - La línea `%%sql` al principio de la celda (llamada *magic*) indica que se debe usar el entorno de ejecución del lenguaje Spark SQL para ejecutar el código en esta celda en lugar de PySpark.
    - El código SQL hace referencia a la vista **salesorder** que creaste anteriormente mediante PySpark.
    - La salida de la consulta SQL se muestra automáticamente como resultado en la celda.

> **Nota**: Para más información sobre Spark SQL y los objetos DataFrame, consulte la [documentación de Spark SQL](https://spark.apache.org/docs/2.2.0/sql-programming-guide.html).

## Visualización de datos con Spark

Proverbialmente, una imagen vale más que mil palabras, y un gráfico suele ser mejor que mil filas de datos. Aunque los cuadernos de Azure Synapse Analytics incluyen una vista de gráfico integrada para los datos que se muestran de un objeto dataframe o una consulta de Spark SQL, no están diseñados para crear gráficos completos. Sin embargo, puede usar bibliotecas de gráficos de Python como **matplotlib** y **seaborn** para crear gráficos a partir de datos de objetos DataFrame.

### Visualización de los resultados en un gráfico

1. Agregue una nueva celda de código al cuaderno y, luego, escriba en ella el código siguiente:

    ```sql
    %%sql
    SELECT * FROM salesorders
    ```

2. Ejecute el código y observe que devuelve los datos de la vista **salesorders** que creó anteriormente.
3. En la sección de resultados debajo de la celda, cambie la opción **Ver** de **Tabla** a **Gráfico**.
4. Use el botón **Opciones de vista** situado en la parte superior derecha del gráfico para mostrar el panel de opciones del gráfico. A continuación, establezca las opciones como se indica a continuación y seleccione **Aplicar**:
    - **Tipo de gráfico:**  Gráfico de barras.
    - **Clave**: Elemento.
    - **Valores**: Cantidad.
    - **Grupo de series**: *déjelo en blanco*.
    - **Agregación**: Suma.
    - **Apilado**: *No seleccionado*.

5. Compruebe que el gráfico se parece a este:

    ![Un gráfico de barras de productos por cantidades totales de pedidos](./images/notebook-chart.png)

### Introducción a **matplotlib**

1. Agregue una nueva celda de código al cuaderno y, luego, escriba en ella el código siguiente:

    ```Python
    sqlQuery = "SELECT CAST(YEAR(OrderDate) AS CHAR(4)) AS OrderYear, \
                    SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue \
                FROM salesorders \
                GROUP BY CAST(YEAR(OrderDate) AS CHAR(4)) \
                ORDER BY OrderYear"
    df_spark = spark.sql(sqlQuery)
    df_spark.show()
    ```

2. Ejecute el código y observe que devuelve un objeto DataFrame de Spark que contiene los ingresos anuales.

    Para visualizar los datos en un gráfico, comenzaremos usando la biblioteca **matplotlib** de Python. Esta biblioteca es la biblioteca de trazado principal en la que se basan muchas otras y proporciona una gran flexibilidad en la creación de gráficos.

3. Agregue una nueva celda de código al cuaderno y escriba en ella el código siguiente:

    ```Python
    from matplotlib import pyplot as plt

    # matplotlib requires a Pandas dataframe, not a Spark one
    df_sales = df_spark.toPandas()

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'])

    # Display the plot
    plt.show()
    ```

4. Ejecute la celda y revise los resultados, que constan de un gráfico de columnas con los ingresos brutos totales de cada año. Observe las siguientes características del código usado para generar este gráfico:
    - La biblioteca **matplotlib** requiere un objeto DataFrame de *Pandas*, por lo que debe convertir a este formato el objeto DataFrame de *Spark* devuelto en la consulta de Spark SQL.
    - En el centro de la biblioteca **matplotlib** se encuentra el objeto **pyplot**. Esta es la base de la mayor parte de la funcionalidad de trazado.
    - La configuración predeterminada da como resultado un gráfico utilizable, pero hay un margen considerable para personalizarla.

5. Modifique el código para trazar el gráfico de la siguiente manera:

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

    # Customize the chart
    plt.title('Revenue by Year')
    plt.xlabel('Year')
    plt.ylabel('Revenue')
    plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
    plt.xticks(rotation=45)

    # Show the figure
    plt.show()
    ```

6. Vuelva a ejecutar la celda de código y observe los resultados. El gráfico ahora incluye un poco más de información.

    Un gráfico está técnicamente contenido con una **Figura**. En los ejemplos anteriores, la figura se creó implícitamente; pero puede crearla explícitamente.

7. Modifique el código para trazar el gráfico de la siguiente manera:

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a Figure
    fig = plt.figure(figsize=(8,3))

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

    # Customize the chart
    plt.title('Revenue by Year')
    plt.xlabel('Year')
    plt.ylabel('Revenue')
    plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
    plt.xticks(rotation=45)

    # Show the figure
    plt.show()
    ```

8. Vuelve a ejecutar la celda de código y observa los resultados. La figura determina la forma y el tamaño del trazado.

    Una figura puede contener varios subtrazados, cada uno en su propio *eje*.

9. Modifica el código para trazar el gráfico de la siguiente manera:

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a figure for 2 subplots (1 row, 2 columns)
    fig, ax = plt.subplots(1, 2, figsize = (10,4))

    # Create a bar plot of revenue by year on the first axis
    ax[0].bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')
    ax[0].set_title('Revenue by Year')

    # Create a pie chart of yearly order counts on the second axis
    yearly_counts = df_sales['OrderYear'].value_counts()
    ax[1].pie(yearly_counts)
    ax[1].set_title('Orders per Year')
    ax[1].legend(yearly_counts.keys().tolist())

    # Add a title to the Figure
    fig.suptitle('Sales Data')

    # Show the figure
    plt.show()
    ```

10. Vuelva a ejecutar la celda de código y observe los resultados. La figura contiene las subtrazados especificados en el código.

> **Nota**: Para más información sobre el trazado con matplotlib, consulte la [documentación de matplotlib](https://matplotlib.org/).

### Uso de la biblioteca **seaborn**

Aunque **matplotlib** permite crear gráficos complejos de varios tipos, puede que sea necesario código complejo para lograr los mejores resultados. Por esta razón, a lo largo de los años, se han creado muchas bibliotecas nuevas sobre la base de matplotlib para abstraer su complejidad y mejorar sus capacidades. Una de estas bibliotecas es **seaborn**.

1. Agregue una nueva celda de código al cuaderno y, luego, escriba en ella el código siguiente:

    ```Python
    import seaborn as sns

    # Clear the plot area
    plt.clf()

    # Create a bar chart
    ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)
    plt.show()
    ```

2. Ejecute el código y observe que se muestra un gráfico de barras usando la biblioteca seaborn.
3. Agregue una nueva celda de código al cuaderno y, luego, escriba en ella el código siguiente:

    ```Python
    # Clear the plot area
    plt.clf()

    # Set the visual theme for seaborn
    sns.set_theme(style="whitegrid")

    # Create a bar chart
    ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)
    plt.show()
    ```

4. Ejecuta el código y observa que seaborn te permite establecer un tema de color coherente para tus trazados.

5. Agregue una nueva celda de código al cuaderno y, luego, escriba en ella el código siguiente:

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a bar chart
    ax = sns.lineplot(x="OrderYear", y="GrossRevenue", data=df_sales)
    plt.show()
    ```

6. Ejecuta el código para ver los ingresos anuales en un gráfico de líneas.

> **Nota**: Para más información sobre el trazado con seaborn, consulte la [documentación de seaborn](https://seaborn.pydata.org/index.html).

## Eliminación de recursos de Azure

Si ha terminado de explorar Azure Synapse Analytics, debe eliminar los recursos que ha creado para evitar costos innecesarios de Azure.

1. Cierre la pestaña del explorador de Synapse Studio y vuelva a Azure Portal.
2. En Azure Portal, en la página **Inicio**, seleccione **Grupos de recursos**.
3. Selecciona el grupo de recursos **dp203-*xxxxxxx*** para tu área de trabajo de Synapse Analytics (no el grupo de recursos administrados) y verifica que contiene el área de trabajo de Synapse, la cuenta de almacenamiento y el grupo de Spark para tu área de trabajo.
4. En la parte superior de la página **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
5. Especifica el nombre del grupo de recursos **dp203-*xxxxxxx*** para confirmar que quieres eliminarlo y selecciona **Eliminar**.

    Después de unos minutos, se eliminarán el grupo de recursos de área de trabajo de Azure Synapse y el grupo de recursos de área de trabajo administrado asociado a él.
