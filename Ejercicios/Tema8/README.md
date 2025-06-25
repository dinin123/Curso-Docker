# Ejercicios avanzados

- [Ejercicio 1: Contenedor SSH seguro con Alpine, solo acceso clave RSA y usuario sudo](#ejercicio-1-contenedor-ssh-seguro-con-alpine-solo-acceso-clave-rsa-y-usuario-sudo)


---


# Ejercicio 1: Contenedor SSH seguro con Alpine, solo acceso clave RSA y usuario sudo

Otra alternativa,con chroot, de un proyecto de Github: [SSH con CHROOT](https://github.com/masonarchhsieh/chroot-jail-for-ssh)

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
(dejar solo "-D" después de las pruebas para eliminar mensajes de depuración).

```dockerfile
FROM alpine:latest
RUN apk update && apk add --no-cache openssh sudo
RUN echo 'PasswordAuthentication no' >> /etc/ssh/sshd_config && \
    echo 'PermitRootLogin no' >> /etc/ssh/sshd_config && \
    echo 'PubkeyAuthentication yes' >> /etc/ssh/sshd_config && \
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

- Solo podrá acceder usuario1 (no root).
- Solo con clave pública.
- El usuario1 puede hacer `sudo` sin contraseña.
- Importante, los usuarios sin password están bloqueados por defecto en las imágenes de Alpine!!

---

## 7. Explicación de cada paso

- **Alpine**: imagen ligera.
- **sshd_config**:
  - Solo clave pública (`PasswordAuthentication no`).
  - `PermitRootLogin no` (prohíbe acceso root).
- **Clave pública**: se copia desde tu proyecto al contenedor, solo acceso mediante esa clave.
- **Usuario y grupo**: usuario1 es sudoer (`/etc/sudoers` modificado).

