---
layout: post
title:  "Como desplegar vRealize automation 8.x paso a paso"
author: limberth
categories: [ Cloud, VMware ]
image: assets/images/18.png
---

# Como instalar vRealize Automation 8.x paso a paso.

El siguiente post describe la instalación paso a paso de VMware vRealize Automation 8.x (vRA) utilizando vRealize Suite Lifecycle Manager (vRLCM) Easy Installer, estoy utilizando específicamente la versión 8.4.2 pero el proceso es prácticamente igual para cualquier versión 8.x incluyendo la más reciente al momento de escribir este artículo que es la 8.6.2.

 

A continuación, se describe el único método soportado (vRLCM Easy Installer) ya que desplegar directamente desde el OVA no es admitido, durante este proceso se despliegan básicamente los siguientes 3 componentes:

- vRealize Automation (vRA)
- vRealize Suite Lifecycle Manager (vRLCM)
- VMware Identity Manager (vIDM).

 

## Tipos de despliegue:

 

Existen 2 tipos de despliegues de vRA, pero para ambos despliegues el proceso es el mismo, únicamente cambia la cantidad de recursos requeridos ya que se deben desplegar VMs adicionales

 

- ###   **Standard deployment:**

Despliega únicamente 1 nodo primario de vRA y un nodo primario de vIDM, no existe redundancia a nivel de sus componentes, por lo que es requerido contar con vSphere HA para proveer otro nivel disponibilidad, por lo que a nivel de ambientes productivos se recomienda utilizar el despliegue en clúster.

 

- ### **Cluster deployment:**

Se realiza el despliegue de un nodo de primario de vRA y 2 nodos secundarios, de igual manera para vIDM, está opción es la recomendada para ambientes de producción debido al nivel de disponibilidad/escalabilidad que ofrece. Se requiere de un balanceador de carga, incluso puede ser utilizado el balanceador de carga de NSX. La configuración del balanceador de carga en caso de utilizar configuración en cluster la pueden ver en  [vRealize Automation Load Balancing](https://docs.vmware.com/en/vRealize-Automation/8.0/vRA_load-balancing_80.pdf)

 



 

## Recursos de Sistema:

 

| **Requisitos**                    | **vRealize Suite   Lifecycle Manager** | **VMware Identity   Manager** | **vRealize   Automation**                                    |                                                              |
| --------------------------------- | -------------------------------------- | ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                   |                                        |                               | Perfil intermedio                                            | Perfil extragrande                                           |
| Disco                             | 78 GB                                  | 100 GB                        | 246 GB (solo para la instalación con un solo nodo)           | 246 GB (solo para la instalación con un solo nodo)           |
| vCPU                              | 2                                      | 8                             | 12                                                           | 24                                                           |
| RAM                               | 6 GB                                   | 16 GB                         | 42 GB                                                        | 96 GB                                                        |
| Max latencia red                  |                                        |                               | 5 ms entre cada nodo del clúster                             | 5 ms entre cada nodo del clúster                             |
| Máxima latencia de almacenamiento |                                        |                               | 20 ms para cada operación de E/S de disco desde cualquier  nodo de vRA | 20 ms para cada operación de E/S de disco desde cualquier  nodo de vRA |

 

 

## Requisitos previos:

 

-  Descargar la imagen ISO de vRealize Suite Lifecycle Manager Easy installer desde el sitio Customer Connect de VMware de la versión que deseamos instalar.
- Previamente a iniciar el despliegue contar con las IPs estáticas y entradas de DNS para cada uno de los componentes (tanto para los nodos/appliances como VIPs en caso de ambientes de cluster). 
- Todas las direcciones de red deben encontrarse en el mismo segmento de red de capa 2.

***Importante, luego de hacer el despliegue no se pueden cambiar los hostnames de los appliances.

Proceso de despliegue:

 

1. Montar la imagen ISO del easy installer y ejecutar la versión del installer de acuerdo a la versión del SO que estén utilizando, para  iniciar el asistente de instalación.

​      

​      <cd-rom>:\vrlcm-ui-installer\<os-version>

​      **Ejemplo en windows:**

​      E:\vrlcm-ui-installer\win32\installer.exe

 

2. Seleccionar la opción instalar (Install) en el asistente

 

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/1.png)



 

3. Next

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/2.png)



 

4. Aceptar las condiciones y dar next, CEIP es opcional.

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/3.png)

 

5. Indicar la información y las credenciales del vCenter donde se desean desplegar los appliances.

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/4.png)

  

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/5.png)

 

6. Seleccionar el folder donde se van a desplegar los virtual appliances en el vCenter

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/6.png)
 

7. Elegir el clúster donde se van a desplegar los virtual appliances.

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/7.png)

8. Seleccionar el datastore donde se van a almacenar los virtual appliance, se puede utilizar modo Think Disk.

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/8.png)

 

9. En este paso se deben colocar los datos generales en relación a la red y el dominio, se debe seleccionar la red e indicar los tipo de asignamiento de IPs (estático o DHCP) que por lo general es estático, la dirección IP del Gateway y de los servidores DNS separados por comas “,”, nombre de dominio y el tema del NTP es muy importante, incluso más aún en ambientes de clúster en donde los nodos deben mantenerse sincronizados. 

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/9.png)

 

10. El password que se debe configurar aquí corresponde al password que se le asignará al usuario root y al usuario admin del Lifecycle Manager.

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/10.png)
 

11. Ahora es necesario especificar el hostname (FQDN) y la IP que se asignara al Lifecycle Manager, virtual machine name es el nombre correspondiente a la VM a nivel de inventario de vCenter.

Data Center Name y vCenter Name es como se identificarán estos 2 componentes dentro de la consola del vRLCM, la opción de **Increase Disk** es opcional incrementar o no el espacio, en este caso estoy asignando 100 GB para poder subir binarios de futuras actualizaciones de cualquier producto administrado por vRLCM.

FIPS mode se deja off a menos que se tenga una razón específica de compliance para habilitarlo.

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/11.png)


12. Al igual que el paso anterior, se deben indicar los valores de IP, hostname (FQDN) y nombre de la VM pero esta vez para el vIDM.

El vIDM también puede ser configurado en clúster luego desde vRLCM.

También se debe indicar el “Default configuration admin” que es el nombre del usuario administrador local que se va a crear en el identity manager, este usuario funcionara para loguearse por primera vez dentro del vRA.

 

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/12.png)


![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/13.png)

 

13. En este paso se debe elegir si se va a desplegar vRA en modo Standard Deployment (1 master node) o Clustered Deployment (1 Master Node y 2 Secondary Nodes) y se debe asignar un nodo al environment, este nombre es únicamente para efectos de vRLCM.

Acá mismo se debe indicar la llave de la licencia obligatoriamente.

El tamaño del nodo va a depender del consumo estimado de recursos con base en los siguientes aspectos: [Valores máximos de escalabilidad y concurrencia.](https://docs.vmware.com/en/vRealize-Automation/8.6/reference-architecture/GUID-9DD443EA-0F7A-43B3-AD0A-8370B56109BE.html)

Al igual que el vRLCM y vIDM se debe indicar la dirección IP, hostname (FQDN) y el nombre de la VM.

Y por último existe la posibilidad de modificar el rango de IPs por default que utilizará vRA para su cluster de Kubernetes internos, este rango no es necesario sino hace overlap con ningun otra subred en el ambiente.

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/14.png)

 

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/15.png)

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/16.png)
 

14. Por último simplemente se deben validar los datos ingresados en los pasos anteriores y si todo está correcto iniciar el proceso de instalación dando click en **Submit**.

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/17.png)

15. Una vez finalizado el proceso de despliegue, podemos ingresar tanto al vRA como al vIDM o vRLCM para validar que todo este funcionando de manera correcta.

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/18.png)


![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/19.png)

![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/20.png)

 
![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/211.png)


![Easy Installer]({{ site.baseurl }}/assets/images/2022-07-01-deploy-vra-step-by-step/22.png)

 

## Conclusiones

Una vez validado que todo esta funcionando correctamente, este fue solo el primer paso ya que ahora se debe empezar con la configuración de vRA, Cloud Accounts, Projects, Profiles, RBAC, Integraciones, etc. Todo esto para poder luego iniciar con lo que realmente agrega valor que es la automatización de procesos de entrega de recursos atravez del portal.
