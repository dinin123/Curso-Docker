 Comandos Básicos de Docker para Volúmenes 💾

## Gestión de Volúmenes

| Acción | Comando |
|--------|---------|
| Listar todos los volúmenes | `docker volume ls` |
| Ver detalles de un volumen | `docker volume inspect <nombre_volumen>` |
| Crear un volumen | `docker volume create <nombre_volumen>` |
| Crear volumen con etiqueta | `docker volume create --label <clave=valor> <nombre_volumen>` |
| Eliminar un volumen | `docker volume rm <nombre_volumen>` |
| Eliminar todos los volúmenes no utilizados | `docker volume prune` |

## Uso de Volúmenes con Contenedores

| Acción | Comando |
|--------|---------|
| Montar volumen en un contenedor | `docker run -v <nombre_volumen>:<ruta_contenedor> <imagen>` |
| Montar volumen como solo lectura | `docker run -v <nombre_volumen>:<ruta_contenedor>:ro <imagen>` |
| Usar múltiples volúmenes | `docker run -v vol1:/data -v vol2:/backup <imagen>` |
| Usar volumen y bind mount juntos | `docker run -v <nombre_volumen>:/data -v $(pwd)/config:/config <imagen>` |

## Información y Depuración

| Acción | Comando |
|--------|---------|
| Ver espacio usado por volúmenes | `docker system df -v` |
| Inspeccionar el contenido del volumen (vía contenedor) | `docker run -it --rm -v <nombre_volumen>:/data alpine sh` |

## Notas

- Los volúmenes se almacenan en `/var/lib/docker/volumes/` por defecto.
- Los volúmenes persisten aunque se borren los contenedores.
- `bind mounts` usan rutas absolutas del host y no se gestionan con `docker volume`.
