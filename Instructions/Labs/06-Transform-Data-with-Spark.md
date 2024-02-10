---
lab:
  title: Transformar datos con Spark en Synapse Analytics
  ilt-use: Lab
---

# Transformar datos con Spark en Synapse Analytics

Los ingenieros de *datos* suelen usar los cuadernos de Spark como una de sus herramientas preferidas para realizar actividades de *extracción, transformación y carga (ETL)* o *extracción, carga y transformación (ELT)* que transforman los datos de un formato o estructura a otro.

En este ejercicio, usarás un cuaderno de Spark en Azure Synapse Analytics para transformar datos en archivos.

Este ejercicio debería tardar en completarse **30** minutos aproximadamente.

## Antes de empezar

Necesitará una [suscripción de Azure](https://azure.microsoft.com/free) en la que tenga acceso de nivel administrativo.

## Aprovisionar un área de trabajo de Azure Synapse Analytics

Necesitarás un área de trabajo de Azure Synapse Analytics con acceso a Data Lake Storage y a un grupo de Spark.

En este ejercicio, usarás una combinación de un script de PowerShell y una plantilla de ARM para aprovisionar un área de trabajo de Azure Synapse Analytics.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) en `https://portal.azure.com`.
2. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** y crear almacenamiento si se solicita. Cloud Shell proporciona una interfaz de línea de comandos en un panel situado en la parte inferior de Azure Portal, como se muestra a continuación:

    ![Azure Portal con un panel de Cloud Shell](./images/cloud-shell.png)

    > **Nota**: Si creaste anteriormente un Cloud Shell que usa un entorno de *Bash*, usa el menú desplegable situado en la parte superior izquierda del panel de Cloud Shell para cambiarlo a ***PowerShell***.

3. Tenga en cuenta que puede cambiar el tamaño de Cloud Shell arrastrando la barra de separación en la parte superior del panel, o usando los iconos **&#8212;** , **&#9723;** y **X** en la parte superior derecha para minimizar, maximizar y cerrar el panel. Para obtener más información sobre el uso de Azure Cloud Shell, consulte la [documentación de Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. En el panel de PowerShell, esscribe los siguientes comandos para clonar este repositorio:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Una vez clonado el repositorio, escribe los siguientes comandos para cambiar a la carpeta de este ejercicio y ejecutar el script **setup.ps1** que contiene:

    ```
    cd dp-203/Allfiles/labs/06
    ./setup.ps1
    ```

6. Si se te solicita, elige qué suscripción quieres usar (esto solo ocurrirá si tienes acceso a varias suscripciones de Azure).
7. Cuando se te solicite, especifica una contraseña adecuada para el grupo de Azure Synapse SQL.

    > **Nota**: Asegúrate de recordar la contraseña.

8. Espera a que se complete el script: normalmente tarda unos 10 minutos, pero en algunos casos puede tardar más. Mientras esperas, revisa el artículo [Conceptos básicos de Apache Spark en Azure Synapse Analytics](https://learn.microsoft.com/azure/synapse-analytics/spark/apache-spark-concepts) de la documentación de Azure Synapse Analytics.

## Usar un cuaderno de Spark para transformar datos

1. Una vez completado el script de implementación, en Azure Portal, vaya al grupo de recursos **dp203-*xxxxxxx*** que creó y observa que este grupo de recursos contiene el área de trabajo de Synapse, una cuenta de almacenamiento para el lago de datos y un grupo de Apache Spark.
2. Selecciona el área de trabajo de Synapse y en tu página **Información general**, en la tarjeta **Abrir Synapse Studio**, selecciona **Abrir** para abrir Synapse Studio en una nueva pestaña del explorador; inicia sesión si se solicita.
3. En el lado izquierdo de Synapse Studio, usa el icono **&rsaquo;&rsaquo;** para expandir el menú: lo que mostrará las diferentes páginas de Synapse Studio que usarás para administrar recursos y realizar tareas de análisis de datos.
4. En la página **Administrar**, selecciona la pestaña **Grupos de Apache Spark** y observa que se ha aprovisionado un grupo de Spark con un nombre similar a **spark*xxxxxxx*** en el área de trabajo.
5. En la página **Datos**, consulta la pestaña **Vinculado** y comprueba que tu área de trabajo incluye un vínculo a tu cuenta de almacenamiento de Azure Data Lake Storage Gen2, que debería tener un nombre similar a **synapse*xxxxxxx* (Primary - datalake*xxxxxxx*)**.
6. Expande tu cuenta de almacenamiento y verifica que contiene un contenedor de sistema de archivos denominado **files (Primary)**.
7. Selecciona el contenedor **files** y observa que contiene carpetas denominadas **data** y **synapse**. Azure Synapse usa la carpeta synapse, y la carpeta **data** contiene los archivos de datos que vas a consultar.
8. Abre la carpeta **datos** y observa que contiene archivos .csv de tres años de datos de ventas.
9. Haz clic con el botón derecho en cualquiera de los archivos y selecciona **Vista previa** para ver los datos que contiene. Ten en cuenta que los archivos contienen una fila de encabezado, por lo que puedes seleccionar la opción de mostrar los encabezados de columna.
10. Cierre la vista preliminar. A continuación, descargue **Spark Transform.ipynb** desde [Allfiles/labs/06/notebooks](https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/tree/master/Allfiles/labs/06/notebooks)

    > **Nota**: Es mejor copiar este texto con ***Ctrl+a*** y luego ***Ctrl+c*** y pegarlo en una herramienta con ***Ctrl+v***, como por ejemplo, el cuaderno y luego con archivo, guardar como **Spark Transform.ipynb** con el tipo de archivo ***Todos los archivos***. También puede descargar el archivo haciendo clic en él, seleccionando los puntos suspensivos y, a continuación, descargarlo; debe recordar dónde lo guardó.
    ![Descarga del cuaderno de Spark desde GitHub](./images/select-download-notebook.png)

11. Después, en la página **Desarrollar**, despliega **Cuaderno** y haz clic en las opciones + Importar

    ![Importar un cuaderno de Spark](./image/../images/spark-notebook-import.png)
        
12. Selecciona el archivo que acabas de descargar y guardar como **Spark Transfrom.ipynb**.
13. Asocia el cuaderno a tu grupo **spark*xxxxxxx*** de Spark.
14. Revisa las notas del cuaderno y ejecuta las celdas de código.

    > **Nota**: La primera celda de código tardará unos minutos en ejecutarse porque debe iniciarse el grupo de Spark. Las células posteriores se ejecutarán más rápido.

## Eliminación de recursos de Azure

Si ha terminado de explorar Azure Synapse Analytics, debe eliminar los recursos que ha creado para evitar costos innecesarios de Azure.

1. Cierre la pestaña del explorador de Synapse Studio y vuelva a Azure Portal.
2. En Azure Portal, en la página **Inicio**, seleccione **Grupos de recursos**.
3. Selecciona el grupo de recursos **dp203-*xxxxxxx*** para tu área de trabajo de Synapse Analytics (no el grupo de recursos administrados) y verifica que contiene el área de trabajo de Synapse, la cuenta de almacenamiento y el grupo de Spark para tu área de trabajo.
4. En la parte superior de la página **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
5. Especifica el nombre del grupo de recursos **dp203-*xxxxxxx*** para confirmar que quieres eliminarlo y selecciona **Eliminar**.

    Después de unos minutos, se eliminarán el grupo de recursos de área de trabajo de Azure Synapse y el grupo de recursos de área de trabajo administrado asociado a él.
