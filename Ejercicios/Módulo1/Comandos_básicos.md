# Comandos Básicos de Docker 🐳

## Gestión de Contenedores

| Acción | Comando |
|--------|---------|
| Ejecutar un contenedor | `docker run <opciones> <imagen>` |
| Ejecutar en segundo plano (modo daemon) | `docker run -d <imagen>` |
| Ejecutar con nombre personalizado | `docker run --name <nombre> <imagen>` |
| Ejecutar con puerto mapeado | `docker run -p <host>:<contenedor> <imagen>` |
| Ejecutar con volumen | `docker run -v <host_dir>:<contenedor_dir> <imagen>` |
| Entrar en un contenedor en ejecución | `docker exec -it <contenedor> bash` |
| Crear un contenedor (sin ejecutarlo) | `docker create <imagen>` |
| Iniciar un contenedor detenido | `docker start <contenedor>` |
| Detener un contenedor en ejecución | `docker stop <contenedor>` |
| Reiniciar un contenedor | `docker restart <contenedor>` |
| Eliminar un contenedor detenido | `docker rm <contenedor>` |
| Eliminar todos los contenedores detenidos | `docker container prune` |

## Listados e Información

| Acción | Comando |
|--------|---------|
| Listar contenedores en ejecución | `docker ps` |
| Listar todos los contenedores (activos e inactivos) | `docker ps -a` |
| Ver detalles de un contenedor | `docker inspect <contenedor>` |
| Ver logs de un contenedor | `docker logs <contenedor>` |
| Ver estadísticas en tiempo real | `docker stats` |

## Utilidades Adicionales

| Acción | Comando |
|--------|---------|
| Ver procesos dentro de un contenedor | `docker top <contenedor>` |
| Detener todos los contenedores | `docker stop $(docker ps -q)` |
| Eliminar todos los contenedores | `docker rm $(docker ps -aq)` |
