# Imágenes Docker y Dockerfiles: Índice de ejercicios

- [Ejercicio 1: Explora las imágenes disponibles](#ejercicio-1-explora-las-imágenes-disponibles)
- [Ejercicio 2: Descarga una imagen desde Docker Hub](#ejercicio-2-descarga-una-imagen-desde-docker-hub)
- [Ejercicio 3: Crea un Dockerfile mínimo y construye una imagen](#ejercicio-3-crea-un-dockerfile-mínimo-y-construye-una-imagen)
- [Ejercicio 4: Construye una imagen personalizada con dependencias](#ejercicio-4-construye-una-imagen-personalizada-con-dependencias)
- [Ejercicio 5: Usa COPY para añadir archivos al contenedor](#ejercicio-5-usa-copy-para-añadir-archivos-al-contenedor)
- [Ejercicio 6: Diferencia entre CMD y ENTRYPOINT](#ejercicio-6-diferencia-entre-cmd-y-entrypoint)
- [Ejercicio 7: Usa RUN para instalar varias dependencias y compara capas](#ejercicio-7-usa-run-para-instalar-varias-dependencias-y-compara-capas)
- [Ejercicio 8: Usa variables de build (build args) en un Dockerfile](#ejercicio-8-usa-variables-de-build-build-args-en-un-dockerfile)
- [Ejercicio 9: Usa un Dockerfile para crear una imagen multi-usuario](#ejercicio-9-usa-un-dockerfile-para-crear-una-imagen-multi-usuario)
- [Ejercicio 10: Limpieza de imágenes y recursos](#ejercicio-10-limpieza-de-imágenes-y-recursos)

---

## Ejercicio 1: Explora las imágenes disponibles

**Planteamiento:**  
Lista todas las imágenes locales disponibles en tu sistema.

**Desarrollo y resolución:**
```bash
docker images
```
Ves una tabla con las imágenes, sus tags, IDs, tamaño, etc.

---

## Ejercicio 2: Descarga una imagen desde Docker Hub

**Planteamiento:**  
Descarga la imagen oficial de Node.js, sin ejecutarla aún.

**Desarrollo y resolución:**
```bash
docker pull node:18
```
Confirma con `docker images` que la imagen ya está localmente.

---

## Ejercicio 3: Crea un Dockerfile mínimo y construye una imagen

**Planteamiento:**  
Haz un Dockerfile que simplemente imprima “Hola Dockerfile”.

**Desarrollo:**

Crea el archivo `Dockerfile`:
```dockerfile
FROM alpine
CMD ["echo", "Hola Dockerfile"]
```

Construye la imagen:
```bash
docker build -t saludo-dockerfile .
```

Ejecuta la imagen:
```bash
docker run saludo-dockerfile
```

**Resolución:**  
Se muestra `Hola Dockerfile` por pantalla.

**Limpieza:**
```bash
docker rmi saludo-dockerfile
```

---

## Ejercicio 4: Construye una imagen personalizada con dependencias

**Planteamiento:**  
Haz una imagen basada en Python que instale el paquete `requests` al construirla.

**Desarrollo:**

Crea `Dockerfile`:
```dockerfile
FROM python:3.11-slim
RUN pip install requests
CMD ["python3"]
```

Construye la imagen:
```bash
docker build -t python-requests .
```

Comprueba que `requests` está instalado:
```bash
docker run python-requests python3 -c "import requests; print(requests.__version__)"
```

**Resolución:**  
Ves la versión instalada de requests, por ejemplo, `2.31.0`.

---

## Ejercicio 5: Usa COPY para añadir archivos al contenedor

**Planteamiento:**  
Copia un archivo local llamado `mensaje.txt` al contenedor usando el Dockerfile.

**Desarrollo:**

Crea `mensaje.txt`:
```
¡Este archivo está dentro del contenedor!
```

Crea `Dockerfile`:
```dockerfile
FROM alpine
COPY mensaje.txt /mensaje.txt
CMD ["cat", "/mensaje.txt"]
```

Construye y ejecuta:
```bash
docker build -t mensaje-copy .
docker run mensaje-copy
```

**Resolución:**  
Ves el contenido del archivo por pantalla.

---

## Ejercicio 6: Diferencia entre CMD y ENTRYPOINT

**Planteamiento:**  
Haz un Dockerfile en el que la instrucción por defecto se pueda sobrescribir con argumentos, usando ENTRYPOINT y CMD.

**Desarrollo:**

Crea `Dockerfile`:
```dockerfile
FROM alpine
ENTRYPOINT ["echo"]
CMD ["Hola por defecto"]
```

Construye la imagen:
```bash
docker build -t entrypoint-cmd .
```

Ejecútalo tal cual:
```bash
docker run entrypoint-cmd
```
Salida: `Hola por defecto`

Ejecútalo con argumento:
```bash
docker run entrypoint-cmd "¡Sobrescribiendo CMD!"
```
Salida: `¡Sobrescribiendo CMD!`

---

## Ejercicio 7: Usa RUN para instalar varias dependencias y compara capas

**Planteamiento:**  
Crea dos Dockerfiles: uno instalando dos paquetes con dos RUN, y otro combinando los comandos en un RUN. Compara las capas.

**Desarrollo:**

**Dockerfile A (más capas):**
```dockerfile
FROM alpine
RUN apk add --no-cache curl
RUN apk add --no-cache nano
```

**Dockerfile B (una capa):**
```dockerfile
FROM alpine
RUN apk add --no-cache curl nano
```

Construye ambas imágenes:
```bash
docker build -t multi-run -f DockerfileA .
docker build -t single-run -f DockerfileB .
docker history multi-run
docker history single-run
```

**Resolución:**  
La imagen “multi-run” tiene más capas (una por RUN).  
“single-run” tiene menos capas y suele ocupar menos espacio.

---

## Ejercicio 8: Usa variables de build (build args) en un Dockerfile

**Planteamiento:**  
Permite que el usuario elija qué versión de Alpine usar con un ARG en el Dockerfile.

**Desarrollo:**

Crea `Dockerfile`:
```dockerfile
ARG ALPINE_VERSION=3.18
FROM alpine:${ALPINE_VERSION}
CMD ["cat", "/etc/alpine-release"]
```

Construye la imagen con la versión por defecto:
```bash
docker build -t alpine-arg .
docker run alpine-arg
```

Construye con otra versión:
```bash
docker build --build-arg ALPINE_VERSION=3.17 -t alpine-arg-317 .
docker run alpine-arg-317
```

**Resolución:**  
Ves el número de versión de Alpine según el valor pasado por argumento.

---

## Ejercicio 9: Usa un Dockerfile para crear una imagen multi-usuario

**Planteamiento:**  
Crea una imagen Alpine donde añades un usuario llamado “usuario” y que la shell por defecto sea con ese usuario.

**Desarrollo:**

`Dockerfile`:
```dockerfile
FROM alpine
RUN adduser -D usuario
USER usuario
CMD ["whoami"]
```

Construye y ejecuta:
```bash
docker build -t alpine-usuario .
docker run alpine-usuario
```

**Resolución:**  
Muestra `usuario` como salida, y la shell corre como ese usuario, no como root.

---

## Ejercicio 10: Limpieza de imágenes y recursos

**Planteamiento:**  
Elimina todas las imágenes y contenedores que creaste durante estos ejercicios para dejar limpio tu entorno.

**Desarrollo:**
```bash
docker ps -a # Revisa qué contenedores hay
docker rm -f $(docker ps -aq) # Borra todos los contenedores
docker rmi -f saludo-dockerfile python-requests mensaje-copy entrypoint-cmd multi-run single-run alpine-arg alpine-arg-317 alpine-usuario mensaje-copy node:18
docker system prune -f
```

**Resolución:**  
Tu entorno queda limpio, sin restos de pruebas.
