---
lab:
  title: "Migración de bases de datos de SQL\_Server a Azure SQL Managed Instance mediante Log Replay Service"
---

# Migración de bases de datos de SQL Server a Azure SQL Managed Instance mediante Log Replay Service

En este ejercicio, aprenderá a migrar una base de datos de SQL Server a Azure SQL Managed Instance mediante Log Replay Service. 

Para empezar, implementará una instancia de Azure SQL Managed Instance. A continuación, usará Log Replay Service para realizar una migración en línea de una base de datos de SQL Server a Azure SQL Managed Instance. También aprenderá a supervisar el proceso de migración en PowerShell.

Este ejercicio dura aproximadamente **45** minutos.

> **Nota**: Para realizar este ejercicio, necesita acceso a una suscripción de Azure a fin de crear recursos de Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?azure-portal=true) antes de empezar.

## Antes de comenzar

Para ejecutar este ejercicio, necesitará lo siguiente:

| Elemento | Descripción |
| --- | --- |
| **Servidor de destino** | Una Azure SQL Managed Instance. Lo crearemos durante este ejercicio.|
| **Servidor de origen** | Una instancia de SQL Server 2019 o una [versión más reciente](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) instalada en un servidor de su preferencia. |
| **Base de datos de origen** | La base de datos ligera [AdventureWorks](https://learn.microsoft.com/sql/samples/adventureworks-install-configure) que se va a restaurar en la instancia de SQL Server. |
| **Azure Data Studio** | Instale [Azure Data Studio](https://learn.microsoft.com/sql/azure-data-studio/download-azure-data-studio) en el mismo servidor donde se encuentra la base de datos de origen. Si ya está instalado, actualícelo para asegurarse de que usa la versión más reciente. |

## Restaurar una base de datos de SQL Server

Vamos a restaurar la base de datos *AdventureWorks2016* en la instancia de SQL Server. Esta base de datos sirve como base de datos de origen para este ejercicio de laboratorio. Puede omitir estos pasos si la base de datos ya está restaurada.

1. Seleccione el botón Inicio de Windows y escriba SSMS. Seleccione **Microsoft SQL Server Management Studio 18** en la lista.  

1. Cuando se abra SSMS, observará que el cuadro de diálogo **Conectar al servidor** se rellena previamente con el nombre de la instancia predeterminada. Seleccione **Conectar**.

1. Seleccione la carpeta **Bases de datos** y, a continuación, **Nueva consulta**.

1. En la ventana de consulta, copie y pegue el código T-SQL siguiente. Ejecute la consulta para restaurar la base de datos.

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

## Implementación de una instancia de Azure SQL Managed Instance

Siga estos pasos para crear una instancia de Azure SQL Managed Instance:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true) y, a continuación, seleccione **Crear un recurso** en la esquina superior izquierda.
1. Busque la opción de **instancia administrada**, seleccione **Azure SQL Managed Instance** y, después, elija **Crear**.
1. Rellene el formulario de la instancia administrada de SQL con la información de la tabla siguiente:

    |  | Valor sugerido |
    |---|---|
    | **Suscripción** | Su suscripción. |
    | **nombre de la instancia administrada** | Cualquier nombre válido. |
    | **inicio de sesión de administrador de instancia administrada** | Cualquier nombre de usuario válido. No utilice "serveradmin", ya es un rol de nivel de servidor reservado. |
    | **Contraseña** | Cualquier contraseña de más de 16 caracteres y que cumple los requisitos de complejidad. |
    | **Zona horaria** | La zona horaria que debe observar la instancia administrada. |
    | **Intercalación** | Intercalación que se quiere usar para la instancia administrada. Si migra bases de datos desde SQL Server, compruebe la intercalación de origen mediante SELECT SERVERPROPERTY(N'Collation') y use ese valor. |
    | **Ubicación** | Región de Azure en la que se quiere crear la instancia administrada. |
    | **Red virtual** | Seleccione "Crear una nueva red virtual" o una subred y red virtual válidas. |
    | **Habilitar el punto de conexión público** | Active esta opción para habilitar un punto de conexión público, lo que ayuda a los clientes fuera de Azure a acceder a la base de datos. |
    | **Permitir acceso desde** | Seleccione entre los servicios de Azure, Internet o sin acceso. |
    | **Tipo de conexión** | Elija entre el tipo de conexión: Proxy o Redirigir. |
    | **Grupo de recursos** | un grupo de recursos nuevo o existente. |

1. Seleccione **Plan de tarifa** para cambiar el tamaño de los recursos de almacenamiento y de proceso, así como para revisar las opciones del plan de tarifa. 
1. Cuando haya terminado, seleccione **Aplicar** para guardar la selección y, después, elija **Crear** para implementar la instancia administrada.
1. Seleccione el icono de **Notificaciones** para ver el estado de la implementación.
1. Seleccione **Implementación en curso** para abrir la ventana de la instancia administrada y supervisar el progreso de la implementación.

## Creación de una cuenta y un contenedor de Azure Blob Storage

Cree una cuenta de Azure Blob Storage en la misma región que la instancia de Azure SQL Managed Instance. Aquí es donde almacenará las copias de seguridad de la base de datos para la migración.

1. Vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de su cuenta.
1. En el menú izquierdo, seleccione **Todos los servicios** y busque *"Cuentas de almacenamiento"*. Seleccione **Cuentas de almacenamiento** para abrir la página de cuentas de almacenamiento.
1. En la página Cuentas de almacenamiento, seleccione **+Agregar** para crear una nueva cuenta de almacenamiento.
1. En la pestaña **Aspectos básicos** de la página **Crear cuenta de almacenamiento**, seleccione la suscripción que quiere usar para la cuenta de almacenamiento. A continuación, seleccione el grupo de recursos que contiene la instancia de Azure SQL Managed Instance.
1. Escriba un nombre único para la cuenta de almacenamiento. 
    
    > **Nota:** El nombre debe tener entre 3 y 24 caracteres, y solo puede contener letras minúsculas y números.

1. Seleccione la ubicación (región) donde se encuentra la instancia de Azure SQL Managed Instance.
1. Seleccione el nivel de rendimiento de la cuenta de almacenamiento.
1. Seleccione **BlobStorage** como el tipo de cuenta de almacenamiento. 
1. Seleccione **Almacenamiento con redundancia local (LRS)** como opción de replicación de la cuenta de almacenamiento.
1. Revise los datos especificados y seleccione **Revisar y crear** para crear la cuenta de almacenamiento.
1. Una vez creada la cuenta de almacenamiento, vaya a la página de la cuenta de almacenamiento y seleccione la opción **Contenedores** en el menú izquierdo. Luego, seleccione **+Contenedor** para crear un contenedor. Escriba un nombre para el contenedor y seleccione el nivel de acceso público. 
1. Seleccione el botón **Crear** para crear el contenedor.

Después de completar estos pasos, tendrá una cuenta de Azure Blob Storage en la misma región que la instancia de Azure SQL Managed Instance, y un contenedor donde puede almacenar las copias de seguridad de base de datos para la migración.

## Copia de seguridad de una base de datos SQL Server

Vamos a crear una copia de seguridad completa de la base de datos *AdventureWorksLT* en la instancia de SQL Server, seguida de una copia de seguridad diferencial y de registros con `CHECKSUM` habilitado. 

1. Seleccione el botón Inicio de Windows y escriba SSMS. Seleccione **Microsoft SQL Server Management Studio 18** en la lista.  
1. Cuando se abra SSMS, observará que el cuadro de diálogo **Conectar al servidor** se rellena previamente con el nombre de la instancia predeterminada. Seleccione **Conectar**.
1. Seleccione la carpeta **Bases de datos** y, a continuación, **Nueva consulta**.
1. En la ventana Nueva consulta, copie y pegue T-SQL. Ejecute la consulta para restaurar la base de datos.

    ```sql
    BACKUP DATABASE AdventureWorksLT
    TO DISK = 'C:\LabFiles\AdventureWorksLT_full.bak'
    WITH CHECKSUM;

    BACKUP DATABASE AdventureWorksLT
    TO DISK = 'C:\LabFiles\AdventureWorksLT_diff.dif'
    WITH DIFFERENTIAL, CHECKSUM;

    BACKUP LOG AdventureWorksLT
    TO DISK = 'C:\LabFiles\AdventureWorksLT_log.trn'
    WITH CHECKSUM;
    ```

    > **Nota**: Asegúrese de que la ruta de acceso del archivo del ejemplo anterior coincida con la ruta de acceso del archivo real. Si no lo hacen, es posible que se produzca un error en el comando.

1. Debería ver un mensaje correcto una vez completada la restauración.
1. Si ejecuta una versión de SQL Server (a partir de SQL Server 2012 SP1 CU2 y SQL Server 2014), puede realizar copias de seguridad de SQL Server directamente en la cuenta de Blob Storage mediante la opción nativa de SQL Server `BACKUP TO URL`. 

    ```sql
    CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/<containername>] 
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE',  
    SECRET = '<SAS_TOKEN>';  
    GO
    
    -- Take a full database backup to a URL
    BACKUP DATABASE [AdventureWorksLT]
    TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<databasefolder>/AdventureWorksLT_full.bak'
    WITH INIT, COMPRESSION, CHECKSUM
    GO
    
    -- Take a differential database backup to a URL
    BACKUP DATABASE [AdventureWorksLT]
    TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<databasefolder>/AdventureWorksLT_diff.bak'  
    WITH DIFFERENTIAL, COMPRESSION, CHECKSUM
    GO
    
    -- Take a transactional log backup to a URL
    BACKUP LOG [AdventureWorksLT]
    TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<databasefolder>/AdventureWorksLT_log.trn'  
    WITH COMPRESSION, CHECKSUM
    ```

    > **Nota:** Si decide usar esta opción, puede omitir la siguiente sección **Copia de archivos de copia de seguridad en la cuenta de Azure Storage**.

## Copia de archivos de copia de seguridad en la cuenta de Azure Storage

Ahora vamos a copiar los archivos de copia de seguridad en la cuenta de Azure Blob Storage que creó anteriormente.

1. Vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de su cuenta.
1. En el menú de la izquierda, seleccione **Cuentas de almacenamiento** y, luego, elija la cuenta de almacenamiento que creó anteriormente.
1. En la página de información general de la cuenta de almacenamiento, desplácese hacia abajo hasta la sección **Blob service** y seleccione **Contenedores**. Seleccione el contenedor que creó anteriormente.
1. Seleccione **Cargar** en la parte superior de la página del contenedor. En la página **Cargar blob**, seleccione **Carpeta** para seleccionar la carpeta que contiene los archivos de copia de seguridad o elija **Archivos** para elegir archivos de copia de seguridad individuales. Cuando haya seleccionado los archivos, seleccione **Cargar** para iniciar el proceso de carga.

## Validación del acceso

Es importante validar si su servidor de SQL Server y su instancia de SQL Managed Instance pueden acceder correctamente a la cuenta de Blob Storage. Para ello, ejecute una consulta de prueba de ejemplo para determinar si la instancia administrada puede acceder a la copia de seguridad del contenedor.

1. Conéctese a su instancia de SQL Managed Instance a través de SSMS.
1. Abra un nuevo editor de consultas y ejecute el comando.

```sql
CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/databases] 
WITH IDENTITY = 'SHARED ACCESS SIGNATURE' 
, SECRET = '<sastoken>' 

RESTORE HEADERONLY 
FROM URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<backup_file_name>.bak'
```
1. Repita este proceso conectado a la instancia de SQL Server.

## Uso de Log Replay Service para restaurar archivos de copia de seguridad

Usará Log Replay Service (LRS) para restaurar los archivos de copia de seguridad de Azure Blob Storage en la instancia de Azure SQL Managed Instance. LRS es un servicio gratuito basado en la tecnología de trasvase de registros de SQL Server.

1. En la página de información general de la cuenta de almacenamiento, desplácese hacia abajo hasta la sección **Blob service** y seleccione **Contenedores**. Seleccione el contenedor donde se almacenan los archivos de copia de seguridad.
1. Seleccione **Generar SAS** en la parte superior de la página del contenedor. En la página **Generar firma de acceso compartido**, seleccione los permisos que quiere conceder, establezca la hora de inicio y expiración del token de SAS y, luego, elija **Generar la cadena de conexión y SAS**. El token de SAS se mostrará en el campo **Token de SAS**; cópielo.
1. Use PowerShell para conectarse a su cuenta de Azure ejecutando el cmdlet `Connect-AzAccount`.

    ```powershell
    Login-AzAccount
    Select-AzSubscription -SubscriptionId <subscription ID>
    ```

1. Use el cmdlet `Start-AzSqlInstanceDatabaseLogReplay` para iniciar Log Replay Service para la base de datos que quiere restaurar. Deberá proporcionar el nombre del grupo de recursos, el nombre de la instancia, el nombre de la base de datos, el URI del contenedor de almacenamiento y el token de SAS que copió anteriormente.

```PowerShell
Import-Module Az.Sql

Start-AzSqlInstanceDatabaseLogReplay -ResourceGroupName "YourResourceGroupName" -InstanceName "YourInstanceName" -Name "YourDatabaseName" -StorageContainerUri "https://yourstorageaccount.blob.core.windows.net/yourcontainer" -StorageContainerSasToken "YourSasToken"
```

## Supervisión del progreso de la migración

Puede usar el cmdlet `Get-AzSqlInstanceDatabaseLogReplay` para supervisar el progreso de Log Replay Service. Este cmdlet devuelve información sobre el estado actual del servicio, incluido el último archivo de copia de seguridad de registros que se restauró.

1. Ejecute el siguiente código de PowerShell.

```powershell
# Import the Az.Sql module
Import-Module Az.Sql

# Set the resource group name, instance name, and database name
$resourceGroupName = "YourResourceGroupName"
$instanceName = "YourInstanceName"
$databaseName = "YourDatabaseName"

# Get the log replay status
$logReplayStatus = Get-AzSqlInstanceDatabaseLogReplay -ResourceGroupName $resourceGroupName -InstanceName $instanceName -Name $databaseName

# Display the log replay status
$logReplayStatus | Format-List
```

## Realización de la migración total

Una vez restaurada la copia de seguridad de la base de datos completa en la instancia de destino de la instancia administrada de Azure SQL Database, la base de datos está disponible para realizar una migración total.

1. Cuando esté listo para completar la migración de la base de datos en línea, seleccione **Iniciar transición**.
1. Detenga todo el tráfico de entrada a las bases de datos de origen.
1. Realice la copia del final del registro, proporcione el archivo de copia de seguridad en el recurso compartido de red SMB y espere a que se restaure la copia de seguridad del registro de transacciones final.
1. En ese momento, verá que el valor de **Cambios pendientes** es 0.
1. Seleccione **Confirmar** y, después, **Aplicar**.

    ![Pantalla del total de migración](../media/3-migration-cutover-screen.png)

1. Cuando el estado de la migración de la base de datos muestre **Completado**, conecte las aplicaciones a la nueva instancia destino de la instancia administrada de Azure SQL Database.