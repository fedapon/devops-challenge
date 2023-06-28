## Prueba 2 - Despliegue de una aplicación Django y React.js

Elaborar el deployment dockerizado de una aplicación en django (backend) con frontend en React.js contenida en el repositorio. Es necesario desplegar todos los servicios en un solo docker-compose.

Se deben entregar los Dockerfiles pertinentes para elaborar el despliegue y justificar la forma en la que elabora el deployment (supervisor, scripts, docker-compose, kubernetes, etc).

Subir todo lo elabora a un repositorio (github, gitlab, bitbucket, etc). En el repositorio se debe incluir el código de la aplicación y un README.md con instrucciones detalladas para compilar y desplegar la aplicación tanto en una PC local como en la nube (AWS o GCP).

### Solución propuesta:

Backend
Se detectó que la versión de Djando utilizada es la 2.1.4. Esta versión está deprecada y no debería utilzarse por motivos de seguridad. Sin embargo, para no realizar una migración del proyecto, se procedió verificar la documentación y se encontró que esta versión es compatible con las versiones de Python 3.5, 3.6 y 3.7, por lo que se seleccionó la versión 3.7 para construir el dockerfile.
Inicialmente la imágen se trató de crear desde node:18-alpine, pero por poblema de compatibilidad con las dependencias hubo que bajar a node:14-alpine.
