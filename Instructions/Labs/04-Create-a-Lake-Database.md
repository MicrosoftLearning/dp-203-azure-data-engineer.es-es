---
lab:
  title: Análisis de datos en una base de datos de lago
  ilt-use: Suggested demo
---

# Análisis de datos en una base de datos de lago

Azure Synapse Analytics permite combinar la flexibilidad de almacenamiento de archivos en un lago de datos con las funcionalidades de consulta de SQL y esquema estructurado de una base de datos relacional mediante la capacidad de crear una *base de datos de lago*. Una base de datos de lago es un esquema de base de datos relacional definido en un almacén de archivos de lago de datos que permite separar el almacenamiento de datos del proceso que se usa para consultarlo. Las bases de datos de lago combinan las ventajas de un esquema estructurado que incluye compatibilidad con tipos de datos, relaciones y otras características que normalmente solo se encuentran en sistemas de bases de datos relacionales, con la flexibilidad de almacenar datos en archivos que se pueden usar independientemente de un almacén de bases de datos relacionales. Básicamente, la base de datos de lago "superpone" un esquema relacional a los archivos de carpetas del lago de datos.

Este ejercicio debería tardar en completarse **45** minutos aproximadamente.

## Antes de empezar

Necesitará una [suscripción de Azure](https://azure.microsoft.com/free) en la que tenga acceso de nivel administrativo.

## Aprovisionar un área de trabajo de Azure Synapse Analytics

Para admitir una base de datos de lago, necesita un área de trabajo de Azure Synapse Analytics con acceso a Data Lake Storage. No es necesario un grupo de SQL dedicado, ya que puede definir la base de datos de lago mediante el grupo de SQL sin servidor integrado. Opcionalmente, también puedes usar un grupo de Spark para trabajar con datos en la base de datos de lago.

En este ejercicio, usarás una combinación de un script de PowerShell y una plantilla de ARM para aprovisionar un área de trabajo de Azure Synapse Analytics.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) en `https://portal.azure.com`.
2. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** y crear almacenamiento si se solicita. Cloud Shell proporciona una interfaz de línea de comandos en un panel situado en la parte inferior de Azure Portal, como se muestra a continuación:

    ![Azure Portal con un panel de Cloud Shell](./images/cloud-shell.png)

    > **Nota**: Si ha creado previamente un cloud shell que usa un entorno de *Bash*, use el menú desplegable de la parte superior izquierda del panel de cloud shell para cambiarlo a ***PowerShell***.

3. Tenga en cuenta que puede cambiar el tamaño de Cloud Shell arrastrando la barra de separación en la parte superior del panel, o usando los iconos **&#8212;** , **&#9723;** y **X** en la parte superior derecha para minimizar, maximizar y cerrar el panel. Para obtener más información sobre el uso de Azure Cloud Shell, consulta la [documentación de Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. En el panel de PowerShell, esscribe los siguientes comandos para clonar este repositorio:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Una vez clonado el repositorio, escribe los siguientes comandos para cambiar a la carpeta de este ejercicio y ejecutar el script **setup.ps1** que contiene:

    ```
    cd dp-203/Allfiles/labs/04
    ./setup.ps1
    ```

6. Si se solicita, elige la suscripción que quieres usar (esto solo ocurrirá si tienes acceso a varias suscripciones de Azure).
7. Cuando se te solicite, escribe una contraseña adecuada que se va a establecer para el grupo de SQL de Azure Synapse.

    > **Nota**: Asegúrate de recordar esta contraseña.

8. Espera a que se complete el script: normalmente tarda unos 10 minutos, pero en algunos casos puede tardar más. Mientras esperas, revisa los artículos sobre las [base de datos de lago](https://docs.microsoft.com/azure/synapse-analytics/database-designer/concepts-lake-database)y [plantillas de base de datos de lago](https://docs.microsoft.com/azure/synapse-analytics/database-designer/concepts-database-templates) en la documentación de Azure Synapse Analytics.

## Modificación de los permisos del contenedor

1. Una vez finalizada la secuencia de comandos de implementación, en Azure Portal, vaya al grupo de recursos **dp203-*xxxxxxx*** que ha creado y observe que este grupo de recursos contiene su área de trabajo de Synapse, una cuenta de almacenamiento para su lago de datos y un grupo de Apache Spark.
1. Seleccione la **Cuenta de almacenamiento** para su lago de datos denominada **datalakexxxxxxx** 

     ![Navegación del lago de datos al contenedor](./images/datalakexxxxxx-storage.png)

1. En el contenedor **datalakexxxxxx** selecciona la **carpeta de archivos**

    ![Selección de la carpeta de archivos en el contenedor de lago de datos](./images/dp203-Container.png)

1. Dentro de la **carpeta de archivos** observará que el **Método de autenticación:** aparece en la lista como ***Clave de acceso (cambiar a cuenta de usuario de Entra)*** haga clic en ella para cambiar a cuenta de usuario de Entra.

    ![Cambia a la cuenta de usuario de Azure AD.](./images/dp203-switch-to-aad-user.png)
## Creación de una base de datos de lago

Una base de datos de lago es un tipo de base de datos que puedes definir en el área de trabajo y trabajar con el grupo de SQL sin servidor integrado.

1. Selecciona el área de trabajo de Synapse y en tu página **Información general**, en la tarjeta **Abrir Synapse Studio**, selecciona **Abrir** para abrir Synapse Studio en una nueva pestaña del explorador; inicia sesión si se solicita.
2. En el lado izquierdo de Synapse Studio, usa el icono **&rsaquo;&rsaquo;** para expandir el menú, lo que revelará las diferentes páginas dentro de Synapse Studio que usarás para administrar recursos y realizar tareas de análisis de datos.
3. En la página **Datos**, ve a la pestaña **Vinculado** y comprueba que el área de trabajo incluye un vínculo a la cuenta de almacenamiento de Azure Data Lake Storage Gen2.
4. En la página **Datos**, vuelve a la pestaña **Área de trabajo** y observa que no hay ninguna base de datos en tu área de trabajo.
5. En el menú **+**, selecciona **Base de datos de Lake** para abrir una nueva pestaña en la que puedes diseñar el esquema de la base de datos (aceptando los términos de uso de las plantillas de base de datos si se solicita).
6. En el panel **Propiedades** de la nueva base de datos, cambia el **Nombre** por **RetailDB** y comprueba que la propiedad **carpeta de entrada** se actualiza automáticamente a **archivos/RetailDB**. Deje el **Formato de datos** como **Texto delimitado** (también puedes usar  le formato *Parquet* y puedes invalidar el formato de archivo para tablas individuales: usaremos datos delimitados por comas en este ejercicio).
7. En la parte superior del panel **RetailDB**, selecciona **Publicar** para guardar la base de datos hasta ese momento.
8. En el panel **Datos** de la izquierda, visualiza la pestaña **Vinculado**. Después, expande **Azure Data Lake Storage Gen2** y el almacén **datalake*xxxxxxx*** principal para el área de trabajo **Synapse*xxxxxxx*** y selecciona el sistema de archivos **files**, que actualmente contiene una carpeta llamada **synapse**.
9.  En la pestaña **archivos** que se ha abierto, usa el botón **+ Nueva carpeta** para crear una carpeta llamada **RetailDB**; esta será la carpeta de entrada de los archivos de datos usados por las tablas de la base de datos.

## Creación de una tabla

Ahora que has creado una base de datos de lago, puedes definir su esquema mediante la creación de tablas.

### Definir el esquema de la tabla

1. Vuelve a la pestaña **RetailDB** para definir tu base de datos y, en la lista **+ Tabla**, selecciona **Personalizada**, y observa que se agrega una nueva tabla denominada **Tabla_1** a tu base de datos.
2. Con **Tabla_1** seleccionada, en la pestaña **General** bajo el lienzo de diseño de la base de datos, cambia la propiedad **Nombre** por **Cliente**.
3. Expande la sección **Configuración de Storage para la tabla** y observa que la tabla se almacenará como texto delimitado en la carpeta **files/RetailDB/Cliente** en el almacenamiento de lago de datos predeterminado para tu área de trabajo de Synapse.
4. En la pestaña **Columnas**, observa que, de forma predeterminada, la tabla contiene una columna denominada **Columna_1**. Edita la definición de la columna para que coincida con las siguientes propiedades:

    | Nombre | Claves | Descripción | Nulabilidad | Tipo de datos | Formato / Duración |
    | ---- | ---- | ----------- | ----------- | --------- | --------------- |
    | CustomerId | PK &#128505; | Id. de cliente único | &#128454;  | long | |

5. En la lista **+ Columna**, selecciona **Nueva columna** y modifica la definición de la nueva columna para agregar una columna **Nombre** a la tabla de la siguiente manera:

    | Nombre | Claves | Descripción | Nulabilidad | Tipo de datos | Formato / Duración |
    | ---- | ---- | ----------- | ----------- | --------- | --------------- |
    | CustomerId | PK &#128505; | Id. de cliente único | &#128454;  | long | |
    | **Nombre** | **PK &#128454;** | **Nombre del cliente** | **&#128454;** | **string** | **256** |

6. Agrega más columnas nuevas hasta que la definición de la tabla tenga este aspecto:

    | Nombre | Claves | Descripción | Nulabilidad | Tipo de datos | Formato / Duración |
    | ---- | ---- | ----------- | ----------- | --------- | --------------- |
    | CustomerId | PK &#128505; | Id. de cliente único | &#128454;  | long | |
    | Nombre | PK &#128454; | Nombre del cliente | &#128454; | string | 256 |
    | Apellidos | PK &#128454; | Apellidos del cliente | &#128505; | string | 256 |
    | EmailAddress | PK &#128454; | Correo electrónico del cliente | &#128454; | string | 256 |
    | Teléfono | PK &#128454; | Teléfono del cliente | &#128505; | string | 256 |

7. Cuando hayas agregado todas las columnas, vuelve a publicar la base de datos para guardar los cambios.
8. En el panel **Datos** de la izquierda, vuelve a la pestaña **Área de trabajo** para que puedas ver la base de datos del lago **RetailDB**. A continuación, expándela y actualiza su carpeta **Tablas** para ver la tabla **Customer** recién creada.

### Carga de datos en la ruta de acceso de almacenamiento de la tabla

1. En el panel principal, vuelve a la pestaña **archivos**, que contiene el sistema de archivos con la carpeta **RetailDB**. A continuación, abre la carpeta **RetailDB** y crea en ella una nueva carpeta denominada **Cliente**. Aquí es de donde la tabla **Cliente** obtendrá sus datos.
2. Abre la nueva carpeta **Cliente**, que debe estar vacía.
3. Descarga el archivo de datos **customer.csv** desde [https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/04/data/customer.csv](https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/04/data/customer.csv) y guárdalo en una carpeta del equipo local (no importa dónde). A continuación, en la carpeta **Cliente** del Explorador de Synapse, usa el botón **⤒ Cargar** para cargar el archivo **customer.csv** en la carpeta **RetailDB/Customer** del lago de datos.

    > **Nota**: En un escenario de producción real, probablemente crearías una canalización para ingerir datos en la carpeta de los datos de la tabla. En este ejercicio, lo más práctico es cargarlos directamente en la interfaz de usuario de Synapse Studio.

4. En el panel **Datos** de la izquierda, en la pestaña **Área de trabajo** del menú **...** de la tabla **Cliente**, selecciona **Nuevo script SQL** > **Seleccionar las primeras 100 filas**. A continuación, en el nuevo panel **Script de SQL 1** que se ha abierto, asegúrate de que el grupo SQL **integrado** está conectado y usa el botón **▷ Ejecutar** para ejecutar el código SQL. Los resultados deben incluir las primeras 100 filas de la tabla **Cliente**, en función de los datos almacenados en la carpeta subyacente del lago de datos.
5. Cierra la pestaña **Script de SQL 1** y descarta los cambios.

## Creación de una tabla desde una platilla de base de datos

Como has visto, puedes crear las tablas que necesitas en la base de datos del lago desde cero. Sin embargo, Azure Synapse Analytics también proporciona numerosas plantillas de base de datos basadas en cargas de trabajo y entidades de base de datos comunes que puedes usar como punto de partida para el esquema de la base de datos.

### Definir el esquema de la tabla

1. En el panel principal, vuelve al panel **RetailDB**, que contiene el esquema de la base de datos (actualmente solo contiene la tabla **Cliente**).
2. En el menú **+ Tabla**, selecciona **Desde plantilla**. A continuación, en la página **Agregar desde plantilla**, selecciona **Comercial** y haz clic en **Continuar**.
3. En la página **Agregar desde plantilla (Comercial),** espera a que se rellene la lista de tablas y, a continuación, expande **Producto** y selecciona **RetailProduct**. A continuación, haga clic en **Agregar**. Esto agrega una nueva tabla basada en la plantilla **RetailProduct** a la base de datos.
4. En el panel **RetailDB**, selecciona la nueva tabla **RetailProduct**. A continuación, en el panel debajo del lienzo de diseño, en la pestaña **General**, cambia el nombre a **Producto** y comprueba que la configuración de almacenamiento de la tabla especifica la carpeta de entrada **files/RetailDB/Product**.
5. En la pestaña **Columnas** de la tabla **Producto**, ten en cuenta que la tabla ya incluye un gran número de columnas heredadas de la plantilla. Hay más columnas de las obligatorias en esta tabla, por lo que tendrás que quitar algunas.
6. Selecciona la casilla junto a **Nombre** para seleccionar todas las columnas y después <u>anula</u> la selecciona las siguientes columnas (que debes conservar):
    - ProductId
    - ProductName
    - IntroductionDate
    - ActualAbandonmentDate
    - ProductGrossWeight
    - ItemSku
7. En la barra de herramientas del panel **Columnas**, selecciona **Eliminar** para quitar las columnas seleccionadas. Así obtendrías las siguientes columnas:

    | Nombre | Claves | Descripción | Nulabilidad | Tipo de datos | Formato / Duración |
    | ---- | ---- | ----------- | ----------- | --------- | --------------- |
    | ProductId | PK &#128505; | El identificador único de un producto. | &#128454;  | long | |
    | ProductName | PK &#128454; | El nombre del producto... | &#128505; | string | 128 |
    | IntroductionDate | PK &#128454; | La fecha en que el producto se puso a la venta. | &#128505; | date | YYYY-MM-DD |
    | ActualAbandonmentDate | PK &#128454; | Fecha real en la que se retiró el marketing del producto... | &#128505; | date | AAAA-MM-DD |
    | ProductGrossWeight | PK &#128454; | El peso bruto del producto. | &#128505; | Decimal | 18.8 |
    | ItemSku | PK &#128454; | Identificador de la unidad de mantenimiento de existencias... | &#128505; | string | 20 |

8. Agrega una nueva columna denominada **ListPrice** a la tabla como se muestra aquí:

    | Nombre | Claves | Descripción | Nulabilidad | Tipo de datos | Formato / Duración |
    | ---- | ---- | ----------- | ----------- | --------- | --------------- |
    | ProductId | PK &#128505; | El identificador único de un producto. | &#128454;  | long | |
    | ProductName | PK &#128454; | El nombre del producto... | &#128505; | string | 128 |
    | IntroductionDate | PK &#128454; | La fecha en que el producto se puso a la venta. | &#128505; | date | YYYY-MM-DD |
    | ActualAbandonmentDate | PK &#128454; | Fecha real en la que se retiró el marketing del producto... | &#128505; | date | AAAA-MM-DD |
    | ProductGrossWeight | PK &#128454; | El peso bruto del producto. | &#128505; | Decimal | 18.8 |
    | ItemSku | PK &#128454; | Identificador de la unidad de mantenimiento de existencias... | &#128505; | string | 20 |
    | **ListPrice** | **PK &#128454;** | **El precio del producto.** | **&#128454;** | **decimal** | **18,2** |

9. Cuando hayas modificado las columnas como se muestra anteriormente, vuelva a publicar la base de datos para guardar los cambios.
10. En el panel **Datos** de la izquierda, vuelve a la pestaña **Área de trabajo** para que puedas ver la base de datos del lago **RetailDB**. Luego usa el menú **...** de la carpeta **Tablas** para actualizar la vista y ver la tabla **Producto** recién creada.

### Carga de datos en la ruta de acceso de almacenamiento de la tabla

1. En el panel principal, vuelve a la pestaña **archivos**, que contiene el sistema de archivos, y ve a la carpeta **files/RetailDB**, que contiene actualmente la carpeta **Cliente** de la tabla que creaste anteriormente.
2. En la carpeta **RetailDB**, crea una carpeta denominada **Producto**. Aquí es de donde la tabla **Producto** obtendrá sus datos.
3. Abre la nueva carpeta **Producto**, que debe estar vacía.
4. Descarga el archivo de datos **product.csv** desde [https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/04/data/product.csv](https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/04/data/product.csv) y guárdalo en una carpeta del equipo local (no importa dónde). A continuación, en la carpeta **Producto** del Explorador de Synapse, usa el botón **⤒ Cargar** para cargar el archivo **product.csv** en la carpeta **RetailDB/Product** del lago de datos.
5. En el panel **Datos** de la izquierda, en la pestaña **Área de trabajo** del menú **...** de la tabla **Producto**, selecciona **Nuevo script SQL** > **Seleccionar las primeras 100 filas**. A continuación, en el nuevo panel **Script de SQL 1** que se ha abierto, asegúrate de que el grupo SQL **integrado** está conectado y usa el botón **▷ Ejecutar** para ejecutar el código SQL. Los resultados deberían incluir las primeras 100 filas de la tabla **Producto**, en función de los datos almacenados en la carpeta subyacente del lago de datos.
6. Cierra la pestaña **Script de SQL 1** y descarta los cambios.

## Creación de una tabla a partir de datos existentes

Hasta ahora, has creado tablas y, a continuación, las has rellenado con datos. En algunos casos, es posible que ya tengas datos en un lago de datos a partir del cual quieras derivar una tabla.

### Carga de datos

1. En el panel principal, vuelve a la pestaña **archivos**, que contiene el sistema de archivos, y ve a la carpeta **files/RetailDB**, que contiene actualmente las carpetas **Cliente** y **Producto** de las tablas que creaste anteriormente.
2. En la carpeta **RetailDB**, crea una carpeta denominada **SalesOrder**.
3. Abra la nueva carpeta **SalesOrder**, que debe estar vacía.
4. Descarga el archivo de datos **salesorder.csv** desde [https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/04/data/salesorder.csv](https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/04/data/salesorder.csv) y guárdalo en una carpeta del equipo local (no importa dónde). A continuación, en la carpeta **SalesOrder** del Explorador de Synapse, usa el botón **⤒ Cargar** para cargar el archivo **salesorder.csv** en la carpeta **RetailDB/SalesOrder** del lago de datos.

### Creación de una tabla

1. En el panel principal, vuelve al panel **RetailDB**, que contiene el esquema de la base de datos (que contiene actualmente las tablas **Cliente** y **Producto**).
2. En el menú **+ Tabla**, selecciona **Desde lago de datos**. A continuación, en el panel **Crear tabla externa desde lago de datos**, especifica las siguientes opciones:
    - **Nombre de la tabla externa**: SalesOrder
    - **Servicio vinculado**: selecciona **synapse*xxxxxxx*-WorkspaceDefautStorage(datalake*xxxxxxx*)**
    - **Archivo de entrada de la carpeta**: files/RetailDB/SalesOrder
3. Continúa con la página siguiente y, a continuación, crea la tabla con las siguientes opciones:
    - **Tipo de archivo**: CSV
    - **Terminador de campo**: predeterminado (coma ,)
    - **Primera fila**: deja *Inferir nombres de columna*<u>sin</u> seleccionar.
    - **Delimitador de cadena**: predeterminado (cadena vacía)
    - **Usar el tipo predeterminado**: tipo predeterminado (true,false)
    - **Longitud de cadena máxima**: 4000

4. Cuando se haya creado la tabla, ten en cuenta que incluye columnas denominadas **C1**, **C2**, etc. y que los tipos de datos se han inferido a partir de los datos de la carpeta. Modifica las definiciones de columna de la siguiente manera:

    | Nombre | Claves | Descripción | Nulabilidad | Tipo de datos | Formato / Duración |
    | ---- | ---- | ----------- | ----------- | --------- | --------------- |
    | SalesOrderId | PK &#128505; | El identificador único de un pedido. | &#128454;  | long | |
    | OrderDate | PK &#128454; | La fecha del pedido. | &#128454; | timestamp | yyyy-MM-dd |
    | LineItemId | PK &#128505; | El id. de un elemento de línea individual. | &#128454; | long | |
    | CustomerId | PK &#128454; | El cliente. | &#128454; | long | |
    | ProductId | PK &#128454; | El producto. | &#128454; | long | |
    | Cantidad | PK &#128454; | La cantidad del pedido. | &#128454; | long | |

    > **Nota**: La tabla contiene un registro para cada elemento individual pedido e incluye una clave principal compuesta formada por **SalesOrderId** y **LineItemId**.

5. En la pestaña **Relaciones** de la tabla **SalesOrder**, en la lista **+ Relación**, selecciona **Tabla de destino** y luego define la siguiente relación:

    | Desde la tabla | Desde la columna | Tabla de destino | A la columna |
    | ---- | ---- | ----------- | ----------- |
    | Customer | CustomerId | Pedido de ventas | CustomerId |

6. Agrega una segunda relación *Tabla de destino* con la siguiente configuración:

    | Desde la tabla | Desde la columna | Tabla de destino | A la columna |
    | ---- | ---- | ----------- | ----------- |
    | Producto | ProductId | Pedido de ventas | ProductId |

    La capacidad de definir relaciones entre tablas ayuda a aplicar la integridad referencial entre entidades de datos relacionadas. Se trata de una característica común de las bases de datos relacionales que, de lo contrario, serían difíciles de aplicar a los archivos de un lago de datos.

7. Vuelve a publicar la base de datos para guardar los cambios.
8. En el panel **Datos** de la izquierda, vuelve a la pestaña **Área de trabajo** para que puedas ver la base de datos del lago **RetailDB**. A continuación, usa el menú **...** de su carpeta **Tablas** para actualizar la vista y ver la tabla **SalesOrder** recién creada.

## Trabajar con tablas de bases de datos de lago

Ahora que tienes algunas tablas en tu base de datos, puedes usarlas para trabajar con los datos subyacentes.

### Consulta de tablas con SQL

1. En Synapse Studio, seleccione la página **Desarrollar**.
2. En el panel **Desarrollar**, en el menú **+**, selecciona **Script SQL**.
3. En el nuevo panel **SQL script 1**, asegúrate de que el script está conectado al grupo de SQL **integrado** y en la lista **Base de datos del usuario**, selecciona **RetailDB**.
4. Introduce el siguiente código SQL:

    ```sql
    SELECT o.SalesOrderID, c.EmailAddress, p.ProductName, o.Quantity
    FROM SalesOrder AS o
    JOIN Customer AS c ON o.CustomerId = c.CustomerId
    JOIN Product AS p ON o.ProductId = p.ProductId
    ```

5. Usa el botón **▷ Ejecutar** para ejecutar el código SQL.

    Los resultados muestran los detalles del pedido con información sobre el cliente y el producto.

6. Cierra el panel **SQL script 1** y descarta los cambios.

### Insertar datos con Spark

1. En el panel **Desarrollar**, en el menú **+**, selecciona **Cuaderno**.
2. En el nuevo panel **Notebook 1**, asocia el cuaderno al grupo **spark*xxxxxxx**** fr Spark.
3. Especifica el siguiente código en la celda vacía del cuaderno:

    ```
    %%sql
    INSERT INTO `RetailDB`.`SalesOrder` VALUES (99999, CAST('2022-01-01' AS TimeStamp), 1, 6, 5, 1)
    ```

4. Usa el botón **▷** a la izquierda de la celda para ejecutarla y espera a que termine de ejecutarse. Ten en cuenta que el grupo de Spark tardará algún tiempo en iniciarse.
5. Usa el botón **+ Código** para agregar una nueva celda al cuaderno.
6. En la celda nueva, escribe el código siguiente:

    ```
    %%sql
    SELECT * FROM `RetailDB`.`SalesOrder` WHERE SalesOrderId = 99999
    ```
7. Usa el botón **▷** situado a la izquierda de la celda para ejecutarla y comprueba que se ha insertado una fila para el pedido de ventas 99999 en la tabla **SalesOrder**.
8. Cierra el panel **Cuaderno 1** para detener sesión de Spark y descartar los cambios.

## Eliminación de recursos de Azure

Si ha terminado de explorar Azure Synapse Analytics, debe eliminar los recursos que ha creado para evitar costos innecesarios de Azure.

1. Cierre la pestaña del explorador de Synapse Studio y vuelva a Azure Portal.
2. En Azure Portal, en la página **Inicio**, seleccione **Grupos de recursos**.
3. Selecciona el grupo de recursos **dp203-*xxxxxxx*** para tu área de trabajo de Synapse Analytics (no el grupo de recursos administrados) y verifica que contiene el área de trabajo de Synapse, la cuenta de almacenamiento y el grupo de Spark para tu área de trabajo.
4. En la parte superior de la página **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
5. Especifica el nombre del grupo de recursos **dp203-*xxxxxxx*** para confirmar que quieres eliminarlo y selecciona **Eliminar**.

    Después de unos minutos, se eliminarán el grupo de recursos de área de trabajo de Azure Synapse y el grupo de recursos de área de trabajo administrado asociado a él.
