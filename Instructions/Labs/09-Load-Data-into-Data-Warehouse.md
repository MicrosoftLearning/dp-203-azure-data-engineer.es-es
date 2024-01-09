---
lab:
  title: Carga de datos en un almacenamiento de datos relacional
  ilt-use: Lab
---

# Carga de datos en un almacenamiento de datos relacional

En este ejercicio, vas a cargar datos en un grupo de SQL dedicado.

Este ejercicio debería tardar en completarse **30** minutos aproximadamente.

## Antes de empezar

Necesitará una [suscripción de Azure](https://azure.microsoft.com/free) en la que tenga acceso de nivel administrativo.

## Aprovisionar un área de trabajo de Azure Synapse Analytics

Necesitarás un área de trabajo de Azure Synapse Analytics con acceso a Data Lake Storage y un grupo de SQL dedicado que hospede un almacenamiento de datos.

En este ejercicio, usarás una combinación de un script de PowerShell y una plantilla de ARM para aprovisionar un área de trabajo de Azure Synapse Analytics.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) en `https://portal.azure.com`.
2. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** y crear un almacenamiento si se te solicita. Cloud Shell proporciona una interfaz de línea de comandos en un panel situado en la parte inferior de Azure Portal, como se muestra a continuación:

    ![Azure Portal con un panel de Cloud Shell](./images/cloud-shell.png)

    > **Nota**: Si has creado previamente un Cloud Shell que usa un entorno de *Bash*, usa el menú desplegable situado en la parte superior izquierda del panel de Cloud Shell para cambiarlo a ***PowerShell***.

3. Ten en cuenta que puedes cambiar el tamaño de Cloud Shell arrastrando la barra de separación en la parte superior del panel, o usando los iconos —, **◻** y **X** en la parte superior derecha para minimizar, maximizar y cerrar el panel. Para obtener más información sobre el uso de Azure Cloud Shell, consulte la [documentación de Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. En el panel de PowerShell, escribe los siguientes comandos para clonar este repositorio:

    ```powershell
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Una vez clonado el repositorio, escribe los siguientes comandos para cambiar a la carpeta de este ejercicio y ejecuta el script **setup.ps1** que contiene:

    ```powershell
    cd dp-203/Allfiles/labs/09
    ./setup.ps1
    ```

6. Si se te solicita, elige la suscripción que deseas usar (esta opción solo se producirá si tienes acceso a varias suscripciones de Azure).
7. Cuando se te solicite, escribe una contraseña adecuada para el grupo de SQL de Azure Synapse.

    > **Nota**: No olvides la contraseña.

8. Espere a que se complete el script: normalmente tarda unos 10 minutos, pero en algunos casos tardará más. Mientras esperas, revisa el artículo [Estrategias de carga de datos para un grupo de SQL dedicado en Azure Synapse Analytics](https://learn.microsoft.com/azure/synapse-analytics/sql-data-warehouse/design-elt-data-loading) en la documentación de Azure Synapse Analytics.

## Preparación para cargar datos

1. Una vez completado el script, en Azure Portal, ve al grupo de recursos **dp203-*xxxxxxx*** que creaste y selecciona el área de trabajo de Synapse.
2. En la **página Información general** del área de trabajo de Synapse, en la pestaña **Abrir Synapse Studio**, selecciona **Abrir** para abrir Synapse Studio en una nueva pestaña del explorador; inicia sesión si se te solicita.
3. En el lado izquierdo de Synapse Studio, usa el icono ›› para expandir el menú; se mostrarán las distintas páginas de Synapse Studio que usarás para administrar recursos y realizar tareas de análisis de datos.
4. En la página **Administrar**, en la pestaña **Grupos de SQL**, selecciona la fila del grupo de SQL dedicado **sql*xxxxxxx***, que hospeda el almacenamiento de datos de este ejercicio, y usa el icono **▷** para iniciarlo; confirma que quieres reanudarlo cuando se te solicite.

    La reanudación de un grupo puede tardar varios minutos. Puedes usar el botón **↻ Actualizar** para comprobar su estado periódicamente. El estado aparecerá como **En línea** cuando esté listo. Mientras esperas, continúe con los pasos siguientes para ver los archivos de datos que cargarás.

5. En la página **Datos**, consulta la pestaña **Vinculado** y comprueba que el área de trabajo incluye un vínculo a la cuenta de almacenamiento de Azure Data Lake Storage Gen2, que debe tener un nombre similar a **synapsexxxxxxxxxx (Primary - datalakexxxxxxx)**.
6. Expande la cuenta de almacenamiento y comprueba que incluye un contenedor del sistema de archivos denominado **files (primary)**.
7. Selecciona el contenedor de archivos y observa que contiene una carpeta denominada **datos**. Esta carpeta contiene los archivos de datos que vas a cargar en el almacenamiento de datos.
8. Abre la carpeta de **data** y observa que contiene archivos .csv de datos de clientes y productos.
9. Haz clic con el botón derecho en cualquiera de los archivos y selecciona **Vista previa** para ver los datos que contiene. Ten en cuenta que los archivos contienen una fila de encabezado, por lo que puedes seleccionar la opción para mostrar encabezados de columna.
10. Vuelve a la página **Administrar** y comprueba que el grupo de SQL dedicado está en línea.

## Carga de tablas de almacenamiento de datos

Echemos un vistazo a algunos enfoques basados en SQL para cargar datos en el almacenamiento de datos.

1. En la página **Datos**, selecciona la pestaña **Área de trabajo**.
2. Expande **Bases de datos de SQL** y selecciona tu base de datos **sql*xxxxxxx***. Luego, en su menú <bpt ctype="x-unknown" id="1" rid="1"><bpt xmlns="urn:oasis:names:tc:xliff:document:1.2" id="p1">**</bpt></bpt>...<ept id="2" rid="1"><ept xmlns="urn:oasis:names:tc:xliff:document:1.2" id="p1">**</ept></ept>, selecciona <bpt ctype="x-unknown" id="3" rid="2"><bpt xmlns="urn:oasis:names:tc:xliff:document:1.2" id="p2">**</bpt></bpt>Nuevo script de SQL<ept id="4" rid="2"><ept xmlns="urn:oasis:names:tc:xliff:document:1.2" id="p2">**</ept></ept><ph ctype="x-unknown" id="5"><ph xmlns="urn:oasis:names:tc:xliff:document:1.2" id="ph1"> > 
</ph></ph><bpt ctype="x-unknown" id="6" rid="3"><bpt xmlns="urn:oasis:names:tc:xliff:document:1.2" id="p3">**</bpt></bpt>Empty Script<ept id="7" rid="3"><ept xmlns="urn:oasis:names:tc:xliff:document:1.2" id="p3">**</ept></ept>.

Ahora tienes una página de SQL en blanco, que está conectada a la instancia para los ejercicios siguientes. Usarás este script para explorar varias técnicas de SQL que puedes usar para cargar datos.

### Carga de datos desde un lago de datos mediante la instrucción COPY

1. En tu script SQL, escribe el siguiente código en la ventana.

    ```sql
    SELECT COUNT(1) 
    FROM dbo.StageProduct
    ```

2. En la barra de herramientas, usa el botón **▷ Ejecutar** para ejecutar el código SQL y confirmar que hay **0** filas actualmente en la tabla **StageProduct**.
3. Reemplaza el código por la siguiente instrucción COPY (cambiando **datalake*xxxxxx*** por el nombre de tu lago de datos):

    ```sql
    COPY INTO dbo.StageProduct
        (ProductID, ProductName, ProductCategory, Color, Size, ListPrice, Discontinued)
    FROM 'https://datalakexxxxxx.blob.core.windows.net/files/data/Product.csv'
    WITH
    (
        FILE_TYPE = 'CSV',
        MAXERRORS = 0,
        IDENTITY_INSERT = 'OFF',
        FIRSTROW = 2 --Skip header row
    );


    SELECT COUNT(1) 
    FROM dbo.StageProduct
    ```

4. Ejecuta el script y revisa los resultados. Se deberían haber cargado 11 filas en la tabla **StageProduct**.

    Ahora vamos a usar la misma técnica para cargar otra tabla, registrando los errores que puedan producirse.

5. Reemplaza el código SQL del panel de script por el siguiente código, cambiando **datalake*xxxxxx*** por el nombre de tu lago de datos tanto en ```FROM``` como en las cláusulas ```ERRORFILE```:

    ```sql
    COPY INTO dbo.StageCustomer
    (GeographyKey, CustomerAlternateKey, Title, FirstName, MiddleName, LastName, NameStyle, BirthDate, 
    MaritalStatus, Suffix, Gender, EmailAddress, YearlyIncome, TotalChildren, NumberChildrenAtHome, EnglishEducation, 
    SpanishEducation, FrenchEducation, EnglishOccupation, SpanishOccupation, FrenchOccupation, HouseOwnerFlag, 
    NumberCarsOwned, AddressLine1, AddressLine2, Phone, DateFirstPurchase, CommuteDistance)
    FROM 'https://datalakexxxxxx.dfs.core.windows.net/files/data/Customer.csv'
    WITH
    (
    FILE_TYPE = 'CSV'
    ,MAXERRORS = 5
    ,FIRSTROW = 2 -- skip header row
    ,ERRORFILE = 'https://datalakexxxxxx.dfs.core.windows.net/files/'
    );
    ```

6. Ejecuta el script y revisa el mensaje resultante. El archivo de origen contiene una fila con datos no válidos, por lo que se rechaza una fila. El código anterior especifica un máximo de **5** errores, por lo que un único error no debería haber impedido que se cargaran las filas válidas. Puedes ver las filas que se *han* cargado ejecutando la siguiente consulta.

    ```sql
    SELECT *
    FROM dbo.StageCustomer
    ```

7. En la pestaña **archivos**, consulta la carpeta raíz de tu lago de datos y comprueba que se ha creado una nueva carpeta denominada **_rejectedrows** (si no ves esta carpeta, en el menú **Más**, selecciona **Actualizar** para actualizar la vista).
8. Abre la carpeta **_rejectedrows** y la subcarpeta específica de fecha y hora que contiene, y observa que se han creado archivos con nombres similares a ***QID123_1_2*.Error.Txt** y ***QID123_1_2*.Row.Txt**. Puede hacer clic con el botón derecho en cada uno de estos archivos y seleccionar **Vista previa** para ver los detalles del error y la fila rechazada.

    El uso de tablas de almacenamiento provisional permite validar o transformar los datos antes de moverlos o usarlos para anexarlos o insertarlos en cualquier tabla de dimensiones existente. La instrucción COPY proporciona una técnica sencilla, pero de alto rendimiento, que puedes usar para cargar fácilmente datos de archivos de un lago de datos en tablas de almacenamiento provisional y, como has visto, identificar y redireccionar filas no válidas.

### Usar una instrucción CREATE TABLE AS (CTAS)

1. Vuelve al panel de script y reemplaza el código que contiene por el siguiente código:

    ```sql
    CREATE TABLE dbo.DimProduct
    WITH
    (
        DISTRIBUTION = HASH(ProductAltKey),
        CLUSTERED COLUMNSTORE INDEX
    )
    AS
    SELECT ROW_NUMBER() OVER(ORDER BY ProductID) AS ProductKey,
        ProductID AS ProductAltKey,
        ProductName,
        ProductCategory,
        Color,
        Size,
        ListPrice,
        Discontinued
    FROM dbo.StageProduct;
    ```

2. Ejecuta el script, que creará una nueva tabla denominada **DimProduct** a partir de los datos de producto provisionales que usan **ProductAltKey** como clave de distribución hash y tienen un índice de almacén de columnas agrupado.
4. Usa la siguiente consulta para ver el contenido de la nueva tabla **DimProduct**:

    ```sql
    SELECT ProductKey,
        ProductAltKey,
        ProductName,
        ProductCategory,
        Color,
        Size,
        ListPrice,
        Discontinued
    FROM dbo.DimProduct;
    ```

    La expresión CREATE TABLE AS SELECT (CTAS) tiene varios usos, entre los que se incluyen los siguientes:

    - Redistribuir la clave hash de una tabla y que se vincule con otras tablas para mejorar el rendimiento de las consultas.
    - Asignar una clave suplente a una tabla de almacenamiento provisional en función de los valores existentes después de realizar un análisis delta.
    - Crear tablas de agregado rápidamente con fines informativos.

### Combinar las instrucciones INSERT y UPDATE para cargar una tabla de dimensiones de variación lenta

La tabla **DimCustomer** admite dimensiones de variación lenta de tipo 1 y tipo 2 (SCD), donde los cambios de tipo 1 dan lugar a una actualización local a una fila existente y los cambios de tipo 2 dan lugar a una nueva fila para indicar la versión más reciente de una instancia de entidad de dimensión determinada. Para cargar esta tabla se necesita una combinación de instrucciones INSERT (para cargar nuevos clientes) e instrucciones UPDATE (para aplicar cambios de tipo 1 o tipo 2).

1. En el panel de consulta, reemplaza el código SQL existente por el siguiente código:

    ```sql
    INSERT INTO dbo.DimCustomer ([GeographyKey],[CustomerAlternateKey],[Title],[FirstName],[MiddleName],[LastName],[NameStyle],[BirthDate],[MaritalStatus],
    [Suffix],[Gender],[EmailAddress],[YearlyIncome],[TotalChildren],[NumberChildrenAtHome],[EnglishEducation],[SpanishEducation],[FrenchEducation],
    [EnglishOccupation],[SpanishOccupation],[FrenchOccupation],[HouseOwnerFlag],[NumberCarsOwned],[AddressLine1],[AddressLine2],[Phone],
    [DateFirstPurchase],[CommuteDistance])
    SELECT *
    FROM dbo.StageCustomer AS stg
    WHERE NOT EXISTS
        (SELECT * FROM dbo.DimCustomer AS dim
        WHERE dim.CustomerAlternateKey = stg.CustomerAlternateKey);

    -- Type 1 updates (change name, email, or phone in place)
    UPDATE dbo.DimCustomer
    SET LastName = stg.LastName,
        EmailAddress = stg.EmailAddress,
        Phone = stg.Phone
    FROM DimCustomer dim inner join StageCustomer stg
    ON dim.CustomerAlternateKey = stg.CustomerAlternateKey
    WHERE dim.LastName <> stg.LastName OR dim.EmailAddress <> stg.EmailAddress OR dim.Phone <> stg.Phone

    -- Type 2 updates (address changes triggers new entry)
    INSERT INTO dbo.DimCustomer
    SELECT stg.GeographyKey,stg.CustomerAlternateKey,stg.Title,stg.FirstName,stg.MiddleName,stg.LastName,stg.NameStyle,stg.BirthDate,stg.MaritalStatus,
    stg.Suffix,stg.Gender,stg.EmailAddress,stg.YearlyIncome,stg.TotalChildren,stg.NumberChildrenAtHome,stg.EnglishEducation,stg.SpanishEducation,stg.FrenchEducation,
    stg.EnglishOccupation,stg.SpanishOccupation,stg.FrenchOccupation,stg.HouseOwnerFlag,stg.NumberCarsOwned,stg.AddressLine1,stg.AddressLine2,stg.Phone,
    stg.DateFirstPurchase,stg.CommuteDistance
    FROM dbo.StageCustomer AS stg
    JOIN dbo.DimCustomer AS dim
    ON stg.CustomerAlternateKey = dim.CustomerAlternateKey
    AND stg.AddressLine1 <> dim.AddressLine1;
    ```

2. Ejecuta el script y revisa la salida.

## Realizar la optimización posterior a la carga

Después de cargar nuevos datos en el almacenamiento de datos, es recomendable recompilar los índices de las tablas y actualizar las estadísticas de las columnas consultadas habitualmente.

1. Reemplaza el código en el panel de script por el siguiente código:

    ```sql
    ALTER INDEX ALL ON dbo.DimProduct REBUILD;
    ```

2. Ejecuta el script para recompilar los índices de la tabla **DimProduct**.
3. Reemplaza el código en el panel de script por el siguiente código:

    ```sql
    CREATE STATISTICS customergeo_stats
    ON dbo.DimCustomer (GeographyKey);
    ```

4. Ejecuta el script para crear o actualizar estadísticas en la columna **GeographyKey** de la tabla **DimCustomer**.

## Eliminación de recursos de Azure

Si ha terminado de explorar Azure Synapse Analytics, debe eliminar los recursos que ha creado para evitar costos innecesarios de Azure.

1. Cierre la pestaña del explorador de Synapse Studio y vuelva a Azure Portal.
2. En Azure Portal, en la página **Inicio**, seleccione **Grupos de recursos**.
3. Selecciona el grupo de recursos **dp203-*xxxxxxx*** para tu área de trabajo de Synapse Analytics (no el grupo de recursos administrados) y verifica que contiene el área de trabajo de Synapse, la cuenta de almacenamiento y el grupo de Spark para tu área de trabajo.
4. En la parte superior de la página **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
5. Especifica el nombre del grupo de recursos **dp203-*xxxxxxx*** para confirmar que quieres eliminarlo y selecciona **Eliminar**.

    Después de unos minutos, tu grupo de recursos del área de trabajo de Azure Synapse y el grupo de recursos del área de trabajo administrada asociada se eliminarán.
