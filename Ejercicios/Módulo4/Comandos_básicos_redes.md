# Comandos Básicos de Docker para Redes 🌐

## Tipos de Redes en Docker

Docker soporta varios drivers de red:

- `bridge` (por defecto)
- `host`
- `none`
- `overlay` (para Swarm)
- `macvlan`

## Gestión de Redes

| Acción | Comando |
|--------|---------|
| Listar todas las redes | `docker network ls` |
| Ver detalles de una red | `docker network inspect <nombre_red>` |
| Conectar un contenedor a una red | `docker network connect <nombre_red> <contenedor>` |
| Desconectar un contenedor de una red | `docker network disconnect <nombre_red> <contenedor>` |

## Creación de Redes

| Acción | Comando |
|--------|---------|
| Crear una red bridge (por defecto) | `docker network create <nombre_red>` |
| Crear red con driver específico | `docker network create --driver <bridge|host|overlay|macvlan> <nombre_red>` |
| Crear red con subred y gateway | `docker network create --subnet <subred> --gateway <ip_gateway> <nombre_red>` |
| Crear red y etiquetarla | `docker network create --label <clave=valor> <nombre_red>` |

## Eliminación de Redes

| Acción | Comando |
|--------|---------|
| Eliminar una red | `docker network rm <nombre_red>` |
| Eliminar todas las redes no usadas | `docker network prune` |

## Redes al Ejecutar Contenedores

| Acción | Comando |
|--------|---------|
| Usar una red específica al ejecutar contenedor | `docker run --network <nombre_red> <imagen>` |
| Ejecutar sin conexión de red | `docker run --network none <imagen>` |
| Usar red del host | `docker run --network host <imagen>` |

## Información Adicional

| Acción | Comando |
|--------|---------|
| Ver configuración de red de un contenedor | `docker inspect <contenedor>` |
| Ver estadísticas de red en tiempo real | `docker stats` (muestra tráfico de red entre otras cosas) |
