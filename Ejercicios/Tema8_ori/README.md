# Ejercicios avanzados

- [Ejercicio 1: Contenedor SSH seguro con Alpine, solo acceso clave RSA y usuario sudo](#ejercicio-1-contenedor-ssh-seguro-con-alpine-solo-acceso-clave-rsa-y-usuario-sudo)
- [Ejercicio 2: Stack WordPress escalable](#ejercicio-2-stack-wordpress-escalable) 


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

## Estructura del directorio del proyecto

```
proyecto-ssh/
├── Dockerfile
├── docker-compose.yml
└── id_rsa_usuario1.pub
```

##  Explicación de cada paso

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


# Ejercicio 2: Stack WordPress escalable

**Planteamiento:**
Despliega un stack compuesto por MySQL, WordPress (inicialmente 1 réplica, escalable hasta 5), phpMyAdmin y HAProxy como balanceador en el puerto 8080, todo unido por redes personalizadas. HAProxy debe estar preconfigurado para balancear hasta 5 WordPress y la conexión a PHPMyAdmin.

Hay que definir las siguientes redes:

**Pública**: Haproxy  
**Interna**: Haproxy - Wordpress - PHPMyAdmin  
**backend**: Wordpress - DB - PHPMyAdmin  

Hay que definir los siguintes almacenamientos:

**vol-db**: Almacenamiento persistente para los datos de BBDD.  
**vol-web**: Almacenamiento persistente para los datos de Wordpress. Todos los contenedores de wordpress tienen que compartirlo.  

---

