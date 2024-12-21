---
lab:
  title: Uso de un almacén de SQL en Azure Databricks
  ilt-use: Optional demo
---

SQL es un lenguaje estándar del sector para consultar y manipular datos. Muchos analistas de datos realizan análisis de datos mediante SQL para consultar tablas en una base de datos relacional. Azure Databricks incluye funcionalidad de SQL que se basa en tecnologías de Spark y Delta Lake para proporcionar una capa de base de datos relacional a través de archivos en un lago de datos.

Este ejercicio debería tardar en completarse **30** minutos aproximadamente.

## Aprovisiona un área de trabajo de Azure Databricks.

> **Sugerencia**: si ya tienes un espacio de trabajo *premium* o de *prueba* Azure Databricks, puedes omitir este procedimiento y usar tu espacio de trabajo existente.

En este ejercicio, se incluye un script para aprovisionar una nueva área de trabajo de Azure Databricks. El script intenta crear un recurso de área de trabajo de Azure Databricks de nivel *Premium* en una región en la que la suscripción de Azure tiene cuota suficiente para los núcleos de proceso necesarios en este ejercicio, y da por hecho que la cuenta de usuario tiene permisos suficientes en la suscripción para crear un recurso de área de trabajo de Azure Databricks. Si se produjese un error en el script debido a cuota o permisos insuficientes, intenta [crear un área de trabajo de Azure Databricks de forma interactiva en Azure Portal](https://learn.microsoft.com/azure/databricks/getting-started/#--create-an-azure-databricks-workspace).

1. En un explorador web, inicia sesión en [Azure Portal](https://portal.azure.com) en `https://portal.azure.com`.
2. Usa el botón **[\>_]** a la derecha de la barra de búsqueda en la parte superior de la página para crear un nuevo Cloud Shell en Azure Portal, selecciona un entorno de ***PowerShell*** y crea almacenamiento si se te solicita. Cloud Shell proporciona una interfaz de línea de comandos en un panel situado en la parte inferior de Azure Portal, como se muestra a continuación:

    ![Azure Portal con un panel de Cloud Shell](./images/cloud-shell.png)

    > **Nota**: Si ha creado previamente un cloud shell que usa un entorno de *Bash*, use el menú desplegable de la parte superior izquierda del panel de cloud shell para cambiarlo a ***PowerShell***.

3. Tenga en cuenta que puede cambiar el tamaño de Cloud Shell arrastrando la barra de separación en la parte superior del panel, o usando los iconos **&#8212;** , **&#9723;** y **X** en la parte superior derecha para minimizar, maximizar y cerrar el panel. Para obtener más información sobre el uso de Azure Cloud Shell, consulta la [documentación de Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. En el panel de PowerShell, introduce los siguientes comandos para clonar este repositorio:

    ```
    rm -r mslearn-databricks -f
    git clone https://github.com/MicrosoftLearning/mslearn-databricks
    ```

5. Una vez clonado el repositorio, escribe el siguiente comando para ejecutar el script **setup.ps1**, que aprovisiona un área de trabajo de Azure Databricks en una región disponible:

    ```
    ./mslearn-databricks/setup.ps1
    ```

6. Si se solicita, elige la suscripción que quieres usar (esto solo ocurrirá si tienes acceso a varias suscripciones de Azure).
7. Espera a que se complete el script: normalmente puede tardar entre 5 y 10 minutos, pero en algunos casos puede tardar más. Mientras esperas, revisa el artículo [¿Qué es el almacenamiento de datos en Azure Databricks?](https://learn.microsoft.com/azure/databricks/sql/) en la documentación de Azure Databricks.

## Visualización e inicio de una instancia de un almacén de SQL

1. Cuando se haya implementado el recurso del área de trabajo de Azure Databricks, ve a él en Azure Portal.
1. En la página **Información general** del área de trabajo de Azure Databricks, usa el botón **Iniciar área de trabajo** para abrir el área de trabajo de Azure Databricks en una nueva pestaña del explorador; inicia sesión si se solicita.

    > **Sugerencia**: al usar el portal del área de trabajo de Databricks, se pueden mostrar varias sugerencias y notificaciones. Descarta estos elementos y sigue las instrucciones proporcionadas para completar las tareas de este ejercicio.

1. Visualiza el portal del área de trabajo de Azure Databricks y observa que la barra lateral del lado izquierdo contiene los nombres de las categorías de tareas.
1. En la barra lateral, en **SQL**, selecciona **Almacenes de SQL**.
1. Observa que el área de trabajo ya incluye una instancia de Almacén de SQL denominada **Almacén de inicio**.
1. En el menú **Acciones** (**⁝**) del Almacén de SQL, selecciona **Editar**. Después, establece la propiedad **Tamaño del clúster** en **2X-Small** y guarda los cambios.
1. Usa el botón **Iniciar** para iniciar el Almacén de SQL (que puede tardar un par de minutos).

> **Nota**: si el Almacén de SQL no se inicia, es posible que la suscripción no tenga cuota suficiente en la región donde se aprovisiona el área de trabajo de Azure Databricks. Para más información, consulta [Cuota necesaria de vCPU de Azure](https://docs.microsoft.com/azure/databricks/sql/admin/sql-endpoints#required-azure-vcpu-quota). Si esto sucede, puedes intentar pedir un aumento de cuota, tal como se detalla en el mensaje de error cuando el almacenamiento no se inicia. Como alternativa, puedes intentar eliminar el área de trabajo y crear una nueva en otra región. Puedes especificar una región como parámetro para el script de configuración de la siguiente manera: `./setup.ps1 eastus`

## Creación de un esquema de la base de datos

1. Cuando el Almacén de SQL está *en ejecución*, selecciona **Editor de SQL** en la barra lateral.
2. En el panel **Explorador de esquema**, observa que el catálogo *hive_metastore* contiene una base de datos denominada **predeterminado**.
3. En el panel **Nueva consulta**, escribe el siguiente código SQL:

    ```sql
   CREATE DATABASE retail_db;
    ```

4. Usa el botón **► Ejecutar (1000)** para ejecutar el código SQL.
5. Cuando el código se haya ejecutado correctamente, en el panel **Explorador de esquema**, usa el botón Actualizar situado en la parte superior del panel para actualizar la lista. Después amplía **hive_metastore** y **retail_db**, y observa que la base de datos se ha creado, pero no contiene tablas.

Puedes usar la base de datos **predeterminada** para las tablas, pero al compilar un almacén de datos analíticos es mejor para crear bases de datos personalizadas para datos específicos.

## Creación de una tabla

1. Descarga el archivo [`products.csv`](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-databricks/main/data/products.csv) en tu equipo local y guárdalo como **products.csv**.
1. En el portal del área de trabajo de Azure Databricks, en la barra lateral, selecciona **(+) Nuevo** y después selecciona **Datos**.
1. En la página **Agregar datos**, selecciona **Crear o modificar tabla** y carga el archivo **products.csv** que descargaste en el equipo.
1. En la página **Crear o modificar tabla del archivo cargado**, selecciona el esquema **retail_db** y establece el nombre de la tabla en **products**. Después, selecciona **Crear tabla** en la esquina inferior derecha de la página.
1. Cuando la tabla esté creada, revisa sus detalles.

La capacidad de crear una tabla mediante la importación de datos desde un archivo facilita rellenar una base de datos. También puedes usar Spark SQL para crear tablas mediante código. Las propias tablas son definiciones de metadatos en la tienda de metadatos Hive y los datos que contienen se almacenan en formato Delta en el almacenamiento del sistema de archivos de Databricks (DBFS).

## Crear un panel

1. Haz clic en **(+) Nuevo** en la barra lateral y luego selecciona **Panel**.
2. Selecciona el nombre del nuevo panel y cámbialo a `Retail Dashboard`.
3. En la pestaña **Datos**, selecciona **Crear desde SQL** y usa la consulta siguiente:

    ```sql
   SELECT ProductID, ProductName, Category
   FROM retail_db.products; 
    ```

4. Selecciona **Ejecutar** y, después, cambia el nombre del conjunto de datos sin título a `Products and Categories`.
5. Selecciona la pestaña **Lienzo** y después selecciona **Agregar una visualización**.
6. En el editor de visualización, establece las siguientes propiedades:
    
    - **Conjunto de datos**: Productos y categorías
    - **Visualización**: barra
    - **Eje X**: COUNT(ProductID)
    - **Eje Y**: categoría

7. Selecciona **Publicar** para ver el panel como lo verán los usuarios.

Los paneles son una excelente manera de compartir tablas de datos y visualizaciones con usuarios profesionales. Puedes programar que los paneles se actualicen periódicamente y se envíen por correo electrónico a los suscriptores.

## Limpiar

En el portal de Azure Databricks, en la página **Almacenes SQL**, seleccione su almacén SQL y seleccione **&#9632; Detener** para apagarlo.

Si has terminado de explorar Azure Databricks, puedes eliminar los recursos que has creado para evitar costes innecesarios de Azure y liberar capacidad en la suscripción.
