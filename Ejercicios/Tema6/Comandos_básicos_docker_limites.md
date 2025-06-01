 Ejercicios: Limitar y forzar recursos en Docker

---

## Ejercicio 1: Limita la memoria RAM y fuerza OOM (Out-Of-Memory)

**Planteamiento:**  
Lanza un contenedor Ubuntu limitado a 64MB de RAM. Intenta consumir más memoria de la permitida y observa el resultado.

**Desarrollo:**
```bash
docker run --name mem_limit --memory="64m" ubuntu bash -c "apt update && apt install -y stress && stress --vm 2 --vm-bytes 128M --timeout 10"
```

**Comprobación:**
- El proceso dentro del contenedor intentará usar 128MB de RAM.
- Docker terminará (“matando”) el proceso con un error OOM.
- Comprueba el estado y logs:
```bash
docker ps -a | grep mem_limit
docker logs mem_limit
```

**Resolución:**  
El contenedor termina con un error y verás mensajes del kernel/Docker sobre el límite de memoria (“Killed” o “OOM”).

---

## Ejercicio 2: Limita el uso de CPU y fuerza consumo

**Planteamiento:**  
Lanza un contenedor con sólo 0.5 CPU y ejecuta una carga intensiva para comprobar el límite.

**Desarrollo:**
```bash
docker run --name cpu_limit --cpus="0.5" ubuntu bash -c "apt update && apt install -y stress && stress --cpu 2 --timeout 10"
```

**Comprobación:**
- Mientras el contenedor está activo:
```bash
docker stats cpu_limit
```
- Verás que la columna “CPU %” nunca supera el 50% de un núcleo.

**Resolución:**  
El proceso tarda el doble o más en acabar comparado con la ejecución sin límite de CPU.

---

## Ejercicio 3: Limita la cantidad de procesos y fuerza error de fork

**Planteamiento:**  
Lanza un contenedor con un máximo de 10 procesos. Intenta crear más para forzar error.

**Desarrollo:**
```bash
docker run --name pids_limit --pids-limit=10 ubuntu bash -c "apt update && apt install -y stress && stress --fork 20 --timeout 10"
```

**Comprobación:**
- Observa los logs del contenedor:
```bash
docker logs pids_limit
```
- Aparecen errores “fork: Resource temporarily unavailable”.

**Resolución:**  
El contenedor no puede crear más de 10 procesos y los intentos adicionales fallan con error.

---

## Ejercicio 4: Limita archivos abiertos y fuerza error de “too many open files”

**Planteamiento:**  
Lanza un contenedor con un límite de 32 archivos abiertos y usa un script para abrir muchos archivos a la vez.

**Desarrollo:**
Crea un archivo de prueba llamado `test_openfiles.sh`:
```bash
for i in $(seq 1 50); do
  exec 3<> "file_$i.txt"
done
echo "Fin"
```
Lanza el contenedor y ejecuta el script:
```bash
docker run --name nofiles_limit --ulimit nofile=32:32 -v $(pwd)/test_openfiles.sh:/test.sh ubuntu bash -c "apt update && apt install -y bash && bash /test.sh"
```

**Comprobación:**
- El script falla con “Too many open files”.

**Resolución:**  
El límite se ha forzado y sólo se pueden abrir hasta 32 archivos simultáneos.

---

## Ejercicio 5: Limita la velocidad de escritura en disco (I/O) y observa el efecto

**Planteamiento:**  
Limita la escritura en disco a 1MB/s y prueba escribir un archivo grande, midiendo el tiempo.

**Desarrollo:**
> Cambia `/dev/sda` por tu disco real si es necesario.
```bash
docker run --name io_limit --device-write-bps /dev/sda:1mb ubuntu bash -c "dd if=/dev/zero of=/tmp/prueba_io bs=1M count=50 oflag=direct"
```

**Comprobación:**
- El comando `dd` tarda **al menos 50 segundos** en escribir 50MB.
- Sin el límite, sería casi instantáneo.

**Resolución:**  
El tiempo de ejecución refleja el límite aplicado.

---

## Ejercicio 6: Limita la red a 1Mbit/s y comprueba con una descarga

**Planteamiento:**  
Usa un contenedor con acceso root en red, instala `tc` y limita la red a 1Mbit/s, luego descarga un archivo grande y mide la velocidad.

**Desarrollo:**
```bash
docker run --rm -it --cap-add=NET_ADMIN ubuntu bash
# Dentro del contenedor:
apt update && apt install -y iproute2 curl
tc qdisc add dev eth0 root tbf rate 1mbit burst 10kb latency 70ms
curl -o /dev/null http://speedtest.tele2.net/10MB.zip
```

**Comprobación:**
- La descarga tarda unos ~80 segundos (10MB a 1Mbit/s).
- Si repites el test **sin** tc, la descarga será mucho más rápida.

**Resolución:**  
El tiempo de descarga evidencia el límite de red.

---

## Ejercicio 7: Limita RAM y SWAP, y fuerza swap-out

**Planteamiento:**  
Limita un contenedor a 64MB de RAM y 128MB de swap, y ejecuta un proceso que intenta usar más memoria.

**Desarrollo:**
```bash
docker run --name swap_limit --memory="64m" --memory-swap="192m" ubuntu bash -c "apt update && apt install -y stress && stress --vm 2 --vm-bytes 160M --timeout 10"
```

**Comprobación:**
- El proceso sobrevive hasta que el uso conjunto RAM+SWAP llega a 192MB; después es terminado.

**Resolución:**  
El proceso es “matado” cuando se intenta usar más memoria de la permitida (RAM+swap).

---

## Ejercicio 8: Limita CPU a un solo núcleo específico y comprueba afinidad

**Planteamiento:**  
Lanza un contenedor asignado solo al core 0, genera carga y verifica que se usa solo ese núcleo.

**Desarrollo:**
```bash
docker run --name cpu_affinity --cpuset-cpus="0" ubuntu bash -c "apt update && apt install -y stress && stress --cpu 1 --timeout 10"
```

**Comprobación:**
- En el host, observa el uso de CPU con `htop` o `top` y verás que sólo el core 0 sube al 100%.

**Resolución:**  
El contenedor nunca usa otros cores; sólo el core asignado.

