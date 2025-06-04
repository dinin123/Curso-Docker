# Ejercicios avzanzados: Índice de ejercicios

- [Ejercicio 1: Contenedor SSH seguro con Alpine, solo acceso clave RSA y usuario sudo](#ejercicio-1-contenedor-ssh-seguro-con-alpine-solo-acceso-clave-rsa-y-usuario-sudo)
- [Ejercicio 2: Simulación y recuperación ante fallo grave en Docker (falta de directorio crítico)](#ejercicio-2-simulación-y-recuperación-ante-fallo-grave-en-docker-falta-de-directorio-crítico)
- [Ejercicio 3: Auditoría de seguridad de contenedores Docker desde otro contenedor](#ejercicio-3--auditoría-de-seguridad-de-contenedores-docker-desde-otro-contenedor)

---


# Ejercicio 1: Contenedor SSH seguro con Alpine, solo acceso clave RSA y usuario sudo

## 1. Prepara la clave pública

Supón que tienes un archivo llamado `id_rsa_usuario1.pub` en el raíz del proyecto.  
Si necesitas generarlo:

```bash
ssh-keygen -t rsa -b 4096 -f id_rsa_usuario1
# No pongas passphrase si quieres automatizar login
```

---

## 2. Dockerfile

Crea un archivo `Dockerfile` así: 
(dejar solo "-D" después de las pruebas para eliminar debbug).

```dockerfile
FROM alpine:latest
RUN apk update && apk add --no-cache openssh sudo
RUN echo 'PasswordAuthentication no' >> /etc/ssh/sshd_config && \
    echo 'PermitRootLogin no' >> /etc/ssh/sshd_config && \
    echo 'PubkeyAuthentication yes' >> /etc/ssh/sshd_config && \
    echo 'MaxAuthTries 15' >> /etc/ssh/sshd_config
RUN adduser -D usuario1
RUN passwd -u usuario1
RUN echo 'usuario1 ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
RUN mkdir -p /home/usuario1/.ssh && \
    chown usuario1:usuario1 /home/usuario1/.ssh && \
    chmod 700 /home/usuario1/.ssh
COPY .id_rsa_usuario1.pub /home/usuario1/.ssh/authorized_keys
RUN chown usuario1:usuario1 /home/usuario1/.ssh/authorized_keys && \
    chmod 600 /home/usuario1/.ssh/authorized_keys
RUN ssh-keygen -A
EXPOSE 22
CMD ["/usr/sbin/sshd", "-Ded"]

```

---

## 3. docker-compose.yml

En la raíz del proyecto:

```yaml
services:
  ssh:
    build: .
    container_name: sshseguro
    ports:
      - "10022:22"
    restart: always
```

---

## 4. Estructura del directorio del proyecto

```
proyecto-ssh/
├── Dockerfile
├── docker-compose.yml
└── id_rsa_usuario1.pub
```

---

## 5. Construcción y arranque

Desde el directorio del proyecto:

```bash
docker compose up -d --build
```

---

## 6. Conexión desde cliente

```bash
ssh -p 10022 usuario1@localhost -i id_rsa_usuario1
```

- Solo podrá acceder usuario1 (no root).
- Solo con clave pública.
- El usuario1 puede hacer `sudo` sin contraseña.
- Importante, los usuarios sin password están bloqueados en /etc/shadow!!

---

## 7. Explicación de cada paso

- **Alpine**: imagen ligera.
- **sshd_config**:
  - Puerto cambiado a 10022.
  - Solo clave pública (`PasswordAuthentication no`).
  - `PermitRootLogin no` (prohíbe acceso root).
  - Solo permite usuario1.
- **Clave pública**: se copia desde tu proyecto al contenedor, solo acceso mediante esa clave.
- **Usuario y grupo**: usuario1 es sudoer (`/etc/sudoers` modificado).
- **restart: always** en Compose: el contenedor se reiniciará si falla o si el host se reinicia.

---

## Limpieza

```bash
docker compose down
```


# Ejercicio 2: Simulación y recuperación ante fallo grave en Docker (falta de directorio crítico)

**Planteamiento:**  
Levanta un contenedor. Después, de forma intencionada desde el sistema host, elimina un directorio crítico que impida que Docker funcione (por ejemplo, el directorio de almacenamiento de volúmenes o contenedores). Simula el fallo y explica el proceso para diagnosticar y recuperar la funcionalidad de Docker y los contenedores afectados.

---

## 1. Levanta un contenedor de prueba

```bash
docker run -d --name test_nginx nginx
```

Comprueba que está en ejecución:

```bash
docker ps
```

---

## 2. Simula un fallo grave: elimina un directorio crítico de Docker

**ATENCIÓN: Este paso es destructivo, solo para laboratorio. ¡NO lo hagas en sistemas productivos!**

Por ejemplo, elimina el directorio de volúmenes de Docker (esto impedirá arrancar contenedores con volúmenes y puede afectar otros recursos):

```bash
sudo rm -rf /var/lib/docker/volumes
```

---

## 3. Observa el fallo

Intenta parar o arrancar el contenedor:

```bash
docker stop test_nginx
docker start test_nginx
```

**Verás un error como:**

```
Error response from daemon: open /var/lib/docker/volumes/xxx/_data: no such file or directory
```

O incluso:

```
docker: Error response from daemon: failed to mount local volume: mount /var/lib/docker/volumes/...: no such file or directory.
```

---

## 4. Diagnóstico del fallo

- Comprueba el estado del demonio Docker:
  ```bash
  sudo systemctl status docker
  ```
- Revisa los logs de Docker para encontrar errores específicos:
  ```bash
  sudo journalctl -u docker
  ```
- Observa los errores relativos a volúmenes, capas o datos faltantes.

---

## 5. Proceso de recuperación

**a) Si el directorio eliminado no se puede recuperar de una copia de seguridad:**

- **Crea nuevamente el directorio necesario:**
  ```bash
  sudo mkdir -p /var/lib/docker/volumes
  sudo chown root:root /var/lib/docker/volumes
  sudo chmod 711 /var/lib/docker/volumes
  ```

- **Reinicia el demonio Docker:**
  ```bash
  sudo systemctl restart docker
  ```

- **Nota:** Los contenedores o volúmenes cuyos datos fueron borrados no podrán recuperarse a menos que tengas un backup previo.

**b) Si tienes una copia de seguridad del directorio:**

- **Restaura la copia de seguridad:**
  ```bash
  sudo systemctl stop docker
  sudo tar -xzvf backup_volumes.tar.gz -C /var/lib/docker/
  sudo systemctl start docker
  ```

- **Comprueba el estado de los contenedores y volúmenes:**
  ```bash
  docker ps -a
  ```

---

## 6. Resumen de buenas prácticas

- **Nunca borres directorios internos de Docker sin comprender el impacto.**
- **Haz copias de seguridad periódicas de `/var/lib/docker/volumes` si usas volúmenes persistentes.**
- **Revisa los logs del sistema y de Docker para encontrar la causa real del fallo.**
- **Si el directorio faltante no existe, recrearlo puede permitir arrancar Docker de nuevo, pero los datos previos se habrán perdido.**
- **Para sistemas críticos, considera usar herramientas de backup y snapshots de volúmenes.**

---

## Limpieza

```bash
docker rm -f test_nginx
```

---


# Ejercicio 3.  Auditoría de seguridad de contenedores Docker desde otro contenedor

---

# 1. Auditar imágenes locales con Trivy (desde un contenedor)

Trivy es una herramienta muy usada para detectar vulnerabilidades, paquetes desactualizados y malas configuraciones en imágenes y contenedores Docker.

**Ejemplo:**
```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image nginx:latest
```

---

## 2. Auditar una imagen o contenedor en ejecución

Para escanear una imagen o un contenedor ya en uso (por nombre o ID):

```bash
docker ps  # para ver el nombre o ID
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image nginx_logs
```

---

## 3. Auditar todas las imágenes locales con Trivy

Puedes escanear todas las imágenes almacenadas en tu host:

```bash
docker images --format '{{.Repository}}:{{.Tag}}' | xargs -L1 docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image
```

---

## 4. Auditar buenas prácticas y hardening con Dockle

Dockle detecta problemas como uso de usuario root, falta de HEALTHCHECK, puertos innecesarios, etc.

```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock goodwithtech/dockle:latest nginx:latest
```

---

## 5. Chequeos manuales en contenedores (básicos de seguridad)

Puedes acceder al contenedor y comprobar cosas manualmente:

```bash
docker exec -it <contenedor> bash
# Dentro del contenedor:
id                # Comprobar si corre como root
cat /etc/os-release
apt list --upgradable  # Si es Ubuntu/Debian
netstat -tulnp         # Puertos abiertos
ls -l /root            # Acceso a home de root
ps aux                 # Procesos activos
```
Esto te permite auditar configuraciones, paquetes y permisos.

---

## 6. Recomendaciones y buenas prácticas de auditoría

- Ejecuta auditorías antes y después de desplegar imágenes (shift-left security).
- Integra Trivy, Dockle u otros en tus pipelines de CI/CD.
- Audita también las configuraciones de Docker Compose, secretos y redes.
- No des acceso de escritura al Docker socket a usuarios sin privilegios.
- Centraliza reportes y planifica parches para vulnerabilidades detectadas.

---

## 7. Otros escáneres y métodos recomendados

- **Clair:** Escáner de vulnerabilidades que puede integrarse con registries privados.
- **Snyk, Grype:** Alternativas útiles, se pueden usar en CI o localmente.
- **Análisis manual:** Accede a los contenedores y revisa manualmente configuraciones y software.

---

## 8. Auditoría automatizada y continua

Lo ideal es programar auditorías periódicas con cron, o usarlas como etapa automática en tus pipelines de CI/CD.

**Ejemplo de script simple (trivy todas las imágenes):**
```bash
docker images --format '{{.Repository}}:{{.Tag}}' | xargs -L1 docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image
```

---

> **Recuerda:** La auditoría de seguridad es un proceso continuo, no un evento único. Mantén tus imágenes y contenedores monitorizados, y actualiza siempre ante vulnerabilidades críticas.


