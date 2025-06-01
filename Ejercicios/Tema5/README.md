 Módulo 6: Administración avanzada y solución de problemas

---

## Ejercicio 1: Limita recursos de CPU y memoria en un contenedor

**Planteamiento:**  
Lanza un contenedor de NGINX con límite de 256MB de RAM y 0.5 CPUs.

**Desarrollo:**
```bash
docker run -d --name nginx_limitado --memory="256m" --cpus="0.5" nginx
docker stats nginx_limitado
```

**Resolución:**  
Verifica que el uso de RAM y CPU nunca supera los límites asignados.

---

## Ejercicio 2: Cambia la política de reinicio de un contenedor

**Planteamiento:**  
Haz que un contenedor de NGINX se reinicie siempre que falle o al reiniciar el host.

**Desarrollo:**
```bash
docker run -d --name nginx_auto --restart=always nginx
docker ps -a --filter "name=nginx_auto"
```

**Resolución:**  
La columna STATUS muestra si el contenedor se reinicia tras pararlo o reiniciar Docker.

---

## Ejercicio 3: Visualiza logs en tiempo real y gestiona la salida

**Planteamiento:**  
Lanza un contenedor que escriba la fecha en un bucle y observa sus logs en directo.

**Desarrollo:**
```bash
docker run -d --name reloj busybox sh -c "while true; do date; sleep 1; done"
docker logs -f reloj
```

**Resolución:**  
Ves la hora actualizándose cada segundo.

---

## Ejercicio 4: Configura el driver de logs y comprueba la rotación

**Planteamiento:**  
Lanza un contenedor con el driver de logs json-file y límite de tamaño de log a 1MB, 2 archivos rotativos.

**Desarrollo:**
```bash
docker run -d --name logs_limited --log-opt max-size=1m --log-opt max-file=2 busybox sh -c "while true; do echo 'Log rotando...'; sleep 0.01; done"
# Espera 1-2 minutos y para el contenedor:
docker stop logs_limited
docker logs logs_limited
```

**Resolución:**  
El log nunca ocupa más de 2MB en total. Los archivos rotan automáticamente.

---

## Ejercicio 5: Realiza y restaura un backup de volumen

**Planteamiento:**  
Haz backup de un volumen, bórralo y recupéralo usando Alpine y tar.

**Desarrollo:**
```bash
docker volume create backupvol
docker run --rm -v backupvol:/data ubuntu bash -c "echo 'backup ok' > /data/info.txt"
docker run --rm -v backupvol:/data -v $(pwd):/backup alpine tar czvf /backup/backupvol.tar.gz -C /data .
docker volume rm backupvol
docker volume create backupvol
docker run --rm -v backupvol:/data -v $(pwd):/backup alpine tar xzvf /backup/backupvol.tar.gz -C /data
docker run --rm -v backupvol:/data ubuntu cat /data/info.txt
```

**Resolución:**  
Recuperas el archivo info.txt con el mensaje original.

---

## Ejercicio 6: Usa healthchecks para controlar el estado

**Planteamiento:**  
Configura un healthcheck en un contenedor MySQL y observa su estado de salud.

**Desarrollo:**
```bash
docker run -d --name mysql_hc -e MYSQL_ROOT_PASSWORD=1234 --health-cmd='mysqladmin ping -h localhost -p1234' --health-interval=10s --health-retries=3 mysql
docker inspect --format='{{json .State.Health}}' mysql_hc
```

**Resolución:**  
El estado Health pasa de starting a healthy cuando el servidor MySQL responde.

---

## Ejercicio 7: Monitoriza recursos en tiempo real

**Planteamiento:**  
Lanza dos contenedores y observa su consumo con docker stats.

**Desarrollo:**
```bash
docker run -d --name stress1 busybox sh -c "while true; do true; done"
docker run -d --name stress2 busybox sh -c "while true; do true; done"
docker stats
```

**Resolución:**  
Ves las columnas de CPU, memoria, red y disco para ambos contenedores.

---

## Ejercicio 8: Inspecciona eventos de Docker en vivo

**Planteamiento:**  
Observa en directo los eventos del demonio Docker al arrancar y parar contenedores.

**Desarrollo:**
```bash
docker events
# (En otra terminal, ejecuta contenedores o deténlos para ver los eventos generados)
```

**Resolución:**  
Ves líneas en vivo del tipo: container start, container stop, etc.

---

## Ejercicio 9: Diagnostica un error de imagen inexistente

**Planteamiento:**  
Intenta lanzar un contenedor con una imagen que no existe y analiza el error.

**Desarrollo:**
```bash
docker run --name failme noexiste:1.0
docker ps -a --filter "name=failme"
docker logs failme
```

**Resolución:**  
docker logs failme muestra un error de “manifest for noexiste:1.0 not found”.

---

## Ejercicio 10: Limpia todos los recursos no utilizados

**Planteamiento:**  
Libera espacio eliminando contenedores, redes, imágenes y volúmenes que no se usen.

**Desarrollo:**
```bash
docker system prune -af --volumes
```

**Resolución:**  
El comando muestra cuánto espacio se ha liberado y el sistema Docker queda limpio.

---

## Ejercicio 11: Inspecciona la configuración de una red y de un contenedor

**Planteamiento:**  
Consulta toda la información de red, volúmenes y configuración de un contenedor en formato JSON.

**Desarrollo:**
```bash
docker inspect nginx_limitado
```

**Resolución:**  
Ves la configuración completa, incluyendo redes conectadas, puertos, rutas, mounts y mucho más.

---

## Ejercicio 12: Escanea vulnerabilidades en imágenes Docker

**Planteamiento:**  
Analiza una imagen Docker en busca de vulnerabilidades usando Trivy (requiere instalación previa).

**Desarrollo:**
```bash
sudo dnf install trivy     # O el gestor correspondiente
trivy image nginx
```

**Resolución:**  
Obtienes un listado de posibles vulnerabilidades de paquetes y configuraciones de la imagen.

---

## Ejercicio 13: Comprueba la rotación de logs del daemon Docker con logrotate

**Planteamiento:**  
Asegura que los logs de Docker se roten automáticamente para no llenar el disco.

**Desarrollo:**
1. Comprueba el archivo `/etc/logrotate.d/docker-container`
2. Fuerza rotación:
```bash
sudo logrotate -f /etc/logrotate.d/docker-container
```
3. Verifica que aparecen archivos `.gz` de logs antiguos.

**Resolución:**  
Los logs rotan, se guardan como archivos comprimidos y no saturan el disco.

---

## Ejercicio 14: Debug avanzado de procesos dentro de un contenedor (strace)

**Planteamiento:**  
Analiza las llamadas a sistema de un proceso dentro de un contenedor con `strace` para troubleshooting avanzado.

**Desarrollo:**
```bash
docker run -it --name debug_test ubuntu bash
apt-get update && apt-get install -y strace
strace ls /
exit
docker rm debug_test
```

**Resolución:**  
Ves todas las llamadas al sistema que hace el comando `ls /`.

---

## Ejercicio 15: Limpieza general

```bash
docker rm -f nginx_limitado nginx_auto reloj logs_limited mysql_hc stress1 stress2 failme debug_test
docker volume rm backupvol
```

