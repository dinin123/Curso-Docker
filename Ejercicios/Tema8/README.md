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

```dockerfile
FROM alpine:latest

RUN apk update &&     apk add --no-cache openssh sudo

# Crea usuario usuario1 y lo agrega a sudoers
RUN adduser -D usuario1 &&     echo "usuario1 ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Prepara claves y directorios
RUN mkdir -p /home/usuario1/.ssh
COPY id_rsa_usuario1.pub /home/usuario1/.ssh/authorized_keys
RUN chown -R usuario1:usuario1 /home/usuario1/.ssh &&     chmod 700 /home/usuario1/.ssh &&     chmod 600 /home/usuario1/.ssh/authorized_keys

# Configuración SSHD: solo clave pública, sin root login
RUN sed -i 's/#Port 22/Port 10022/' /etc/ssh/sshd_config &&     sed -i 's/#PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config &&     sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config &&     echo "PubkeyAuthentication yes" >> /etc/ssh/sshd_config &&     echo "AllowUsers usuario1" >> /etc/ssh/sshd_config

# Arranca el servicio SSH
EXPOSE 10022
CMD ["/usr/sbin/sshd", "-D"]
```

---

## 3. docker-compose.yml

En la raíz del proyecto:

```yaml
version: "3.8"
services:
  ssh:
    build: .
    container_name: sshseguro
    ports:
      - "10022:10022"
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

---



