# Laboratorio Práctico: Auditoría de Seguridad en Docker

## Objetivo
Auditar una aplicación web sencilla en Docker usando herramientas de análisis estático, escaneo de vulnerabilidades y monitoreo en tiempo real.

---

## Paso 1: Crear una aplicación vulnerable de ejemplo

```bash
git clone https://github.com/darinpope/vulnerable-docker.git
cd vulnerable-docker
```

> Este repositorio contiene un Dockerfile vulnerable y un servidor web simple con errores comunes.

---

## Paso 2: Auditar el Dockerfile con Hadolint

### Instalar Hadolint (Debian/Ubuntu)

```bash
sudo apt install hadolint -y
```

### Ejecutar auditoría

```bash
hadolint Dockerfile
```

> Verás advertencias sobre buenas prácticas: uso de `latest`, falta de `USER`, etc.

---

## Paso 3: Escanear vulnerabilidades con Trivy

### Instalar Trivy

```bash
sudo apt install trivy -y
```

### Construir imagen y escanear

```bash
docker build -t vulnerable-app .
trivy image vulnerable-app
```

> Trivy detecta CVEs en el sistema operativo y los paquetes instalados.

---

## Paso 4: Revisar cumplimiento con Docker Bench for Security

```bash
git clone https://github.com/docker/docker-bench-security.git
cd docker-bench-security
sudo bash docker-bench-security.sh
```

> El script revisa más de 100 controles de seguridad basados en el estándar **CIS Docker Benchmark**.

---

## Paso 5: Monitorear comportamiento en tiempo real con Falco

```bash
docker run -i -t --name falco --privileged \
  -v /var/run/docker.sock:/host/var/run/docker.sock \
  -v /dev:/host/dev \
  -v /proc:/host/proc:ro \
  -v /boot:/host/boot:ro \
  -v /lib/modules:/host/lib/modules:ro \
  -v /usr:/host/usr:ro \
  falcosecurity/falco
```

> Falco alertará si un contenedor ejecuta comandos sospechosos o accede a archivos sensibles.

---

## Conclusión

| Herramienta     | Función                                   |
|-----------------|--------------------------------------------|
| **Hadolint**     | Auditoría de Dockerfiles                  |
| **Trivy**        | Escaneo de vulnerabilidades               |
| **Docker Bench** | Auditoría de configuración de Docker      |
| **Falco**        | Detección en tiempo real de amenazas      |

---

## Recursos

- [Trivy](https://github.com/aquasecurity/trivy)
- [Hadolint](https://github.com/hadolint/hadolint)
- [Docker Bench for Security](https://github.com/docker/docker-bench-security)
- [Falco](https://falco.org/)

