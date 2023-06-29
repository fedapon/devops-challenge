## Prueba 2 - Despliegue de una aplicación Django y React.js

Elaborar el deployment dockerizado de una aplicación en django (backend) con frontend en React.js contenida en el repositorio. Es necesario desplegar todos los servicios en un solo docker-compose.

Se deben entregar los Dockerfiles pertinentes para elaborar el despliegue y justificar la forma en la que elabora el deployment (supervisor, scripts, docker-compose, kubernetes, etc).

Subir todo lo elabora a un repositorio (github, gitlab, bitbucket, etc). En el repositorio se debe incluir el código de la aplicación y un README.md con instrucciones detalladas para compilar y desplegar la aplicación tanto en una PC local como en la nube (AWS o GCP).

### Solución propuesta:

##### Frontend

Se observó que el proyecto está realizado en javascript con tecnologías como React y Webpack. Con este input se intentó crear el dockerfile con la imagen **node:18-alpine**. Dado que el proyecto está realizado con versiones antiguas, se presentaron problemas de incompatibilidad entre los paquetes al momento de instalarlos. Luego de probar distintas versiones se encontró que la versión **node:14-alpine** no generaba problemas.

Dentro de la carpeta /config hubo que corregir dentro de los archivos de webpack varios lugares donde la url apuntaba a un IP 0.0.0.0 por el de localhost.

##### Backend

Se encontró que la versión de Djando utilizada es la 2.1.4. Esta versión está deprecada y no debería utilizarse por motivos de seguridad. Sin embargo, para no realizar una migración del proyecto, se procedió verificar la documentación y se encontró que esta versión es compatible con las versiones de Python 3.5, 3.6 y 3.7, por lo que se seleccionó la versión 3.7 para construir el dockerfile.

Hubo que agregar dentro del archivo settings.py de Django la configuración CORS_ALLOW_CREDENTIALS, dado que si no el frontend no se podía autenticar con el backend. De este modo, cuando el frontend intenta acceder al backend, aparece en la pantalla del usuario la posibilidad de ingresar el nombre de usuario y contraseña.

