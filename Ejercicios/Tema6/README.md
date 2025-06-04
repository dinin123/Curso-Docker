# Limitar recursos en Docker: Índice de ejercicios

- [Ejercicio 1: Limita la memoria RAM y fuerza OOM (Out-Of-Memory)](#ejercicio-1-limita-la-memoria-ram-y-fuerza-oom-out-of-memory)
- [Ejercicio 2: Limita el uso de CPU y fuerza consumo](#ejercicio-2-limita-el-uso-de-cpu-y-fuerza-consumo)
- [Ejercicio 3: Limita la cantidad de procesos y fuerza error de fork](#ejercicio-3-limita-la-cantidad-de-procesos-y-fuerza-error-de-fork)
- [Ejercicio 4: Limita la velocidad de escritura en disco (IO) y observa el efecto](#ejercicio-5-limita-la-velocidad-de-escritura-en-disco-io-y-observa-el-efecto)
- [Ejercicio 5: Limita RAM y SWAP, y fuerza swap-out](#ejercicio-7-limita-ram-y-swap-y-fuerza-swap-out)
- [Ejercicio 6: Limita CPU a un solo núcleo específico y comprueba afinidad](#ejercicio-8-limita-cpu-a-un-solo-núcleo-específico-y-comprueba-afinidad)

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
docker run --name pids_limit --pids-limit=10 ubuntu bash -c "apt update && apt install -y stress-ng && stress-ng --fork 20 --timeout 100"
```

**Comprobación:**
- Observa los logs del contenedor:
```bash
docker logs pids_limit
```
- Se ve a stress-ng que solo puede abrir X procesos y va mátando otros:

```bash
stress-ng: info:  [1] dispatching hogs: 30 fork
stress-ng: debug: [1] starting stressors
stress-ng: debug: [577] fork: [577] started (instance 0 on CPU 1)
stress-ng: debug: [578] fork: [578] started (instance 1 on CPU 1)
stress-ng: debug: [580] fork: [580] started (instance 3 on CPU 1)
stress-ng: debug: [582] fork: [582] started (instance 4 on CPU 1)
stress-ng: debug: [583] fork: [583] started (instance 5 on CPU 1)
stress-ng: debug: [585] fork: [585] started (instance 6 on CPU 0)
stress-ng: debug: [579] fork: [579] started (instance 2 on CPU 0)
stress-ng: debug: [5272] fork: [5272] started (instance 7 on CPU 1)
stress-ng: debug: [9438] fork: [9438] started (instance 8 on CPU 0)
stress-ng: debug: [580] fork: [580] exited (instance 3 on CPU 1)
stress-ng: debug: [582] fork: [582] exited (instance 4 on CPU 0)
stress-ng: debug: [577] fork: [577] exited (instance 0 on CPU 1)
stress-ng: debug: [578] fork: [578] exited (instance 1 on CPU 1)
stress-ng: debug: [585] fork: [585] exited (instance 6 on CPU 0)
```

- Podemos ver el límite aplicado a 

```cat
docker exec pids_limit cat /sys/fs/cgroup/pids.max
10
```


**Resolución:**  
El contenedor no puede crear más de 10 procesos.

---

---

## Ejercicio 4: Limita la velocidad de escritura en disco (I/O) y observa el efecto

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

## Ejercicio 5: Limita RAM y SWAP, y fuerza swap-out

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

## Ejercicio 6: Limita CPU a un solo núcleo específico y comprueba afinidad

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




