# Instalación de Docker en Rocky Linux 9

En esta práctica aprenderás a instalar Docker en un sistema operativo Rocky Linux 9. Docker es una plataforma para desarrollar, enviar y ejecutar aplicaciones dentro de contenedores.

## Requisitos previos

- Tener acceso a una máquina con Rocky Linux 9 (física o virtual).
- Privilegios de superusuario (root) o acceso a `sudo`.

---

## Paso 1: Actualizar el sistema

Antes de comenzar, es recomendable actualizar los paquetes del sistema.

```bash
sudo dnf update -y
```

---

## Paso 2: Instalar dependencias necesarias

Instala las utilidades que Docker necesita para funcionar correctamente:

```bash
sudo dnf install -y dnf-utils device-mapper-persistent-data lvm2
```

---

## Paso 3: Añadir el repositorio oficial de Docker

Agrega el repositorio de Docker CE (Community Edition):

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

Aunque el repositorio es para CentOS, también es compatible con Rocky Linux.

---

## Paso 4: Instalar Docker Engine

Instala Docker utilizando el repositorio que acabas de configurar:

```bash
sudo dnf install -y docker-ce docker-ce-cli containerd.io
```

---

## Paso 5: Habilitar e iniciar el servicio Docker

Activa e inicia el servicio de Docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Comprueba que el servicio esté funcionando:

```bash
sudo systemctl status docker
```

---

## Paso 6: Verificar la instalación

Ejecuta el siguiente comando para verificar que Docker está instalado correctamente:

```bash
sudo docker run hello-world
```

Este comando descarga una imagen de prueba y la ejecuta en un contenedor. Si todo está correctamente instalado, verás un mensaje de bienvenida similar a este:

```bash
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

### ¿Qué significa esta salida?

Este mensaje no solo confirma que Docker está instalado correctamente, sino que **explica de forma clara cómo funciona Docker internamente**. Los pasos que describe son esenciales:

1. **El cliente Docker se comunica con el demonio Docker**: demuestra que los componentes principales están funcionando.
2. **El demonio descarga una imagen desde Docker Hub**: prueba que tienes acceso a internet y a los repositorios.
3. **Se crea y ejecuta un contenedor a partir de esa imagen**: comprueba que el sistema puede crear contenedores.
4. **La salida del contenedor llega a tu terminal**: verifica que el flujo completo funciona.

>  Esta simple prueba resume el ciclo de vida básico de un contenedor: *descargar imagen → crear contenedor → ejecutar → mostrar resultado*. Es una excelente primera validación del entorno.

---

## Paso 7 (Opcional): Usar Docker sin sudo

Para ejecutar Docker sin anteponer `sudo`:

```bash
sudo usermod -aG docker $USER
```

Después de ejecutar este comando, **cierra la sesión y vuelve a iniciarla** para que los cambios surtan efecto.

---

## Conclusión

Has instalado correctamente Docker en Rocky Linux 9. A partir de ahora puedes empezar a crear y gestionar contenedores con facilidad. En prácticas posteriores aprenderás a usar imágenes, redes y volúmenes con Docker.



# Ejercicio 1: Ejecutar contenedores

En este ejercicio, aprenderemos lo básico sobre cómo descargar imágenes, iniciar, detener y eliminar contenedores.

### Descargar una imagen

Para ejecutar contenedores, primero necesitamos descargar algunas imágenes.

1. Veamos qué imágenes tenemos actualmente en nuestra máquina ejecutando `docker images`:

    ```
    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    ```

2. En una instalación nueva de Docker, no deberíamos tener imágenes. Descarguemos una desde DockerHub.

    Normalmente descargamos imágenes desde DockerHub usando etiquetas (tags). Estas se ven así:

    ```
    # Imágenes oficiales de Docker
    <repositorio>:<etiqueta>
    # ubuntu:16.04
    # elasticsearch:5.2
    # nginx:latest

    # Imágenes creadas por usuarios u organizaciones
    <usuario o organización>/<repositorio>:<etiqueta>
    # peter/ubuntu:16.04
    # bitnami/rails:latest
    ```

    Podemos buscar imágenes con `docker search <palabra clave>`

    También puedes encontrar imágenes en línea en [DockerHub](https://hub.docker.com/).

    Ejecuta `docker pull ubuntu:18.04` para descargar una imagen de Ubuntu 16.04 desde DockerHub.

3. También podemos descargar diferentes versiones de la misma imagen.

    Ejecuta `docker pull ubuntu:24.04` para descargar la imagen de Ubuntu 16.10.

    Luego, si ejecutamos `docker images` nuevamente, deberíamos ver:

    ```
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    ubuntu              18.04               ...                 ...                 ...
    ubuntu              24.04               ...                 ...                 ...
    ```

4. Con el tiempo, tu máquina puede acumular muchas imágenes, así que conviene eliminar las que no se usan.

    Ejecuta `docker rmi <IMAGE ID>` para eliminar la imagen de Ubuntu 18.04.

    También puedes eliminar imágenes por etiqueta o por ID parcial:

     - `docker rmi 31`
     - `docker rmi ubuntu:18.04`

    Un atajo útil para eliminar todas las imágenes del sistema es:

    ```
    docker rmi $(docker images -a -q)
    ```

### Ejecutar nuestro contenedor

Usando la imagen de Ubuntu 24.04 que descargamos, podemos ejecutar nuestro primer contenedor.

1. Ejecutemos un ejemplo sencillo:

    ```
    $ docker run ubuntu:24.04 /bin/echo '¡Hola mundo!'
    ¡Hola mundo!
    ```

2. Verifiquemos qué contenedores tenemos tras esto:

    ```
    $ docker ps
    ```

    Usa `-a` para ver también los detenidos:

    ```
    $ docker ps -a
    ```

3. Ejecutemos algo más interactivo:

    ```
    $ docker run -it ubuntu:24.04 /bin/bash
    root@<container_id>:/# 
    ```

    Puedes usar comandos como `pwd` y `ls`, y salir con `exit`.

4. Para evitar que el terminal quede adjunto al contenedor, usa `-d`:

    ```
    $ docker run -d ubuntu:24.04 /bin/sleep 3600
    ```

5. Para acceder a un contenedor en ejecución, usa `docker exec`:

    ```
    $ docker exec -it <ID o nombre> /bin/bash
    ```

    Luego puedes usarlo como una sesión SSH y salir con `exit`.

6. Para detener un contenedor:

    ```
    $ docker stop <ID>
    ```

### Eliminar contenedores

7. Para eliminar contenedores detenidos:

    ```
    $ docker rm <ID>
    ```

    O todos a la vez:

    ```
    docker rm $(docker ps -a -q)
    ```

    Para eliminar automáticamente un contenedor tras su ejecución:

    ```
    $ docker run --rm ubuntu:24.04 /bin/echo '¡Hola y adiós!'
    ```

---

**FIN DEL EJERCICIO 1**

