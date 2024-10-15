---
lab:
  title: Creación de un informe en tiempo real con Azure Stream Analytics y Microsoft Power BI
  ilt-use: Suggested demo
---

# Creación de un informe en tiempo real con Azure Stream Analytics y Microsoft Power BI

Las soluciones de análisis de datos suelen incluir un requisito para ingerir y procesar *secuencias* de datos. El procesamiento de secuencias difiere del procesamiento por lotes en que las secuencias suelen ser *sin límites*; es decir, son orígenes continuos de datos que se deben procesar perpetuamente en lugar de a intervalos fijos.

Azure Stream Analytics proporciona un servicio en la nube que puedes usar para definir una *consulta* que opera en un flujo de datos de un origen de secuencia, como Azure Event Hubs o Azure IoT Hub. Puedes usar una consulta de Azure Stream Analytics para procesar un flujo de datos y enviar los resultados directamente a Microsoft Power BI para la visualización en tiempo real.

En este ejercicio, usarás Azure Stream Analytics para procesar un flujo de datos de pedidos de ventas como el que podría generarse en una aplicación comercial en línea. Los datos de pedidos se enviarán a Azure Event Hubs, desde donde el trabajo de Azure Stream Analytics leerá y resumirá los datos antes de enviarlos a Power BI, donde visualizarás los datos en un informe.

Este ejercicio debería tardar en completarse **45** minutos aproximadamente.

## Antes de empezar

Necesitará una [suscripción de Azure](https://azure.microsoft.com/free) en la que tenga acceso de nivel administrativo.

También necesitará acceso al servicio Microsoft Power BI. Es posible que en el centro educativo o la organización ya se proporcione, o bien puede [registrarse para obtener el servicio Power BI como individuo](https://learn.microsoft.com/power-bi/fundamentals/service-self-service-signup-for-power-bi).

## Aprovisionamiento de los recursos de Azure

En este ejercicio, necesitarás un área de trabajo de Azure Synapse Analytics con acceso a Data Lake Storage y a un grupo de SQL dedicado. También necesitarás un espacio de nombres de Azure Event Hubs al que se puedan enviar los datos de pedidos de la secuencia.

Usarás una combinación de un script de PowerShell y una plantilla de ARM para aprovisionar estos recursos.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) en `https://portal.azure.com`.
2. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** y crear almacenamiento si se solicita. Cloud Shell proporciona una interfaz de línea de comandos en un panel situado en la parte inferior de Azure Portal, como se muestra a continuación:

    ![Captura de pantalla de Azure Portal con un panel de Cloud Shell.](./images/cloud-shell.png)

    > **Nota**: Si has creado previamente un Cloud Shell que usa un entorno de *Bash*, usa el menú desplegable situado en la parte superior izquierda del panel de Cloud Shell para cambiarlo a ***PowerShell***.

3. Ten en cuenta que puedes cambiar el tamaño de Cloud Shell arrastrando la barra de separación en la parte superior del panel, o usando los iconos **&#8212;** , **&#9723;** y **X** en la parte superior derecha para minimizar, maximizar y cerrar el panel. Para obtener más información sobre el uso de Azure Cloud Shell, consulte la [documentación de Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. En el panel de PowerShell, escribe los siguientes comandos para clonar el repositorio que contiene este ejercicio:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Una vez clonado el repositorio, escribe los siguientes comandos para cambiar a la carpeta de este ejercicio y ejecuta el script **setup.ps1** que contiene:

    ```
    cd dp-203/Allfiles/labs/19
    ./setup.ps1
    ```

6. Si se te solicita, elige la suscripción que deseas usar (esto solo ocurrirá si tienes acceso a varias suscripciones de Azure).

7. Mientras esperas a que se complete el script, continúa con la siguiente tarea.

## Creación de un área de trabajo de Power BI

En el servicio Power BI, organizarás conjuntos de datos, informes y otros recursos en *áreas de trabajo*. Cada usuario de Power BI tiene un área de trabajo predeterminada denominada **Mi área de trabajo**, que puedes usar en este ejercicio; no obstante, generalmente es recomendable crear un área de trabajo para cada solución de informes discreta que quieras administrar.

1. Inicia sesión en el servicio Power BI en [https://app.powerbi.com/](https://app.powerbi.com/) con sus credenciales de servicio Power BI.
2. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
3. Crea un área de trabajo con un nombre descriptivo (por ejemplo, *mslearn-streaming*), seleccionando el modo de licencia **Profesional**.

    > **Nota**: Si usas una cuenta de prueba, es posible que tengas que habilitar características de prueba adicionales.

4. Cuando veas el área de trabajo, fíjate en tu identificador único global (GUID) en la dirección URL de la página (que debe ser similar a `https://app.powerbi.com/groups/<GUID>/list`). Necesitarás este GUID más adelante.

## Utilizar Azure Stream Analytics para procesar transmisiones de datos

Un trabajo de Azure Stream Analytics define una consulta perpetua que funciona en transmisiones de datos de una o varias entradas y envía los resultados a una o varias salidas.

### Creación de un trabajo de Stream Analytics

1. Vuelve a la pestaña del explorador de Azure Portal y, cuando termine el script, anota la región donde se aprovisionó el grupo de recursos **dp203-*xxxxxxx***.
2. En la página **Inicio** de Azure Portal, selecciona **+ Crear un recurso** y busca `Stream Analytics job`. Después, crea un trabajo de **Stream Analytics** con las siguientes propiedades:
    - **Suscripción** : su suscripción a Azure.
    - **Grupo de recursos**: selecciona el grupo de recursos **dp203-*xxxxxxx*** existente.
    - **Nombre**: `stream-orders`
    - **Región**: selecciona la región donde se aprovisiona el área de trabajo de Synapse Analytics.
    - **Entorno de hospedaje**: nube
    - **Unidades de streaming**: 1
3. Espera a que finalice la implantación y después ve al recurso del trabajo de Stream Analytics implementado.

### Crear una entrada para el flujo de datos de eventos

1. En la página de información general **stream-orders**, selecciona la página **Entradas** y usa el menú **Agregar entrada** para agregar una entrada de **Event Hub** con las siguientes propiedades:
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

### Crear una salida para el área de trabajo de Power BI

1. Consulta la página **Salidas** del trabajo **stream-orders** de Stream Analytics. Después, usa el menú **Agregar salida** para agregar una salida **Power BI** con las siguientes propiedades:
    - **Alias de salida**: `powerbi-dataset`
    - **Seleccionar manualmente la configuración de Power BI**: seleccionada
    - **Area de trabajo del grupo**: *el GUID para tu área de trabajo*
    - **Modo de autenticación**: *selecciona***Token de usuario*** y usa el botón ***Autorizar*** de la parte inferior para iniciar sesión en tu cuenta de Power BI*
    - **Nombre del conjunto de datos**: `realtime-data`
    - **Nombre de la tabla**: `orders`

2. Guarda la salida y espera mientras se crea. Verás varias notificaciones. Espera a una notificación de **prueba de conexión correcta**.

### Crear una consulta para resumir el flujo de eventos

1. Consulta la página **Consulta** para ver el trabajo **stream-orders** de Stream Analytics.
2. Modifique la consulta predeterminada de la siguiente manera:

    ```
    SELECT
        DateAdd(second,-5,System.TimeStamp) AS StartTime,
        System.TimeStamp AS EndTime,
        ProductID,
        SUM(Quantity) AS Orders
    INTO
        [powerbi-dataset]
    FROM
        [orders] TIMESTAMP BY EventEnqueuedUtcTime
    GROUP BY ProductID, TumblingWindow(second, 5)
    HAVING COUNT(*) > 1
    ```

    Observa que esta consulta usa **System.Timestamp** (basado en el campo **EventEnqueuedUtcTime**) para definir el inicio y el final de cada *salto* de ventana (secuencial no solapado) de 5 segundos en la que se calcula la cantidad total para cada id. de producto.

3. Guarde la consulta.

### Ejecutar el trabajo de streaming para procesar los datos del pedido

1. Consulta la página **Información general** del trabajo **stream-orders** de Stream Analytics y, en la pestaña **Propiedades**, revisa las opciones **Entradas**, **Consulta**, **Salidas** y **Funciones** del trabajo. Si el número de **Entradas** y **Salidas** es 0, usa el botón **↻ Actualizar** de la página **Información general** para mostrar la entrada **pedidos** y la salida **powerbi-dataset**.
2. Selecciona el botón **▷ Iniciar** e inicia ahora el trabajo de streaming. Espera a recibir la notificación de que el trabajo de streaming se ha iniciado correctamente.
3. Vuelve a abrir el panel de Cloud Shell y ejecuta el siguiente comando para enviar 100 pedidos.

    ```
    node ~/dp-203/Allfiles/labs/19/orderclient
    ```

4. Mientras se ejecuta la aplicación cliente de pedidos, ve a la pestaña del explorador de la aplicación de Power BI y consulta el área de trabajo.
5. Actualiza la página de la aplicación de Power BI hasta que veas el conjunto de datos **realtime-data** en tu área de trabajo. El trabajo de Azure Stream Analytics genera este conjunto de datos.

## Visualizar los datos de streaming en Power BI

Ahora que tienes un conjunto de datos para los datos del pedido de streaming, puedes crear un panel de Power BI que lo represente visualmente.

1. Vuelve a la pestaña del explorador de PowerBI.

2. En el menú desplegable **+ Nuevo** de tu área de trabajo, selecciona **Panel** y crea un nuevo panel denominado **Seguimiento de pedidos**.

3. En el panel **Seguimiento de pedidos**, selecciona el menú **✏️ Editar** y después selecciona **+ Agregar ventana**. Después, en el panel **Agregar ventana**, selecciona **Datos de streaming personalizados** y selecciona **Siguiente**:

4. En el panel **Agregar un icono de datos de streaming personalizados**, en **Sus conjuntos de datos**, selecciona el conjunto de datos **realtime-data** y selecciona **Siguiente**.

5. Cambia el tipo de visualización predeterminado a **Gráfico de líneas**. Después, configura las siguientes propiedades y selecciona **Siguiente**:
    - **Eje**: EndTime
    - **Valor**: Pedidos
    - **Periodo de tiempo para mostrar**: 1 minuto

6. En el panel **Detalles del icono**, establece el **Título** en **Recuento de pedidos en tiempo real** y selecciona **Aplicar**.

7. Vuelve a la pestaña del explorador que contiene Azure Portal y, si es necesario, vuelva a abrir el panel de Cloud Shell. Después, vuelve a ejecutar el siguiente comando para enviar otros 100 pedidos.

    ```
    node ~/dp-203/Allfiles/labs/19/orderclient
    ```

8. Mientras se ejecuta el script de envío de pedidos, vuelve a la pestaña del explorador que contiene el panel de **Seguimiento de pedidos** de Power BI y observa que la visualización se actualiza para reflejar los nuevos datos de los pedidos a medida que los procesa el trabajo de Stream Analytics (que debería seguir ejecutándose).

    ![Captura de pantalla de un informe de Power BI que muestra un flujo de datos de pedido en tiempo real.](./images/powerbi-line-chart.png)

    Puedes volver a ejecutar el script **orderclient** y observar los datos que se capturan en el panel en tiempo real.

## Eliminar recursos

Si has terminado de explorar Azure Stream Analytics y Power BI, debes eliminar los recursos que creaste para evitar costos innecesarios de Azure.

1. Cierra la pestaña del explorador que contiene el informe de Power BI. Después, en el panel **Áreas de trabajo**, en el menú **⋮** de tu área de trabajo, selecciona **Configuración del área de trabajo** y elimina el área de trabajo.
2. Vuelve a la pestaña del explorador que contiene Azure Portal, cierra el panel de Cloud Shell y usa el botón ** Detener** para detener el trabajo de Stream Analytics. Espera a recibir la notificación de que el trabajo de Stream Analytics se detuvo correctamente.
3. En Azure Portal, en la página **Inicio**, seleccione **Grupos de recursos**.
4. Selecciona el grupo de recursos **dp203-*xxxxxxx*** que contiene tus recursos de Azure Event Hub y Stream Analytics.
5. En la parte superior de la página **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
6. Especifica el nombre del grupo de recursos **dp203-*xxxxxxx*** para confirmar que quieres eliminarlo y selecciona **Eliminar**.

    Después de unos minutos, se eliminarán los recursos creados en este ejercicio.
