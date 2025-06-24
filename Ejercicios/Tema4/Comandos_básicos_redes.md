# Comandos Básicos de Docker para Redes

## Tipos de Redes en Docker

Docker admite varios tipos de redes que se pueden usar con diferentes propósitos:

### 1. **Bridge (puente)**
- Tipo por defecto en Docker.
- Crea una red virtual aislada para contenedores.
- Ideal para entornos de un solo host.

**Ejemplo:**
```bash
docker network create --driver bridge mi_red_bridge
docker run -d --name web1 --network mi_red_bridge nginx
```

### 2. **Host**
- El contenedor comparte la pila de red del host.
- No hay aislamiento de red entre host y contenedor.
- Disponible solo en Linux.

**Ejemplo:**
```bash
docker run --network host nginx
```

### 3. **None**
- El contenedor no tiene red.
- Útil para contenedores aislados.

**Ejemplo:**
```bash
docker run --network none nginx
```

### 4. **Overlay**
- Permite comunicación entre contenedores en diferentes hosts (requiere Docker Swarm).
- Ideal para arquitecturas distribuidas.

**Ejemplo:**
```bash
docker network create -d overlay mi_red_overlay
```

### 5. **Macvlan**
- Asigna una dirección MAC y IP desde la red física del host al contenedor.
- Útil para conectividad directa con la red física.

**Ejemplo:**
```bash
docker network create -d macvlan   --subnet=192.168.1.0/24   --gateway=192.168.1.1   -o parent=eth0 mi_red_macvlan
```

---

## Gestión de Redes

| Acción | Comando |
|--------|---------|
| Listar todas las redes | `docker network ls` |
| Ver detalles de una red | `docker network inspect <nombre_red>` |
| Conectar un contenedor a una red | `docker network connect <nombre_red> <contenedor>` |
| Desconectar un contenedor de una red | `docker network disconnect <nombre_red> <contenedor>` |

---

## Creación de Redes

| Acción | Comando |
|--------|---------|
| Crear red por defecto (bridge) | `docker network create <nombre_red>` |
| Crear red con driver específico | `docker network create --driver <bridge|host|overlay|macvlan> <nombre_red>` |
| Crear red con subred y gateway | `docker network create --subnet <subred> --gateway <ip_gateway> <nombre_red>` |
| Crear red con etiqueta | `docker network create --label <clave=valor> <nombre_red>` |

---

## Eliminación de Redes

| Acción | Comando |
|--------|---------|
| Eliminar una red | `docker network rm <nombre_red>` |
| Eliminar todas las redes no usadas | `docker network prune` |

---

## Redes al Ejecutar Contenedores

| Acción | Comando |
|--------|---------|
| Usar una red específica | `docker run --network <nombre_red> <imagen>` |
| Ejecutar sin conexión de red | `docker run --network none <imagen>` |
| Usar red del host | `docker run --network host <imagen>` |

---

## Información Adicional

| Acción | Comando |
|--------|---------|
| Ver configuración de red de un contenedor | `docker inspect <contenedor>` |
| Ver estadísticas de red en tiempo real | `docker stats` (incluye tráfico de red) |
