# Chat-app
Bienvenidos a la aplicación chat de ejemplo 
Esta aplicación tiene una arquitectura sencilla, orientada a que entiendas los conceptos básicos de conteinerización usando Docker y despliegue en Openshift. 
Sigue los pasos a continuación para construir y desplegar la aplicación 
Este workshop te ayudará a entender las interioridades de la tecnología de contenedores, y como ayuda al desarrollo y despliegue de una aplicación.
Utilizaremos Red Hat Openshift como nuestra plataforma de referencia para desplegar la aplicación conteinerizada y así entender también parte de sus capacidades.

## Arquitectura y flujo de acciones

<p align="center">
  <img src="images/flow.png" width="70%"></img>
</p>

-	El usuario interactúa con el bot en la plataforma Discord.
-	El bot utiliza la aplicación Node.js para recibir los comandos.
-	Dentro de la app Node.js se realizan las llamadas a la APIs de los servicios.
-	Las respuestas de las APIs son recibidas y acomodadas por la app.
-	El bot entrega las respuestas al usuario en Discord.


## Lab 1. Conteninerizando nuestra aplicación.

### Paso 1. Generando el Dockerfile

Lo primero que tenemos que hacer es clonar este repositorio en tu máquina:

```
git clone https://github.com/luisreyesoliva/simple-chat-app.git
```

Ahora abriremos una sesión de terminal en el directorio donde hemos clonado la aplicación cliente (React Chat App):

```bash
cd <location-path>/react-chat-client
```

Vamos a crear un fichero llamado Dockerfile (sin extensión) donde indicaremos las instrucciones para generar la imagen del contenedor de nuestra aplicación de chat. 

Cada linea que se ejecute en este Dockerfile generará nuevas capas sobre la imagen base de node de la cual partimos para conteninerizar nuestra aplicación y generar todo lo necesario para ejecutar la aplicación React en nuestro entorno.
El Docker file está configurado en dos stages (Build y Run) para generar la aplicación y ejecutarla sobre un servidor de Nginx (que también es un contenedor). Copia las siguientes instrucciones en el fichero que has creado: 

```bash
#Build Steps
FROM node:alpine3.10 as build-step

RUN mkdir /app
WORKDIR /app

COPY package.json /app
RUN npm install
COPY . /app

RUN npm run build

#Run Steps
FROM nginx:1.19.8-alpine  
COPY --from=build-step /app/build /usr/share/nginx/html
```

Para aprender más sobre el proceso de conteinerización de una aplicación React, te recomiendo que visites esta página: [Dockers and Dad Jokes: How to Containerize a ReactJS Application](https://ibm.biz/how-to-containerize-react-app-031821-bradstondev)

## Paso 2: Creando el .dockerignore

Lo siguiente será crear un fichero _.dockerignore_. Este fichero nos permitirá "ignorar" determinados ficheros a la creación de nuestra imagen y así ahorrar algo de tiempo al construirla y evitar sobreescrituras accidentales. Es habitual ver este fichero en un entorno Docker.

```bash
node_modules
build
.dockerignore
Dockerfile
Dockerfile.prod
```

## Paso 3: Construir la imagen Docker

Lo siguiente va a ser construir la imagen Docker que definirá lo que queremos ejecutar en nuestro contenedor. Este es el formato del comando Docker que debemos usar en nuestro terminal.

```bash
docker build -t <image-name>:<tag> .
```
Esto es lo que va a suceder:
* _docker build_ inicia el proces de construcción de la imagen
* _-t_ permite tagear la imagen en formato 'nombre:tag'
* _image-name_ es el nombre que queremos dar a la imagen
* _tag_ is es el tag para la versión de la imagen que vamos a construir. Se suele usar para identificar diferentes versiones de la imagen a desplegar.
* _._ indica el path desde el que todo será construido (y que contiene un Dockerfile). NOTA: Es imprescindible para que el comando funcione, no lo olvides!

En nuestro caso, este es el comando que usaremos (pon otro nombre si lo prefieres).

```bash
docker build -t chat-app-client:v1 .
```

## Paso 4: Arrancando la aplicación

Ahora pongamos nuestro contenedor en acción. 

Para ello ejecuta el siguiente comando:

```bash
docker run -p 8080:80/tcp -d <image-name>:<tag>
```

Esto es lo que va a pasar:
* _docker run_ ejecuta nuestra imagen Docker en forma de contenedor
* _-p_ se usa para indicar el puerto de nuestro host en que queremos exponer nuestro contenedor
* _8000:80/tcp expone la aplicación, que está hosteada en un servidor de Nginx en el puerto 80 de nuestro contenedor, sobre el puerto 8000 de nuestra máquina local.
* _-d_ hace que el contenedor se ejecute en background, permitiendo que podamos seguir usando nuestra sesión de terminal.

En nuestro caso, el comando será:

```bash
docker run -p 8080:80/tcp -d chat-app-client:v1
```

## Paso 5: Verificar que el contenedor está corriendo en nuestro máquina

 Ejecutar el siguiente comando en la terminal:

```bash
docker ps
```

Esencialmente, el comando _docker ps_ lista todos los contenedores en ejecución dentro de nuesto Docker host (el que está instalado en nuestra máquina).

## Paso 6: Acceder a nuestra aplicación de chat

<p align="center">
  <img src="images/chat-client.png" width="70%"></img>
</p>

 Simplemente acceder a través de un navegador a localhost:8000 y verás la aplicación en ejecución. Ojo, todavía no funciona porque hay que conectarla al servidor, esto lo veremos en el siguiente Lab. 

