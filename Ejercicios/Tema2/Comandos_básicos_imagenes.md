# Comandos Básicos Docker para Creación y Gestión de Imágenes

## Creación de Imágenes

| Acción | Comando |
|--------|---------|
| Crear una imagen a partir de un Dockerfile | `docker build -t <nombre_imagen> <directorio>` |
| Crear imagen sin usar cache | `docker build --no-cache -t <nombre_imagen> <directorio>` |
| Crear imagen con archivo Dockerfile personalizado | `docker build -f <ruta/Dockerfile> -t <nombre_imagen> <directorio>` |
| Crear imagen y establecer build args | `docker build --build-arg <clave=valor> -t <nombre_imagen> <directorio>` |

## Gestión de Imágenes

| Acción | Comando |
|--------|---------|
| Ver todas las imágenes locales | `docker images` |
| Ver imágenes filtradas por nombre | `docker images <nombre>` |
| Ver detalles de una imagen | `docker inspect <imagen>` |
| Etiquetar una imagen existente | `docker tag <imagen_origen> <nuevo_nombre>:<tag>` |
| Eliminar una imagen | `docker rmi <imagen>` |
| Eliminar múltiples imágenes | `docker rmi <img1> <img2> ...` |
| Eliminar todas las imágenes sin etiqueta (dangling) | `docker image prune` |
| Eliminar todas las imágenes no usadas | `docker image prune -a` |
| Forzar eliminación de imagen | `docker rmi -f <imagen>` |

## Exportar, Importar y Compartir Imágenes

| Acción | Comando |
|--------|---------|
| Guardar una imagen como archivo tar | `docker save -o <archivo.tar> <imagen>` |
| Cargar una imagen desde archivo tar | `docker load -i <archivo.tar>` |
| Exportar un contenedor como archivo tar | `docker export <contenedor> > <archivo.tar>` |
| Importar una imagen desde archivo tar | `docker import <archivo.tar>` |
| Subir imagen a Docker Hub | `docker push <usuario>/<imagen>:<tag>` |
| Descargar imagen de Docker Hub | `docker pull <usuario>/<imagen>:<tag>` |
| Iniciar sesión en Docker Hub | `docker login` |

## Utilidades

| Acción | Comando |
|--------|---------|
| Comprobar capas de una imagen | `docker history <imagen>` |
| Crear contenedor sin ejecutar (para preparar imagen) | `docker create <imagen>` |
| Crear imagen desde contenedor en ejecución | `docker commit <contenedor> <nombre_imagen>` |
