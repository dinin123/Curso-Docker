# Módulo 4: Redes en Docker

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
```

**Resolución:**  
Recibes respuesta del servidor HTTP, aunque no mapeaste ningún puerto.

---

## Ejercicio 7: Desconecta y reconecta un contenedor de una red

**Planteamiento:**  
Desconecta un contenedor de una red personalizada y comprueba que deja de responder al ping; luego reconéctalo.

**Desarrollo:**
```bash
docker network disconnect redtest reduno
docker exec reddos ping -c 2 reduno   # Falla
docker network connect redtest reduno
docker exec reddos ping -c 2 reduno   # Funciona
```

**Resolución:**  
El ping solo responde cuando los contenedores están en la misma red.

---

## Ejercicio 8: Crea una red interna (--internal)

**Planteamiento:**  
Crea una red interna y comprueba que los contenedores en esa red no pueden acceder a Internet.

**Desarrollo:**
```bash
docker network create --internal redprivada
docker run -dit --name solo_interna --network redprivada ubuntu bash
docker exec solo_interna apt-get update   # Falla
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
docker run -dit --name multi_net --network redtest ubuntu bash
docker network connect redextra multi_net
docker run -dit --name extra1 --network redextra ubuntu bash
docker exec multi_net apt-get update && docker exec multi_net apt-get install -y iputils-ping
docker exec extra1 apt-get update && docker exec extra1 apt-get install -y iputils-ping
docker exec multi_net ping -c 2 extra1    # Funciona
docker exec multi_net ping -c 2 reddos    # Funciona
```

**Resolución:**  
El contenedor multi_net responde a ping tanto de extra1 como de reddos.

---

## Ejercicio 10: Limpia redes no usadas

**Planteamiento:**  
Elimina todas las redes que no estén en uso por contenedores activos.

**Desarrollo:**
```bash
docker network prune -f
```

**Resolución:**  
Ves qué redes se han eliminado y tu sistema queda más ordenado.

---

## Ejercicio 11: Crea y usa una red Macvlan (ejemplo avanzado)

**Planteamiento:**  
Crea una red macvlan para que un contenedor obtenga una IP real en tu LAN, permitiendo que otros dispositivos de la red accedan directamente al contenedor.

**Nota:**  
Debes saber tu interfaz de red (por ejemplo, `eth0`) y el rango de IPs de tu red local. ¡Esto requiere permisos de administrador y puede interferir con la red si no lo configuras bien!

**Desarrollo:**
```bash
docker network create -d macvlan   --subnet=192.168.1.0/24   --gateway=192.168.1.1   -o parent=eth0 macvlan_demo
```

```bash
docker run -d --name nginx_macvlan --network macvlan_demo --ip 192.168.1.50 nginx
```

Desde otro dispositivo en la misma red, prueba acceder a http://192.168.1.50

**Resolución:**  
La IP 192.168.1.50 es accesible desde la LAN. El contenedor es como "un servidor físico" en la red.

---

## Ejercicio 12: (Opcional) Simula comunicación entre diferentes redes

**Planteamiento:**  
Crea dos redes personalizadas y conecta dos contenedores a ambas, simulando un "router" Docker entre subredes.

**Desarrollo:**
```bash
docker network create redA
docker network create redB
docker run -dit --name nodoA --network redA ubuntu bash
docker run -dit --name nodoB --network redB ubuntu bash
docker run -dit --name router --network redA ubuntu bash
docker network connect redB router
```

Instala ping en todos:
```bash
for c in nodoA nodoB router; do docker exec $c apt-get update && docker exec $c apt-get install -y iputils-ping; done
```

Prueba ping:
```bash
docker exec nodoA ping -c 2 router  # OK
docker exec nodoB ping -c 2 router  # OK
docker exec nodoA ping -c 2 nodoB   # NO responde (a menos que actives forwarding en router)
```

**Resolución:**  
El contenedor router puede hablar con ambas redes; los otros solo con la suya. Así puedes simular topologías complejas.

---

## Limpieza general

```bash
docker rm -f reduno reddos nginxbridge solo_interna multi_net extra1 nodoA nodoB router nginx_macvlan
docker network rm redtest redprivada redextra redA redB macvlan_demo
```

---

> **¡Recuerda!** El ejercicio de macvlan puede necesitar adaptar la IP, el rango y la interfaz a tu entorno real, y requiere permisos de red en el host.
