---
lab:
  title: Comparación de los costos de migración de Azure
---

# Comparación de los costos de migración de Azure

La Calculadora de precios de Azure es una herramienta útil que puede usar durante un proyecto de modernización de la plataforma de datos para calcular los costes de diferentes servicios de Azure y enfoques de migración.

En su minorista global, se espera que el proyecto de modernización de la plataforma de datos obtenga ahorros significativos, pero la junta de administración le ha pedido que calcule los costes de diferentes opciones de migración de Azure de la forma más precisa posible.

Aquí, calculará los costes estimados de migración a Azure mediante la [Calculadora de precios de Azure](https://azure.microsoft.com/en-us/pricing/calculator/).

Este ejercicio dura aproximadamente **30** minutos.

## Cálculo de los costos estimados de Azure

1. Abra una nueva pestaña del explorador y vaya a [Calculadora de precios de Azure](https://azure.microsoft.com/en-us/pricing/calculator/).
1. La Calculadora de precios de Azure le ayuda a calcular los costes de los servicios de Azure. Calcularemos el coste de migrar la carga de trabajo de la base de datos a Azure.

## Agregue la carga de trabajo de la base de datos

1. En la sección **Productos**, busque y seleccione **Azure SQL Database** (puede usar la búsqueda o examinar la categoría **Bases de datos**).
1. En el panel de configuración de **Azure SQL Database** que aparece más abajo de la página, escriba estos valores:

    | Propiedad | Valor |
    | --- | --- |
    | Region | **Este de EE. UU.** (o su región preferida) |
    | Tipo | **Base de datos única** |
    | Modelo de compra | **vCore** |
    | Nivel de servicio | **Uso general** |
    | Nivel de proceso | **aprovisionado** |
    | Tipo de hardware | **Serie estándar (Gen5)** |
    | Instancia | **4 núcleos virtuales** |
    | Recuperación ante desastres | **Réplica principal o geográfica** |
    | Proceso | **Con redundancia local** |
    | Storage | **32 GB** |
    | Almacenamiento de copia de seguridad | **RA-GRS** |

1. Revise la estimación del coste mensual que se muestra para SQL Database.

## Adición de una máquina virtual para comparar

1. De nuevo en la sección **Productos**, busque y seleccione **Virtual Machines**.
1. En el panel configuración de **Virtual Machines**, escriba estos valores:

    | Propiedad | Valor |
    | --- | --- |
    | Region | **Este de EE. UU.** (igual que la base de datos) |
    | Sistema operativo | **Windows** |
    | Tipo | **SQL Server** |
    | Nivel | **Estándar** |
    | Instancia | **D4s v3** (4 vCPU, 16 GB RAM) |
    | Máquinas virtuales | **1** |
    | Licencia | **SQL Standard** |

1. Expanda **Managed Disks** y agregue:
   - Nivel: **SSD Premium**, **128 GB**

## Adición de almacenamiento para copias de seguridad

1. En la sección **Productos**, busque y seleccione **Cuentas de almacenamiento**.
1. En el panel de configuración **Cuentas de almacenamiento**, escriba estos valores:

    | Propiedad | Valor |
    | --- | --- |
    | Region | **Este de EE. UU.** |
    | Tipo | **Almacenamiento de blobs en bloques** |
    | Rendimiento | **Estándar** |
    | Tipo de cuenta de almacenamiento | **Uso general V2**. |
    | Estructura de archivos | **Espacio de nombres plano** |
    | Nivel de acceso | **Acceso frecuente** |
    | Redundancia | **LRS** |
    | Capacity | **1 TB** |

## Revisión y comparación de los costes

1. Revise los costes mensuales estimados totales de todos los servicios que ha agregado:
   - **SQL Database**: Costes del servicio de base de datos administrado
   - **Máquinas virtuales**: Costes de infraestructura de SQL Server en VM
   - **Cuentas de almacenamiento**: Costes de copia de seguridad y almacenamiento adicional

1. Considere las siguientes preguntas a medida que revise las estimaciones:
   - ¿Cómo se comparan los costes entre SQL Database (PaaS) y SQL Server en VM (IaaS)?
   - ¿Cuáles son las desventajas entre los servicios administrados y los servicios de infraestructura?
   - ¿Cómo pueden aumentar estos costes con sus requisitos de carga de trabajo?

1. Para guardar esta estimación, seleccione **Guardar** o **Exportar** para compartirla con las partes interesadas.

## Exploración de las opciones de optimización de costes

1. En cada configuración del servicio, explore diferentes opciones para ver cómo afectan a los costes:
   - **SQL Database**: pruebe diferentes niveles de servicio (Básico, Estándar, Premium) y tamaños de proceso.
   - **Máquinas virtuales**: compare diferentes tamaños de máquina virtual y opciones de almacenamiento.
   - **Almacenamiento**: compare diferentes opciones de redundancia y niveles de acceso.

1. Observe cómo el cambio de estas opciones afecta a la estimación mensual general.

Ha usado la Calculadora de precios de Azure para calcular los costes para migrar el servidor de contabilidad de Adatum Corporation y sus bases de datos asociadas a diferentes servicios de Azure. Esto le proporciona una base para comparar las opciones de infraestructura como servicio (IaaS) frente a las opciones de plataforma como servicio (PaaS) y planear el presupuesto de migración.

