# Módulo 3: Volúmenes y Almacenamiento

---

## Ejercicio 1: Crea y usa un volumen nombrado

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

## Limpieza general

```bash
docker rm -f cont_vol2 sqlite_test2 multi_vol
docker volume rm vol1 vol2 sqlite_vol volumen_prueba
rm -rf ~/docker_bind_test
rm volumen_prueba_backup.tar.gz
```

