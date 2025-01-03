---
lab:
  title: Transformación de archivos mediante un grupo de SQL sin servidor
  ilt-use: Lab
---

# Transformación de archivos mediante un grupo de SQL sin servidor

Los *analistas* de datos suelen usar SQL para consultar datos de análisis e informes. Los* ingenieros* de datos también pueden usar SQL para manipular y transformar datos; a menudo como parte de una canalización de ingesta de datos o de un proceso de extracción, transformación y carga (ETL).

En este ejercicio, usarás una agrupación de SQL sin servidor en Azure Synapse Analytics para transformar datos en archivos.

Este ejercicio debería tardar en completarse **30** minutos aproximadamente.

## Antes de empezar

Necesitará una [suscripción de Azure](https://azure.microsoft.com/free) en la que tenga acceso de nivel administrativo.

## Aprovisionar un área de trabajo de Azure Synapse Analytics

Necesitarás un área de trabajo de Azure Synapse Analytics con acceso a Data Lake Storage. Puedes usar el grupo de SQL sin servidor integrado para consultar archivos en el lago de datos.

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
    cd dp-203/Allfiles/labs/03
    ./setup.ps1
    ```

6. Si se solicita, elige la suscripción que quieres usar (esto solo ocurrirá si tienes acceso a varias suscripciones de Azure).
7. Cuando se te solicite, escribe una contraseña adecuada que se va a establecer para el grupo de SQL de Azure Synapse.

    > **Nota**: Asegúrate de recordar esta contraseña.

8. Espera a que se complete el script: normalmente tarda unos 10 minutos, pero en algunos casos puede tardar más. Mientras esperas, revisa el artículo [CETAS con Synapse SQL](https://docs.microsoft.com/azure/synapse-analytics/sql/develop-tables-cetas) en la documentación de Azure Synapse Analytics.

## Consulta de datos en archivos

El script aprovisiona un área de trabajo de Azure Synapse Analytics y una cuenta de Azure Storage para hospedar el lago de datos y luego carga algunos archivos de datos en el lago de datos.

### Visualización de archivos en el lago de datos

1. Una vez completado el script, en Azure Portal, ve al grupo de recursos **dp203-*xxxxxxx*** que creó y selecciona el área de trabajo de Synapse.
2. En la página **Información general** de tu área de trabajo de Synapse, en la tarjeta **Abrir Synapse Studio**, selecciona **Abrir** para abrir Synapse Studio en una nueva pestaña del explorador e inicia sesión si se te solicita.
3. En el lado izquierdo de Synapse Studio, usa el icono **&rsaquo;&rsaquo;** para expandir el menú. Esta acción mostrará las diferentes páginas de Synapse Studio que usarás para administrar recursos y realizar tareas de análisis de datos.
4. En la página **Datos**, consulta la pestaña **Vinculado** y comprueba que el área de trabajo incluye un vínculo a la cuenta de almacenamiento de Azure Data Lake Storage Gen2, que debe tener un nombre similar a **synapse*xxxxxxx* (Primary - datalake*xxxxxxx*)**.
5. Expande tu cuenta de almacenamiento y comprueba que contiene un contenedor del sistema de archivos denominado **files**.
6. Selecciona el contenedor **files** y observa que contiene una carpeta denominada **sales**. Esta carpeta contiene los archivos de datos que vas a consultar.
7. Abre la carpeta **sales** y la carpeta **csv** que contiene, y observa que esta carpeta contiene archivos .csv que corresponden a tres años de datos de ventas.
8. Haz clic con el botón derecho en cualquiera de los archivos y selecciona **Vista previa** para ver los datos que contiene. Observa que los archivos contienen una fila de encabezado.
9. Cierre la vista previa y luego usa el botón **↑** para volver a la carpeta **ventas**.

### Uso de SQL para consultar archivos CSV

1. Selecciona la carpeta **csv** y luego, en la lista **Nuevo script SQL** de la barra de herramientas, selecciona **Seleccionar las 100 primeras filas**.
2. En la lista **Tipo de archivo**, selecciona **Formato de texto** y después aplica la configuración para abrir un nuevo script SQL que consulta los datos en la carpeta.
3. En el panel **Propiedades** de **SQL Script 1** que se crea, cambie el nombre por **Consultar archivos CSV de ventas** y cambia la configuración de resultados para mostrar **Todas las filas**. A continuación, en la barra de herramientas, selecciona **Publicar** para guardar el script y utiliza el botón **Propiedades** (que tiene un aspecto similar a **<sub>*</sub>**) en el extremo derecho de la barra de herramientas para ocultar el panel **Propiedades**.
4. Revisa el código SQL que se ha generado, que debe ser similar a lo siguiente:

    ```SQL
    -- This is auto-generated code
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/csv/**',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0'
        ) AS [result]
    ```

    Este código usa OPENROWSET para leer datos de los archivos CSV de la carpeta ventas y recupera las 100 primeras filas de datos.

5. En este caso, los archivos de datos incluyen los nombres de columna en la primera fila; por lo tanto, modifica la consulta para añadir un `HEADER_ROW = TRUE` parámetro a la cláusula `OPENROWSET`, como se muestra aquí (no olvides añadir una coma después del parámetro anterior):

    ```SQL
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/csv/**',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0',
            HEADER_ROW = TRUE
        ) AS [result]
    ```

6. En la lista **Conectar a**, asegúrese de que **Integrado** está seleccionado: representa el grupo de SQL integrado que se creó con el área de trabajo. En la barra de herramientas, usa el botón **▷ Ejecutar** para ejecutar el código SQL y revisa los resultados, que deben tener un aspecto similar a lo siguiente:

    | SalesOrderNumber | SalesOrderLineNumber | OrderDate | CustomerName | EmailAddress | Elemento | Quantity | UnitPrice | TaxAmount |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | SO43701 | 1 | 2019-07-01 | Christy Zhu | christy12@adventure-works.com |Mountain-100 Silver, 44 | 1 | 3399,99 | 271.9992 |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... |

7. Publica los cambios en el script y luego cierra el panel de scripts.

## Transformación de datos mediante instrucciones CREATE EXTERNAL TABLE AS SELECT (CETAS)

Una manera sencilla de usar SQL para transformar datos en un archivo y conservar los resultados en otro archivo es usar una instrucción CREATE EXTERNAL TABLE AS SELECT (CETAS). Esta instrucción crea una tabla basada en las solicitudes de una consulta, pero los datos de la tabla se almacenan como archivos en un lago de datos. A continuación, se pueden consultar los datos transformados a través de la tabla externa o acceder directamente al sistema de archivos (por ejemplo, para incluirlos en un proceso descendente para cargar los datos transformados en un almacén de datos).

### Crear un origen de datos externo y un formato de archivo

Al definir un origen de datos externo en una base de datos, puedes utilizarlo para hacer referencia a la ubicación del lago de datos en la que deseas almacenar los archivos de las tablas externas. Un formato de archivo externo permite definir el formato de esos archivos: por ejemplo, Parquet o CSV. Para utilizar estos objetos y trabajar con tablas externas, necesitas crearlos en una base de datos distinta de la base de datos **maestra** predeterminada.

1. En Synapse Studio, en la página **Desarrollo**, en el menú **+**, selecciona **Script SQL**.
2. En el nuevo panel de script, agrega el código siguiente (reemplazando *datalakexxxxxxx* por el nombre de la cuenta de almacenamiento del lago de datos) para crear una nueva base de datos y agregarle un origen de datos externo.

    ```sql
    -- Database for sales data
    CREATE DATABASE Sales
      COLLATE Latin1_General_100_BIN2_UTF8;
    GO;
    
    Use Sales;
    GO;
    
    -- External data is in the Files container in the data lake
    CREATE EXTERNAL DATA SOURCE sales_data WITH (
        LOCATION = 'https://datalakexxxxxxx.dfs.core.windows.net/files/'
    );
    GO;
    
    -- Format for table files
    CREATE EXTERNAL FILE FORMAT ParquetFormat
        WITH (
                FORMAT_TYPE = PARQUET,
                DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'
            );
    GO;
    ```

3. Modifica las propiedades del script para cambiar su nombre a **Crear base de datos de ventas** y publícalo.
4. Asegúrate de que el script está conectado al grupo de SQL **integrado** y a la base de datos **maestra** y, a continuación, ejecútalo.
5. Vuelve a la página **Datos** y usa el botón **↻** situado en la parte superior derecha de Synapse Studio para actualizar la página. A continuación, ve la pestaña **Área de trabajo** en el panel **Datos**, donde ahora se muestra una lista de **bases de datos SQL**. Expande esta lista para comprobar que se ha creado la base de datos **Sales**.
6. Amplía la base de datos **Ventas**, su carpeta **Recursos externos** y la carpeta **Orígenes de datos externos** que hay debajo para ver el origen de datos externo **sales_data** que creaste.

### Crear una tabla externa

1. In Synapse Studio, en la página **Desarrollo**, en el menú **+**, selecciona **Script SQL**.
2. En el nuevo panel de script, agrega el siguiente código para recuperar y agregar datos de los archivos de ventas CSV utilizando el origen de datos externo, teniendo en cuenta que la ruta **BULK** es relativa a la ubicación de la carpeta en la que está definido el origen de datos:

    ```sql
    USE Sales;
    GO;
    
    SELECT Item AS Product,
           SUM(Quantity) AS ItemsSold,
           ROUND(SUM(UnitPrice) - SUM(TaxAmount), 2) AS NetRevenue
    FROM
        OPENROWSET(
            BULK 'sales/csv/*.csv',
            DATA_SOURCE = 'sales_data',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0',
            HEADER_ROW = TRUE
        ) AS orders
    GROUP BY Item;
    ```

3. Ejecute el script. El resultado debería ser similar al siguiente:

    | Producto | ItemsSold | NetRevenue |
    | -- | -- | -- |
    | AWC Logo Cap | 1063 | 8791,86 |
    | ... | ... | ... |

4. Modifica el código SQL para guardar los resultados de la consulta en una tabla externa, de la siguiente manera:

    ```sql
    CREATE EXTERNAL TABLE ProductSalesTotals
        WITH (
            LOCATION = 'sales/productsales/',
            DATA_SOURCE = sales_data,
            FILE_FORMAT = ParquetFormat
        )
    AS
    SELECT Item AS Product,
        SUM(Quantity) AS ItemsSold,
        ROUND(SUM(UnitPrice) - SUM(TaxAmount), 2) AS NetRevenue
    FROM
        OPENROWSET(
            BULK 'sales/csv/*.csv',
            DATA_SOURCE = 'sales_data',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0',
            HEADER_ROW = TRUE
        ) AS orders
    GROUP BY Item;
    ```

5. Ejecute el script. Esta vez no hay ninguna salida, pero el código debería haber creado una tabla externa basada en los resultados de la consulta.
6. Asigna al script el nombre **Crear tabla ProductSalesTotals** y publícalo.
7. En la página **datos**, en la pestaña **Área de trabajo**, consulta el contenido de la carpeta **Tablas externas** de la base de datos SQL **Ventas** para comprobar que se ha creado una nueva tabla denominada **ProductSalesTotals**.
8. En el menú **...** de la tabla **ProductSalesTotals**, selecciona **Nuevo script SQL** > **Seleccionar las 100 primeras filas**. A continuación, ejecuta el script resultante y comprueba que devuelve los datos agregados de ventas de productos.
9. En la pestaña **archivos** que contiene el sistema de archivos del lago de datos, consulta el contenido de la carpeta **ventas** (actualizando la vista si es necesario) y comprueba que se ha creado una nueva carpeta **productsales**.
10. En la carpeta **productsales**, observa que se han creado uno o varios archivos con nombres similares a ABC123DE----.parquet. Estos archivos contienen los datos agregados de ventas de productos. Para demostrarlo, puedes seleccionar uno de los archivos y utilizar el menú **Nuevo script SQL** > **Seleccionar las 100 primeras filas** para consultarlo directamente.

## Encapsular una transformación de datos en un procedimiento almacenado

Si necesitas transformar datos con frecuencia, puedes utilizar un procedimiento almacenado para encapsular una instrucción CETAS.

1. En Synapse Studio, en la página **Desarrollo**, en el menú **+**, selecciona **Script SQL**.
2. En el nuevo panel de script, agrega el siguiente código para crear un procedimiento almacenado en la base de datos **Ventas** que agregue las ventas por año y guarde los resultados en una tabla externa:

    ```sql
    USE Sales;
    GO;
    CREATE PROCEDURE sp_GetYearlySales
    AS
    BEGIN
        -- drop existing table
        IF EXISTS (
                SELECT * FROM sys.external_tables
                WHERE name = 'YearlySalesTotals'
            )
            DROP EXTERNAL TABLE YearlySalesTotals
        -- create external table
        CREATE EXTERNAL TABLE YearlySalesTotals
        WITH (
                LOCATION = 'sales/yearlysales/',
                DATA_SOURCE = sales_data,
                FILE_FORMAT = ParquetFormat
            )
        AS
        SELECT YEAR(OrderDate) AS CalendarYear,
                SUM(Quantity) AS ItemsSold,
                ROUND(SUM(UnitPrice) - SUM(TaxAmount), 2) AS NetRevenue
        FROM
            OPENROWSET(
                BULK 'sales/csv/*.csv',
                DATA_SOURCE = 'sales_data',
                FORMAT = 'CSV',
                PARSER_VERSION = '2.0',
                HEADER_ROW = TRUE
            ) AS orders
        GROUP BY YEAR(OrderDate)
    END
    ```

3. Ejecuta el script para crear el procedimiento almacenado.
4. En el código que acabas de ejecutar, agrega el siguiente código para llamar al procedimiento almacenado:

    ```sql
    EXEC sp_GetYearlySales;
    ```

5. Selecciona solo la instrucción `EXEC sp_GetYearlySales;` que acabas de agregar y utiliza el botón **▷ Ejecutar** para ejecutarla.
6. En la pestaña **archivos** que contiene el sistema de archivos del lago de datos, consulta el contenido de la carpeta **ventas** (actualizando la vista si es necesario) y comprueba que se creó una nueva carpeta **yearlysales**.
7. En la carpeta **yearlysales**, observa que se creó un archivo parquet que contiene los datos agregados de ventas anuales.
8. Vuelve al script SQL, vuelve a ejecutar la instrucción `EXEC sp_GetYearlySales;` y observa que se produce un error.

    Aunque el script elimina la tabla externa, la carpeta que contiene los datos no se elimina. Para volver a ejecutar el procedimiento almacenado (por ejemplo, como parte de una canalización de transformación de datos programada), debes eliminar los datos antiguos.

9. Vuelve a la pestaña **archivos** y consulta la carpeta **sales**. Después, selecciona la carpeta **yearlysales** y elimínala.
10. Vuelve al script SQL y vuelve a ejecutar la instrucción `EXEC sp_GetYearlySales;`. Esta vez, la operación se realiza correctamente y se genera un nuevo archivo de datos.

## Eliminación de recursos de Azure

Si ha terminado de explorar Azure Synapse Analytics, debe eliminar los recursos que ha creado para evitar costos innecesarios de Azure.

1. Cierre la pestaña del explorador de Synapse Studio y vuelva a Azure Portal.
2. En Azure Portal, en la página **Inicio**, seleccione **Grupos de recursos**.
3. Selecciona el grupo de recursos **dp203-*xxxxxxx*** del área de trabajo de Synapse Analytics (no el grupo de recursos administrado) y comprueba que contiene el área de trabajo de Synapse, la cuenta de almacenamiento y el grupo de Spark del área de trabajo.
4. En la parte superior de la página **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
5. Especifica el nombre del grupo de recursos **dp203-*xxxxxxx*** para confirmar que quieres eliminarlo y selecciona **Eliminar**.

    Después de unos minutos, se eliminarán el grupo de recursos de área de trabajo de Azure Synapse y el grupo de recursos de área de trabajo administrado asociado a él.
