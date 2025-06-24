# Redes en Docker: Índice de ejercicios

- [Ejercicio 1: Lista redes existentes](#ejercicio-1-lista-redes-existentes)
- [Ejercicio 2: Inspecciona una red](#ejercicio-2-inspecciona-una-red)
- [Ejercicio 3: Conectividad entre contenedores en red por defecto y red personalizada](#ejercicio-3-conectividad-entre-contenedores-en-red-por-defecto-y-red-personalizada)
- [Ejercicio 4: Mapea un puerto y accede desde el host](#ejercicio-4-mapea-un-puerto-y-accede-desde-el-host)
- [Ejercicio 5: Usa --network none para máxima seguridad](#ejercicio-5-usa---network-none-para-máxima-seguridad)
- [Ejercicio 6: Prueba la red host](#ejercicio-6-prueba-la-red-host)
- [Ejercicio 7: Desconecta y reconecta un contenedor de una red](#ejercicio-7-desconecta-y-reconecta-un-contenedor-de-una-red)
- [Ejercicio 8: Crea una red interna (--internal)](#ejercicio-8-crea-una-red-interna---internal)
- [Ejercicio 9: Usa varias redes en un mismo contenedor](#ejercicio-9-usa-varias-redes-en-un-mismo-contenedor)
- [Limpieza general](#limpieza-general)

---

## Ejercicio 1: Lista redes existentes

**Planteamiento:**  
Lista todas las redes existentes en tu sistema Docker y observa cuáles son predeterminadas.

**Desarrollo:**
```bash
docker network ls
```

**Resolución:**  
Ves una lista de redes, normalmente: bridge, host, none, y otras que hayas creado.

---

## Ejercicio 2: Inspecciona una red

**Planteamiento:**  
Inspecciona la red bridge y observa qué información muestra Docker.

**Desarrollo:**
```bash
docker network inspect bridge
```

**Resolución:**  
Ves un JSON con detalles como: ID, nombre, tipo de driver, subred, puerta de enlace, contenedores conectados.

---


## Ejercicio 3: Conectividad entre contenedores en red por defecto y red personalizada

**Planteamiento:**  
Comprueba si los contenedores pueden comunicarse entre sí usando el nombre de contenedor en la red por defecto (`bridge`) y en una red personalizada.

---

### 1. Prueba en la red por defecto (`bridge`)

**Desarrollo:**
```bash
docker run -dit --name defecto1 ubuntu bash
docker run -dit --name defecto2 ubuntu bash
docker exec defecto1 apt-get update && docker exec defecto1 apt-get install -y iputils-ping
docker exec defecto2 apt-get update && docker exec defecto2 apt-get install -y iputils-ping
```

**Desde defecto1 intenta hacer ping por nombre a defecto2:**
```bash
docker exec defecto1 ping -c 2 defecto2
```

**Desde defecto1, haz ping por IP a defecto2:**
1. Descubre la IP de defecto2:
    ```bash
    docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' defecto2
    ```
2. Haz ping a esa IP:
    ```bash
    docker exec defecto1 ping -c 2 <IP_DE_DEFECTO2>
    ```

**Resolución:**  
- **Ping por nombre:**  
  Falla (`ping: defecto2: Name or service not known`).
- **Ping por IP:**  
  Funciona (hay respuesta).

---

### 2. Prueba en una red personalizada

**Desarrollo:**
```bash
docker network create redtest
docker run -dit --name personalizado1 --network redtest ubuntu bash
docker run -dit --name personalizado2 --network redtest ubuntu bash
docker exec personalizado1 apt-get update && docker exec personalizado1 apt-get install -y iputils-ping
docker exec personalizado2 apt-get update && docker exec personalizado2 apt-get install -y iputils-ping
```

**Desde personalizado1, ping por nombre a personalizado2:**
```bash
docker exec personalizado1 ping -c 2 personalizado2
```

**Desde personalizado1, ping por ip a personalizado2:**

1. Descubre la IP de personalizado2:
    ```bash
    docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' personalizado2
    ```
2. Haz ping a esa IP:
    ```bash
    docker exec personalizado1 ping -c 2 <IP_DE_PERSONALIZADO2>
    ```


**Resolución:**  
- **Ping por nombre:**  
  Funciona, los contenedores se resuelven correctamente por nombre.
- **Ping por IP:**  
  También funciona.

---

**Conclusión:**  
- **Red por defecto (`bridge`)**: los contenedores NO pueden resolverse por nombre, solo por IP.
- **Red personalizada:** los contenedores SÍ pueden resolverse por nombre, facilitando la comunicación entre servicios.


## Ejercicio 4: Mapea un puerto y accede desde el host

**Planteamiento:**  
Lanza un NGINX en la red bridge por defecto, mapeando el puerto 8081 del host al 80 del contenedor. Accede desde tu navegador.

**Desarrollo:**
```bash
docker run -d --name nginxbridge -p 8081:80 nginx
```

**Resolución:**  
Entra a http://localhost:8081 y ves la página de NGINX.

---

## Ejercicio 5: Usa --network none para máxima seguridad

**Planteamiento:**  
Lanza un contenedor Ubuntu sin acceso a ninguna red y comprueba que no puede conectarse a Internet.

**Desarrollo:**
```bash
docker run -it --network none ubuntu bash
# Dentro:
apt-get update   # Falla
ping 8.8.8.8     # Falla
exit
```

**Resolución:**  
El contenedor no puede acceder a ninguna red, ni siquiera al host.

---

## Ejercicio 6: Prueba la red host

**Planteamiento:**  
Ejecuta un contenedor Python con un servidor HTTP en la red host. Accede al puerto desde el host sin usar -p.

**Desarrollo:**
```bash
docker run --rm --network host -d python:3 python -m http.server 9000
curl http://localhost:9000

<!DOCTYPE HTML>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href=".dockerenv">.dockerenv</a></li>
<li><a href="bin/">bin@</a></li>
<li><a href="boot/">boot/</a></li>
<li><a href="dev/">dev/</a></li>
<li><a href="etc/">etc/</a></li>
<li><a href="home/">home/</a></li>
<li><a href="lib/">lib@</a></li>
<li><a href="lib64/">lib64@</a></li>
<li><a href="media/">media/</a></li>
<li><a href="mnt/">mnt/</a></li>
<li><a href="opt/">opt/</a></li>
<li><a href="proc/">proc/</a></li>
<li><a href="root/">root/</a></li>
<li><a href="run/">run/</a></li>
<li><a href="sbin/">sbin@</a></li>
<li><a href="srv/">srv/</a></li>
<li><a href="sys/">sys/</a></li>
<li><a href="tmp/">tmp/</a></li>
<li><a href="usr/">usr/</a></li>
<li><a href="var/">var/</a></li>
</ul>
<hr>
</body>
</html>


```

**Resolución:**  
Recibes respuesta del servidor HTTP, aunque no mapeaste ningún puerto.

---

## Ejercicio 7: Desconecta y reconecta un contenedor de una red

**Planteamiento:**  
Desconecta un contenedor de una red personalizada y comprueba que deja de responder al ping; luego reconéctalo.

**Desarrollo:**
```bash
docker network disconnect redtest personalizado2
docker exec personalizado1 ping -c 2 personalizado2   # Falla
docker network connect redtest personalizado2
docker exec personalizado1 ping -c 2 personalizado2  # Funciona
```

**Resolución:**  
El ping solo responde cuando los contenedores están en la misma red. Al desconectar el contenedor, deja de tener conectividad. 

---

## Ejercicio 8: Crea una red interna (--internal)

**Planteamiento:**  
Crea una red interna y comprueba que los contenedores en esa red no pueden acceder a Internet.

**Desarrollo:**
```bash
docker network create --internal redprivada
docker run -dit --name solo_interna --network redprivada ubuntu bash
docker exec solo_interna apt-get update   # Falla

docker exec solo_interna apt-get update
Ign:1 http://security.ubuntu.com/ubuntu noble-security InRelease
Ign:2 http://archive.ubuntu.com/ubuntu noble InRelease
Ign:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Ign:4 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Ign:2 http://archive.ubuntu.com/ubuntu noble InRelease
Ign:1 http://security.ubuntu.com/ubuntu noble-security InRelease
Ign:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Ign:4 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Ign:2 http://archive.ubuntu.com/ubuntu noble InRelease
Ign:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Ign:1 http://security.ubuntu.com/ubuntu noble-security InRelease
Ign:4 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Err:2 http://archive.ubuntu.com/ubuntu noble InRelease
  Temporary failure resolving 'archive.ubuntu.com'
Err:1 http://security.ubuntu.com/ubuntu noble-security InRelease
  Temporary failure resolving 'security.ubuntu.com'
Err:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease
  Temporary failure resolving 'archive.ubuntu.com'
Err:4 http://archive.ubuntu.com/ubuntu noble-backports InRelease
  Temporary failure resolving 'archive.ubuntu.com'
Reading package lists...
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/noble/InRelease  Temporary failure resolving 'archive.ubuntu.com'
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/noble-updates/InRelease  Temporary failure resolving 'archive.ubuntu.com'
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/noble-backports/InRelease  Temporary failure resolving 'archive.ubuntu.com'
W: Failed to fetch http://security.ubuntu.com/ubuntu/dists/noble-security/InRelease  Temporary failure resolving 'security.ubuntu.com'
W: Some index files failed to download. They have been ignored, or old ones used instead.

```

**Resolución:**  
El contenedor no puede hacer update ni acceder fuera de la red.

---

## Ejercicio 9: Usa varias redes en un mismo contenedor

**Planteamiento:**  
Conecta un contenedor a dos redes distintas y verifica que puede comunicarse con contenedores de ambas.

**Desarrollo:**
```bash
docker network create redextra
docker network create redextra2
docker run -dit --name multi_net --network redtest ubuntu bash
docker network connect redextra multi_net
docker network connect redextra2 multi_net
docker run -dit --name extra1 --network redextra ubuntu bash
docker run -dit --name extra2 --network redextra2 ubuntu bash
docker exec multi_net apt-get update && docker exec multi_net apt-get install -y iputils-ping
docker exec extra1 apt-get update && docker exec extra1 apt-get install -y iputils-ping
docker exec extra2 apt-get update && docker exec extra1 apt-get install -y iputils-ping
docker exec multi_net ping -c 2 extra1    # Funciona
docker exec multi_net ping -c 2 extra2    # Funciona
```

**Resolución:**  
El contenedor multi_net responde a ping tanto de extra1 como de extra2.

---


## Limpieza general

Para y elimina todos los conternedores:

```bash
docker stop $(docker ps -q)
docker rm $(docker ps -a -q)
```

Elimina todas las redes:

```bash
docker network rm redextra redextra2 redprivada redtest
```

---

