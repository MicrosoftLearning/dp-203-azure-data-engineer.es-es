---
lab:
  title: Uso de Apache Spark en Azure Databricks
  ilt-use: Lab
---

# Uso de Apache Spark en Azure Databricks

Azure Databricks es una versión basada en Microsoft Azure de la conocida plataforma de código abierto Databricks. Azure Databricks se basa en Apache Spark y ofrece una solución altamente escalable para tareas de ingeniería y análisis de datos que implican trabajar con datos en archivos. Una de las ventajas de Spark es la compatibilidad con una amplia variedad de lenguajes de programación, como Java, Scala, Python y SQL, lo que lo convierte en una solución muy flexible para cargas de trabajo de procesamiento de datos, incluida la limpieza y manipulación de datos, el análisis estadístico y el aprendizaje automático, y el análisis y la visualización de datos.

Este ejercicio debería tardar en completarse **45** minutos aproximadamente.

## Antes de empezar

Necesitará una [suscripción de Azure](https://azure.microsoft.com/free) en la que tenga acceso de nivel administrativo.

## Aprovisiona un área de trabajo de Azure Databricks.

En este ejercicio, usarás un script para aprovisionar un nuevo área de trabajo de Azure Databricks.

1. En un explorador web, inicia sesión en [Azure Portal](https://portal.azure.com) en `https://portal.azure.com`.
2. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***Bash*** y crear almacenamiento si se solicita. Cloud Shell proporciona una interfaz de línea de comandos en un panel situado en la parte inferior de Azure Portal, como se muestra a continuación:

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
    cd dp-203/Allfiles/labs/24
    ./setup.ps1
    ```

6. Si se solicita, elige la suscripción que quieres usar (esto solo ocurrirá si tienes acceso a varias suscripciones de Azure).

7. Espera a que se complete el script: normalmente tarda unos 5 minutos, pero en algunos casos puede tardar más. Mientras esperas, revisa el artículo [Análisis de datos exploratorios en Azure Databricks](https://learn.microsoft.com/azure/databricks/exploratory-data-analysis/) en la documentación de Azure Databricks.

## Crear un clúster

Azure Databricks es una plataforma de procesamiento distribuido que usa clústeres* de Apache Spark *para procesar datos en paralelo en varios nodos. Cada clúster consta de un nodo de controlador para coordinar el trabajo y los nodos de trabajo para realizar tareas de procesamiento.

> **Nota**: en este ejercicio, creará un clúster de un *solo nodo* para minimizar los recursos de proceso usados en el entorno de laboratorio (en el que se pueden restringir los recursos). En un entorno de producción, normalmente crearías un clúster con varios nodos de trabajo.

1. En Azure Portal, ve al grupo de recursos **dp203-*xxxxxxx*** que creó el script que ejecutaste.
2. Selecciona el recurso Azure Databricks Service **databricks*xxxxxxx***.
3. En la página **Información general** de **databricks*xxxxxxx***, usa el botón **Iniciar área de trabajo** para abrir el área de trabajo de Azure Databricks en una nueva pestaña del explorador; inicie sesión si se solicita.
4. Si se muestra un mensaje **¿Cuál es el proyecto de datos actual?**, selecciona **Finalizar** para cerrarlo. Después, visualiza el portal del área de trabajo de Azure Databricks y observa que la barra lateral del lado izquierdo contiene iconos para las distintas tareas que puede realizar.

    >**Sugerencia**: al usar el portal del área de trabajo de Databricks, se pueden mostrar varias sugerencias y notificaciones. Descarta estos elementos y sigue las instrucciones proporcionadas para completar las tareas de este ejercicio.

1. Selecciona la tarea **Nuevo (+)** y luego selecciona **Clúster**.
1. En la página **Nuevo clúster**, crea un clúster con la siguiente configuración:
    - **Nombre del clúster**: clúster del *Nombre de usuario*  (el nombre del clúster predeterminado)
    - **Modo de clúster** de un solo nodo
    - **Modo de acceso**: usuario único (*con la cuenta de usuario seleccionada*)
    -  **versión de Databricks Runtime**: 11.3 (Scala 2.12, Spark 3.3.0) o en una versión posterior.
    - **Usar aceleración de Photon**: seleccionado
    - **Tipo de nodo**: Standard_DS3_v2.
    - **Finaliza después de ***30***minutos de inactividad**

7. Espera a que se cree el clúster. Esto puede tardar un par de minutos.

> **Nota**: si el clúster no se inicia, es posible que la suscripción no tenga cuota suficiente en la región donde se aprovisiona el área de trabajo de Azure Databricks. Para más información consulta [El límite de núcleos de la CPU impide la creación de clústeres](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). Si esto sucede, puedes intentar eliminar el área de trabajo y crear una nueva en otra región. Puedes especificar una región como parámetro para el script de configuración de la siguiente manera: `./setup.ps1 eastus`

## Exploración de datos mediante un cuaderno

Como en muchos entornos de Spark, Databricks es compatible con el uso de cuadernos para combinar notas y celdas de código interactivo que puedes usar para explorar los datos.

1. En la barra de menús de la izquierda, selecciona el **Área de trabajo**. Después, selecciona la carpeta **⌂ Inicio**.
1. En la parte superior de la página, en el menú **⋮** junto a tu nombre de usuario, selecciona **Importar**. A continuación, en el cuadro de diálogo **Importar**, selecciona **URL** e importeael cuaderno de `https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/raw/master/Allfiles/labs/24/Databricks-Spark.ipynb`
1. Conectar el cuaderno del clúster y sigue las instrucciones que contiene; ejecuta las celdas que contiene para explorar los datos en archivos.

## Eliminar recursos de Azure Databricks

Ahora que has terminado de explorar Azure Databricks, debes eliminar los recursos que has creado para evitar costos innecesarios de Azure y liberar capacidad en tu suscripción.

1. Cierra la pestaña del explorador y vuelve a la pestaña de Azure Portal.
2. En Azure Portal, en la página **Inicio**, seleccione **Grupos de recursos**.
3. Selecciona el grupo de recursos ** dp203-*xxxxxxx*** (no el grupo de recursos administrado) y comprueba que contiene el área de trabajo de Azure Databricks.
4. En la parte superior de la página **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
5. Escribe el nombre del grupo de recursos **dp203-*xxxxxxx*** para confirmar que quieres eliminarlo y selecciona **Eliminar**.

    Después de unos minutos, tu grupo de recursos y el grupo de recursos del área de trabajo administrada asociado a él se eliminarán.
