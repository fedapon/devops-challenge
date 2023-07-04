## Prueba 2 - Despliegue de una aplicación Django y React.js

Elaborar el deployment dockerizado de una aplicación en django (backend) con frontend en React.js contenida en el repositorio. Es necesario desplegar todos los servicios en un solo docker-compose.

Se deben entregar los Dockerfiles pertinentes para elaborar el despliegue y justificar la forma en la que elabora el deployment (supervisor, scripts, docker-compose, kubernetes, etc).

Subir todo lo elabora a un repositorio (github, gitlab, bitbucket, etc). En el repositorio se debe incluir el código de la aplicación y un README.md con instrucciones detalladas para compilar y desplegar la aplicación tanto en una PC local como en la nube (AWS o GCP).

### Solución propuesta:

##### Frontend

Se observó que el proyecto está realizado en javascript con tecnologías como React y Webpack. Con este input se intentó crear el dockerfile con la imagen **node:18-alpine**. Dado que el proyecto está realizado con versiones antiguas de react, se presentaron problemas de compatibilidad entre los paquetes al momento de instalarlos. Luego de probar distintas versiones se encontró que la versión **node:14-alpine** no generaba problemas.

Dentro de la carpeta /config hubo que corregir los archivos de webpack en varios lugares donde la url apuntaba a un IP "0.0.0.0" por el de "localhost".

Inicialmente la imagen creada se servía mediante el servidor de desarrollo ("yarn run start") lo que no era eficiente en cuanto a tamaño y velocidad de arranque de un contenedor. Por lo tanto, dado que una vez hecho el build con webpack los archivos a servir eran estáticos, se procedió a modificar el Dockerfile para generar la imagen en 2 etapas: una que haga el build con node:14-alpine, y una segunda a partir de una imagen de **nginx** donde se copiaban los archivos generados en la etapa de build. Para que esto funcione hubo que modificar en el archivo webpack.config.prod.js cambiando el publicPath (línea 20) a "/" para que los links en los archivos generados no presenten ningún error.

##### Backend

Se encontró que la versión de Djando utilizada es la 2.1.4. Esta versión está deprecada y no debería utilizarse por motivos de seguridad. Sin embargo, para no realizar una migración del proyecto, se procedió verificar la documentación y se encontró que esta versión es compatible con las versiones de Python 3.5, 3.6 y 3.7, por lo que se seleccionó la versión 3.7 para construir el Dockerfile. Además, según las dependencias que tenía el proyecto se dedujo que el motor de la base de datos utilizado es posgres.

Por lo tanto para la construcción de la imagen se partió de **python:3.7-alpine** y hubo que instalar librerías adicionales de postgres antes de instalar las dependencias del proyecto (django).

Hubo que agregar dentro del archivo settings.py de Django la configuración "CORS_ALLOW_CREDENTIALS" para que el frontend solicite las credenciales al usuario para que se pueda autenticar con el backend. De este modo, cuando el frontend intenta acceder al backend, aparece en la pantalla del usuario la posibilidad de ingresar el nombre de usuario y contraseña.

##### Docker compose

Para el armado del docker-compose que configura el despliegue del proyecto se crearon 3 servicios: "frontend", "backend" y "database". Tanto frontend como backend parten de la imagen generada por los dockerfile creado para cada uno. Además se importa un archivo con las variables de entorno de cada uno de los proyectos. Finalmente el servicio "database" se construye a partir de la imagen pública de postgres que se encuentra disponible en docker hub. Este último tiene configurado un volumen para que los datos en la base de datos persistan por más que el contenedor se detenga o se destruya.

#### Despliegue de la aplicación en Local

##### Requisitos

Se debe tener instalado docker y docker-compose.

##### Pasos a seguir

1. Clonamos el repositorio y accedemos a la carpeta **prueba-2**:

​ `git clone https://github.com/fedapon/devops-challenge.git`

​ `cd devops-challenge/prueba-2`

2. Ejecutamos docker-compose para iniciar el proceso de despliegue:

​ `docker-compose -d`

3. Ejecutamos las migraciones en la base de datos y creamos el usuario administrador. Para ello nos conectaremos primero a la consola del contenedor que está corriendo el backend.

​ `docker exec -it craftech-backend /bin/sh`

Una vez conectados, ejecutaremos los siguientes comandos para crear la estructura del proyecto en la base de datos :

​ `python manage.py makemigrations`

`	python manage.py migrate`

Y finalmente crearemos el usuario administrador:

​ `	python manage.py createsuperuser`

Se nos solicitará un nombre de usuario, email y contraseña. Anotarlas ya que se solicitaran en el momento de que el frontend se comunique con el backend.

4. Una vez finalizados los pasos anteriores, se puede proceder a ingresar a:

http://localhost:3000

Si todos los pasos anteriores se realizaron correctamente al ingresar a "Solicitudes" el sistema deberá solicitar el nombre de usuario y contraseña generado previamente. Una vez realizado, el sistema traerá desde el back todas las solicitudes encontradas (recordar que inicialmente no hay ninguna).

#### Despliegue de la aplicación en AWS ECS

Para este despliegue utilizaremos un cluster administrado por el servicio de ECS.

##### Pasos a seguir

1. A través de la consola de administración buscamos el servicio Elastic Container Service y procedemos a crear un cluster. Para utilizar el tier gratuito en la selección de infraestructura seleccionar "Amazon EC2 instances" y el tipo es muy importante que sea **t2.micro**. En capacidad deseada seleccionar un mínimo de 1 y un máximo de 3.

2. Antes de continuar publicaremos en docker hub las imágenes del proyecto para el frontend y el backend. Para ello haremos

​ `docker build --tag fedapon/craftech-frontend /frontend`

​ `docker build --tag fedapon/craftech-backend /backend`

Es muy importante reemplazar fedapon por su nombre de usuario para poder subir las imágines a su repositorio.

3. A continuación nos loguearemos en docker hub y procederemos a subir las imágene.

​ `docker login`

​ `docker push fedapon/craftech-frontend `

​ `docker push fedapon/craftech-backend `

4. Una vez que las imágenes ya están disponibles en docker hub, volvemos a la consola de AWS para definir las tareas. En este caso definiremos las imágenes que acabamos de crear para el frontend y el backend, y para la base de datos, la imagen será simplemente posgres. A las variables de entorno los cargaremos ahí (cada tarea con sus respectivas variables de entorno). Y en red seleccionaremos el modo "bridge" que nos permitirá exponer directamente los puertos utilizados en la instancia que esté corriendo.

5. Una vez creadas las tareas (definiciones), debe ingresar al cluster creado en el paso uno y crear un servicio. Se debe crear un servicio seleccionando que vamos a desplegarlo en instancias EC2 y para ello seleccionaremos la definición de las tareas creadas en el paso 4. Se debe crear un servicio por cada tarea, es decir una para el frontend, otra para el backend y finalmente una para la base de datos.

6. Finalmente si los 3 servicios se encuentran operativos, podremos acceder al ip público de la instancia EC2 que creó el cluster y conectarnos por consola para realizar las migraciones de la base de datos (ver paso 3 de instalación local).

7. Una vez corrida las migraciones, podremos acceder al servicio a través de:

   http://ip-publico-instancia-ec2:3000
