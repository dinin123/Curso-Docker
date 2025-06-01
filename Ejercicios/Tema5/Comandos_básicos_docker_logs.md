# Comandos de Docker para Gestión de Logs, Eventos y Auditorías 📋

## Logs de Contenedores

Docker permite acceder a los logs generados por los procesos dentro de los contenedores.

| Acción | Comando |
|--------|---------|
| Ver logs de un contenedor | `docker logs <contenedor>` |
| Ver logs en tiempo real | `docker logs -f <contenedor>` |
| Incluir marcas de tiempo | `docker logs -t <contenedor>` |
| Ver las últimas líneas (por defecto 10) | `docker logs --tail <n> <contenedor>` |
| Ver logs desde una hora específica | `docker logs --since "2024-06-01T10:00:00"` |
| Ver logs hasta una hora específica | `docker logs --until "2024-06-01T12:00:00"` |
| Combinar seguimiento y marcas de tiempo | `docker logs -ft <contenedor>` |

---

## Eventos de Docker

Puedes monitorizar eventos en tiempo real relacionados con contenedores, imágenes, volúmenes, redes y más.

| Acción | Comando |
|--------|---------|
| Ver todos los eventos del sistema Docker | `docker events` |
| Filtrar eventos por contenedor | `docker events --filter container=<id|nombre>` |
| Filtrar por tipo de objeto | `docker events --filter type=container` |
| Filtrar por acción específica | `docker events --filter event=start` |
| Filtrar por tiempo | `docker events --since "2024-06-01T00:00:00"` |

---

## Auditoría con Docker

### Ejemplo 1: Auditar eventos del sistema durante una ventana de tiempo

```bash
docker events --since "2024-06-01T00:00:00" --until "2024-06-01T23:59:59" > eventos_01jun.log
```

### Ejemplo 2: Auditar el historial de una imagen

```bash
docker history <nombre_imagen>
```

### Ejemplo 3: Auditar acceso a contenedores (requiere journald)

```bash
journalctl -u docker.service | grep 'container start'
```

### Ejemplo 4: Guardar logs de un contenedor a fichero

```bash
docker logs <contenedor> > logs_<contenedor>_$(date +%F).log
```

### Ejemplo 5: Auditar eventos de tipo exec (ejecución de comandos)

```bash
docker events --filter event=exec_create > auditoria_exec.log
```

---

## Automatización con cron

Puedes automatizar auditorías diarias creando scripts que se ejecuten con `cron`.

### Ejemplo de script `/usr/local/bin/auditar_docker.sh`

```bash
#!/bin/bash
# Guardar eventos del día anterior
FECHA=$(date --date="yesterday" +%F)
docker events --since "${FECHA}T00:00:00" --until "${FECHA}T23:59:59" > /var/log/docker/eventos_${FECHA}.log

# Guardar logs de contenedor específico
docker logs mi_contenedor --since "${FECHA}T00:00:00" --until "${FECHA}T23:59:59" > /var/log/docker/logs_mi_contenedor_${FECHA}.log
```

### Añadir al cron (ejecutar diariamente a las 00:30)

```bash
crontab -e
```

Agregar:

```
30 0 * * * /usr/local/bin/auditar_docker.sh
```

---

## Buenas Prácticas

- Redireccionar la salida de eventos y logs a archivos en `/var/log/docker/`.
- Rotar logs con `logrotate` para evitar llenar el disco.
- Usar herramientas como `auditd`, ELK, Grafana Loki o syslog para auditorías centralizadas.
- Mantener el timestamp en nombres de archivo para trazabilidad.

---
