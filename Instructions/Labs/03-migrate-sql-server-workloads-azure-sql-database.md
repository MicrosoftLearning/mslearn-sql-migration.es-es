---
lab:
  title: "Migración de bases de datos de SQL\_Server a Azure SQL Database"
---

# Migración de bases de datos de SQL Server a Azure SQL Database

En este ejercicio, aprenderá a migrar una base de datos de SQL Server a Azure SQL Database mediante la extensión de migración de Azure para Azure Data Studio. Empezará instalando e iniciando la extensión de migración de Azure para Azure Data Studio. Después, aprenderá a realizar una migración sin conexión de una base de datos de SQL Server a Azure SQL Database. También aprenderá a supervisar el proceso de migración en Azure Portal.

Este ejercicio dura aproximadamente **45** minutos.

> **Nota**: Para realizar este ejercicio, necesita acceso a una suscripción de Azure a fin de crear recursos de Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?azure-portal=true) antes de empezar.

## Antes de comenzar

Para ejecutar este ejercicio, necesitará lo siguiente:

| Elemento | Descripción |
| --- | --- |
| **Servidor de destino** | Un servidor de Azure SQL Database. Lo crearemos durante este ejercicio.|
| **Base de datos de destino** | Una base de datos en un servidor de SQL Database V12. Lo crearemos durante este ejercicio.|
| **Servidor de origen** | Una instancia de SQL Server 2019 o una [versión más reciente](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) instalada en un servidor de su preferencia. |
| **Base de datos de origen** | La base de datos ligera [AdventureWorks](https://learn.microsoft.com/sql/samples/adventureworks-install-configure) que se va a restaurar en la instancia de SQL Server. |
| **Azure Data Studio** | Instale [Azure Data Studio](https://learn.microsoft.com/sql/azure-data-studio/download-azure-data-studio) en el mismo servidor donde se encuentra la base de datos de origen. Si ya está instalado, actualícelo para asegurarse de que usa la versión más reciente. |
| **Microsoft Data Migration Assistant** | Instale [Data Migration Assistant](https://www.microsoft.com/en-us/download/details.aspx?id=53595) en el mismo servidor donde se encuentra la base de datos de origen. |
| Proveedor de recursos **Microsoft.DataMigration** | Asegúrese de que la suscripción esté registrada para usar el espacio de nombres **Microsoft.DataMigration**. Para saber cómo realizar un registro de proveedor de recursos, consulte [Registrar el proveedor de recursos](https://learn.microsoft.com/azure/dms/quickstart-create-data-migration-service-portal#register-the-resource-provider). |
| **Microsoft Integration Runtime**. | Instale [Microsoft Integration Runtime](https://aka.ms/sql-migration-shir-download). |

## Restaurar una base de datos de SQL Server

Vamos a restaurar la base de datos *AdventureWorks2016* en la instancia de SQL Server. Esta base de datos servirá como base de datos de origen para este ejercicio de laboratorio. Puede omitir estos pasos si la base de datos ya está restaurada.

1. Seleccione el botón Inicio de Windows y escriba SSMS. Seleccione **Microsoft SQL Server Management Studio 18** en la lista.  

1. Cuando se abra SSMS, observe que el cuadro de diálogo **Conectar al servidor** se rellenará previamente con el nombre de instancia predeterminado. Seleccione **Conectar**.

1. Seleccione la carpeta **Bases de datos** y, a continuación, **Nueva consulta**.

1. En la ventana Nueva consulta, copie y pegue T-SQL. Ejecute la consulta para restaurar la base de datos.

    ```sql
    RESTORE DATABASE AdventureWorksLT
    FROM DISK = 'C:\LabFiles\AdventureWorksLT2019.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorksLT2019_Data' 
            TO 'C:\LabFiles\AdventureWorksLT2019.mdf',
          MOVE 'AdventureWorksLT2019_Log'
            TO 'C:\LabFiles\AdventureWorksLT2019.ldf';
    ```

    > **Nota**: Asegúrese de que el nombre y la ruta de acceso del archivo de copia de seguridad de la base de datos del ejemplo anterior coincidan con el archivo de copia de seguridad real. Si no lo hacen, es posible que se produzca un error en el comando.

1. Debería ver un mensaje correcto una vez completada la restauración.

## Registro del espacio de nombres Microsoft.DataMigration

Omita estos pasos si el espacio de nombres **Microsoft.DataMigration** ya está registrado en la suscripción.

1. En Azure Portal, busque **Suscripción** en el cuadro de búsqueda de la parte superior y seleccione **Suscripciones**. Seleccione su suscripción en la hoja **Suscripciones**.

1. En la página de la suscripción, en **Configuración**, seleccione **Proveedores de recursos**. Busque **Microsoft.DataMigration** en el cuadro de búsqueda de la parte superior y, después, seleccione **Microsoft.DataMigration**. 

    > **Nota**: Si se abre la barra lateral **Detalles del proveedor de recursos**, puede cerrarla.

1. Seleccione **Registrar**.

## Aprovisionamiento de una base de datos de Azure SQL

Vamos a configurar una instancia de Azure SQL Database que servirá como entorno de destino.

1. En Azure Portal, busque **Bases de datos SQL** en el cuadro de búsqueda de la parte superior y seleccione **Bases de datos SQL**.

1. En la hoja **Bases de datos SQL**, seleccione **+ Crear**.

1. En la página **Crear base de datos SQL**, seleccione las opciones siguientes.

    - **Suscripción**: &lt;Su suscripción&gt;
    - **Grupo de recursos:** &lt;su grupo de recursos&gt;.
    - **Nombre de la base de datos**: AdventureWorksLT.
    - **Servidor:** seleccione el vínculo **Crear nuevo**. Proporcione los detalles del servidor en la página **Crear servidor de SQL Database**.
        - **Nombre del servidor:** &lt;elija un nombre de servidor&gt;. Este nombre debe ser único globalmente.
        - **Ubicación:** &lt;su región, la misma región que la del grupo de recursos&gt;
        - **Método de autenticación**: use la autenticación de SQL
        - **Inicio de sesión del administrador del servidor:** sqladmin
        - **Contraseña**: &lt;Su contraseña&gt;
        - **Confirmar contraseña:** &lt;su contraseña&gt;
        
            **Nota:** Anote este nombre de servidor y las credenciales. Los usará en tareas posteriores.

    - **¿Quiere usar un grupo elástico de SQL?** No
    - **Entorno de carga de trabajo:** producción

1. En **Proceso y almacenamiento**, seleccione **Configurar base de datos**. En la página **Configurar**, en la lista desplegable **Nivel de servicio**, seleccione **Básico** y, a continuación, **Aplicar**.

1. Para la opción **Redundancia de almacenamiento de copia de seguridad**, mantenga el valor predeterminado: **Almacenamiento de copia de seguridad con redundancia geográfica**. Seleccione **Siguiente: Redes**.

1. En la pestaña **Redes**, seleccione **Siguiente: Seguridad** y, a continuación, **Siguiente: Configuración adicional**.

1. En la página **Configuración adicional**, seleccione **Revisar y crear**.

1. Revise la configuración y seleccione **Crear**.

1. Una vez finalizada la implementación, seleccione **Ir al recurso**.

## Habilitar el acceso a Azure SQL Database

Vamos a habilitar el acceso a Azure SQL Database para poder conectarse a ella a través de las herramientas de cliente.

1. En la página de la **base de datos SQL**, seleccione la sección **Información general** y, a continuación, seleccione el vínculo para el nombre del servidor en la sección superior:

1. En la hoja **SQL Server**, seleccione **Redes** en la sección **Seguridad** .

1. En la pestaña **Acceso público**, seleccione **Redes seleccionadas**. 

1. En **Reglas de firewall**, seleccione **+ Agregar la dirección IPv4 del cliente**.

1. En **Excepciones**, active la propiedad **Permitir que los servicios y recursos de Azure accedan a este servidor**. 

1. Seleccione **Guardar**.

## Conexión a Azure SQL Database en Azure Data Studio

Antes de empezar a usar la extensión de migración de Azure, vamos a conectarnos a la base de datos de destino.

1. Inicie Azure Data Studio.

1. Seleccione **Conexiones** y **Agregar conexión**.

1. Rellene los campos **Detalles de conexión** con el nombre de SQL Server y otra información.

    > **Nota**: Escriba el nombre del servidor SQL Server creado anteriormente. El formato debe estar en el formato **<server>.database.windows.net**.

1. En **Tipo de autenticación**, seleccione **Inicio de sesión de SQL** y proporcione el nombre de usuario y la contraseña.

1. Seleccione **Conectar**.

## Instale e inicie la extensión de migración de Azure para Azure Data Studio

Para instalar la extensión de migración, siga estos pasos: Si la extensión ya está instalada, puede omitir estos pasos.

1. Abra el administrador de extensiones en Azure Data Studio.

1. Busque ***Migración de Azure SQL*** y seleccione la extensión.

1. Instale la extensión. Una vez que la instale, encontrará la extensión de migración de Azure SQL en la lista de extensiones instaladas.

1. Conéctese a una instancia de SQL Server en Azure Data Studio. En la nueva pestaña de conexión, seleccione **Opcional (Falso)** para la opción **Cifrar**.

1. Para iniciar la extensión de migración de Azure, haga clic con el botón derecho en el nombre de la instancia de SQL Server y seleccione **Administrar** para acceder al panel y la página de aterrizaje de la extensión de migración de Azure SQL.

    > **Nota**: Si **Azure SQL Migration** no está visible en la barra lateral del panel del servidor, vuelva a abrir Azure Data Studio.
 
## Generación del esquema de la base de datos con DMA

Antes de comenzar la migración, es necesario asegurarse de que el esquema existe en la base de datos de destino. Usamos DMA para crear el esquema a partir del origen y aplicarlo al destino.

1. Inicie Data Migration Assistant.

1. Cree un nuevo proyecto de migración y establezca el tipo de origen en **SQL Server**, el tipo de servidor de destino en **Azure SQL Database** y el ámbito de la migración en **Solo esquema**. Seleccione **Crear**.

    ![Captura de pantalla que muestra cómo iniciar un nuevo proyecto de migración en Data Migration Assistant.](../media/3-data-migration-schema.png) 

1. En la pestaña **Seleccionar origen**, escriba el nombre de la instancia de SQL Server de origen y seleccione **Autenticación de Windows** para **Tipo de autenticación**. Desactive **Cifrar conexión**. 

1. Seleccione **Conectar**.

1. Seleccione la base de datos **AdventureWorksLT** y, después, seleccione **Siguiente**.

1. En la pestaña **Seleccionar destino**, escriba el nombre del servidor de Azure SQL Server de destino, seleccione **Autenticación de SQL Server** para **Tipo de autenticación** y proporcione las credenciales de usuario de SQL. 

1. Seleccione la base de datos **AdventureWorksLT** y, después, seleccione **Siguiente**.

1. En la pestaña **Seleccionar objetos**, seleccione todos los objetos de esquema de la base de datos de origen. Seleccione **Generar script SQL**. 

    ![Captura de pantalla que muestra la pestaña seleccionar objetos en Data Migration Assistant.](../media/3-data-migration-generate.png)

1. Una vez generado el esquema, dedique algún tiempo a revisarlo. Por lo general, este paso implica realizar los ajustes necesarios en el script para los objetos que no se pueden crear en su estado actual en la ubicación de destino, que no es el caso en este escenario.
 
1. Puede ejecutar el script manualmente mediante Azure Data Studio, SQL Management Studio o seleccionando **Implementar esquema**. Continúe con uno de los métodos.

    ![Captura de pantalla que muestra el script generado en Data Migration Assistant.](../media/3-data-migration-script.png)

## Realizar una migración sin conexión de una base de datos de SQL Server a Azure SQL Database.

Ahora estamos listos para migrar los datos. Para realizar una migración sin conexión con Azure Data Studio, siga estos pasos:

1. Inicie el asistente para **migrar a Azure SQL** desde la extensión Azure Data Studio y, después, seleccione **Migrar a Azure SQL**.

1. En **Paso 1: Bases de datos para valoración**, seleccione la base de datos *AdventureWorks* y, a continuación, seleccione **Siguiente**.

1. En **Paso 2: Resultados y recomendaciones de la evaluación**, espere a que se complete la evaluación y, a continuación, seleccione **Azure SQL Database** como destino de **Azure SQL**.

1. En la parte inferior de la página **Paso 2: Resultados y recomendaciones de la evaluación**, seleccione **Ver/Seleccionar** para ver los resultados de la evaluación. Seleccione la base de datos que se va a migrar.

    > **Nota**: Dedique un momento a revisar los resultados de la evaluación que aparecen en el lado derecho.

1. En **Paso 3: Destino de Azure SQL**, si la cuenta aún no está vinculada, asegúrese de agregar una cuenta seleccionando el vínculo **Vincular cuenta**. A continuación, seleccione una cuenta de Azure, un inquilino de AD, una suscripción, una ubicación, un grupo de recursos, un servidor de Azure SQL Database y credenciales de Azure SQL Database.

1. Seleccione **Conectar** y, a continuación, seleccione la base de datos *AdventureWorks* como **Base de datos de destino**. Seleccione **Siguiente**.

1. En **Paso 4: Azure Database Migration Service**, seleccione el vínculo **Crear nuevo** para crear un servicio Azure Database Migration Service mediante el asistente. Siga los pasos proporcionados por el asistente para configurar un nuevo entorno de ejecución de integración autohospedado. Si ha creado anteriormente uno, puede reutilizarlo.

1. En **Paso 5: Configuración del origen de datos**, escriba las credenciales para conectarse a la instancia de SQL Server desde el entorno de ejecución de integración autohospedado. 

1. Seleccione las tablas que se van a migrar desde el origen al destino. 

1. Seleccione **Ejecutar validación**.

    ![Captura de pantalla de la ejecución del paso de validación en la extensión de migración de Azure para Azure Data Studio.](../media/3-run-validation.png) 

1. Una vez que se haya completado el proceso de validación, seleccione **Siguiente**.

1. En **Paso 6: Resumen**, seleccione **Iniciar migración**.

1. Seleccione **Migraciones de base de datos en curso** en el panel de migración para ver el estado de la migración. 

    ![Captura de pantalla del panel de migración en la extensión de migración de Azure para Azure Data Studio.](../media/3-data-migration-dashboard.png)

1. Seleccione el nombre de la base de datos *AdventureWorks* para obtener más detalles.

    ![Captura de pantalla de los detalles de migración en la extensión de migración de Azure para Azure Data Studio.](../media/3-dashboard-sqldb.png)

1. Después de que el estado sea **Correcto**, vaya al servidor de destino y valide la base de datos de destino. Compruebe el esquema y los datos migrados de la base de datos.

Ha aprendido a instalar la extensión de migración y a generar el esquema de la base de datos mediante Data Migration Assistant. En este ejercicio, ha aprendido a migrar una base de datos de SQL Server a Azure SQL Database mediante la extensión de migración de Azure para Azure Data Studio. Una vez completada la migración, puede empezar a usar el nuevo recurso de Azure SQL Database. 

## Limpieza

Cuando trabaje con su propia suscripción, es una buena idea al final de un proyecto identificar si todavía se necesitan los recursos que ha creado. 

Dejar que los recursos se ejecuten innecesariamente puede dar lugar a costes adicionales. Puede eliminar recursos de forma individual o bien eliminar el grupo de recursos completo desde [Azure Portal](https://portal.azure.com?azure-portal=true).

## Más información

Para obtener más información sobre Azure SQL Database, consulte [¿Qué es Azure SQL Database?](https://learn.microsoft.com/en-us/azure/azure-sql/database/sql-database-paas-overview).
