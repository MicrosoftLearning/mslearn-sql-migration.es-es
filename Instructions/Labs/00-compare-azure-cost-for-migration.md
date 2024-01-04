---
lab:
  title: Comparación de los costes locales de Azure para la migración
---

# Comparación de los costes locales de Azure para la migración

El costo total de propiedad (TCO) es una herramienta que puede usar durante un proyecto de modernización de la plataforma de datos para evaluar la diferencia de costos que puede implicar la migración.

En su distribuidor global, se espera que la modernización de la plataforma de datos genere ahorros importantes, pero la junta directiva le pidió calcular estos ahorros con la mayor precisión posible.

Aquí se calculará el costo total de propiedad (TCO) de la migración a Azure mediante la calculadora de TCO.

Este ejercicio dura aproximadamente **30** minutos.

## Calcular el coste total de propiedad

1. Abra una nueva pestaña del explorador y vaya a [Calculadora de TCO de Azure](https://azure.microsoft.com/pricing/tco/calculator/).
1. En **Definir las cargas de trabajo**, elimine las cargas de trabajo existentes en la sección **Servidores**.

## Agregue la carga de trabajo de la base de datos

1. En **Bases de datos**, seleccione **+ Agregar base de datos**.
1. En el cuadro de texto **Nombre**, escriba **Contabilidad**.
1. En la sección **Origen**, elija estos valores:

    | Propiedad | Valor |
    | --- | --- |
    | Base de datos | **Microsoft SQL Server** |
    | Licencia | **Empresa** |
    | Entorno | **Servidores físicos** |
    | Sistema operativo | **Windows** |
    | Licencia del sistema operativo | **Centro de datos** |
    | Servidores | **1** |
    | Procesadores por servidor | **1** |
    | Núcleos por procesador | **4** |
    | RAM (GB) | **64** |
    | Optimizar por | **CPU** |
    | SQL Server 2008/2008R2 | **Alternar para seleccionar este valor** |

1. En la sección **Destino**, elija estos valores:

    | Propiedad | Valor |
    | --- | --- |
    | Servicio | **VM con SQL Server** |
    | Tipo de disco | **SSD** |
    | E/S | **5000** |
    | Almacenamiento de SQL Server | **32 GB** |
    | Copia de seguridad de SQL Server | **32 GB** |

    > [!NOTE]
    > Se recomienda usar discos SSD para las cargas de trabajo de producción en Azure.

## Agregue las cargas de trabajo de red y almacenamiento

1. En **Almacenamiento**, seleccione **+ Agregar almacenamiento**.
1. En el cuadro de texto **Nombre**, escriba **Discos locales de contabilidad** y, luego, escriba estos valores:

    | Propiedad | Valor |
    | --- | --- |
    | Tipo de almacenamiento | **Disco local/SAN** |
    | Tipo de disco | **HDD** |
    | Capacity | **3 TB** |
    | Copia de seguridad | **1 TB** |
    | Archivar | **0 TB** |

1. En **Redes**, en los controles **Ancho de banda de salida**, seleccione **1 GB**.
1. En la parte inferior de la página, seleccione **Siguiente**.

## Ajustar supuestos

1. En la sección **Ajustar supuestos**, en la lista **Moneda**, seleccione la moneda que prefiera.
1. En **Cobertura de Software Assurance (proporciona Ventaja híbrida de Azure)**, habilite la alternancia para seleccionar **Cobertura de Software Assurance de Windows Server**.
1. Habilite la alternancia para seleccionar **Cobertura de Software Assurance de SQL Server**.

    > [!NOTE]
    > Puede usar los vínculos que se proporcionan en la sección **Software Assurance** si quiere obtener más información sobre el aseguramiento que está disponible. 

1. En **Almacenamiento con redundancia geográfica (GRS)**, asegúrese de que la opción **GRS replica sus datos a una región secundaria que se encuentra a cientos de kilómetros de distancia de la región primaria** no esté habilitada.
1. En **Costos de las máquinas virtuales**, asegúrese de que la opción **Habilitar esta opción para que la calculadora no recomiende máquinas virtuales de la serie Bs** esté habilitada.

    > [!NOTE]
    > Las máquinas virtuales de la serie B no tienen la proporción de memoria a núcleo virtual de 8 que se recomienda para las cargas de trabajo de SQL Server.

1. En **Costos de electricidad**, en el cuadro de texto **Price per KW hour** (Precio por hora de kW), escriba un valor realista para su ubicación.

    > [!NOTE]
    > Puede encontrar precios de electricidad aproximados en [Global electricity prices](https://www.statista.com/statistics/263492/electricity-prices-in-selected-countries/) (Precios de electricidad global). Estos precios están en USD ($). Puede convertirlos a un valor aproximado en la moneda que prefiera.

1. En **Costos de almacenamiento**, deje todos los valores predeterminados o ajústelos si no parecen razonables.
1. En **Costos de personal de TI**, deje todos los valores predeterminados o ajústelos si no parecen razonables.
1. En **Other assumption** (Otro supuesto), expanda cada sección y examine los costos asociados.
1. En la parte inferior de la página, seleccione **Siguiente**.

## Investigación del informe de 5 años

1. En la página **Ver el informe**, tenga en cuenta que el **Período de tiempo** predeterminado es de **5 años**.
1. Desplácese hacia abajo en el informe e investigue el desglose estimado de los costos de los sistemas locales y Azure. Anote esta información:

    - ¿Cuál es el componente más significativo de los costos del entorno local?
    - Si decide migrar a Azure, ¿cuál será el mayor ahorro de costos?

1. Expanda cada sección por turnos e investigue el desglose de los costos.

## Investigación del informe de 3 años

1. Desplácese hasta la parte superior de la página y en el cuadro de texto **Período de tiempo**, seleccione **3 años**.
1. Desplácese hacia abajo en el informe e investigue el desglose estimado de los costos de los sistemas locales y Azure. Anote esta información:

    - ¿Cuál es el componente más significativo de los costos del entorno local?
    - Si decide migrar a Azure, ¿cuál será el mayor ahorro de costos?

1. Expanda cada sección por turnos e investigue el desglose de los costos.

Ha usado la calculadora de TCO de Azure para identificar las diferencias de costos entre las implementaciones locales y de Azure para el servidor de Contabilidad de Adatum Corporation y las bases de datos asociadas.
