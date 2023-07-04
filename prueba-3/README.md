## Prueba 3 - CI/CD

Dockerizar un nginx con el index.html default. Elaborar un pipeline que ante cada cambio realizado sobre el index.html buildee la nueva imagen y la actualice en la plataforma elegida (docker-compose, swarm, kubernetes, etc). Para la creación del CI/CD se puede utilizar cualquier plataforma (CircleCI, Gitlab, Github, Bitbucket).

##### Requisitos y desables:

La solución al ejercicio debe mostrarnos que usted puede:

Automatizar la parte del proceso de despliegue. Usar conceptos de CI para aprovisionar el software necesario para que los entregables que se ejecuten usen cualquier herramienta de CI de su elección para implementar el entregable.

### Solución propuesta:

Dado que el código de challenge se encontraba en Github y luego hice un fork del mismo a mi usuario, para la implementación de esta prueba utilicé Github Actions.

El despliegue lo realicé en una instancia EC2 de AWS al que le instalé Docker trabajando en modo Swarm. Docker Swarm permite la actualización de la imagen sin tener pérdida de servicio, ya que primero inicializa el nuevo contenedor y una vez que está operativo se elimina el anterior.

El proyecto se encontrará disponible solo por un tiempo en http://craftech.blackbird.ar

#### Descripción del workflow

El mismo está configurado para ejecutarse únicamente cuando se realiza un push dentro de la carpeta "prueba-3", es decir un cambio realizado tanto en el index.html, el Dockerfile o si se agregaran nuevos archivos disparará la tarea.

Este flujo de trabajo cuenta con 2 jobs:

**build:** este job se encarga de crear la nueva imagen del proyecto a partir del Dockerfile. Una vez creado se loguea con las credenciales del usuario a docker hub y sube la imagen creada al mismo. De esta manera la imagen ya está disponible para ser consumida por quien lo requiera.

**deploy:** una vez que el "build" se realizó correctamente, se procede a conectarse por ssh a la instancia EC2 que está corriendo Docker Swarm. Este orquestador de contenedores tiene la posibilidad de una vez creado el servicio, actualizar la imagen utilizada sin que exista un downtime de servicio. Esto lo realiza inicializando primero el nuevo contenedor y a continuación finalizando el anterior con la versión más antigua.

En este ejemplo el comando para la creación del servicio (realizado conectándome directamente a la consola de la instancia EC2) es:

`docker service create -d --name craftech-nginx -p 80:80 fedapon/craftech-nginx`

El comando ejecutado por el job deploy que actualiza la imagen del servicio es:

`docker service update -d --update-order start-first --image fedapon/craftech-nginx craftech-nginx`
