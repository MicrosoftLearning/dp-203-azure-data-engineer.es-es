---
lab:
  title: Uso de un almacén de SQL en Azure Databricks
  ilt-use: Optional demo
---

# Uso de un almacén de SQL en Azure Databricks

SQL es un lenguaje estándar del sector para consultar y manipular datos. Muchos analistas de datos realizan análisis de datos mediante SQL para consultar tablas en una base de datos relacional. Azure Databricks incluye funcionalidad de SQL que se basa en tecnologías de Spark y Delta Lake para proporcionar una capa de base de datos relacional a través de archivos en un lago de datos.

Este ejercicio debería tardar en completarse **30** minutos aproximadamente.

## Antes de comenzar

Necesitarás una [suscripción de Azure](https://azure.microsoft.com/free) en la que tengas acceso de nivel administrativo y cuota suficiente en al menos una región para aprovisionar una instancia del almacén de SQL de Azure Databricks.

## Aprovisiona un área de trabajo de Azure Databricks.

En este ejercicio, necesitarás un área de trabajo de Azure Databricks de nivel Premium.

1. En un explorador, inicia sesión en [Azure Portal](https://portal.azure.com) en `https://portal.azure.com`.
2. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear un nuevo Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** y crear almacenamiento si es necesario. Cloud Shell proporciona una interfaz de línea de comandos en un panel situado en la parte inferior de Azure Portal, como se muestra a continuación:

    ![Azure Portal con un panel de Cloud Shell](./images/cloud-shell.png)

    > **Nota**: si has creado previamente un Cloud Shell que usa un entorno de *Bash*, usa el menú desplegable situado en la parte superior izquierda del panel de Cloud Shell para cambiarlo a ***PowerShell***.

3. Tenga en cuenta que puede cambiar el tamaño de Cloud Shell arrastrando la barra de separación en la parte superior del panel, o usando los iconos **&#8212;** , **&#9723;** y **X** en la parte superior derecha para minimizar, maximizar y cerrar el panel. Para obtener más información sobre el uso de Azure Cloud Shell, consulte la [documentación de Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. En el panel de PowerShell, escribe los siguientes comandos para clonar este repositorio:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Una vez clonado el repositorio, escribe los siguientes comandos para cambiar a la carpeta de este laboratorio y ejecuta el script **setup.ps1** que contiene:

    ```
    cd dp-203/Allfiles/labs/26
    ./setup.ps1
    ```

6. Si se solicita, elige la suscripción que quieres usar (esto solo ocurrirá si tienes acceso a varias suscripciones de Azure).

7. Espera a que se complete el script: normalmente puede tardar entre 5 y 10 minutos, pero en algunos casos puede tardar más. Mientras esperas, revisa el artículo [¿Qué es el almacenamiento de datos en Azure Databricks?](https://learn.microsoft.com/azure/databricks/sql/) en la documentación de Azure Databricks.

## Visualización e inicio de una instancia de un almacén de SQL

1. Cuando se haya implementado el recurso del área de trabajo de Azure Databricks, ve a él en Azure Portal.
2. En la página **Información general** del área de trabajo de Azure Databricks, usa el botón **Iniciar área de trabajo** para abrir el área de trabajo de Azure Databricks en una nueva pestaña del explorador; inicie sesión si se solicita.
3. Si aparece un mensaje **¿Cuál es el proyecto de datos actual?**, selecciona **Finalizar** para cerrarlo. A continuación, ve el portal del área de trabajo de Azure Databricks y observa que la barra lateral del lado izquierdo contiene los nombres de las categorías de tareas.

    >**Sugerencia**: al usar el portal del área de trabajo de Databricks, se pueden mostrar varias sugerencias y notificaciones. Descarta estos elementos y sigue las instrucciones proporcionadas para completar las tareas de este ejercicio.

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
    CREATE SCHEMA adventureworks;
    ```

4. Usa el botón **► Ejecutar (1000)** para ejecutar el código SQL.
5. Cuando el código se haya ejecutado correctamente, en el panel **Explorador de esquema**, usa el botón Actualizar situado en la parte inferior del panel para actualizar la lista. A continuación, expande **hive_metastore** y **adventureworks** y observa que se ha creado la base de datos, pero no contiene tablas.

Puedes usar la base de datos **predeterminada** para las tablas, pero al compilar un almacén de datos analíticos es mejor para crear bases de datos personalizadas para datos específicos.

## Creación de una tabla

1. Descarga el archivo [**products.csv**](https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/26/data/products.csv) en el equipo local y guárdalo como **products.csv**.
1. En el portal del área de trabajo de Azure Databricks, en la barra lateral, selecciona **(+) Nuevo** y después selecciona **Carga de archivos** y carga el archivo **products.csv** que has descargado en el equipo.
1. En la página **Cargar datos**, selecciona el esquema **adventureworks** y establece el nombre de la tabla en **productos**. Después, selecciona **Crear tabla** en la esquina inferior izquierda de la página.
1. Cuando la tabla esté creada, revisa sus detalles.

La capacidad de crear una tabla mediante la importación de datos desde un archivo facilita rellenar una base de datos. También puedes usar Spark SQL para crear tablas mediante código. Las propias tablas son definiciones de metadatos en la tienda de metadatos Hive y los datos que contienen se almacenan en formato Delta en el almacenamiento del sistema de archivos de Databricks (DBFS).

## Creación de una consulta

1. En la barra lateral, selecciona **(+) Nuevo** y luego selecciona **Consulta**.
2. En el panel **Explorador de esquema** expande **hive_metastore** y **adventureworks** y comprueba que aparece la tabla **productos**.
3. En el panel **Nueva consulta**, escribe el siguiente código SQL:

    ```sql
    SELECT ProductID, ProductName, Category
    FROM adventureworks.products; 
    ```

4. Usa el botón **► Ejecutar (1000)** para ejecutar el código SQL.
5. Una vez completada la consulta, revisa la tabla de resultados.
6. Usa el botón **Guardar** situado en la parte superior derecha del editor de consultas para guardar la consulta como **Productos y categorías**.

Guardar una consulta facilita recuperar de nuevo los mismos datos más adelante.

## Crear un panel

1. Haz clic en **(+) Nuevo** en la barra lateral y luego selecciona **Panel**.
2. En el cuadro de diálogo **Nuevo panel**, escribe el nombre **Productos de Adventure Works** y selecciona **Guardar**.
3. En el panel **Productos** de Adventure Works, en la lista desplegable **Agregar**, selecciona **Visualización**.
4. En el cuadro de diálogo **Agregar widget de visualización**, selecciona la consulta **Productos y categorías**. Después, selecciona **Crear nueva visualización** y establece el título en **Productos por categoría**. y selecciona **Crear visualización**.
5. En el editor de visualización, establece las siguientes propiedades:
    - **Tipo de visualización**: barra
    - **Gráfico horizontal**: seleccionado
    - **Columna Y**: categoría
    - **Columnas X**: Id. de producto: Recuento
    - **Agrupar por**: categoría
    - **Posición de la leyenda**: automático (flexible)
    - **Orden de los elementos de leyenda**: normal
    - **Apilamiento**: pila
    - **Normalizar valores en porcentaje**: <u>no</u> seleccionado
    - **Valores que faltan y NULL**: no mostrar en el gráfico

6. Guarda la visualización y visualízala en el panel.
7. Selecciona **Edición finalizada** para ver el panel como lo verán los usuarios.

Los paneles son una excelente manera de compartir tablas de datos y visualizaciones con usuarios profesionales. Puedes programar que los paneles se actualicen periódicamente y se envíen por correo electrónico a los suscriptores.

## Eliminación de recursos de Azure Databricks

Ahora que has terminado de explorar Almacenes de SQL en Azure Databricks, deberás eliminar los recursos que has creado para evitar costes innecesarios de Azure y liberar capacidad en la suscripción.

1. Cierra la pestaña del explorador y vuelve a la pestaña de Azure Portal.
2. En Azure Portal, en la página **Inicio**, seleccione **Grupos de recursos**.
3. Selecciona el grupo de recursos que contiene el área de trabajo de Azure Databricks (no el grupo de recursos administrado).
4. En la parte superior de la página **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
5. Escriba el nombre del grupo de recursos para confirmar que quiere eliminarlo y seleccione **Eliminar**.

    Después de unos minutos, tu grupo de recursos y el grupo de recursos del área de trabajo administrada asociado se eliminarán.
