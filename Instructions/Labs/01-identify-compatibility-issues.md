---
lab:
  title: Identificación de problemas de compatibilidad para la migración de SQL
---

# Identificación de problemas de compatibilidad para la migración de SQL

En nuestro escenario, se le ha pedido que evalúe la preparación de una base de datos de SQL Server heredada para la migración a Azure SQL Database. Su tarea consiste en realizar una valoración de la base de datos heredada e identificar los posibles problemas de compatibilidad o cambios que deban realizarse antes de la migración. También debe revisar el esquema de la base de datos e identificar las características o configuraciones que no sean compatibles con Azure SQL Database.

Este ejercicio dura aproximadamente **15** minutos.

> **Nota**: Para realizar este ejercicio, necesita acceso a una suscripción de Azure a fin de crear recursos de Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?azure-portal=true) antes de empezar.

## Antes de comenzar

Para ejecutar este ejercicio, asegúrese de que cumple los siguientes requisitos previos antes de continuar:

- Necesitará SQL Server 2019 o una versión posterior, junto con la base de datos ligera [**AdventureWorksLT**](https://learn.microsoft.com/sql/samples/adventureworks-install-configure#download-backup-files) compatible con su instancia específica de SQL Server.
- Descargar e instalar [Azure Data Studio.](https://learn.microsoft.com/sql/azure-data-studio/download-azure-data-studio) Si ya está instalado, actualícelo para asegurarse de que usa la versión más reciente.
- Un usuario de SQL con acceso de lectura a la base de datos de origen.

## Restaurar una base de datos de SQL Server y ejecutar un comando

1. Seleccione el botón Inicio de Windows y escriba SSMS. Seleccione **Microsoft SQL Server Management Studio 18** en la lista.  

1. Cuando se abra SSMS, observe que el cuadro de diálogo **Conectar al servidor** se rellenará previamente con el nombre de instancia predeterminado. Seleccione **Conectar**.

1. Seleccione la carpeta **Bases de datos** y, a continuación, **Nueva consulta**.

1. En la ventana Nueva consulta, copie y pegue T-SQL. Asegúrese de que la ruta de acceso y el nombre de archivo de copia de seguridad de la base de datos coincidan con el archivo de copia de seguridad real. En caso contrario, se producirá un error en el comando. Ejecute la consulta para restaurar la base de datos.

    ```sql
    RESTORE DATABASE AdventureWorksLT
    FROM DISK = 'C:\<FolderName>\AdventureWorksLT2019.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorksLT2019_Data' 
            TO 'C:\<FolderName>\AdventureWorksLT2019.mdf',
          MOVE 'AdventureWorksLT2019_Log'
            TO 'C:\<FolderName>\AdventureWorksLT2019.ldf';
    ```

    > **Nota**: Asegúrese de tener el archivo de copia de seguridad [AdventureWorks](https://learn.microsoft.com/sql/samples/adventureworks-install-configure#download-backup-files) ligero en la máquina SQL Server antes de ejecutar el comando T-SQL.

1. Debería ver un mensaje correcto una vez completada la restauración.

1. Ejecute el comando siguiente en la base de datos **AdventureWorksLT** de la instancia de SQL Server.

```sql
ALTER TABLE [SalesLT].[Customer] ADD [Next] VARCHAR(5);
```

## Instale e inicie la extensión de migración de Azure para Azure Data Studio

Para instalar la extensión de migración, siga estos pasos: Si la extensión de migración de Azure ya está instalada, puede omitir estos pasos.

1. Abra el administrador de extensiones en Azure Data Studio. s

1. Busque ***Migración de Azure SQL*** e instale la extensión. Una vez que la instale, encontrará la extensión de migración de Azure SQL en la lista de extensiones instaladas.

1. Seleccione el icono **Conexiones** y, después, seleccione **Nueva conexión**. 

1. En la pestaña **Nueva conexión**, escriba el nombre del servidor. Seleccione **Opcional (Falso)** para la opción **Cifrar**.

1. Seleccione **Conectar**. 

1. Para iniciar la extensión de migración de Azure, haga clic con el botón derecho en el nombre de la instancia de origen y seleccione **Administrar**. 

1. En el menú del servidor, en **General**, seleccione **Migración de Azure SQL**. Esto le llevará a la página principal de la extensión Migración de Azure SQL.

    > **Nota**: Si no puede ver la opción **Migración de Azure SQL** en el menú servidor o si la página Migración de Azure SQL no se carga, vuelva a abrir Azure Data Studio.

## Ejecución de la valoración de compatibilidad

La evaluación de compatibilidad ayuda a identificar posibles problemas de migración y proporciona instrucciones detalladas sobre cómo resolverlos antes de que comience el proceso de migración. Esto puede ahorrar tiempo y recursos significativos. 

Ejecutará la extensión de migración de Azure para Azure Data Studio y el de valoraciones de compatibilidad y, a continuación, verá los resultados para una instancia de Azure SQL Database de destino.

1. En el panel Azure SQL Migration, seleccione **Migrar a Azure SQL** para abrir el Asistente para la migración.

1. En **Paso 1: Bases de datos para valoración**, seleccione la base de datos *AdventureWorks* y, a continuación, seleccione **Siguiente**.

1. En **Paso 2: Resultados y recomendaciones de la valoración**, espere a que se complete la valoración.

## Revisión de los resultados de la valoración

Ahora puede revisar las recomendaciones generadas por la extensión de migración.

1. En **Paso 2: Resultados y recomendaciones de la valoración**, seleccione **Azure SQL Database** como plataforma de destino.

1. En la parte inferior de la página, elija **Ver/Seleccionar** para ver los resultados de la valoración. 

1. Seleccione la base de datos *AdventureWorks*. Dedique un momento a revisar los resultados de la valoración que aparecen en el lado derecho.
    
    > **Nota:** Podemos ver que la columna `Next` que se agregó anteriormente se ha marcado, ya que puede provocar un error en Azure SQL Database.

1. Seleccione **Cancelar** y elija **Azure SQL Managed Instance** en su lugar como plataforma de destino de **Azure SQL**.
    
    > **Nota**: La columna `Next` que se agregó anteriormente ya no aparece marcada para Azure SQL Managed Instance, ¿cuál es el motivo? 
    >
    >Esto significa que la columna `Next` se puede usar de forma segura en Azure SQL Managed Instance.

1. Seleccione **Guardar informe de la valoración** para guardar el informe en formato JSON.

1. Dedique un momento a revisar el archivo JSON y sus propiedades.

## Corrección del problema

1. Ejecute el comando T-SQL siguiente en la base de datos *AdventureWorks*.

    ```sql
    ALTER TABLE [SalesLT].[Customer] DROP COLUMN [Next];
    ```

1. Vuelva a la página **Paso 2: Resultados y recomendaciones de la evaluación** en el asistente y seleccione **Actualizar evaluación**.

1. Seleccione **Azure SQL Database** como plataforma de destino de **Azure SQL**.

1. Seleccione **Ver/Seleccionar** para ver los resultados de la evaluación.

    > **Nota:** El problema ya no está marcado.

Ha aprendido cómo valorar la preparación de una base de datos de SQL Server para la migración a Azure SQL Database. Al solucionar los problemas de compatibilidad y realizar cambios esenciales en el esquema o notificarlos, ha dado un paso importante para mitigar posibles problemas técnicos que pueden surgir en el futuro en Azure SQL Database.
