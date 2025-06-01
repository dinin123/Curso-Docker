# Índice de ejercicios

- [Ejercicio 1: Visualiza los logs básicos de un contenedor](#ejercicio-1-visualiza-los-logs-básicos-de-un-contenedor)
- [Ejercicio 2: Sigue los logs en tiempo real y haz una prueba](#ejercicio-2-sigue-los-logs-en-tiempo-real-y-haz-una-prueba)
- [Ejercicio 3: Visualiza solo los errores del contenedor](#ejercicio-3-visualiza-solo-los-errores-del-contenedor)
- [Ejercicio 4: Revisa los logs de arranque fallido](#ejercicio-4-revisa-los-logs-de-arranque-fallido)
- [Ejercicio 5: Obtén logs con marcas de tiempo y líneas recientes](#ejercicio-5-obtén-logs-con-marcas-de-tiempo-y-líneas-recientes)
- [Ejercicio 6: Consulta los logs de un periodo concreto](#ejercicio-6-consulta-los-logs-de-un-periodo-concreto)
- [Ejercicio 7: Visualiza eventos Docker en tiempo real](#ejercicio-7-visualiza-eventos-docker-en-tiempo-real)
- [Ejercicio 8: Filtra eventos solo de un contenedor concreto](#ejercicio-8-filtra-eventos-solo-de-un-contenedor-concreto)
- [Ejercicio 9: Guarda los logs de un contenedor a un archivo para auditoría](#ejercicio-9-guarda-los-logs-de-un-contenedor-a-un-archivo-para-auditoría)
- [Ejercicio 10: Audita los eventos del sistema durante un periodo](#ejercicio-10-audita-los-eventos-del-sistema-durante-un-periodo)
- [Ejercicio 11: Configura y revisa logs en formato JSON](#ejercicio-11-configura-y-revisa-logs-en-formato-json)
- [Ejercicio 12: Redirecciona logs de Docker a syslog (y verifícalo)](#ejercicio-12-redirecciona-logs-de-docker-a-syslog-y-verifícalo)
- [Ejercicio 13: Debug de contenedores con logs de aplicaciones](#ejercicio-13-debug-de-contenedores-con-logs-de-aplicaciones)
- [Ejercicio 14: Audita acceso “exec” en contenedores para detectar intrusiones](#ejercicio-14-audita-acceso-exec-en-contenedores-para-detectar-intrusiones)

---


#Logs y Eventos en Docker

---

## Ejercicio 1: Visualiza los logs básicos de un contenedor

**Planteamiento:**  
Lanza un contenedor NGINX y revisa sus logs.

**Desarrollo:**
```bash
docker run -d --name nginx_logs -p 8080:80 nginx
docker logs nginx_logs
```
**Resolución:**  
Ves las peticiones recibidas y mensajes de acceso del servidor web.

---

## Ejercicio 2: Sigue los logs en tiempo real y haz una prueba

**Planteamiento:**  
Abre los logs en modo seguimiento y haz peticiones con curl para verlas aparecer.

**Desarrollo:**
```bash
docker logs -f nginx_logs
# En otra terminal
curl http://localhost:8080
curl http://localhost:8080/loquesea
```
**Resolución:**  
Cada acceso aparece en tiempo real en la salida de logs.

---

## Ejercicio 3: Visualiza solo los errores del contenedor

**Planteamiento:**  
Lanza un contenedor con una app que imprime errores y filtra solo los mensajes de error.

**Desarrollo:**
```bash
docker run --name error_demo --rm alpine sh -c 'echo OK; echo "ERROR: Falla grave" >&2; sleep 3'
docker logs error_demo 2>&1 | grep ERROR
```
**Resolución:**  
Solo ves las líneas con `ERROR`, útil para detectar fallos.

---

## Ejercicio 4: Revisa los logs de arranque fallido

**Planteamiento:**  
Lanza un contenedor con un comando incorrecto y revisa los logs para ver el error.

**Desarrollo:**
```bash
docker run --name fail_start alpine comando_invalido || true
docker logs fail_start
```
**Resolución:**  
En los logs aparece el mensaje de error: `executable file not found in $PATH`.

---

## Ejercicio 5: Obtén logs con marcas de tiempo y líneas recientes

**Planteamiento:**  
Consulta los últimos 5 eventos de log y muestra marcas de tiempo.

**Desarrollo:**
```bash
docker logs -t --tail 5 nginx_logs
```
**Resolución:**  
Ves solo las últimas 5 líneas de log, cada una con su timestamp.

---

## Ejercicio 6: Consulta los logs de un periodo concreto

**Planteamiento:**  
Obtén logs de los últimos 2 minutos para investigar un incidente reciente.

**Desarrollo:**
```bash
docker logs --since 2m nginx_logs
```
**Resolución:**  
Solo ves los logs generados en los últimos 2 minutos.

---

## Ejercicio 7: Visualiza eventos Docker en tiempo real

**Planteamiento:**  
Sigue los eventos del sistema Docker para ver creación, parada y eliminación de contenedores.

**Desarrollo:**
```bash
docker events
# En otra terminal:
docker run --rm --name test1 alpine echo hola
docker rm -f nginx_logs
```
**Resolución:**  
Ves líneas del tipo `container start`, `container die`, `container destroy`, etc.

---

## Ejercicio 8: Filtra eventos solo de un contenedor concreto

**Planteamiento:**  
Observa los eventos solo del contenedor nginx_logs.

**Desarrollo:**
```bash
docker events --filter container=nginx_logs
```
**Resolución:**  
Solo aparecen eventos relacionados con ese contenedor (start, stop, die, etc).

---

## Ejercicio 9: Guarda los logs de un contenedor a un archivo para auditoría

**Planteamiento:**  
Exporta los logs del contenedor nginx_logs a un fichero local, con timestamp.

**Desarrollo:**
```bash
docker logs nginx_logs > logs_nginx_$(date +%F_%H%M%S).log
```
**Resolución:**  
Tienes un archivo en disco con todos los logs para revisarlos o archivarlos.

---

## Ejercicio 10: Audita los eventos del sistema durante un periodo

**Planteamiento:**  
Guarda todos los eventos Docker ocurridos durante los últimos 5 minutos en un fichero para análisis.

**Desarrollo:**
```bash
docker events --since 5m > eventos_recientes.log
```
**Resolución:**  
El archivo `eventos_recientes.log` contiene una lista detallada de los eventos recientes del daemon Docker.

---

## Ejercicio 11: Configura y revisa logs en formato JSON

**Planteamiento:**  
Cambia el formato de logs de un contenedor a JSON y analiza la salida.

**Desarrollo:**
```bash
docker run -d --name json_logger --log-driver=json-file --log-opt format=json nginx
docker logs json_logger
```
**Resolución:**  
Cada línea del log aparece en formato JSON, útil para análisis automatizado o integración con herramientas externas.

---

## Ejercicio 12: Redirecciona logs de Docker a syslog (y verifícalo)

**Planteamiento:**  
Arranca un contenedor cuyo log se envía a syslog, consulta luego los logs del sistema.

**Desarrollo:**
```bash
docker run -d --name syslog_test --log-driver=syslog nginx
# Verifica en el host:
sudo journalctl -t docker | tail -20
```
**Resolución:**  
En los logs del sistema aparecen mensajes de acceso/errores del contenedor.

---

## Ejercicio 13: Debug de contenedores con logs de aplicaciones

**Planteamiento:**  
Lanza un contenedor de Python con una app que escribe logs a stdout y stderr, observa ambos.

**Desarrollo:**
Crea un archivo llamado `app.py`:
```python
import sys
print("INFO: Esto es un log estándar")
print("ERROR: Esto es un log de error", file=sys.stderr)
```
Ejecuta el contenedor:
```bash
docker run --rm -v $(pwd)/app.py:/app.py python:3 bash -c "python /app.py"
```
**Resolución:**  
Ves ambos tipos de mensajes en el log del contenedor. Puedes filtrar con `grep ERROR` para ver solo errores.

---

## Ejercicio 14: Audita acceso “exec” en contenedores para detectar intrusiones

**Planteamiento:**  
Monitorea si alguien ejecuta comandos dentro de un contenedor mediante `docker exec`.

**Desarrollo:**
```bash
docker events --filter event=exec_create --filter event=exec_start --since 1h
```
En otra terminal, haz:
```bash
docker exec -it nginx_logs bash
```
**Resolución:**  
En los eventos aparecen registros de la creación y ejecución del comando “exec”, lo que permite auditar accesos sospechosos.

---

