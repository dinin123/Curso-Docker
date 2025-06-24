# Comandos Básicos de Docker Compose

Docker Compose permite definir y ejecutar aplicaciones Docker multicontenedor a través de un archivo `docker compose.yml`.

---

## Comandos Principales

| Acción | Comando |
|--------|---------|
| Levantar servicios (en foreground) | `docker compose up` |
| Levantar servicios en segundo plano | `docker compose up -d` |
| Detener servicios | `docker compose down` |
| Reconstruir servicios | `docker compose up --build` |
| Eliminar contenedores, redes y volúmenes (todo) | `docker compose down -v` |
| Ver el estado de los servicios | `docker compose ps` |
| Ver los logs de los servicios | `docker compose logs` |
| Ver logs en tiempo real | `docker compose logs -f` |
| Ejecutar un comando en un servicio | `docker compose exec <servicio> <comando>` |
| Acceder a un contenedor con shell | `docker compose exec <servicio> sh` o `bash` |
| Crear contenedores sin iniciar | `docker compose create` |
| Iniciar servicios ya creados | `docker compose start` |
| Detener servicios sin eliminar | `docker compose stop` |
| Eliminar contenedores sin tocar volúmenes o imágenes | `docker compose rm` |

---

## Estructura Básica del `docker compose.yml`

```yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  app:
    build: .
    volumes:
      - .:/code
    depends_on:
      - db
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: ejemplo
```

---

## Gestión de Recursos

| Acción | Comando |
|--------|---------|
| Ver estadísticas de contenedores | `docker compose top` |
| Ver el uso de recursos en tiempo real | `docker stats` (de Docker directamente) |

---

## Archivos y Variables

- El archivo por defecto es `docker compose.yml`
- Puedes especificar otro archivo con `-f`:
  ```bash
  docker compose -f docker compose.dev.yml up
  ```
- Las variables de entorno pueden estar en un archivo `.env` o definirse directamente.

---

## Notas

- Docker Compose simplifica el manejo de múltiples servicios.
- La opción `depends_on` **no** espera a que un servicio esté “listo”, solo a que haya sido iniciado.
- Usa `healthcheck` para esperar que un contenedor esté disponible.

---
