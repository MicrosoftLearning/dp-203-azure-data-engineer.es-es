---
lab:
  title: Explorar Azure Databricks
  ilt-use: Suggested demo
---

# Explorar Azure Databricks

Azure Databricks es una versión basada en Microsoft Azure de la conocida plataforma de código abierto Databricks.

De forma similar a Azure Synapse Analytics, un *área de trabajo* de Azure Databricks proporciona un punto central para administrar clústeres, datos y recursos de Databricks en Azure.

Este ejercicio debería tardar en completarse **30** minutos aproximadamente.

## Antes de empezar

Necesitará una [suscripción de Azure](https://azure.microsoft.com/free) en la que tenga acceso de nivel administrativo.

## Aprovisiona un área de trabajo de Azure Databricks.

En este ejercicio, usarás un script para aprovisionar una nueva área de trabajo de Azure Databricks.

> **Sugerencia**: si ya tienes un área de trabajo de Azure Databricks *Estándar* o de *Evaluación*, puedes omitir este procedimiento y usar el área de trabajo existente.

1. En un explorador web, inicia sesión en [Azure Portal](https://portal.azure.com) en `https://portal.azure.com`.
2. Usa el botón **[\>_]** a la derecha de la barra de búsqueda en la parte superior de la página para crear un nuevo Cloud Shell en Azure Portal, selecciona un entorno de ***PowerShell*** y crea almacenamiento si se te solicita. Cloud Shell proporciona una interfaz de línea de comandos en un panel situado en la parte inferior de Azure Portal, como se muestra a continuación:

    ![Azure Portal con un panel de Cloud Shell](./images/cloud-shell.png)

    > **Nota**: si creaste anteriormente un Cloud Shell que usa un entorno de *Bash*, usa el menú desplegable situado en la parte superior izquierda del panel de Cloud Shell para cambiarlo a ***PowerShell***.

3. Ten en cuenta que puedes cambiar el tamaño de Cloud Shell arrastrando la barra de separación en la parte superior del panel, o usando los iconos **&#8212;** , **&#9723;** y **X** en la parte superior derecha para minimizar, maximizar y cerrar el panel. Para obtener más información sobre el uso de Azure Cloud Shell, consulta la [documentación de Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. En el panel de PowerShell, introduce los siguientes comandos para clonar este repositorio:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Una vez clonado el repositorio, escribe los siguientes comandos para cambiar a la carpeta de este laboratorio y ejecuta el script **setup.ps1** que contiene:

    ```
    cd dp-203/Allfiles/labs/23
    ./setup.ps1
    ```

6. Si se solicita, elige la suscripción que quieres usar (esto solo ocurrirá si tienes acceso a varias suscripciones de Azure).

7. Espera a que se complete el script: normalmente tarda unos 5 minutos, pero en algunos casos puede tardar más. Mientras esperas, consulta el artículo [Qué es Azure Databricks](https://learn.microsoft.com/azure/databricks/introduction/) en la documentación de Azure Databricks.

## Crear un clúster

Azure Databricks es una plataforma de procesamiento distribuido que usa clústeres* de Apache Spark *para procesar datos en paralelo en varios nodos. Cada clúster consta de un nodo de controlador para coordinar el trabajo y nodos de trabajo para hacer tareas de procesamiento.

En este ejercicio, crearás un clúster de *nodo único* para minimizar los recursos de proceso usados en el entorno de laboratorio (en los que se pueden restringir los recursos). En un entorno de producción, normalmente crearías un clúster con varios nodos de trabajo.

> **Sugerencia**: si ya tienes un clúster con una versión en tiempo de ejecución 13.3 LTS en el área de trabajo de Azure Databricks, puedes usarlo para completar este ejercicio y omitir este procedimiento.

1. En Azure Portal, ve al grupo de recursos ** dp203-*xxxxxxx*** que ha creado el script (o el grupo de recursos que contiene el área de trabajo de Azure Databricks existente)
1. Selecciona el recurso de Azure Databricks Service (denominado **databricks*xxxxxxx*** si has usado el script de instalación para crearlo).
1. En la página **Información general** del área de trabajo, usa el botón **Inicio del área de trabajo** para abrir el área de trabajo de Azure Databricks en una nueva pestaña del explorador; inicia sesión si se solicita.

    > **Sugerencia**: al usar el portal del área de trabajo de Databricks, se pueden mostrar varias sugerencias y notificaciones. Descarta estos elementos y sigue las instrucciones proporcionadas para completar las tareas de este ejercicio.

1. Visualiza el portal del área de trabajo de Azure Databricks y observa que la barra lateral del lado izquierdo contiene vínculos para los distintos tipos de tareas que puedes realizar.

1. Selecciona el vínculo **(+) Nuevo** en la barra lateral y luego selecciona **Clúster**.
1. En la página **Nuevo clúster**, crea un clúster con la siguiente configuración:
    - **Nombre del clúster**: clúster del *Nombre de usuario*  (el nombre del clúster predeterminado)
    - **Modo de clúster** de un solo nodo
    - **Modo de acceso**: usuario único (*con la cuenta de usuario seleccionada*)
    - **Versión del entorno de ejecución de Databricks**: 13.3 LTS (Spark 3.4.1, Scala 2.12)
    - **Usar aceleración de Photon**: seleccionado
    - **Tipo de nodo**: Standard_DS3_v2.
    - **Finaliza después de ***30***minutos de inactividad**

1. Espera a que se cree el clúster. Esto puede tardar un par de minutos.

> **Nota**: si el clúster no se inicia, es posible que la suscripción no tenga cuota suficiente en la región donde se aprovisiona el área de trabajo de Azure Databricks. Para obtener más información, consulta [El límite de núcleos de la CPU impide la creación de clústeres](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). Si esto sucede, puedes intentar eliminar el área de trabajo y crear una nueva en otra región. Puedes especificar una región como parámetro para el script de configuración de la siguiente manera: `./setup.ps1 eastus`

## Usar Spark para analizar un archivo de datos

Como en muchos entornos de Spark, Databricks admite el uso de cuadernos para combinar notas y celdas de código interactivas que puedes usar para explorar datos.

1. En la barra lateral, usa el vínculo **(+) Nuevo** para crear un **cuaderno**.
1. Cambia el nombre predeterminado del cuaderno (**Cuaderno sin título *[fecha]***) por **Explorar productos** y en la lista desplegable **Conectar**, selecciona el clúster si aún no está seleccionado. Si el clúster no se está ejecutando, puede tardar un minuto en iniciarse.
1. Descarga el archivo [**products.csv**](https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/23/adventureworks/products.csv) en el equipo local y guárdalo como **products.csv**. Después, en el cuaderno **Explorar productos**, en el menú **Archivo**, selecciona **Cargar datos a DBFS**.
1. En el cuadro de diálogo **Cargar datos**, fíjate en el **Directorio de destino de DBFS** al que se cargará el archivo. A continuación, selecciona el área **Archivos** y carga en tu equipo el archivo **products.csv** que descargaste. Una vez cargado el archivo, selecciona **Siguiente**.
1. En el panel **Acceso a archivos desde cuadernos**, selecciona el código PySpark de muestra y cópialo en el portapapeles. Lo usarás para cargar los datos del archivo en un DataFrame. A continuación, seleccione **Done** (Listo).
1. En el cuaderno **Explorar productos**, en la celda de código vacía, pega el código que copiaste; que debería tener un aspecto similar al siguiente:

    ```python
    df1 = spark.read.format("csv").option("header", "true").load("dbfs:/FileStore/shared_uploads/user@outlook.com/products.csv")
    ```

1. Usa la opción de menú **▸ Ejecutar celda** en la parte superior derecha de la celda para ejecutarla e inicia y asocia el clúster si se te solicita.
1. Espera a que el código ejecute el trabajo de Spark. El código creó un objeto *dataframe* denominado **df1** a partir de los datos del archivo que cargaste.
1. Debajo de la celda de código existente, usa el icono **+** para agregar una nueva celda de código. Después, en la nueva celda, escribe el siguiente código:

    ```python
    display(df1)
    ```

1. Usa la opción de menú **▸ Ejecutar celda** situada en la parte superior derecha de la nueva celda para ejecutarla. Este código muestra el contenido de dataframe, que debería tener el siguiente aspecto:

    | ProductID | ProductName | Category | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | Bicicletas de montaña | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | Bicicletas de montaña | 3399.9900 |
    | ... | ... | ... | ... |

1. Encima de la tabla de resultados, selecciona **+** y luego **Visualización** para ver el editor de visualización y luego aplica las siguientes opciones:
    - **Tipo de visualización**: barra
    - **Columna X**: categoría
    - **Columna Y**: *agrega una nueva columna y selecciona ***ProductID**. *Aplica la **agregación* de la **cuenta**.

    Guarda la visualización y observa que aparece en el cuaderno, del siguiente modo:

    ![Un gráfico de barras con los recuentos de productos por categoría](./images/databricks-chart.png)

## Crear y consultar una tabla

Aunque muchos análisis de datos pueden usar cómodamente lenguajes como Python o Scala para trabajar con datos en archivos, muchas soluciones de análisis de datos se basan en bases de datos relacionales, en las que los datos se almacenan en tablas y se manipulan con SQL.

1. En el cuaderno **Explorar productos**, bajo la salida del gráfico de la celda de código que se ejecutó anteriormente, usa el icono **+** para agregar una nueva celda.
2. Escribe y ejecuta el siguiente código en la nueva celda:

    ```python
    df1.write.saveAsTable("products")
    ```

3. Cuando la celda se haya completado, agrega una nueva celda debajo con el siguiente código:

    ```sql
    %sql

    SELECT ProductName, ListPrice
    FROM products
    WHERE Category = 'Touring Bikes';
    ```

4. Ejecuta la nueva celda, que contiene código SQL, para devolver el nombre y el precio de los productos en la categoría *Bicicletas de paseo*.
5. En la barra lateral, selecciona el vínculo **Catálogo** y comprueba que la tabla **productos** se ha creado en el esquema de base de datos predeterminado (que, como era de esperar, se llama **predeterminado**). Es posible usar código Spark para crear esquemas de bases de datos personalizados y un esquema de tablas relacionales que los analistas de datos pueden usar para explorar datos y generar informes analíticos.

## Eliminar los recursos de Azure Databricks

Ahora que has terminado de explorar Azure Databricks, debes eliminar los recursos que has creado para evitar costos innecesarios de Azure y liberar capacidad en tu suscripción.

1. Cierra la pestaña del explorador del área de trabajo de Azure Databricks y vuelve a Azure Portal.
2. En Azure Portal, en la página **Inicio**, seleccione **Grupos de recursos**.
3. Selecciona el grupo de recursos ** dp203-*xxxxxxx*** (no el grupo de recursos administrado) y comprueba que contiene el área de trabajo de Azure Databricks.
4. En la parte superior de la página **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
5. Escribe el nombre del grupo de recursos **dp203-*xxxxxxx*** para confirmar que quieres eliminarlo y selecciona **Eliminar**.

    Después de unos minutos, tu grupo de recursos y el grupo de recursos del área de trabajo administrada asociado a él se eliminarán.
