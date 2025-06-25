# Orquestación básica con Docker Compose:  Índice de ejercicios

1. [Ejercicio 1: Crea un docker-compose.yml mínimo y lanza un servicio web](#ejercicio-1-crea-un-docker-composeyml-mínimo-y-lanza-un-servicio-web)
2. [Ejercicio 2: Añade una base de datos y una red personalizada](#ejercicio-2-añade-una-base-de-datos-y-una-red-personalizada)
3. [Ejercicio 3: Usa volúmenes persistentes para la base de datos](#ejercicio-3-usa-volúmenes-persistentes-para-la-base-de-datos)
4. [Ejercicio 4: Modifica la página web desde el host usando bind mount](#ejercicio-4-modifica-la-página-web-desde-el-host-usando-bind-mount)
5. [Ejercicio 5: Usa variables de entorno externas en Compose](#ejercicio-5-usa-variables-de-entorno-externas-en-compose)
6. [Ejercicio 6: Escala el servicio web y observa los resultados](#ejercicio-6-escala-el-servicio-web-y-observa-los-resultados)
7. [Ejercicio 7: Usa depends_on para el orden de arranque](#ejercicio-7-usa-depends_on-para-el-orden-de-arranque)
8. [Ejercicio 8: Añade un healthcheck al servicio de base de datos](#ejercicio-8-añade-un-healthcheck-al-servicio-de-base-de-datos)
9. [Ejercicio 9: Añade un servicio adicional solo para desarrollo](#ejercicio-9-añade-un-servicio-adicional-solo-para-desarrollo)
10. [Ejercicio 10: Limpia todos los recursos de Compose](#ejercicio-10-limpia-todos-los-recursos-de-compose)
11. [Ejercicio 11: Orquesta y mide el rendimiento de acceso a disco en diferentes servicios](#ejercicio-11-orquesta-y-mide-el-rendimiento-de-acceso-a-disco-en-diferentes-servicios)
12. [Ejercicio 12: Docker Compose con Build de Imagen](#ejercicio-12-docker-compose-con-build-de-imagen)
---

## Ejercicio 1: Crea un `docker-compose.yml` mínimo y lanza un servicio web

**Planteamiento:**  
Crea un archivo Compose con un solo servicio NGINX que escuche en el puerto 8082 del host.

**Desarrollo:**  
docker-compose.yml  
```yaml
version: "3.8"
services:
  web:
    image: nginx
    ports:
      - "8082:80"
```
```bash
docker compose up -d
```

**Resolución:**  
Accede a http://localhost:8082 y ves la página de NGINX.

---

## Ejercicio 2: Añade una base de datos y una red personalizada

**Planteamiento:**  
Amplía el archivo Compose para que incluya un servicio MySQL y estén conectados en una red personalizada.

**Desarrollo:**  
docker-compose.yml  
```yaml
version: "3.8"
services:
  web:
    image: nginx
    ports:
      - "8082:80"
    networks:
      - redapp
  db:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: clave123
    networks:
      - redapp
networks:
  redapp:
```
```bash
docker compose up -d
```

**Resolución:**  
Ambos servicios aparecen conectados en la red redapp.  
Comprueba con:  
```bash
docker compose ps
docker network inspect <proyecto>_redapp
```

---

## Ejercicio 3: Usa volúmenes persistentes para la base de datos

**Planteamiento:**  
Configura un volumen para que los datos de MySQL se conserven aunque se borre el contenedor.

**Desarrollo:**  
En el servicio db:  
```yaml
volumes:
  - datos_mysql:/var/lib/mysql
```
Al final del archivo:  
```yaml
volumes:
  datos_mysql:
```
```bash
docker compose up -d
```

**Resolución:**  
Comprueba con:  
```bash
docker volume ls
```
El volumen existe y persiste tras parar los servicios.

---

## Ejercicio 4: Modifica la página web desde el host usando bind mount

**Planteamiento:**  
Haz que el directorio ./html del host sobrescriba el contenido web de NGINX.

**Desarrollo:**  
```bash
mkdir html
echo "Hola desde Compose" > html/index.html
```
En el servicio web:  
```yaml
volumes:
  - ./html:/usr/share/nginx/html
```
```bash
docker compose up -d
```

**Resolución:**  
Accede a http://localhost:8082 y ves el mensaje personalizado.

---

## Ejercicio 5: Usa variables de entorno externas en Compose

**Planteamiento:**  
Guarda la contraseña de MySQL en un archivo .env y haz que Compose la lea.

**Desarrollo:**  
Crea .env:  
```
MYSQL_ROOT_PASSWORD=supersecreta
```
En el Compose:  
```yaml
db:
  environment:
    MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
```
```bash
docker compose up -d
```

**Resolución:**  
La contraseña usada por MySQL es la de .env y no aparece en claro en el docker-compose.yml.

---

## Ejercicio 6: Escala el servicio web y observa los resultados

**Planteamiento:**  
Lanza tres instancias del servicio web y comprueba qué ocurre con los puertos.

**Desarrollo:**  
```bash
docker compose up --scale web=3 -d
docker compose ps
```

**Resolución:**  
Solo una instancia puede usar el puerto 8082 (a menos que uses puertos automáticos).  
Las demás pueden fallar o usar puertos dinámicos si configuras "80" en vez de "8082:80".

---

## Ejercicio 7: Usa depends_on para el orden de arranque

**Planteamiento:**  
Asegúrate de que el servicio web espera a que la base de datos esté en marcha.

**Desarrollo:**  
En el Compose:
```yaml
web:
  image: nginx
  depends_on:
    - db
db:
  image: mysql:latest
  environment:
    MYSQL_ROOT_PASSWORD: root
```
```bash
docker compose up
```

**Resolución:**  
La base de datos se inicia antes que el servicio web.

---

## Ejercicio 8: Añade un healthcheck al servicio de base de datos

**Planteamiento:**  
Incluye un healthcheck para que Compose sepa cuándo la base de datos está lista.

**Desarrollo:**  
En el Compose:
```yaml
db:
  image: mysql:latest
  environment:
    MYSQL_ROOT_PASSWORD: root
  healthcheck:
    test: ["CMD-SHELL", "mysqladmin ping -h localhost -proot"]
    interval: 10s
    retries: 5
```
```bash
docker inspect <proyecto>_db_1 | grep Health -A 4
```

**Resolución:**  
El estado de Health pasa a healthy cuando la base de datos responde correctamente.

---

## Ejercicio 9: Añade un servicio adicional solo para desarrollo

**Planteamiento:**  
Agrega un contenedor adminer (gestor web de bases de datos) solo cuando lo necesites para pruebas.

**Desarrollo:**  
En el Compose:
```yaml
adminer:
  image: adminer
  ports:
    - "8083:8080"
  depends_on:
    - db
```
```bash
docker compose up -d
```

**Resolución:**  
Accede a http://localhost:8083 y administra la BD fácilmente.

---

## Ejercicio 10: Limpia todos los recursos de Compose

**Planteamiento:**  
Borra todos los servicios, volúmenes y redes asociadas al proyecto Compose.

**Desarrollo:**  
```bash
docker compose down --volumes --remove-orphans
```

**Resolución:**  
No quedan contenedores, volúmenes ni redes del proyecto.

---

## Ejercicio 11: Orquesta y mide el rendimiento de acceso a disco en diferentes servicios

**Planteamiento:**  
Crea un stack Compose con un servicio que use un volumen named, otro que use bind mount y otro con tmpfs. Realiza un test de escritura en disco desde cada uno y compara los resultados.

**Desarrollo:**

docker-compose.yml  
```yaml
version: "3.8"
services:
  named:
    image: ubuntu
    command: ["bash", "-c", "dd if=/dev/zero of=/mnt/speed_test bs=1M count=256 oflag=dsync"]
    volumes:
      - named_volume:/mnt
  bind:
    image: ubuntu
    command: ["bash", "-c", "dd if=/dev/zero of=/mnt/speed_test bs=1M count=256 oflag=dsync"]
    volumes:
      - ./benchdir:/mnt
  tmpfs:
    image: ubuntu
    command: ["bash", "-c", "dd if=/dev/zero of=/mnt/speed_test bs=1M count=256 oflag=dsync"]
    tmpfs:
      - /mnt
volumes:
  named_volume:
```
```bash
mkdir -p benchdir
docker compose up
```

**Resolución:**  
Observa el resultado de cada servicio en la salida de Compose (`docker compose logs`).  
Normalmente, tmpfs será el más rápido, seguido de named volume y luego bind mount.

---


## Ejercicio 12: Docker Compose con Build de Imagen

Este es un ejemplo básico de cómo usar `docker-compose.yml` para construir una imagen personalizada a partir de un `Dockerfile`.

## Estructura de Archivos

Supongamos que tienes la siguiente estructura de directorios:

```
mi-proyecto/
│
├── docker-compose.yml
├── Dockerfile
└── html/
    └── index.html
```

## docker-compose.yml

```yaml
version: '3.8'

services:
  webapp:
    build:
      context: .
      dockerfile: Dockerfile
    image: miwebapp:latest
    ports:
      - "8080:80"
```

## Dockerfile

```Dockerfile
# Dockerfile
FROM nginx:alpine
COPY ./html /usr/share/nginx/html
```

Este Dockerfile toma una imagen base de Nginx y copia los archivos del directorio `html/` al directorio de contenido estático de Nginx.

## Comandos

Para construir y ejecutar el contenedor:

```bash
docker-compose up --build
```

Para detener y eliminar los contenedores:

```bash
docker-compose down
```

## Resultado

Después de ejecutar el comando, deberías poder acceder a tu aplicación en:

```
http://localhost:8080
```

## Personalización

Puedes modificar el `Dockerfile` para usar otra imagen base, instalar paquetes o copiar otros archivos, dependiendo de tus necesidades.
