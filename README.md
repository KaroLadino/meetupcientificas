# ¡Hola Científicas de Datos!

Meetup: 2 de septiembre 2019

Gracias por venir al meetup, vamos a aprender cómo configurar ShinyProxy para el despliegue de una app Shiny.

---

## ¿Por qué lo necesitamos?
Al desarrollar herramientas en R, el reto final es la entrega al cliente. ¿Cómo recibe el cliente la herramienta? ¿Cómo la instala? Esto puede ser en la infraestructura de ellos o la propia de la empresa que desarrolla. Pero cuando es en la infraestructura de ellos aparecen retos nuevos, como la autenticación previa al uso de la app. Para esto nos funciona ShinyProxy.

### Casos de uso
Dependiendo lo que el cliente solicite, se verifica con ShinyProxy si permite la autenticación, contamos con LDAP, Kerberos, SPNEGO, Single-Sign On / Keycloak, OpenID Connect (OIDC), incluso autenticación con redes sociales. La idea es escoger la más óptima para ambas partes.

## ¿Qué necesitamos?
### Un servidor
Necesitamos un servidor con la distribución con la que nos encontraremos en la infraestructura del cliente o la que a nuestro juicio nos parezca más estable.

Estando dentro del servidor debemos revisar lo siguiente: 

### Docker
Actualizar la lista de paquetes: 

``` sudo apt update ```

Instalar más paquetes de prerrequisito: 

``` sudo apt install apt-transport-https ca-certificates curl software-properties-common ```

Agregar la llave GPC para el repositorio oficial de Docker:

``` curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - ```

Agregar el repositorio de Docker a las fuentes APT:

``` sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable" ```

Volvemos a actualizar: 

``` sudo apt update ```

Revisamos que instalaremos desde el repo oficial de Docker y no de Ubuntu: 

``` apt-cache policy docker-ce ```

Debería salir algo así: 

``` docker-ce:
  Installed: (none)
  Candidate: 18.03.1~ce~3-0~ubuntu
  Version table:
     18.03.1~ce~3-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages 
        
```

Podemos ver que Docker-ce no está instalado pero tiene un candidato de instalación desde la web de Docker

Ahora sí podemos instalar Docker con el siguiente comando: 

``` sudo apt install docker-ce ```

Al finalizar la instalación, ya contamos con Docker, el daemon se habrá iniciado y estará listo para lo que haremos. Podemos checkear si el servicio está corriendo con el siguiente comando: 

``` sudo systemctl status docker ```

Deberíamos obtener algo así:

```● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2018-07-05 15:08:39 UTC; 2min 55s ago
     Docs: https://docs.docker.com
 Main PID: 10096 (dockerd)
    Tasks: 16
   CGroup: /system.slice/docker.service
           ├─10096 /usr/bin/dockerd -H fd://
           └─10113 docker-containerd --config /var/run/docker/containerd/containerd.toml 
```

Si algo se complica podemos ir a este link de documentación oficial: [Instalando Docker en Ubuntu 18.04LTS](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04)

Por último revisamos la versión de Docker que tenemos, con el siguiente comando:

``` docker -v ```

Debemos obtener algo así:

```
Docker version 17.06.2-ce, 
build cec0b72 
```

Y corremos el hola mundo para saber si todo está en orden:

``` docker run hello-world ```

```
Hello from Docker!  
This message shows that your installation appears to be working correctly.   
```

Para asegurar que funcione debemos agregar una configuración extra en un archivo de Docker

Nos debemos ubicar en el home de nuestro servidor y crear el siguiente directorio:

``` sudo mkdir -p /etc/systemd/system/docker.service.d ```

Luego movernos al mismo:

``` cd /etc/systemd/system/docker.service.d ```

Y dentro crear el siguiente archivo:

``` touch override.conf ```

Editarlo: 

``` nano override.conf ```

Y copiar lo siguiente en el texto: 

``` [Service]
ExecStart=

ExecStart=/usr/bin/dockerd -H unix:// -D -H tcp://127.0.0.1:2375 
```

Para cerrar el editor podemos darle Ctrl+O, enter y Ctrl+X 

Para finalizar debemos recargar y reiniciar los servicios que estamos usando:"

``` sudo systemctl daemon-reload

sudo systemctl restart docker ```

[Link de referencia por corregir con creacción de override.conf](https://docs.docker.com/config/daemon/systemd/)

[Link de soporte sobre ShinyProxy](https://www.shinyproxy.io/getting-started/)

---

###Java y Maven (Gestor de paquetes)

Ahora vamos a instalar Java y Maven para instalar ShinyProxy. 

Miramos si el sistema está listo y actualizado: 

``` apt-get update && apt-get upgrade ```

Revisamos paquetes:

``` apt-get install software-properties-common ```

Agregamos el repositorio de Java: 

``` add-apt-repository ppa:linuxuprising/java ```

Actualizamos nuestra lista de paquetes de nuevo: 

``` apt-get update ```

Ahora instalamos Java :D

``` apt-get install oracle-java11-installer ```

Sí verificamos la versión instalada deberiamos obtener: 

``` java -version openjdk version "1.8.0_212" ``` 

Si hay problemas nos podemos dirigir a este link: [Instalación de JAVA en Ubuntu](https://thishosting.rocks/install-java-ubuntu/)

----

### Instalemos Apache Maven

Necesitamos Maven para instalar el paquete de ShinyProxy

Necesitamos todo actualizado de nuevo: 

``` sudo apt update ```

Instalamos Maven con el siguiente comando: 

``` sudo apt install maven ```

Al finalizar la instalación verificamos la versión: 

``` mvn -version ```

Deberíamos obtener: 

``` Apache Maven 3.5.2
Maven home: /usr/share/maven
Java version: 10.0.2, vendor: Oracle Corporation
Java home: /usr/lib/jvm/java-11-openjdk-amd64
Default locale: en_US, platform encoding: ISO-8859-1
OS name: "linux", version: "4.15.0-36-generic", arch: "amd64", family: "unix" 
```

Cualquier error que se observe podemos ir a la documentación: [Instalación de Maven en Ubuntu 18.04 LTS](https://linuxize.com/post/how-to-install-apache-maven-on-ubuntu-18-04/)

---

### Instalemos Screen

Algunas veces no está disponible por default, así que vamos a instalarlo:

``` sudo apt install screen ```

Para verificar que está instalado: 

``` screen ```

Y obtenemos la versión que instalamos.

Para iniciar una Screen debemos hacer el siguiente comando:

``` screen -S session_name ```

Donde session_name es la pantalla donde vamos a trabajar y dejar funcionando algo.

Para salir de alguna pantalla basta con oprimir Ctrl+ad.

[Más info de como usar Screen](https://linuxize.com/post/how-to-use-linux-screen/)

---

## Empecemos con ShinyProxy

Vamos a instalar todo desde el repositorio oficial de ellos, así que primero nos ubicamos en el servidor donde dejaremos todo consignado, si es un servidor único para la aplicación puede ser en el home, pero si es un servidor compartido organizar es lo mejor. 

``` git clone https://github.com/openanalytics/shinyproxy.git  ```

Entramos a nuestra carpeta

``` cd shinyproxy/ ```

Y compilamos con Maven

``` mvn -U clean install ```

Esto toma un buen rato en lograrse. ~10:28 min 

---


## Manos a la obra con el hola mundo de ShinyProxy

### Aplicaciones listas (ya en docker)

Ya hay varias aplicaciones contenidas para ser utilizadas rápidamente, una de ellas es el demo u hola mundo. 

Halamos el docker:

``` sudo docker pull openanalytics/shinyproxy-demo ```

Revisamos si contamos con la imagen en nuestra lista de imagenes de Docker

```  docker images | grep shinyproxy ```

Ahora podemos correr ShinyProxy con esta imagen, nos ubicamos en la carpeta del repo inicial

``` cd ~/shinyproxy/target/ ```

Y corremos ShinyProxy

``` java -jar shinyproxy-2.3.0.jar ```

En esta posición podemos consultar la dirección IP del servidor con el puerto 8080 y observar el login: donde usamos 'tesla' y 'password' para el ingreso. 


> Quiero hacer un parentesis, este comando de correr ShinyProxy se puede hacer dentro de un Screen en Linux para que no sea facilmente detenido. 

---

### Aplicaciones no listas, dockerfile

Esta es la parte más interesante, intervenimos el application.yml archivo que nos permite lograr la autenticación y además agregamos una app no contenida previamente. 

La manera más fácil de abordar este paso es guiarse con el repo de ShinyProxy llamado template, lo dejo clonado en este mismo repo. 

``` git clone https://github.com/openanalytics/shinyproxy-template.git ```

Nos vemos al repo:

``` cd shinyproxy-template/ ```

Vamos a construir la imagen de Docker con el dockerfile interno que hay

``` docker build -t openanalytics/shinyproxy-template . ```

Verificamos que la imagen esté disponible

``` 
$ docker images | grep "shinyproxy-template\|REPOSITORY"
REPOSITORY                          TAG                 IMAGE ID            CREATED             SIZE  
openanalytics/shinyproxy-template   latest              16e8c49e2261        25 minutes ago      851MB  
```

Vamos a verificar si la app funciona bien antes de entrar al application.yml


``` docker run -it -p 3838:3838  openanalytics/shinyproxy-template ```

Si todo funciona bien, continuamos con la creación del archivo .yml

Nos ubicamos en nuestra carpeta de ShinyProxy:

``` cd ~/shinyproxy/target/ ```

Y copiamos este archivo con el siguiente comando:

``` curl https://raw.githubusercontent.com/openanalytics/shinyproxy/master/src/main/resources/application-demo.yml > application.yml ```

y podremos editarlo con nano:

``` nano application.yml ```

Dentro del archivo debemos agregar los comandos usados arriba para iniciar el docker de Euler: 

``` 
specs:  
  - name: euler
    display-name: Euler's number
    description: Adding another app to shinyproxy
    docker-cmd: ["R", "-e shiny::runApp('/root/euler')"]
    docker-image: openanalytics/shinyproxy-template
    groups: scientists
```

Cerramos con Ctrl+O, enter y Ctrl+X

Así se agrega una app extra a ShinyProxy :)
---

Tips extra: 

Si la imagen ya está construida y queremos entrar a ella para hacer cambios o algo, podemos hacerlo así:

``` docker ps -l ```

Este comando nos muestra todos los containers que tenemos corriendo en el momento, copiaremos el ID y lo completamos en el siguiente comando: 

``` docker exec -it CONTAINER_ID /bin/bash ```

Esto nos dejará dentro de la consola del contenedor y podremos interactuar con el mismo. Para salir Ctrl+C o Ctrl+ad

----

¡Gracias! Espero te sirva este tutorial.