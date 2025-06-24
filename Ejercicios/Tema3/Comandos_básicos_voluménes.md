# Comandos Básicos de Docker para Volúmenes

## Tipos de Volúmenes en Docker

Docker admite tres tipos principales de volúmenes:

### 1. **Volumen gestionado por Docker**
- Docker se encarga de su ubicación y gestión.
- Ideal para persistencia de datos sin preocuparse por rutas del host.
- Se crea automáticamente con `-v volumen:/ruta` si no existe previamente.

**Ejemplo:**
```bash
docker run -v mi_volumen:/app/data nginx
```

### 2. **Bind Mount**
- Monta un directorio o archivo del sistema host dentro del contenedor.
- Requiere ruta absoluta.
- Muy útil en desarrollo, ya que refleja los cambios en tiempo real.

**Ejemplo:**
```bash
docker run -v /home/usuario/proyecto:/app nginx
```

### 3. **Tmpfs Mount**
- Montaje en memoria (RAM), útil para datos temporales.
- No se guarda nada en disco.
- Solo disponible en contenedores Linux.

**Ejemplo:**
```bash
docker run --tmpfs /app/cache:rw,size=64m nginx
```

---

## Gestión de Volúmenes

| Acción | Comando |
|--------|---------|
| Listar todos los volúmenes | `docker volume ls` |
| Ver detalles de un volumen | `docker volume inspect <nombre_volumen>` |
| Crear un volumen | `docker volume create <nombre_volumen>` |
| Crear volumen con etiqueta | `docker volume create --label <clave=valor> <nombre_volumen>` |
| Eliminar un volumen | `docker volume rm <nombre_volumen>` |
| Eliminar todos los volúmenes no utilizados | `docker volume prune` |

---

## Uso de Volúmenes con Contenedores

| Acción | Comando |
|--------|---------|
| Montar volumen en un contenedor | `docker run -v <nombre_volumen>:<ruta_contenedor> <imagen>` |
| Montar volumen como solo lectura | `docker run -v <nombre_volumen>:<ruta_contenedor>:ro <imagen>` |
| Usar múltiples volúmenes | `docker run -v vol1:/data -v vol2:/backup <imagen>` |
| Usar volumen y bind mount juntos | `docker run -v <nombre_volumen>:/data -v $(pwd)/config:/config <imagen>` |

---

## Información y Depuración

| Acción | Comando |
|--------|---------|
| Ver espacio usado por volúmenes | `docker system df -v` |
| Inspeccionar el contenido del volumen (vía contenedor) | `docker run -it --rm -v <nombre_volumen>:/data alpine sh` |

---

## Notas

- Los volúmenes se almacenan en `/var/lib/docker/volumes/` por defecto.
- Los volúmenes persisten aunque se borren los contenedores.
- `bind mounts` usan rutas absolutas del host y no se gestionan con `docker volume`.
- `tmpfs` solo está disponible en sistemas Linux y no se persiste ni aparece en `docker volume ls`.
