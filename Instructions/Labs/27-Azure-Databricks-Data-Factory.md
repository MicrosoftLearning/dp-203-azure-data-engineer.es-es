---
lab:
  title: Automatización de un cuaderno de Azure Databricks con Azure Data Factory
  ilt-use: Suggested demo
---

# Automatización de un cuaderno de Azure Databricks con Azure Data Factory

Puedes usar cuadernos en Azure Databricks para realizar tareas de ingeniería de datos, como procesar archivos de datos y cargar datos en tablas. Cuando necesites organizar estas tareas como parte de una canalización de ingeniería de datos, puedes usar Azure Data Factory.

Este ejercicio debería tardar en completarse **40** minutos aproximadamente.

## Antes de empezar

Necesitará una [suscripción de Azure](https://azure.microsoft.com/free) en la que tenga acceso de nivel administrativo.

## Aprovisionamiento de los recursos de Azure

En este ejercicio, usarás un script para aprovisionar una nueva área de trabajo de Azure Databricks y un recurso de Azure Data Factory en la suscripción de Azure.

> **Sugerencia**: si ya tienes un área de trabajo de Azure Databricks *Estándar* o de *Evaluación*<u> y</u> un recurso de Azure Data Factory v2, puedes omitir este procedimiento.

1. En un explorador web, inicia sesión en [Azure Portal](https://portal.azure.com) en `https://portal.azure.com`.
2. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** y crear almacenamiento si se solicita. Cloud Shell proporciona una interfaz de línea de comandos en un panel situado en la parte inferior de Azure Portal, como se muestra a continuación:

    ![Azure Portal con un panel de Cloud Shell](./images/cloud-shell.png)

    > **Nota**: si has creado previamente un cloud Shell que usa un *entorno de Bash*, usa el menú desplegable situado en la parte superior izquierda del panel de Cloud Shell para cambiarlo a ***PowerShell***.

3. Tenga en cuenta que puede cambiar el tamaño de Cloud Shell arrastrando la barra de separación en la parte superior del panel, o usando los iconos **&#8212;** , **&#9723;** y **X** en la parte superior derecha para minimizar, maximizar y cerrar el panel. Para obtener más información sobre el uso de Azure Cloud Shell, consulte la [documentación de Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. En el panel de PowerShell, introduce los siguientes comandos para clonar este repositorio:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Una vez clonado el repositorio, escribe los siguientes comandos para cambiar a la carpeta de este laboratorio y ejecuta el script **setup.ps1** que contiene:

    ```
    cd dp-203/Allfiles/labs/27
    ./setup.ps1
    ```

6. Si se te solicita, elige la suscripción que quieres usar (esto solo ocurrirá si tienes acceso a varias suscripciones de Azure).

7. Espera a que se complete el script: normalmente tarda unos 5 minutos, pero en algunos casos puede tardar más. Mientras esperas, revisa [¿Qué es Azure Data Factory?](https://docs.microsoft.com/azure/data-factory/introduction).
8. Una vez completado el script, cierra el panel de Cloud Shell y ve al grupo de recursos **dp203-*xxxxxxx*** creado por el script para comprobar que contiene un área de trabajo de Azure Databricks y un recurso de Azure Data Factory (V2) (es posible que tengas que actualizar la vista del grupo de recursos).

## Importación de un cuaderno

Puedes crear cuadernos en el área de trabajo de Azure Databricks para ejecutar código escrito en diferentes lenguajes de programación. En este ejercicio, importarás un cuaderno existente que contiene un poco de código Python.

1. En Azure Portal, ve al grupo de recursos ** dp203-*xxxxxxx*** que ha creado el script (o el grupo de recursos que contiene el área de trabajo de Azure Databricks existente)
1. Selecciona el recurso de Azure Databricks Service (denominado **databricks*xxxxxxx*** si has usado el script de instalación para crearlo).
1. En la página **Información general** del área de trabajo, usa el botón **Inicio del área de trabajo** para abrir el área de trabajo de Azure Databricks en una nueva pestaña del explorador; inicia sesión si se solicita.

    > **Sugerencia**: al usar el portal del área de trabajo de Databricks, se pueden mostrar varias sugerencias y notificaciones. Descarta estos elementos y sigue las instrucciones proporcionadas para completar las tareas de este ejercicio.

1. Visualiza el portal del área de trabajo de Azure Databricks y observa que la barra lateral del lado izquierdo contiene iconos para las distintas tareas que puedes realizar.
1. En la barra de menús de la izquierda, selecciona el **Área de trabajo**. Luego selecciona la carpeta **⌂ Inicio**.
1. En la parte superior de la página, en el menú **⋮** junto a tu nombre de usuario, selecciona **Importar**. Luego, en el cuaderno de diálogo **Importar**, selecciona **dirección URL** e importa el cuaderno de `https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/raw/master/Allfiles/labs/27/Process-Data.ipynb`
1. Revisa el contenido del cuaderno, que incluye algunas celdas de código Python para:
    - Recupera un parámetro denominado **folder** si se ha pasado (de lo contrario, usa un valor predeterminado de *datos*).
    - Descarga datos de GitHub y guárdalos en la carpeta especificada en el sistema de archivos de Databricks (DBFS).
    - Sal del cuaderno y volverás a la ruta de acceso donde se guardaron los datos como salida.

    > **Sugerencia**: el cuaderno podría contener prácticamente cualquier lógica de procesamiento de datos que necesites. Este sencillo ejemplo pretende mostrar los principios fundamentales.

## Habilitación de la integración de Azure Databricks con Azure Data Factory

Para usar Azure Databricks desde una canalización de Azure Data Factory, debes crear un servicio vinculado en Azure Data Factory que permita el acceso al área de trabajo de Azure Databricks.

### Generar token de acceso

1. En el portal de Azure Databricks, en la barra de menús superior derecha, selecciona el nombre de usuario y luego selecciona **Configuración del usuario** en la lista desplegable.
1. En la página **Configuración del usuario**, selecciona **Desarrollador**. Luego, junto a **Tokens de acceso**, selecciona **Administrar**.
1. Selecciona **Generar nuevo token** y genera un nuevo token con el comentario *Data Factory* y deja la duración en blanco (lo cual indica que el token no caduca). Procura **copiar el token cuando aparezca <u>antes</u> de seleccionar *Listo***.
1. Pega el token copiado en un archivo de texto para tenerlo a mano más adelante en este ejercicio.

### Creación de un servicio vinculado en Azure Data Factory

1. Vuelve a Azure Portal y en el grupo de recursos **dp203-*xxxxxxx***, selecciona el recurso **adf*xxxxxxx*** de Azure Data Factory.
2. En la página **Información general**, selecciona **Iniciar estudio** para abrir Azure Data Factory Studio. Inicie sesión si se le pide.
3. En Azure Data Factory Studio, usa el icono **>>** para expandir el panel de navegación de la izquierda. Luego, selecciona la página **Administrar**.
4. En la página **Administrar**, en la pestaña **Servicios vinculados**, selecciona **+ Nuevo** para agregar un nuevo servicio vinculado.
5. En el panel **Nuevo servicio vinculado**, selecciona la pestaña **Proceso**. Luego selecciona **Azure Databricks**.
6. Continúa y crea el servicio vinculado con la siguiente configuración:
    - **Nombre**: AzureDatabricks
    - **Descripción**: área de trabajo de Azure Databricks
    - **Conectar mediante el entorno de ejecución de integración**: AutoResolveInegrationRuntime
    - **Método de selección de cuenta**: desde la suscripción de Azure
    - **Suscripción de Azure**: *selecciona tu suscripción*
    - **Área de trabajo de Databricks**: *selecciona el área de trabajo **databricksxxxxxxx***
    - **Seleccionar clúster**: nuevo clúster de trabajos
    - **URL del área de trabajo de Databricks**: *se establece automáticamente en la URL del área de trabajo de Databricks*
    - **Tipo de autenticación**: token de acceso
    - **** Token de acceso: *pega el token de acceso.*
    - **Versión del clúster**: 13.3 LTS (Spark 3.4.1, Scala 2.12)
    - **Tipo de nodo de clúster**: Standard_DS3_v2
    - **Versión de Python**: 3
    - **Opciones de trabajo**: arregladas
    - **Trabajos**: 1

## Uso de una canalización para ejecutar un cuaderno de Azure Databricks

Ahora que has creado un servicio vinculado, puedes usarlo en una canalización para ejecutar el cuaderno que has visto anteriormente.

### Crear una canalización

1. En Azure Data Factory Studio, en el panel de navegación, selecciona **Autor**.
2. En la página **Autor**, en el panel **Recursos de fábrica**, usa el icono **+** para agregar una **canalización**.
3. En el panel **Propiedades** de la nueva canalización, cambie su nombre por **Procesar datos con Databricks**. Luego usa el botón **Propiedades** (que tiene un aspecto similar a **<sub>*</sub>**) en el extremo derecho de la barra de herramientas para ocultar el panel **Propiedades**.
4. En el panel **Actividades**, expande **Databricks** y arrastra una actividad **Cuaderno** hasta la superficie del diseñador de canalizaciones.
5. Con la nueva actividad **Cuaderno1** seleccionada, establece las siguientes propiedades en el panel inferior:
    - **General:**
        - **Nombre**: Procesar datos
    - **Azure Databricks**:
        - **Servicio vinculado de Databricks**: *selecciona el servicio vinculado **AzureDatabricks** que has creado anteriormente.*
    - **Configuración**:
        - **Ruta de acceso del cuaderno**: *ve a la carpeta **Usuarios/tu nombre de usuario** y selecciona el cuaderno **Procesar datos***.
        - **Parámetros base**: *agrega un nuevo parámetro denominado **folder** con el valor **product_data***
6. Usa el botón **Validar** situado encima de la superficie del diseñador de canalizaciones para validar la canalización. Luego, usa el botón **Publicar todo** para publicarlo (guardarlo).

### Ejecución de la canalización

1. Encima de la superficie del diseñador de canalizaciones, selecciona **Agregar desencadenador** y luego selecciona **Desencadenar ahora**.
2. En el panel **Ejecución de canalización**, selecciona **Aceptar** para ejecutar la canalización.
3. En el panel de navegación de la izquierda, selecciona **Supervisar** y observa la canalización **Procesar datos con Databricks** en la pestaña **Ejecuciones de canalización**. Puede tardar un tiempo en ejecutarse, ya que crea dinámicamente un clúster de Spark y ejecuta el cuaderno. Puedes usar el botón **↻ Actualizar** de la página **Ejecuciones de canalización** para actualizar el estado.

    > **Nota**: si la canalización falla, puede deberse a que la cuota de tu suscripción sea insuficiente para la región donde se aprovisiona el área de trabajo de Azure Databricks para crear un clúster de trabajos. Consulta [El límite de núcleos de la CPU impide la creación de clústeres](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit) para obtener instrucciones. Si esto sucede, puedes intentar eliminar el área de trabajo y crear una nueva en otra región. Puedes especificar una región como parámetro para el script de instalación de la siguiente manera: `./setup.ps1 eastus`

4. Cuando se haya ejecutado correctamente, selecciona su nombre para ver los detalles de la ejecución. Luego, en la página **Procesar datos con Databricks**, en la sección ** Ejecuciones de actividad**, selecciona la actividad **Procesar datos** y usa su icono ***salida*** para ver el JSON de salida de la actividad, que debería parecerse a lo siguiente:
    ```json
    {
        "runPageUrl": "https://adb-..../run/...",
        "runOutput": "dbfs:/product_data/products.csv",
        "effectiveIntegrationRuntime": "AutoResolveIntegrationRuntime (East US)",
        "executionDuration": 61,
        "durationInQueue": {
            "integrationRuntimeQueue": 0
        },
        "billingReference": {
            "activityType": "ExternalActivity",
            "billableDuration": [
                {
                    "meterType": "AzureIR",
                    "duration": 0.03333333333333333,
                    "unit": "Hours"
                }
            ]
        }
    }
    ```

5. Anota el valor **runOutput**, que es la variable de la *ruta de acceso* en la que el cuaderno ha guardado los datos.

## Eliminación de recursos de Azure Databricks

Ahora que has terminado de explorar la integración de Azure Data Factory con Azure Databricks, debes eliminar los recursos que has creado para evitar costes innecesarios de Azure y liberar capacidad en tu suscripción.

1. Cierra las pestañas del explorador del área de trabajo de Azure Databricks y de Azure Data Factory Studio y vuelve a Azure Portal.
2. En Azure Portal, en la página **Inicio**, seleccione **Grupos de recursos**.
3. Selecciona el grupo de recursos **dp203-*xxxxxxx*** que contiene el área de trabajo de Azure Databricks y Azure Data Factory (no el grupo de recursos administrados).
4. En la parte superior de la página **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
5. Escriba el nombre del grupo de recursos para confirmar que quiere eliminarlo y seleccione **Eliminar**.

    Después de unos minutos, tu grupo de recursos y el grupo de recursos administrados del área de trabajo asociada a él se eliminarán.
