---
lab:
  title: Ingesta de datos en tiempo real con Azure Stream Analytics y Azure Synapse Analytics
  ilt-use: Lab
---

# Ingesta de datos en tiempo real con Azure Stream Analytics y Azure Synapse Analytics

Las soluciones de análisis de datos suelen incluir un requisito para ingerir y procesar *flujos* de datos. El procesamiento de secuencias difiere del procesamiento por lotes en que las secuencias suelen ser *sin límites*; es decir, son orígenes continuos de datos que se deben procesar perpetuamente en lugar de a intervalos fijos.

Azure Stream Analytics proporciona un servicio en la nube que puedes usar para definir una *consulta* que opera en un flujo de datos de un origen de streaming, como Azure Event Hubs o Azure IoT Hub. Puedes usar una consulta de Azure Stream Analytics para ingerir el flujo de datos directamente en un almacén de datos para un análisis adicional, o para filtrar, agregar y resumir los datos basados en ventanas temporales.

En este ejercicio, usarás Azure Stream Analytics para procesar un flujo de datos de pedidos de ventas, como el que podría generarse a partir de una aplicación de venta minorista en línea. Los datos del pedido se enviarán a Azure Event Hubs, desde donde tus trabajos de Azure Stream Analytics leerán los datos y los ingerirán en Azure Synapse Analytics.

Este ejercicio debería tardar en completarse **45** minutos aproximadamente.

## Antes de empezar

Necesitará una [suscripción de Azure](https://azure.microsoft.com/free) en la que tenga acceso de nivel administrativo.

## Aprovisionamiento de los recursos de Azure

En este ejercicio, necesitarás un área de trabajo de Azure Synapse Analytics con acceso a Data Lake Storage y a un grupo de SQL dedicado. También necesitarás un espacio de nombres de Azure Event Hubs al que se puedan enviar los datos de pedidos de la secuencia.

Usarás una combinación de un script de PowerShell y una plantilla de ARM para aprovisionar estos recursos.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) en `https://portal.azure.com`.
2. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** y crear almacenamiento si se solicita. Cloud Shell proporciona una interfaz de línea de comandos en un panel situado en la parte inferior de Azure Portal, como se muestra a continuación:

    ![Azure Portal con un panel de Cloud Shell](./images/cloud-shell.png)

    > **Nota**: Si creaste anteriormente un Cloud Shell que usa un entorno de *Bash*, usa el menú desplegable situado en la parte superior izquierda del panel de Cloud Shell para cambiarlo a ***PowerShell***.

3. Tenga en cuenta que puede cambiar el tamaño de Cloud Shell arrastrando la barra de separación en la parte superior del panel, o usando los iconos **&#8212;** , **&#9723;** y **X** en la parte superior derecha para minimizar, maximizar y cerrar el panel. Para obtener más información sobre el uso de Azure Cloud Shell, consulte la [documentación de Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. En el panel de PowerShell, escribe los siguientes comandos para clonar el repositorio que contiene este ejercicio:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Una vez clonado el repositorio, escribe los siguientes comandos para cambiar a la carpeta de este ejercicio y ejecuta el script **setup.ps1** que contiene:

    ```
    cd dp-203/Allfiles/labs/18
    ./setup.ps1
    ```

6. Si se solicita, elige la suscripción que quieres usar (esto solo ocurrirá si tienes acceso a varias suscripciones de Azure).
7. Cuando se te solicite, escribe una contraseña adecuada que se va a establecer para el grupo de SQL de Azure Synapse.

    > **Nota**: Asegúrate de recordar esta contraseña.

8. Espera a que se complete el script: normalmente tarda unos 15 minutos, pero en algunos casos puede tardar más. Mientras esperas, revisa el artículo [Le damos la bienvenida a Azure Stream Analytics](https://learn.microsoft.com/azure/stream-analytics/stream-analytics-introduction) en la documentación de Azure Stream Analytics.

## Ingesta de datos de streaming en un grupo dedicado de SQL

Comencemos por ingerir un flujo de datos directamente en una tabla de un grupo de SQL dedicado de Azure Synapse Analytics.

### Visualización del origen de streaming y la tabla de base de datos

1. Cuando el script de configuración haya terminado de ejecutarse, minimiza el panel de Cloud Shell (volverás a él más adelante). Después, en Azure Portal, ve al grupo de recursos **dp203-*xxxxxxx*** que creaste y observa que este grupo de recursos contiene un área de trabajo de Azure Synapse, una cuenta de almacenamiento para tu lago de datos, un grupo de SQL dedicado y un espacio de nombres de Event Hubs.
2. Selecciona tu área de trabajo de Synapse y, en tu página **Información general**, en la tarjeta **Abrir Synapse Studio**, selecciona **Abrir** para abrir Synapse Studio en una nueva pestaña del explorador. Synapse Studio es una interfaz web que puedes usar para trabajar con tu área de trabajo de Synapse Analytics.
3. En el lado izquierdo de Synapse Studio, usa el icono **&rsaquo;&rsaquo;** para expandir el menú; se muestran las distintas páginas de Synapse Studio que usarás para administrar recursos y realizar tareas de análisis de datos.
4. En la página **Administrar**, en la sección **Grupos de SQL**, selecciona la fila **sql*xxxxxxx*** del grupo de SQL dedicado y después usa su icono **▷** para reanudarlo.
5. Mientras esperas a que se inicie el grupo de SQL, vuelve a la pestaña del explorador que contiene Azure Portal y vuelve a abrir el panel de Cloud Shell.
6. En el panel de Cloud Shell, escribe el siguiente comando para ejecutar una aplicación cliente que envíe 100 pedidos simulados a Azure Event Hubs:

    ```
    node ~/dp-203/Allfiles/labs/18/orderclient
    ```

7. Observa los datos del pedido a medida que se envían: cada pedido consta de un id. de producto y una cantidad.
8. Una vez que la aplicación cliente de pedidos se haya completado, minimiza el panel de Cloud Shell y vuelve a la pestaña del explorador de Synapse Studio.
9. En Synapse Studio, en la página **Administrar**, asegúrate de que tu grupo de SQL dedicado tiene el estado **En línea**, luego ve a la página **datos** y en el panel **Área de trabajo**, expande **Base de datos SQL**, tu grupo de SQL **sql*xxxxxxx*** y **Tablas** para ver la tabla **dbo. FactOrder**.
10. En el menú **...** de la tabla **dbo.FactOrder**, selecciona **Nuevo script de SQL** > **Seleccionar las 100 primeras filas** y revisa los resultados. Observa que la tabla incluye columnas para **OrderDateTime**, **ProductID** y **Quantity**, pero actualmente no hay filas de datos.

### Crear un trabajo de Azure Stream Analytics para ingerir datos de pedidos

1. Vuelve a la pestaña del navegador que contiene Azure Portal y fíjate en la región en la que se aprovisionó tu grupo de recursos **dp203-*xxxxxxx***; crea tu trabajo de Stream Analytics en la <u>misma región</u>.
2. En la página **Inicio** selecciona **+ Crear un recurso** y busca `Stream Analytics job`. Después, crea un trabajo de **Stream Analytics** con las siguientes propiedades:
    - **Aspectos básicos**:
        - **Suscripción** : su suscripción a Azure.
        - **Grupo de recursos**: selecciona el grupo de recursos **dp203-*xxxxxxx*** existente.
        - **Nombre**: `ingest-orders`
        - **Región**: selecciona la <u>misma región</u> donde se aprovisiona el área de trabajo de Synapse Analytics.
        - **Entorno de hospedaje**: nube
        - **Unidades de streaming**: 1
    - **Almacenamiento**:
        - Selecciona **Agregar cuenta de almacenamiento**: seleccionado.
        - **Suscripción** : su suscripción a Azure.
        - **Cuentas de almacenamiento**: selecciona la cuenta de almacenamiento **datalake*xxxxxxx***
        - **Modo de autenticación**: cadena de conexión.
        - **Proteger datos privados en la cuenta de almacenamiento**: seleccionado
    - **Etiquetas:**
        - *Ninguno*
3. Espera a que finalice la implantación y después ve al recurso del trabajo de Stream Analytics implementado.

### Crear una entrada para el flujo de datos de eventos

1. En la página de información general **ingest-orders**, selecciona la página **Entradas**. Usa el menú **Agregar entrada** para agregar un **Centro de eventos** con las siguientes propiedades:
    - **Alias de entrada**: `orders`
    - **Seleccionar Event Hubs de entre tus suscripciones**: seleccionada
    - **Suscripción** : su suscripción a Azure.
    - **Espacio de nombres de Event Hubs**: selecciona el espacio de nombres de Event Hubs **events*xxxxxxx***
    - **Nombre de Event Hubs**: selecciona el Event Hubs **eventhub*xxxxxxx*** existente.
    - **Grupo de consumidores de Event Hub**: selecciona el grupo de consumidores **Usar existente** y luego **$Default**.
    - **Modo de autenticación**: crear una identidad administrada asignada por el sistema
    - **Clave de partición**: *dejar en blanco*
    - **Formato de serialización de eventos**: JSON
    - **Codificación**: UTF-8
2. Guarda la entrada y espera mientras se crea. Verás varias notificaciones. Espera a una notificación de **prueba de conexión correcta**.

### Creación de una salida para la tabla SQL

1. Visualiza la página **Salidas** del trabajo de Stream Analytics **ingest-orders**. A continuación, usa el menú **Agregar salida** para agregar una salida de **Azure Synapse Analytics** con las siguientes propiedades:
    - **Alias de salida**: `FactOrder`
    - **Seleccionar Azure Synapse Analytics desde sus suscripciones**: seleccionado.
    - **Suscripción** : su suscripción a Azure.
    - **Base de datos**: selecciona la base de datos **sql*xxxxxxx* (synapse*xxxxxxx *)**
    - **Modo de autenticación**: autenticación de SQL Server
    - **Nombre de usuario**: SQLUser
    - **Contraseña**: *la contraseña que has especificado para el grupo de SQL al ejecutar el script de configuración*
    - **Tabla**: `FactOrder`
2. Guarda la salida y espera mientras se crea. Verás varias notificaciones. Espera a una notificación de **prueba de conexión correcta**.

### Creación de una consulta para ingerir el flujo de eventos

1. Visualiza la página **Consulta** del trabajo de Stream Analytics **ingest-order**. A continuación, espera unos instantes hasta que se muestre la vista previa de entrada (en función de los eventos de pedido de ventas capturados anteriormente en el centro de eventos).
2. Observa que los datos de entrada incluyen los campos **ProductID** y **Quantity** en los mensajes enviados por la aplicación cliente, así como campos adicionales de Event Hubs, incluido el campo **EventProcessedUtcTime** que indica cuándo se agregó el evento al centro de eventos.
3. Modifique la consulta predeterminada de la siguiente manera:

    ```
    SELECT
        EventProcessedUtcTime AS OrderDateTime,
        ProductID,
        Quantity
    INTO
        [FactOrder]
    FROM
        [orders]
    ```

    Observa que esta consulta toma campos de la entrada (centro de eventos) y los escribe directamente en la salida (tabla SQL).

4. Guarde la consulta.

### Ejecución del trabajo de streaming para ingerir datos de pedido

1. Visualiza la página **Información general** del trabajo de Stream Analytics **ingest-orders** y en la pestaña **Propiedades**, revisa las **Entradas**, **Consulta**, **Salidas** y **Funciones** del trabajo. Si el número de **Entradas** y **Salidas** es 0, usa el botón **↻ Actualizar** de la página **Información general** para mostrar la entrada **pedidos** y la salida **FactTable**.
2. Selecciona el botón **▷ Iniciar** e inicia ahora el trabajo de streaming. Espera a recibir la notificación de que el trabajo de streaming se ha iniciado correctamente.
3. Vuelve a abrir el panel de Cloud Shell y vuelve a ejecutar el siguiente comando para enviar otros 100 pedidos.

    ```
    node ~/dp-203/Allfiles/labs/18/orderclient
    ```

4. Mientras se ejecuta la aplicación cliente de pedidos, cambia a la pestaña del explorador de Synapse Studio y visualiza la consulta que has ejecutado antes para seleccionar las 100 primeras filas de la tabla **.dbo.FactOrder**.
5. Usa el botón **▷ Ejecutar** para volver a ejecutar la consulta y comprobar que la tabla ahora contiene datos de pedido del flujo de eventos (si no es así, espera un minuto y vuelve a ejecutar la consulta). El trabajo de Stream Analytics insertará todos los datos de eventos nuevos en la tabla siempre que el trabajo se esté ejecutando y se envíen eventos de pedido al centro de eventos.
6. En la página **Administrar**, pausa el grupo **sql*xxxxxxx*** (para evitar cargos innecesarios de Azure).
7. Vuelva a la pestaña del explorador que contiene Azure Portal y minimiza el panel de Cloud Shell. Luego usa el botón ** Detener** para detener el trabajo de Stream Analytics y espera a recibir la notificación de que el trabajo de Stream Analytics se ha detenido correctamente.

## Resumen de los datos de streaming en un lago de datos

Hasta ahora, has visto cómo usar un trabajo de Stream Analytics para ingerir mensajes de un origen de streaming en una tabla SQL. Ahora vamos a explorar cómo usar Azure Stream Analytics para agregar datos a través de ventanas temporales, en este caso, para calcular la cantidad total de cada producto vendido cada 5 segundos. También exploraremos cómo usar un tipo diferente de salida para el trabajo escribiendo los resultados en formato CSV en un almacén de blobs de lago de datos.

### Creación de un trabajo de Azure Stream Analytics para agregar datos de pedido

1. En el menú de Azure Portal o en la página **Inicio**, selecciona **+ Crear un recurso** y busca `Stream Analytics job`. Después, crea un trabajo de **Stream Analytics** con las siguientes propiedades:
    - **Aspectos básicos**:
        - **Suscripción** : su suscripción a Azure.
        - **Grupo de recursos**: selecciona el grupo de recursos **dp203-*xxxxxxx*** existente.
        - **Nombre**: `aggregate-orders`
        - **Región**: selecciona la <u>misma región</u> donde se aprovisiona el área de trabajo de Synapse Analytics.
        - **Entorno de hospedaje**: nube
        - **Unidades de streaming**: 1
    - **Almacenamiento**:
        - Selecciona **Agregar cuenta de almacenamiento**: seleccionado.
        - **Suscripción** : su suscripción a Azure.
        - **Cuentas de almacenamiento**: selecciona la cuenta de almacenamiento **datalake*xxxxxxx***
        - **Modo de autenticación**: cadena de conexión.
        - **Proteger datos privados en la cuenta de almacenamiento**: seleccionado
    - **Etiquetas:**
        - *Ninguno*

2. Espera a que finalice la implantación y después ve al recurso del trabajo de Stream Analytics implementado.

### Creación de una entrada para los datos de pedido sin procesar

1. En la página de información general **aggregate-orders**, selecciona la página **Entradas**. Usa el menú **Agregar entrada** para agregar un **Centro de eventos** con las siguientes propiedades:
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

### Creación de una salida del almacén de lago de datos

1. Visualiza la página **Salidas** del trabajo de Stream Analytics **aggregate-orders**. Luego usa el menú **Agregar salida** para agregar una salida **Blob Storage/ADLS Gen2** con las siguientes propiedades:
    - **Alias de salida**: `datalake`
    - **Selecciona Seleccionar almacenamiento de blobs/ADLS Gen2 de entre las suscripciones de las suscripciones**: seleccionado.
    - **Suscripción** : su suscripción a Azure.
    - **Cuenta de almacenamiento**: selecciona la cuenta de almacenamiento **datalake*xxxxxxx***
    - **Contenedor**: selecciona **Usar existente** y en la lista selecciona el contenedor **archivos**
    - **Modo de autenticación**: cadena de conexión.
    - **Formato de serialización de eventos**: CSV - Coma (,)
    - **Codificación**: UTF-8
    - **Modo de escritura**: anexar, a medida que llegan los resultados
    - **Patrón de la ruta de acceso**: `{date}`
    - **Formato de fecha**: AAAA/MM/DD
    - **Formato de hora**: *no aplicable*
    - **Número mínimo de filas**: 20
    - **Tiempo máximo**: 0 horas, 1 minutos y 0 segundos
2. Guarda la salida y espera mientras se crea. Verás varias notificaciones. Espera a una notificación de **prueba de conexión correcta**.

### Creación de una consulta para agregar los datos del evento

1. Visualiza la página **Consulta** del trabajo de Stream Analytics **aggregate-orders**.
2. Modifique la consulta predeterminada de la siguiente manera:

    ```
    SELECT
        DateAdd(second,-5,System.TimeStamp) AS StartTime,
        System.TimeStamp AS EndTime,
        ProductID,
        SUM(Quantity) AS Orders
    INTO
        [datalake]
    FROM
        [orders] TIMESTAMP BY EventProcessedUtcTime
    GROUP BY ProductID, TumblingWindow(second, 5)
    HAVING COUNT(*) > 1
    ```

    Observa que esta consulta usa la ventana **System.Timestamp** (basada en el campo **EventProcessedUtcTime**) para definir el inicio y el final de cada ventana de *saltos de tamaño constante* de 5 segundos (secuencial no superpuesta) en la que se calcula la cantidad total de cada identificador de producto.

3. Guarde la consulta.

### Ejecución del trabajo de streaming para agregar datos de pedido

1. Visualiza la página **Información general** del trabajo de Stream Analytics **aggregate-orders** y en la pestaña **Propiedades**, revisa las funciones **Entradas**, **Consulta**, **Salidas** y **Funciones** del trabajo. Si el número de **Entradas** y **Salidas** es 0, usa el botón **↻ Actualizar** de la página ** Información general** para mostrar la entrada de **pedidos** y la salida de **datalake**.
2. Selecciona el botón **▷ Iniciar** e inicia ahora el trabajo de streaming. Espera a recibir la notificación de que el trabajo de streaming se ha iniciado correctamente.
3. Vuelve a abrir el panel de Cloud Shell y vuelve a ejecutar el siguiente comando para enviar otros 100 pedidos:

    ```
    node ~/dp-203/Allfiles/labs/18/orderclient
    ```

4. Una vez finalizada la aplicación de pedidos, minimiza el panel de Cloud Shell. Después, cambia a la pestaña del explorador de Synapse Studio y, en la página **Datos**, en la pestaña **Vinculado**, expande **Azure Data Lake Storage Gen2** > **synapse*xxxxxxx* (primary - datalake*xxxxxxx *)** y selecciona el contenedor **archivos (principal)**.
5. Si el contenedor **archivos** está vacío, espera un minuto o así y luego usa el botón **↻ Actualizar** para actualizar la vista. Finalmente, se debe mostrar una carpeta con el nombre del año actual. Esta a su vez contiene carpetas del mes y del día.
6. Selecciona la carpeta del año y, en el menú **Nuevo script SQL**, selecciona **Seleccionar las 100 primeras filas**. A continuación, establece el **Tipo de archivo** en **Formato de texto** y aplica la configuración.
7. En el panel de consulta que se abre, modifica la consulta para agregar un parámetro `HEADER_ROW = TRUE` como se muestra aquí:

    ```sql
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/2023/**',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0',
            HEADER_ROW = TRUE
        ) AS [result]
    ```

8. Usa el botón **▷ Ejecutar** para ejecutar la consulta de SQL y ver los resultados, que muestran la cantidad de cada producto solicitado en periodos de cinco segundos.
9. Vuelve a la pestaña del explorador que contiene Azure Portal y usa el botón ** Detener** para detener el trabajo de Stream Analytics y espera la notificación de que el trabajo de Stream Analytics se detuvo correctamente.

## Eliminación de recursos de Azure

Si has terminado de explorar Azure Stream Analytics, debes eliminar los recursos que has creado para evitar costos innecesarios de Azure.

1. Cierra la pestaña del explorador de Azure Synapse Studio y vuelve a Azure Portal.
2. En Azure Portal, en la página **Inicio**, seleccione **Grupos de recursos**.
3. Selecciona el grupo de recursos **dp203-*xxxxxxx*** que contiene tus recursos de Azure Synapse, Event Hubs y Stream Analytics (no el grupo de recursos administrados).
4. En la parte superior de la página **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
5. Especifica el nombre del grupo de recursos **dp203-*xxxxxxx*** para confirmar que quieres eliminarlo y selecciona **Eliminar**.

    Después de unos minutos, se eliminarán los recursos creados en este ejercicio.
