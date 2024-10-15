---
lab:
  title: Uso de Delta Lake en Azure Synapse Analytics
  ilt-use: Lab
---

# Usar Delta Lake con Spark en Azure Synapse Analytics

Delta Lake es un proyecto de código abierto para compilar una capa de almacenamiento de datos transaccional encima de un lago de datos. Delta Lake agrega compatibilidad con la semántica relacional para las operaciones de datos por lotes y de streaming, y permite la creación de una arquitectura de *almacenamiento de lago* en la que se puede usar Apache Spark para procesar y consultar datos en tablas basadas en archivos subyacentes en el lago de datos.

Este ejercicio debería tardar en completarse **40** minutos aproximadamente.

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

4. En el panel de PowerShell, esscribe los siguientes comandos para clonar este repositorio:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Una vez clonado el repositorio, escribe los siguientes comandos para cambiar a la carpeta de este ejercicio y ejecutar el script **setup.ps1** que contiene:

    ```
    cd dp-203/Allfiles/labs/07
    ./setup.ps1
    ```

6. Si se solicita, elige la suscripción que quieres usar (esto solo ocurrirá si tienes acceso a varias suscripciones de Azure).
7. Cuando se te solicite, escribe una contraseña adecuada que se va a establecer para el grupo de SQL de Azure Synapse.

    > **Nota**: Asegúrate de recordar esta contraseña.

8. Espera a que se complete el script: normalmente tarda unos 10 minutos, pero en algunos casos puede tardar más. Mientras esperas, revisa el artículo [Qué es Delta Lake](https://docs.microsoft.com/azure/synapse-analytics/spark/apache-spark-what-is-delta-lake) de la documentación de Azure Synapse Analytics.

## Creación de tablas delta

El script aprovisiona un área de trabajo de Azure Synapse Analytics y una cuenta de Azure Storage para hospedar el lago de datos y luego carga un archivo de datos en el lago de datos.

### Explorar los datos del lago de datos

1. Una vez completado el script, en Azure Portal, ve al grupo de recursos **dp203-*xxxxxxx*** que creaste y selecciona tu área de trabajo de Synapse.
2. En la página **Información general** de tu área de trabajo de Synapse, en la tarjeta **Abrir Synapse Studio**, selecciona **Abrir** para abrir Synapse Studio en una nueva pestaña del explorador e inicia sesión si se te solicita.
3. En el lado izquierdo de Synapse Studio, usa el icono **&rsaquo;&rsaquo;** para expandir el menú. Esta acción mostrará las diferentes páginas de Synapse Studio que usarás para administrar recursos y realizar tareas de análisis de datos.
4. En la página **Datos**, consulta la pestaña **Vinculado** y comprueba que el área de trabajo incluye un vínculo a la cuenta de almacenamiento de Azure Data Lake Storage Gen2, que debe tener un nombre similar a **synapse*xxxxxxx* (Primary - datalake*xxxxxxx*)**.
5. Expande tu cuenta de almacenamiento y comprueba que contiene un contenedor del sistema de archivos denominado **files**.
6. Selecciona el contenedor **files** y observa que contiene una carpeta denominada **products**. Esta carpeta contiene los datos con los que vas a trabajar en este ejercicio.
7. Abre la carpeta **products** y observa que contiene un archivo denominado **products.csv**.
8. Selecciona **products.csv** y después, en la lista **Nuevo cuaderno** de la barra de herramientas, selecciona **Cargar en DataFrame**.
9. En el panel **Cuaderno 1** que se abre, en la lista **Asociar a**, selecciona el grupo **sparkxxxxxxx** de Spark y asegúrate de que el **Lenguaje** está configurado como **PySpark (Python)**.
10. Revise solo el código de la primera celda del cuaderno, que debe tener este aspecto:

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/products/products.csv', format='csv'
    ## If header exists uncomment line below
    ##, header=True
    )
    display(df.limit(10))
    ```

11. Quite la marca de comentario de la línea *,header=True* (porque el archivo products.csv tiene los encabezados de columna en la primera línea), por lo que el código tiene el siguiente aspecto:

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/products/products.csv', format='csv'
    ## If header exists uncomment line below
    , header=True
    )
    display(df.limit(10))
    ```

12. Use el icono **&#9655;** situado a la izquierda de la celda de código para ejecutarlo y espere a obtener los resultados. La primera vez que ejecuta una celda en un cuaderno, se inicia el grupo de Spark, por lo que puede tardar más o menos un minuto en devolver los resultados. Finalmente, los resultados deben aparecer debajo de la celda y deben ser similares a estos:

    | ProductID | ProductName | Category | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | Bicicletas de montaña | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | Bicicletas de montaña | 3399.9900 |
    | ... | ... | ... | ... |

### Cargar los datos del archivo en una tabla Delta

1. En los resultados devueltos por la primera celda de código, usa el botón **+ Código** para agregar una nueva celda de código. A continuación, escriba el código siguiente en la nueva celda y ejecútela:

    ```Python
    delta_table_path = "/delta/products-delta"
    df.write.format("delta").save(delta_table_path)
    ```

2. En la pestaña **archivos**, usa el icono **↑** de la barra de herramientas para volver a la raíz del contenedor **files**, y observa que se ha creado una nueva carpeta denominada **delta**. Abre esta carpeta y la tabla **products-delta** que contiene, donde deberías ver los archivos en formato Parquet que contienen los datos.

3. Vuelve a la pestaña **Cuaderno 1** y agrega otra nueva celda de código. A continuación, en la nueva celda, agregue el código siguiente y ejecútelo:

    ```Python
    from delta.tables import *
    from pyspark.sql.functions import *

    # Create a deltaTable object
    deltaTable = DeltaTable.forPath(spark, delta_table_path)

    # Update the table (reduce price of product 771 by 10%)
    deltaTable.update(
        condition = "ProductID == 771",
        set = { "ListPrice": "ListPrice * 0.9" })

    # View the updated data as a dataframe
    deltaTable.toDF().show(10)
    ```

    Los datos se cargan en un objeto **DeltaTable** y se actualizan. Puedes ver la actualización reflejada en los resultados de la consulta.

4. Agregar otra nueva celda de código con el siguiente código y ejecútalo:

    ```Python
    new_df = spark.read.format("delta").load(delta_table_path)
    new_df.show(10)
    ```

    El código carga los datos de la tabla Delta en un DataFrame desde su ubicación en el lago de datos, verificando que el cambio realizado a través de un objeto **DeltaTable** es persistente.

5. Modifica el código que acabas de ejecutar de la siguiente manera, especificando la opción de usar la característica *time travel* de Delta Lake para ver una versión anterior de los datos.

    ```Python
    new_df = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
    new_df.show(10)
    ```

    Al ejecutar el código modificado, los resultados muestran la versión original de los datos.

6. Agregar otra nueva celda de código con el siguiente código y ejecútalo:

    ```Python
    deltaTable.history(10).show(20, False, True)
    ```

    Se muestra el historial de los últimos 20 cambios en la tabla: debe haber dos (la creación original y la actualización que hiciste).

## Creación de tablas de catálogo

Hasta ahora has trabajado con tablas Delta cargando datos de la carpeta que contiene los archivos Parquet en los que se basa la tabla. Puedes definir *tablas de catálogo* que encapsulan los datos y proporcionar una entidad de tabla denominada a la que puedes hacer referencia en código SQL. Spark admite dos tipos de tablas de catálogo para Delta Lake:

- Tablas *externas* que se definen por la ruta de acceso a los archivos Parquet que contienen los datos de la tabla.
- Tablas *administradas*, que se definen en el metastore de Hive para el grupo de Spark.

### Crear una tabla externa

1. En la nueva celda de código, agregue el código siguiente:

    ```Python
    spark.sql("CREATE DATABASE AdventureWorks")
    spark.sql("CREATE TABLE AdventureWorks.ProductsExternal USING DELTA LOCATION '{0}'".format(delta_table_path))
    spark.sql("DESCRIBE EXTENDED AdventureWorks.ProductsExternal").show(truncate=False)
    ```

    Este código crea una nueva base de datos denominada **AdventureWorks** y después crea una pestaña externa denominada **ProductsExternal** en esa base de datos según la ruta de acceso a los archivos Parquet que definiste anteriormente. Después, muestra una descripción de las propiedades de la tabla. Ten en cuenta que la propiedad **Ubicación** es la ruta de acceso que especificaste.

2. Agrega una nueva celda de código y luego introduce y ejecuta el siguiente código:

    ```sql
    %%sql

    USE AdventureWorks;

    SELECT * FROM ProductsExternal;
    ```

    El código usa SQL para cambiar el contexto a la base de datos **AdventureWorks** (que no devuelve datos) y después consultar la tabla **ProductsExternal** (que devuelve un conjunto de resultados que contiene los datos de productos en la tabla Delta Lake).

### Creación de una tabla administrada

1. En la nueva celda de código, agregue el código siguiente:

    ```Python
    df.write.format("delta").saveAsTable("AdventureWorks.ProductsManaged")
    spark.sql("DESCRIBE EXTENDED AdventureWorks.ProductsManaged").show(truncate=False)
    ```

    Este código crea una tabla administrada denominada **ProductsManaged** basada en el DataFrame que cargaste originalmente desde el archivo **products.csv** (antes de actualizar el precio del producto 771). No se especifica una ruta de acceso para los archivos Parquet usados por la tabla: esto lo administras en el metastore de Hive y se muestra en la propiedad **Ubicación** de la descripción de la tabla (en la ruta **files/synapse/workspaces/synapsexxxxxxx/warehouse**).

2. Agrega una nueva celda de código y luego introduce y ejecuta el siguiente código:

    ```sql
    %%sql

    USE AdventureWorks;

    SELECT * FROM ProductsManaged;
    ```

    El código usa SQL para consultar la tabla **ProductsManaged**.

### Comparar las tablas externas y administradas

1. En la nueva celda de código, agregue el código siguiente:

    ```sql
    %%sql

    USE AdventureWorks;

    SHOW TABLES;
    ```

    En este código se enumeran las tablas de la base de datos **AdventureWorks**.

2. Modifica la celda de código de la siguiente manera y ejecútala:

    ```sql
    %%sql

    USE AdventureWorks;

    DROP TABLE IF EXISTS ProductsExternal;
    DROP TABLE IF EXISTS ProductsManaged;
    ```

    Este código quita las tablas del metastore.

3. Vuelve a la pestaña **archivos** y consulta la carpeta **files/delta/products-delta**. Ten en cuenta que los archivos de datos siguen existiendo en esta ubicación. La eliminación de la tabla externa ha quitado la tabla del metastore, pero deja intactos los archivos de datos.
4. Visualiza la carpeta **files/synapse/workspaces/synapsexxxxxxx/warehouse** y ten en cuenta que no hay ninguna carpeta para los datos de la tabla **ProductsManaged**. Al quitar una tabla administrada se elimina la tabla del metastore y también se eliminan los archivos de datos de la tabla.

### Creación de una tabla mediante SQL

1. Agrega una nueva celda de código y luego introduce y ejecuta el siguiente código:

    ```sql
    %%sql

    USE AdventureWorks;

    CREATE TABLE Products
    USING DELTA
    LOCATION '/delta/products-delta';
    ```

2. Agrega una nueva celda de código y luego introduce y ejecuta el siguiente código:

    ```sql
    %%sql

    USE AdventureWorks;

    SELECT * FROM Products;
    ```

    Observa que la nueva tabla de catálogo se creó para la carpeta de tabla de Delta Lake existente, que refleja los cambios realizados anteriormente.

## Uso de tablas Delta para transmitir datos

Delta Lake admite datos de streaming. Las tablas Delta pueden ser un *receptor* o un *origen* para flujos de datos creados mediante Spark Structured Streaming API. En este ejemplo, usará una tabla Delta como receptor para algunos datos de streaming en un escenario simulado de Internet de las cosas (IoT).

1. Vuelve a la pestaña **Notebook 1** y agrega una nueva celda de código. A continuación, en la nueva celda, agregue el código siguiente y ejecútelo:

    ```python
    from notebookutils import mssparkutils
    from pyspark.sql.types import *
    from pyspark.sql.functions import *

    # Create a folder
    inputPath = '/data/'
    mssparkutils.fs.mkdirs(inputPath)

    # Create a stream that reads data from the folder, using a JSON schema
    jsonSchema = StructType([
    StructField("device", StringType(), False),
    StructField("status", StringType(), False)
    ])
    iotstream = spark.readStream.schema(jsonSchema).option("maxFilesPerTrigger", 1).json(inputPath)

    # Write some event data to the folder
    device_data = '''{"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"error"}
    {"device":"Dev2","status":"ok"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}'''
    mssparkutils.fs.put(inputPath + "data.txt", device_data, True)
    print("Source stream created...")
    ```

    Asegúrese de que se muestra el mensaje *Flujo de origen creado...* El código que acaba de ejecutar ha creado un origen de datos de streaming basado en una carpeta en la que se han guardado algunos datos, que representan lecturas de dispositivos IoT hipotéticos.

2. En la nueva celda de código, agregue el código siguiente:

    ```python
    # Write the stream to a delta table
    delta_stream_table_path = '/delta/iotdevicedata'
    checkpointpath = '/delta/checkpoint'
    deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
    print("Streaming to delta sink...")
    ```

    Este código escribe los datos del dispositivo de streaming en formato Delta.

3. En la nueva celda de código, agregue el código siguiente:

    ```python
    # Read the data in delta format into a dataframe
    df = spark.read.format("delta").load(delta_stream_table_path)
    display(df)
    ```

    Este código lee los datos transmitidos en formato Delta en una trama de datos. Observa que el código para cargar datos de streaming no es diferente al usado para cargar datos estáticos desde una carpeta delta.

4. En la nueva celda de código, agregue el código siguiente:

    ```python
    # create a catalog table based on the streaming sink
    spark.sql("CREATE TABLE IotDeviceData USING DELTA LOCATION '{0}'".format(delta_stream_table_path))
    ```

    Este código crea una tabla de catálogo denominada **IotDeviceData** (en la base de datos **predeterminada**) basada en la carpeta delta. De nuevo, este código es el mismo que se usaría para los datos que no son de streaming.

5. En la nueva celda de código, agregue el código siguiente:

    ```sql
    %%sql

    SELECT * FROM IotDeviceData;
    ```

    Este código consulta la tabla **IotDeviceData**, que contiene los datos del dispositivo del origen de streaming.

6. En la nueva celda de código, agregue el código siguiente:

    ```python
    # Add more data to the source stream
    more_data = '''{"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"error"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}'''

    mssparkutils.fs.put(inputPath + "more-data.txt", more_data, True)
    ```

    Este código escribe datos de dispositivo más hipotéticos en el origen de streaming.

7. En la nueva celda de código, agregue el código siguiente:

    ```sql
    %%sql

    SELECT * FROM IotDeviceData;
    ```

    Este código consulta de nuevo la tabla **IotDeviceData**, que ahora debe incluir los datos adicionales que se agregaron al origen de streaming.

8. En la nueva celda de código, agregue el código siguiente:

    ```python
    deltastream.stop()
    ```

    Este código detiene la transmisión.

## Consulta de una tabla delta desde un grupo de SQL sin servidor

Además de los grupos de Spark, Azure Synapse Analytics incluye un grupo de SQL sin servidor integrado. Puedes usar el motor de base de datos relacional de este grupo para consultar tablas delta mediante SQL.

1. En la pestaña **archivos**, ve a la carpeta **archivos/delta**.
2. Selecciona la carpeta **products-delta** y en la barra de herramientas, en la lista desplegable **Nuevo script SQL**, selecciona **Seleccionar las 100 primeras filas**.
3. En el panel **Seleccionar las 100 primeras filas**, en la lista **Tipo de archivo**, selecciona **Formato Delta** y después selecciona **Aplicar**.
4. Revisa el código SQL que se genera, que debería tener este aspecto:

    ```sql
    -- This is auto-generated code
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/delta/products-delta/',
            FORMAT = 'DELTA'
        ) AS [result]
    ```

5. Usa el icono **▷ Ejecutar** para ejecutar el script y revisa los resultados. La aplicación debe tener un aspecto similar al siguiente:

    | ProductID | ProductName | Category | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | Bicicletas de montaña | 3059.991 |
    | 772 | Mountain-100 Silver, 42 | Bicicletas de montaña | 3399.9900 |
    | ... | ... | ... | ... |

    Esto muestra cómo puedes usar un grupo de SQL sin servidor para consultar archivos de formato Delta creados con Spark y usar los resultados para la generación de informes o análisis.

6. Reemplaza la consulta por el siguiente código SQL:

    ```sql
    USE AdventureWorks;

    SELECT * FROM Products;
    ```

7. Ejecuta el código y observa que también puedes usar el grupo SQL sin servidor para consultar datos de Delta Lake en tablas de catálogo que están definidas en el metastore de Spark.

## Eliminación de recursos de Azure

Si ha terminado de explorar Azure Synapse Analytics, debe eliminar los recursos que ha creado para evitar costos innecesarios de Azure.

1. Cierre la pestaña del explorador de Synapse Studio y vuelva a Azure Portal.
2. En Azure Portal, en la página **Inicio**, seleccione **Grupos de recursos**.
3. Selecciona el grupo de recursos **dp203-*xxxxxxx*** para tu área de trabajo de Synapse Analytics (no el grupo de recursos administrados) y verifica que contiene el área de trabajo de Synapse, la cuenta de almacenamiento y el grupo de Spark para tu área de trabajo.
4. En la parte superior de la página **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
5. Especifica el nombre del grupo de recursos **dp203-*xxxxxxx*** para confirmar que quieres eliminarlo y selecciona **Eliminar**.

    Después de unos minutos, se eliminarán el grupo de recursos de área de trabajo de Azure Synapse y el grupo de recursos de área de trabajo administrado asociado a él.
