# Índice de ejercicios

- [Ejercicio 1: Crea y usa un volumen](#ejercicio-1-crea-y-usa-un-volumen)
- [Ejercicio 2: Persiste datos entre contenedores](#ejercicio-2-persiste-datos-entre-contenedores)
- [Ejercicio 3: Usa bind mounts para compartir carpetas del host](#ejercicio-3-usa-bind-mounts-para-compartir-carpetas-del-host)
- [Ejercicio 4: Usa un volumen para una base de datos simple (SQLite)](#ejercicio-4-usa-un-volumen-para-una-base-de-datos-simple-sqlite)
- [Ejercicio 5: Crea un tmpfs mount y observa qué ocurre](#ejercicio-5-crea-un-tmpfs-mount-y-observa-qué-ocurre)
- [Ejercicio 6: Backup de un volumen Docker](#ejercicio-6-backup-de-un-volumen-docker)
- [Ejercicio 7: Restaura un volumen desde un backup](#ejercicio-7-restaura-un-volumen-desde-un-backup)
- [Ejercicio 8: Limpia volúmenes no usados](#ejercicio-8-limpia-volúmenes-no-usados)
- [Ejercicio 9: Lista y revisa detalles de volúmenes](#ejercicio-9-lista-y-revisa-detalles-de-volúmenes)
- [Ejercicio 10: Usa varios volúmenes en un mismo contenedor](#ejercicio-10-usa-varios-volúmenes-en-un-mismo-contenedor)
- [Ejercicio 11: Compara rendimiento de distintos tipos de volúmenes en Docker](#ejercicio-11-compara-rendimiento-de-distintos-tipos-de-volúmenes-en-docker)
- [Limpieza general](#limpieza-general)

---


# Volúmenes y Almacenamiento

---

## Ejercicio 1: Crea y usa un volumen 

**Planteamiento:**  
Crea un volumen llamado `volumen_prueba`, úsalo en un contenedor Ubuntu y comprueba que puedes guardar datos en él.

**Desarrollo:**
```bash
docker volume create volumen_prueba
docker run -it --name cont_vol1 -v volumen_prueba:/datos ubuntu bash
```

**Dentro del contenedor:**
```bash
echo "Datos de prueba" > /datos/prueba.txt
exit
```

**Resolución:**  
El archivo `/datos/prueba.txt` se guarda en el volumen y persiste aunque el contenedor se elimine.

---

## Ejercicio 2: Persiste datos entre contenedores

**Planteamiento:**  
Comprueba que los datos del volumen siguen existiendo cuando borras y creas un nuevo contenedor con ese volumen.

**Desarrollo:**
```bash
docker rm cont_vol1
docker run -it --name cont_vol2 -v volumen_prueba:/datos ubuntu bash
```

**Dentro del contenedor:**
```bash
cat /datos/prueba.txt
exit
```

**Resolución:**  
Ves el texto "Datos de prueba", demostrando que los datos persisten.

---

## Ejercicio 3: Usa bind mounts para compartir carpetas del host

**Planteamiento:**  
Monta una carpeta de tu host (ej. tu escritorio) dentro de un contenedor y crea un archivo desde el contenedor.

**Desarrollo:**
```bash
mkdir ~/docker_bind_test
docker run -it -v ~/docker_bind_test:/hostdata ubuntu bash
```

**Dentro del contenedor:**
```bash
echo "Archivo creado desde el contenedor" > /hostdata/desde_contenedor.txt
exit
```

**Resolución:**  
El archivo `desde_contenedor.txt` aparece en tu carpeta del host.

---

## Ejercicio 4: Usa un volumen para una base de datos simple (SQLite)

**Planteamiento:**  
Crea un contenedor que guarde una base de datos SQLite en un volumen, elimina el contenedor y verifica que el archivo sigue allí.

**Desarrollo:**
```bash
docker volume create sqlite_vol
docker run -it --name sqlite_test -v sqlite_vol:/db python:3 bash
```

**Dentro:**
```bash
python3 -c "import sqlite3; sqlite3.connect('/db/ejemplo.db').execute('CREATE TABLE t(id INT)')"
exit
```

```bash
docker rm sqlite_test
docker run -it --name sqlite_test2 -v sqlite_vol:/db python:3 bash
ls /db # Debe aparecer ejemplo.db
exit
```

**Resolución:**  
El archivo `ejemplo.db` está en el volumen, la BD persiste aunque borres el contenedor.

---

## Ejercicio 5: Crea un tmpfs mount y observa qué ocurre

**Planteamiento:**  
Lanza un contenedor con un tmpfs en `/tmp`, guarda un archivo, elimina el contenedor y observa si el archivo sigue.

**Desarrollo:**
```bash
docker run -it --tmpfs /tmp:size=16m ubuntu bash
```

**Dentro:**
```bash
echo "Temporal RAM" > /tmp/tmpfile.txt
exit
```

```bash
docker run -it --tmpfs /tmp:size=16m ubuntu bash
ls /tmp # No está tmpfile.txt
exit
```

**Resolución:**  
Los archivos en tmpfs se borran cuando el contenedor termina.

---

## Ejercicio 6: Backup de un volumen Docker

**Planteamiento:**  
Haz backup del volumen `volumen_prueba` a un archivo .tar.gz en tu máquina.

**Desarrollo:**
```bash
docker run --rm -v volumen_prueba:/data -v $(pwd):/backup alpine tar czvf /backup/volumen_prueba_backup.tar.gz -C /data .
```

**Resolución:**  
Tienes el archivo `volumen_prueba_backup.tar.gz` en tu carpeta actual.

---

## Ejercicio 7: Restaura un volumen desde un backup

**Planteamiento:**  
Elimina y vuelve a crear el volumen, luego restaura los datos usando el archivo .tar.gz.

**Desarrollo:**
```bash
docker volume rm volumen_prueba
docker volume create volumen_prueba
docker run --rm -v volumen_prueba:/data -v $(pwd):/backup alpine tar xzvf /backup/volumen_prueba_backup.tar.gz -C /data
docker run -it --rm -v volumen_prueba:/data ubuntu cat /data/prueba.txt
```

**Resolución:**  
Recuperas el archivo original. El backup y restore funcionan.

---

## Ejercicio 8: Limpia volúmenes no usados

**Planteamiento:**  
Elimina todos los volúmenes que no están siendo utilizados por ningún contenedor.

**Desarrollo:**
```bash
docker volume prune -f
```

**Resolución:**  
Ves un resumen de volúmenes eliminados y tu disco queda más limpio.

---

## Ejercicio 9: Lista y revisa detalles de volúmenes

**Planteamiento:**  
Lista los volúmenes y revisa el punto de montaje y otros detalles de un volumen.

**Desarrollo:**
```bash
docker volume ls
docker volume inspect volumen_prueba
```

**Resolución:**  
Ves información como el nombre, la ruta en el host y la fecha de creación.

---

## Ejercicio 10: Usa varios volúmenes en un mismo contenedor

**Planteamiento:**  
Arranca un contenedor con dos volúmenes diferentes y guarda un archivo en cada uno.

**Desarrollo:**
```bash
docker volume create vol1
docker volume create vol2
docker run -it --name multi_vol -v vol1:/data1 -v vol2:/data2 ubuntu bash
```

**Dentro:**
```bash
echo "Hola vol1" > /data1/uno.txt
echo "Hola vol2" > /data2/dos.txt
exit
```

```bash
docker rm multi_vol
docker run -it --rm -v vol1:/data1 ubuntu cat /data1/uno.txt
docker run -it --rm -v vol2:/data2 ubuntu cat /data2/dos.txt
```

**Resolución:**  
Cada archivo está en su volumen correspondiente y persiste de forma independiente.

---

## Ejercicio 11: Compara rendimiento de distintos tipos de volúmenes en Docker

**Planteamiento:**  
Crea y monta en contenedores los tres tipos de volumen de Docker (named volume, bind mount y tmpfs). Realiza un test de escritura en cada uno para comparar velocidades.

**Desarrollo:**

### 1. Crea las rutas necesarias y el archivo de test

```bash
mkdir -p ~/docker_bench
echo "Benchmark de volúmenes Docker" > ~/docker_bench/prueba.txt
```

### 2. Prueba con **named volume** (volumen Docker gestionado)

```bash
docker volume create test_named
docker run --rm -v test_named:/testdir ubuntu bash -c "dd if=/dev/zero of=/testdir/speed_test bs=1M count=512 oflag=dsync"
docker volume rm test_named
```

### 3. Prueba con **bind mount** (carpeta del host)

```bash
docker run --rm -v ~/docker_bench:/testdir ubuntu bash -c "dd if=/dev/zero of=/testdir/speed_test bs=1M count=512 oflag=dsync"
```

### 4. Prueba con **tmpfs** (RAM)

```bash
docker run --rm --tmpfs /testdir:rw,size=600m ubuntu bash -c "dd if=/dev/zero of=/testdir/speed_test bs=1M count=512 oflag=dsync"
```

### 5. Interpreta los resultados

Observa el tiempo total reportado por el comando `dd` (última línea, algo como “x MB/s”).  
Suele observarse:
- **tmpfs** (RAM): velocidad máxima.
- **named volume**: muy rápido, dependiente de Docker y el sistema de archivos.
- **bind mount**: similar o algo más lento, depende del disco y del sistema anfitrión.

**Resolución:**  
Registra los resultados de cada test para comparar.  
Ejemplo de salida esperada:

```
536870912 bytes (537 MB, 512 MiB) copied, 0.692443 s, 775 MB/s  # tmpfs (muy rápido)
536870912 bytes (537 MB, 512 MiB) copied, 2.40977 s, 223 MB/s   # named volume
536870912 bytes (537 MB, 512 MiB) copied, 3.10784 s, 172 MB/s   # bind mount
```

> Nota: Los resultados dependen del hardware y la carga del host.


## Limpieza general

```bash
docker rm -f cont_vol2 sqlite_test2 multi_vol
docker volume rm vol1 vol2 sqlite_vol volumen_prueba
rm -rf ~/docker_bind_test
rm volumen_prueba_backup.tar.gz
```

