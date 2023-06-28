## DevOps Challenge

#### Prueba 1 - Diagrama de Red

Produzca un diagrama de red (puede utilizar lucidchart) de un aplicación web GCP o AWS y escriba una descripción de texto de 1/2 a 1 página de sus elecciones y arquitectura.

El diseño debe soportar:

- Cargas variables
- Contar con HA (alta disponibilidad)
- Fontend en Js.
- Backend con una base de datos relacional y una no relacional.
- La aplicación backend consume 2 microservicios externos.

El diagrama debe hacer un mejor uso de las soluciones distribuidas.



##### Solución propuesta:

![Diagrama de Red en AWS](https://github.com/fedapon/devops-challenge/blob/main/prueba-1/Craftech%20Challenge%20-%20Diagrama%20de%20Red.png)

Los servicios utilizados son:

###### AWS Route 53

Este servicio nos permite administrar los dominios y subdominios del proyecto (DNS).

###### AWS Simple Storage Service

El frontend del proyecto son archivos estáticos que contienen javascript que se ejecuta del lado del cliente, por lo que no es necesario poder de computo en la nube para su funcionamiento. Por lo tanto los archivos se suben a un bucket S3 que será accedido únicamente por el servicio de CloudFront.

###### AWS CloudFront

En conjunto con AWS S3, se configura este servicio para que replique los datos del bucket S3 en todas las zonas de cercanía a los usuarios que utilizarán el servicio. Esto permite reducir la latencia, el tráfico de datos (por compresión) y aumentar la velocidad de entrega de los mismos.

###### AWS Aplication Load Balancer

Nos permite tener un único punto de entrada a nuestra aplicación y este servicio se encargará de balancear la carga de trabajo entre las distintas instancias EC2 que despliegue el Auto Scaling. Se encarga también de monitorear el estado de las instancias en las zonas de disponibilidad, y en caso de una falla en una AZ, derivar todo el tráfico a la zona que se encuentre operativa.

###### AWS Auto Scaling | AWS Elastic Compute Cloud

Es un servicio que permite crear un grupo de auto escalado (a partir de una plantilla de una instancia EC2) que se encarga de escalar (lanzar o terminar instancias) en las distintas AZ de acuerdo a criterios/reglas como uso del CPU, por días/horarios, etc. Esto permitirá que el backend crezca o se reduzca para adaptarse a la carga de trabajo y hacer un uso eficiente de los recurso, es decir en costos.

Las instancias EC2 se encontraran en la red pública protegidas por el "grupo de seguridad" del auto scaling, exponiendo únicamente los puertos requeridos y bloqueando el resto. La decisión de mantener el backend en una red pública es para que puedan hacer uso de los microservicios externos que consume.

###### AWS Relational Database Service | AWS DynamoDB

Estos dos servicios de base de datos se encontrarán dentro de la subred privada, por lo que solamente las instancias de la EC2 van a poder acceder a estos servicios.

La RDS se configura como multi-AZ, es decir que se mantiene una base de datos maestra y se replican los datos a las esclavas. En caso de falla de la AZ donde se encuentra la base de datos maestra, automáticamente la esclava pasará a ser la maestra, por lo que el servicio no es interrumpido.

Con respecto a DynamoDB, dado que es una base de datos no relacional serverless, AWS se encarga de mantiene sincronizada entre las distintas AZ dentro de la región definida.

###### 



