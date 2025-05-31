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

    Ejecuta `docker pull ubuntu:16.04` para descargar una imagen de Ubuntu 16.04 desde DockerHub.

3. También podemos descargar diferentes versiones de la misma imagen.

    Ejecuta `docker pull ubuntu:16.10` para descargar la imagen de Ubuntu 16.10.

    Luego, si ejecutamos `docker images` nuevamente, deberíamos ver:

    ```
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    ubuntu              16.10               ...                 ...                 ...
    ubuntu              16.04               ...                 ...                 ...
    ```

4. Con el tiempo, tu máquina puede acumular muchas imágenes, así que conviene eliminar las que no se usan.

    Ejecuta `docker rmi <IMAGE ID>` para eliminar la imagen de Ubuntu 16.10.

    También puedes eliminar imágenes por etiqueta o por ID parcial:

     - `docker rmi 31`
     - `docker rmi ubuntu:16.10`

    Un atajo útil para eliminar todas las imágenes del sistema es:

    ```
    docker rmi $(docker images -a -q)
    ```

### Ejecutar nuestro contenedor

Usando la imagen de Ubuntu 16.04 que descargamos, podemos ejecutar nuestro primer contenedor.

1. Ejecutemos un ejemplo sencillo:

    ```
    $ docker run ubuntu:16.04 /bin/echo '¡Hola mundo!'
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
    $ docker run -it ubuntu:16.04 /bin/bash
    root@<container_id>:/# 
    ```

    Puedes usar comandos como `pwd` y `ls`, y salir con `exit`.

4. Para evitar que el terminal quede adjunto al contenedor, usa `-d`:

    ```
    $ docker run -d ubuntu:16.04 /bin/sleep 3600
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
    $ docker run --rm ubuntu:16.04 /bin/echo '¡Hola y adiós!'
    ```

---

**FIN DEL EJERCICIO 1**

