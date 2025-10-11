---
lab:
  title: "Migración de bases de datos de SQL\_Server a Azure SQL Database"
---

# Migración de bases de datos de SQL Server a Azure SQL Database

En este ejercicio, aprenderá a migrar tablas concretas de una base de datos de SQL Server a Azure SQL Database mediante la extensión Migración de Azure para Azure Data Studio. 

> **Importante**: Para evitar afectar a la duración de la migración (que puede verse afectada por restricciones de red y conectividad) solo se migrarán algunas tablas (*Customer*, *ProductCategory*, *Product* y *Address*) como ejemplo. El mismo proceso se puede usar para migrar todo el esquema cuando sea necesario.

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
| Proveedor de recursos **Microsoft.DataMigration** | Asegúrese de que la suscripción esté registrada para usar el espacio de nombres **Microsoft.DataMigration**. Para saber cómo realizar un registro de proveedor de recursos, consulte [Registrar el proveedor de recursos](https://learn.microsoft.com/azure/dms/quickstart-create-data-migration-service-portal#register-the-resource-provider). |
| **Microsoft Integration Runtime**. | Instale [Microsoft Integration Runtime](https://aka.ms/sql-migration-shir-download). |

## Restaurar una base de datos de SQL Server

Vamos a restaurar la base de datos *AdventureWorks2016* en la instancia de SQL Server. Esta base de datos servirá como base de datos de origen para este ejercicio de laboratorio. Puede omitir estos pasos si la base de datos ya está restaurada.

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

1. Para la opción **Redundancia de almacenamiento de copia de seguridad**, mantenga el valor predeterminado: **Almacenamiento de copia de seguridad con redundancia geográfica**. Seleccione **Revisar + crear**.

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

## Crear el esquema de destino

Ahora se creará manualmente el esquema para las tablas de destino antes de comenzar la migración de datos. 

1. En Azure Data Studio, conéctese a la instancia de Azure SQL Database.

1. Haz clic con el botón derecho en la base de datos *AdventureWorksLT* y selecciona **Nueva consulta**.

1. Copie y pegue el siguiente script de T-SQL para crear el esquema de las tablas que se van a migrar:

    ```sql
        CREATE SCHEMA [SalesLT]
        GO
        
        CREATE TYPE [dbo].[Name] FROM [nvarchar](50) NULL
        GO
        
        CREATE TYPE [dbo].[NameStyle] FROM [bit] NOT NULL
        GO
        
        CREATE TYPE [dbo].[Phone] FROM [nvarchar](25) NULL
        GO
        
        CREATE TABLE [SalesLT].[Address](
            [AddressID] [int] IDENTITY(1,1) NOT FOR REPLICATION NOT NULL,
            [AddressLine1] [nvarchar](60) NOT NULL,
            [AddressLine2] [nvarchar](60) NULL,
            [City] [nvarchar](30) NOT NULL,
            [StateProvince] [dbo].[Name] NOT NULL,
            [CountryRegion] [dbo].[Name] NOT NULL,
            [PostalCode] [nvarchar](15) NOT NULL,
            [rowguid] [uniqueidentifier] ROWGUIDCOL  NOT NULL,
            [ModifiedDate] [datetime] NOT NULL,
         CONSTRAINT [PK_Address_AddressID] PRIMARY KEY CLUSTERED 
        (
            [AddressID] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY],
         CONSTRAINT [AK_Address_rowguid] UNIQUE NONCLUSTERED 
        (
            [rowguid] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
        ) ON [PRIMARY]
        GO
        
        CREATE TABLE [SalesLT].[Customer](
            [CustomerID] [int] IDENTITY(1,1) NOT FOR REPLICATION NOT NULL,
            [NameStyle] [dbo].[NameStyle] NOT NULL,
            [Title] [nvarchar](8) NULL,
            [FirstName] [dbo].[Name] NOT NULL,
            [MiddleName] [dbo].[Name] NULL,
            [LastName] [dbo].[Name] NOT NULL,
            [Suffix] [nvarchar](10) NULL,
            [CompanyName] [nvarchar](128) NULL,
            [SalesPerson] [nvarchar](256) NULL,
            [EmailAddress] [nvarchar](50) NULL,
            [Phone] [dbo].[Phone] NULL,
            [PasswordHash] [varchar](128) NOT NULL,
            [PasswordSalt] [varchar](10) NOT NULL,
            [rowguid] [uniqueidentifier] ROWGUIDCOL  NOT NULL,
            [ModifiedDate] [datetime] NOT NULL,
         CONSTRAINT [PK_Customer_CustomerID] PRIMARY KEY CLUSTERED 
        (
            [CustomerID] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY],
         CONSTRAINT [AK_Customer_rowguid] UNIQUE NONCLUSTERED 
        (
            [rowguid] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
        ) ON [PRIMARY]
        GO
        
        CREATE TABLE [SalesLT].[Product](
            [ProductID] [int] IDENTITY(1,1) NOT NULL,
            [Name] [dbo].[Name] NOT NULL,
            [ProductNumber] [nvarchar](25) NOT NULL,
            [Color] [nvarchar](15) NULL,
            [StandardCost] [money] NOT NULL,
            [ListPrice] [money] NOT NULL,
            [Size] [nvarchar](5) NULL,
            [Weight] [decimal](8, 2) NULL,
            [ProductCategoryID] [int] NULL,
            [ProductModelID] [int] NULL,
            [SellStartDate] [datetime] NOT NULL,
            [SellEndDate] [datetime] NULL,
            [DiscontinuedDate] [datetime] NULL,
            [ThumbNailPhoto] [varbinary](max) NULL,
            [ThumbnailPhotoFileName] [nvarchar](50) NULL,
            [rowguid] [uniqueidentifier] ROWGUIDCOL  NOT NULL,
            [ModifiedDate] [datetime] NOT NULL,
         CONSTRAINT [PK_Product_ProductID] PRIMARY KEY CLUSTERED 
        (
            [ProductID] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY],
         CONSTRAINT [AK_Product_Name] UNIQUE NONCLUSTERED 
        (
            [Name] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY],
         CONSTRAINT [AK_Product_ProductNumber] UNIQUE NONCLUSTERED 
        (
            [ProductNumber] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY],
         CONSTRAINT [AK_Product_rowguid] UNIQUE NONCLUSTERED 
        (
            [rowguid] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
        ) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
        GO
        
        CREATE TABLE [SalesLT].[ProductCategory](
            [ProductCategoryID] [int] IDENTITY(1,1) NOT NULL,
            [ParentProductCategoryID] [int] NULL,
            [Name] [dbo].[Name] NOT NULL,
            [rowguid] [uniqueidentifier] ROWGUIDCOL  NOT NULL,
            [ModifiedDate] [datetime] NOT NULL,
         CONSTRAINT [PK_ProductCategory_ProductCategoryID] PRIMARY KEY CLUSTERED 
        (
            [ProductCategoryID] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY],
         CONSTRAINT [AK_ProductCategory_Name] UNIQUE NONCLUSTERED 
        (
            [Name] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY],
         CONSTRAINT [AK_ProductCategory_rowguid] UNIQUE NONCLUSTERED 
        (
            [rowguid] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
        ) ON [PRIMARY]
        GO
        
        ALTER TABLE [SalesLT].[Address] ADD  CONSTRAINT [DF_Address_rowguid]  DEFAULT (newid()) FOR [rowguid]
        GO
        
        ALTER TABLE [SalesLT].[Address] ADD  CONSTRAINT [DF_Address_ModifiedDate]  DEFAULT (getdate()) FOR [ModifiedDate]
        GO
        
        ALTER TABLE [SalesLT].[Customer] ADD  CONSTRAINT [DF_Customer_NameStyle]  DEFAULT ((0)) FOR [NameStyle]
        GO
        
        ALTER TABLE [SalesLT].[Customer] ADD  CONSTRAINT [DF_Customer_rowguid]  DEFAULT (newid()) FOR [rowguid]
        GO
        
        ALTER TABLE [SalesLT].[Customer] ADD  CONSTRAINT [DF_Customer_ModifiedDate]  DEFAULT (getdate()) FOR [ModifiedDate]
        GO
        
        ALTER TABLE [SalesLT].[Product] ADD  CONSTRAINT [DF_Product_rowguid]  DEFAULT (newid()) FOR [rowguid]
        GO
        
        ALTER TABLE [SalesLT].[Product] ADD  CONSTRAINT [DF_Product_ModifiedDate]  DEFAULT (getdate()) FOR [ModifiedDate]
        GO
        
        ALTER TABLE [SalesLT].[ProductCategory] ADD  CONSTRAINT [DF_ProductCategory_rowguid]  DEFAULT (newid()) FOR [rowguid]
        GO
        
        ALTER TABLE [SalesLT].[ProductCategory] ADD  CONSTRAINT [DF_ProductCategory_ModifiedDate]  DEFAULT (getdate()) FOR [ModifiedDate]
        GO
        
        ALTER TABLE [SalesLT].[Product]  WITH CHECK ADD  CONSTRAINT [FK_Product_ProductCategory_ProductCategoryID] FOREIGN KEY([ProductCategoryID])
        REFERENCES [SalesLT].[ProductCategory] ([ProductCategoryID])
        GO
        
        ALTER TABLE [SalesLT].[Product] CHECK CONSTRAINT [FK_Product_ProductCategory_ProductCategoryID]
        GO
        
        ALTER TABLE [SalesLT].[ProductCategory]  WITH CHECK ADD  CONSTRAINT [FK_ProductCategory_ProductCategory_ParentProductCategoryID_ProductCategoryID] FOREIGN KEY([ParentProductCategoryID])
        REFERENCES [SalesLT].[ProductCategory] ([ProductCategoryID])
        GO
        
        ALTER TABLE [SalesLT].[ProductCategory] CHECK CONSTRAINT [FK_ProductCategory_ProductCategory_ParentProductCategoryID_ProductCategoryID]
        GO
        
        ALTER TABLE [SalesLT].[Product]  WITH NOCHECK ADD  CONSTRAINT [CK_Product_ListPrice] CHECK  (([ListPrice]>=(0.00)))
        GO
        
        ALTER TABLE [SalesLT].[Product] CHECK CONSTRAINT [CK_Product_ListPrice]
        GO
        
        ALTER TABLE [SalesLT].[Product]  WITH NOCHECK ADD  CONSTRAINT [CK_Product_SellEndDate] CHECK  (([SellEndDate]>=[SellStartDate] OR [SellEndDate] IS NULL))
        GO
        
        ALTER TABLE [SalesLT].[Product] CHECK CONSTRAINT [CK_Product_SellEndDate]
        GO
        
        ALTER TABLE [SalesLT].[Product]  WITH NOCHECK ADD  CONSTRAINT [CK_Product_StandardCost] CHECK  (([StandardCost]>=(0.00)))
        GO
        
        ALTER TABLE [SalesLT].[Product] CHECK CONSTRAINT [CK_Product_StandardCost]
        GO
        
        ALTER TABLE [SalesLT].[Product]  WITH NOCHECK ADD  CONSTRAINT [CK_Product_Weight] CHECK  (([Weight]>(0.00)))
        GO
        
        ALTER TABLE [SalesLT].[Product] CHECK CONSTRAINT [CK_Product_Weight]
        GO
        
    ```

1. Para ejecutar el script, presione **F5** o haga clic en **Ejecutar**.

1. Para comprobar que las tablas se han creado correctamente, expanda la carpeta **Tablas** en el explorador de bases de datos.

## Realizar una migración sin conexión de una base de datos de SQL Server a Azure SQL Database.

Ahora estamos listos para migrar los datos. Para realizar una migración sin conexión con Azure Data Studio, siga estos pasos:

1. Inicie el asistente para **migrar a Azure SQL** desde la extensión Azure Data Studio y, después, seleccione **Migrar a Azure SQL**.

1. En el **paso 1: Bases de datos para la evaluación**, seleccione **No** en *"¿Desea realizar un seguimiento del proceso de migración en Azure Portal?"* y, después, seleccione la base de datos *AdventureWorksLT*. Seleccione **Siguiente**.

1. En el **paso 2: Resumen de evaluación y recomendaciones de SKU**, espere a que se complete la evaluación y revise los resultados. Seleccione **Siguiente**.

1. En el **paso 3: Resultados de la evaluación y plataforma de destino**, seleccione **Azure SQL Database** como tipo de destino. Después de revisar los resultados de la evaluación, seleccione **Siguiente**.

1. En el **paso 4: Destino de Azure SQL**, si la cuenta aún no está vinculada, asegúrese de agregar una cuenta seleccionando el vínculo **Vincular cuenta**. A continuación, seleccione una cuenta de Azure, un inquilino de Microsoft Entra, una suscripción, una ubicación, un grupo de recursos, un servidor de Azure SQL Database y las credenciales de Azure SQL Database.

1. Seleccione **Conectar** y, a continuación, seleccione la base de datos *AdventureWorksLT* como **Base de datos de destino**. Seleccione **Siguiente**.

1. En el **paso 5: Azure Database Migration Service**, seleccione el vínculo **Crear** para crear una instancia de Azure Database Migration Service mediante el asistente. Siga los pasos **Configurar manualmente** proporcionados por el asistente para configurar el entorno de ejecución de integración autohospedado. Si ha creado anteriormente uno, puede reutilizarlo.

1. En el **paso 6: Configuración del origen de datos**, escriba las credenciales para conectarse a la instancia de SQL Server desde el entorno de ejecución de integración autohospedado.

1. Seleccione **Editar** en la columna *Seleccionar tablas* para la base de datos AdventureWorksLT.

1. Desactive la opción Migrar **esquema a destino** (ya que ya el esquema se ha creado manualmente).

1. En la pestaña **Disponible en el destino** se muestra que hay cuatro tablas listas para migrar:
     - **SalesLT.Customer**
     - **SalesLT.ProductCategory** 
     - **SalesLT.Product**
     - **SalesLT.Address**

1. Seleccione **Actualizar** para guardar la selección de tablas.

1. Seleccione **Ejecutar validación**.

    ![Captura de pantalla de la ejecución del paso de validación en la extensión de migración de Azure para Azure Data Studio.](../media/3-run-validation.png) 

1. Una vez que se complete la validación, seleccione **Listo** y luego **Siguiente**.

1. En el **paso 7: Resumen**, seleccione **Iniciar migración**.

1. Seleccione **Migraciones de base de datos en curso** en el panel de migración para ver el estado de la migración. 

    ![Captura de pantalla del panel de migración en la extensión de migración de Azure para Azure Data Studio.](../media/3-data-migration-dashboard.png)

1. Seleccione el nombre de la base de datos *AdventureWorks* para obtener más detalles.

    ![Captura de pantalla de los detalles de migración en la extensión de migración de Azure para Azure Data Studio.](../media/3-dashboard-sqldb.png)

1. Después de que el estado sea **Correcto**, vaya al servidor de destino y valide la base de datos de destino. 

1. Ejecute la consulta siguiente para comprobar que la migración de datos se ha realizado correctamente:

    ```sql
    -- Check row counts for migrated data
    SELECT 'Customer' AS TableName, COUNT(*) AS [RowCount] FROM [SalesLT].[Customer]
    UNION ALL
    SELECT 'ProductCategory' AS TableName, COUNT(*) AS [RowCount] FROM [SalesLT].[ProductCategory]
    UNION ALL
    SELECT 'Product' AS TableName, COUNT(*) AS [RowCount] FROM [SalesLT].[Product]
    UNION ALL
    SELECT 'Address' AS TableName, COUNT(*) AS [RowCount] FROM [SalesLT].[Address];
    ```

También ha aprendido a realizar una migración selectiva de tablas concretas desde una base de datos de SQL Server a Azure SQL Database mediante la extensión Migración de Azure para Azure Data Studio. También aprenderá a supervisar el proceso de migración.

## Limpieza

Cuando trabaje con su propia suscripción, es una buena idea al final de un proyecto identificar si todavía se necesitan los recursos que ha creado. 

Dejar que los recursos se ejecuten innecesariamente puede dar lugar a costes adicionales. Puede eliminar recursos de forma individual o bien eliminar el grupo de recursos completo desde [Azure Portal](https://portal.azure.com?azure-portal=true).

## Más información

Para obtener más información sobre Azure SQL Database, consulte [¿Qué es Azure SQL Database?](https://learn.microsoft.com/en-us/azure/azure-sql/database/sql-database-paas-overview).
