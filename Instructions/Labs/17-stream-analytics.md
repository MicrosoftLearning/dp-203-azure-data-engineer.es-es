---
lab:
  title: Introducción a Azure Stream Analytics
  ilt-use: Suggested demo
---

# Introducción a Azure Stream Analytics

En este ejercicio, aprovisionarás un trabajo de Azure Stream Analytics en tu suscripción de Azure y lo usarás para consultar y resumir un flujo de datos de eventos en tiempo real y almacenar los resultados en Azure Storage.

Este ejercicio debería tardar aproximadamente **15** minutos en completarse.

## Antes de empezar

Necesitará una [suscripción de Azure](https://azure.microsoft.com/free) en la que tenga acceso de nivel administrativo.

## Aprovisionamiento de los recursos de Azure

En este ejercicio, capturarás un flujo de datos de transacciones de ventas simuladas, lo procesarás y almacenarás los resultados en un contenedor de blobs en Azure Storage. Necesitarás un espacio de nombres de Azure Event Hubs al que se puedan enviar datos de streaming y una cuenta de Azure Storage en la que se almacenarán los resultados del procesamiento de flujos.

Usarás una combinación de un script de PowerShell y una plantilla de ARM para aprovisionar estos recursos.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) en `https://portal.azure.com`.
2. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** y crear almacenamiento si se solicita. Cloud Shell proporciona una interfaz de línea de comandos en un panel situado en la parte inferior de Azure Portal, como se muestra a continuación:

    ![Azure Portal con un panel de Cloud Shell](./images/cloud-shell.png)

    > **Nota**: si creaste anteriormente un Cloud Shell que usa un entorno de *Bash*, usa el menú desplegable situado en la parte superior izquierda del panel de Cloud Shell para cambiarlo a ***PowerShell***.

3. Ten en cuenta que puedes cambiar el tamaño de Cloud Shell arrastrando la barra de separación en la parte superior del panel, o usando los iconos **&#8212;** , **&#9723;** y **X** en la parte superior derecha para minimizar, maximizar y cerrar el panel. Para obtener más información sobre el uso de Azure Cloud Shell, consulte la [documentación de Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. En el panel de PowerShell, escribe los siguientes comandos para clonar el repositorio que contiene este ejercicio:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Una vez clonado el repositorio, escribe los siguientes comandos para cambiar a la carpeta de este ejercicio y ejecuta el script **setup.ps1** que contiene:

    ```
    cd dp-203/Allfiles/labs/17
    ./setup.ps1
    ```

6. Si se solicita, elige la suscripción que quieres usar (esto solo ocurrirá si tienes acceso a varias suscripciones de Azure).
7. Espera a que se complete el script: normalmente tarda unos 5 minutos, pero en algunos casos puede tardar más. Mientras esperas, revisa el artículo [Le damos la bienvenida a Azure Stream Analytics](https://learn.microsoft.com/azure/stream-analytics/stream-analytics-introduction) en la documentación de Azure Stream Analytics.

## Visualización del origen de datos de streaming

Antes de crear un trabajo de Azure Stream Analytics para procesar datos en tiempo real, echemos un vistazo al flujo de datos que necesitarás consultar.

1. Cuando el script de configuración haya terminado de ejecutarse, cambia el tamaño o minimiza el panel de Cloud Shell para que puedas ver Azure Portal (volverás a Cloud Shell más adelante). Después, en Azure Portal, ve al grupo de recursos **dp203-*xxxxxxx*** que creaste y observa que este grupo de recursos contiene una cuenta de Azure Storage y un espacio de nombres de Event Hubs.

    Fíjate en la **Ubicación** en la que se han aprovisionado los recursos; más adelante, crearás un trabajo de Azure Stream Analytics en la misma ubicación.

2. Vuelve a abrir el panel de Cloud Shell y especifica el siguiente comando para ejecutar una aplicación cliente que envíe 100 pedidos simulados a Azure Event Hubs:

    ```
    node ~/dp-203/Allfiles/labs/17/orderclient
    ```

3. Observa los datos del pedido de venta a medida que se envían: cada pedido consta de un id. de producto y una cantidad. La aplicación finalizará después de enviar 1000 pedidos, lo que tarda un minuto aproximadamente.

## Creación de un trabajo de Azure Stream Analytics

Ahora ya puedes crear un trabajo de Azure Stream Analytics para procesar los datos de las transacciones de ventas a medida que llegan a Event Hubs.

1. En Azure Portal, en la página **dp203-*xxxxxxx***, selecciona **+ Crear** y busca `Stream Analytics job`. Después, crea un trabajo de **Stream Analytics** con las siguientes propiedades:
    - **Aspectos básicos**:
        - **Suscripción** : su suscripción a Azure.
        - **Grupo de recursos**: selecciona el grupo de recursos **dp203-*xxxxxxx*** existente.
        - **Nombre**: `process-orders`
        - **Región**: selecciona la región en la que se aprovisionan tus otros recursos de Azure.
        - **Entorno de hospedaje**: nube
        - **Unidades de streaming**: 1
    - **Almacenamiento**:
        - **Agregar cuenta de almacenamiento**: no seleccionada
    - **Etiquetas:**
        - *Ninguno*
2. Espera a que finalice la implantación y después ve al recurso del trabajo de Stream Analytics implementado.

## Crear una entrada para el flujo de eventos

El trabajo de Azure Stream Analytics debe obtener datos de entrada de Event Hubs donde se registran los pedidos de ventas.

1. En la página de información general **process-orders**, selecciona **Agregar entrada**. Después, en la página **Entradas**, usa el menú **Agregar entrada de flujo** para agregar una entrada de **Event Hubs** con las siguientes propiedades:
    - **Alias de entrada**: `orders`
    - **Seleccionar Event Hubs de entre tus suscripciones**: seleccionada
    - **Suscripción** : su suscripción a Azure.
    - **Espacio de nombres de Event Hubs**: selecciona el espacio de nombres de Event Hubs **events*xxxxxxx***
    - **Nombre de Event Hubs**: selecciona el Event Hubs **eventhub*xxxxxxx*** existente.
    - **Grupo de consumidores de Event Hub**: selecciona el grupo de consumidores **$Default** existente.
    - **Modo de autenticación**: crear una identidad administrada asignada por el sistema
    - **Clave de partición**: *dejar en blanco*
    - **Formato de serialización de eventos**: JSON
    - **Codificación**: UTF-8
2. Guarda la entrada y espera mientras se crea. Verás varias notificaciones. Espera a una notificación de **prueba de conexión correcta**.

## Creación de una salida para el almacén de blobs

Almacenará los datos agregados de pedidos de ventas en formato JSON en un contenedor de blobs de Azure Storage.

1. Visualiza la página **Salidas** del trabajo de Stream Analytics **process-orders**. Después, usa el menú **Agregar** para agregar una salida **Blob Storage/ADLS Gen2** con las siguientes propiedades:
    - **Alias de salida**: `blobstore`
    - **Selecciona Seleccionar almacenamiento de blobs/ADLS Gen2 de entre las suscripciones de las suscripciones**: seleccionado.
    - **Suscripción** : su suscripción a Azure.
    - **Cuenta de almacenamiento**: selecciona la cuenta de almacenamiento  **store*xxxxxxx***
    - **Contenedor**: selecciona el contenedor de **datos** existente
    - **Modo de autenticación**: identidad administrada: asignada por el sistema.
    - **Formato de serialización de eventos**: JSON
    - **Formato**: separado por líneas
    - **Codificación**: UTF-8
    - **Modo de escritura**: anexar a medida que llegan los resultados
    - **Patrón de la ruta de acceso**: `{date}`
    - **Formato de fecha**: AAAA/MM/DD
    - **Formato de hora**: *no aplicable*
    - **Número mínimo de filas**: 20
    - **Tiempo máximo**: 0 horas, 1 minutos y 0 segundos
2. Guarda la salida y espera mientras se crea. Verás varias notificaciones. Espera a una notificación de **prueba de conexión correcta**.

## Creación de una consulta

Ahora que has definido una entrada y una salida para el trabajo de Azure Stream Analytics, puedes usar una consulta para seleccionar, filtrar y agregar datos de la entrada y enviar los resultados a la salida.

1. Visualiza la página **Consulta** del trabajo **process-orders** de Stream Analytics. A continuación, espera unos instantes hasta que se muestre la vista previa de entrada (en función de los eventos de pedido de ventas capturados anteriormente en el centro de eventos).
2. Observa que los datos de entrada incluyen los campos **ProductID** y **Quantity** en los mensajes enviados por la aplicación cliente, así como campos adicionales de Event Hubs, incluido el campo **EventProcessedUtcTime** que indica cuándo se agregó el evento al centro de eventos.
3. Modifique la consulta predeterminada de la siguiente manera:

    ```
    SELECT
        DateAdd(second,-10,System.TimeStamp) AS StartTime,
        System.TimeStamp AS EndTime,
        ProductID,
        SUM(Quantity) AS Orders
    INTO
        [blobstore]
    FROM
        [orders] TIMESTAMP BY EventProcessedUtcTime
    GROUP BY ProductID, TumblingWindow(second, 10)
    HAVING COUNT(*) > 1
    ```

    Observe que esta consulta usa la ventana **System.Timestamp** (basada en el campo **EventProcessedUtcTime**) para definir el inicio y el final de cada ventana de *saltos de tamaño constante* de 10 segundos (secuencial no superpuesta) en la que se calcula la cantidad total de cada Id. de producto.

4. Usa el botón **▷ Consulta de prueba** para validar la consulta y asegúrate de que el estado **Resultados de la prueba** indica **Success** (aunque no se devuelvan filas).
5. Guarde la consulta.

## Ejecución de un trabajo de streaming

Bien, ahora estás listo para ejecutar el trabajo y procesar algunos datos de pedidos de ventas en tiempo real.

1. Visualiza la página **Información general** del trabajo de Stream Analytics **process-order** y en la pestaña **Propiedades**, revisa las **entradas**, **Consulta**, **Salidas** y **Funciones** del trabajo. Si el número de **Entradas** y **Salidas** es 0, usa el botón **↻ Actualizar** de la página **Información general** para mostrar la entrada **pedidos** y la salida **almacén de blobs**.
2. Selecciona el botón **▷ Iniciar** e inicia ahora el trabajo de streaming. Espera a recibir la notificación de que el trabajo de streaming se ha iniciado correctamente.
3. Vuelve a abrir el panel de Cloud Shell, vuelva a conectarte si es necesario y luego vuelve a ejecutar el siguiente comando para enviar otros 1000 pedidos.

    ```
    node ~/dp-203/Allfiles/labs/17/orderclient
    ```

4. Mientras se ejecuta la simulación, en Azure Portal, vuelva a la página del grupo de recursos e **dp203-*xxxxxxx*** y selecciona la cuenta de almacenamiento **store*xxxxxxxxxxxx***.
6. En el panel de la izquierda de la hoja de la cuenta de almacenamiento, seleccione la pestaña **Contenedores**.
7. Abre el contenedor de **datos** y usa el botón **↻ Actualizar** para actualizar la vista hasta que veas una carpeta con el nombre del año actual.
8. En el contenedor de **datos**, navega por la jerarquía de carpetas, que incluye la carpeta del año actual, con subcarpetas para el mes y el día.
9. En la carpeta de la hora, fíjese en el archivo creado, que debe tener un nombre parecido a **0_xxxxxxxxxxxxxxxx.json**.
10. En el menú **...** para el archivo (a la derecha de los detalles del archivo), selecciona **Ver/editar** y revisa el contenido del archivo; que debería consistir en un registro JSON para cada periodo de 10 segundos, mostrando el número de pedidos procesados para cada id. de producto, como este:

    ```
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":6,"Orders":13.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":8,"Orders":15.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":5,"Orders":15.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":1,"Orders":16.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":3,"Orders":10.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":2,"Orders":25.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":7,"Orders":13.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":4,"Orders":12.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":10,"Orders":19.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":9,"Orders":8.0}
    {"StartTime":"2022-11-23T18:16:35.0000000Z","EndTime":"2022-11-23T18:16:45.0000000Z","ProductID":6,"Orders":41.0}
    {"StartTime":"2022-11-23T18:16:35.0000000Z","EndTime":"2022-11-23T18:16:45.0000000Z","ProductID":8,"Orders":29.0}
    ...
    ```

11. En el panel de Azure Cloud Shell, espera a que finalice la aplicación cliente de los pedidos.
12. De nuevo en Azure Portal, actualiza el archivo una vez más para ver el conjunto completo de resultados que se han producido.
13. Vuelve al grupo de recursos **dp203-*xxxxxxx*** y vuelve a abrir el trabajo **process-orders** de Stream Analytics.
14. En la parte superior de la página del trabajo de Stream Analytics, use el botón **&#11036; Detener** para detener el trabajo y confírmelo cuando se lo solicite.

## Eliminación de recursos de Azure

Si has terminado de explorar Azure Stream Analytics, debes eliminar los recursos que has creado para evitar costos innecesarios de Azure.

1. En Azure Portal, en la página **Inicio**, seleccione **Grupos de recursos**.
2. Selecciona el grupo de recursos **dp203-*xxxxxxx*** que contiene tus recursos de Azure Storage, Event Hubs y Stream Analytics.
3. En la parte superior de la página **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
4. Especifica el nombre del grupo de recursos **dp203-*xxxxxxx*** para confirmar que quieres eliminarlo y selecciona **Eliminar**.

    Después de unos minutos, se eliminarán los recursos creados en este ejercicio.
