# Reto Docker 1: ¡Tu Web App Personalizada en un Contenedor!

## Objetivo

Vas a crear una imagen Docker personalizada capaz de servir una pequeña aplicación web estática (una página HTML) usando Nginx o Apache. Además, añadirás detalles personales y sorpresas para practicar distintas opciones de construcción de imágenes.

---

## Requisitos

1. **Base image:**  
   Usa la imagen oficial de Nginx (`nginx:alpine`).

2. **Página principal personalizada:**  
   Copia a tu imagen una página `index.html` propia que incluya:
   - Un saludo personalizado (¡con tu nombre!).

3. **Variables de entorno:**  
   La página debe mostrar el contenido de una variable de entorno (`AUTOR`) que definas en el Dockerfile.

4. **Archivos adicionales:**  
   - Copia un archivo de texto de bienvenida (`README.txt`) al directorio `/usr/share/nginx/html`.
   - Copia una imagen (`logo.png`) que se muestre en tu web.

5. **Exposición de puertos:**  
   Expón el puerto adecuado en el Dockerfile (por ejemplo, `80`).

6. **Bonus (opcionales):**
   - Usa labels personalizadas en el Dockerfile.

---

## Entrega

- Tu `Dockerfile` y todos los archivos necesarios (`index.html`, `logo.png`, `README.txt`, etc).
- Comando para construir la imagen y ejecutarla.

---

¡Creatividad al poder! No olvides comentar tu Dockerfile.


# Reto Docker 2: ¡Web App Personalizada con Volúmenes!

## Objetivo

Crea una imagen Docker que sirva una página web personalizada usando Nginx o Apache, pero ahora debes usar **volúmenes** para gestionar el contenido web de forma flexible.

---

## Requisitos

1. **Base image:**  
   Usa la imagen oficial de Nginx (`nginx:alpine`).

2. **Página principal personalizada:**  
   El archivo `index.html` **NO** debe copiarse dentro de la imagen en el build, sino que se proporcionará desde un volumen al ejecutar el contenedor.

3. **Archivos adicionales gestionados con volúmenes:**  
   - Tanto `index.html`, como una imagen (`logo.png`) y un archivo de texto de bienvenida (`README.txt`) deben almacenarse en una carpeta de tu máquina (por ejemplo, `./contenido-web/`).
   - Al ejecutar el contenedor, debes montar esa carpeta como un volumen en `/usr/share/nginx/html`.

4. **Variables de entorno:**  
   Puedes definir variables de entorno en el Dockerfile o al ejecutar el contenedor, pero su uso en la web será opcional (puedes mostrar la variable usando un script JavaScript en el HTML si lo deseas).

5. **Exposición de puertos:**  
   Expón el puerto adecuado en el Dockerfile (`EXPOSE 80`).

---

## Ejemplo de estructura

```Estructura

tu-proyecto/
├── Dockerfile
├── contenido-web/
│ ├── index.html
│ ├── logo.png
│ └── README.txt

```

---

## Entrega

- Tu `Dockerfile` y todos los archivos necesarios.
- Comando para construir la imagen y ejecutarla, usando volúmenes.

---

¡Recuerda que si cambias el contenido del directorio de tu máquina, los cambios se reflejan al instante en el contenedor!  



# Reto Docker 3: MySQL + phpMyAdmin en Red y con Volumen Persistente

## Objetivo

Levanta un entorno multi-contenedor con Docker, usando una **red personalizada** y un **volumen persistente** para la base de datos.  
El objetivo es comprobar que puedes acceder a MySQL a través de phpMyAdmin y crear bases de datos, todo gestionado mediante Docker.

---

## Requisitos

1. **Red personalizada:**
   - Crea una red Docker propia (por ejemplo, llamada `red_mysql`).

2. **MySQL con volumen persistente:**
   - Lanza un contenedor con la imagen oficial de MySQL.
   - Usa una contraseña segura para root (`MYSQL_ROOT_PASSWORD=miclave123`).
   - Crea y usa un volumen persistente para `/var/lib/mysql` para que los datos no se pierdan aunque borres el contenedor.
   - Ejemplo de nombre para el volumen: `mysql_datos`.

3. **phpMyAdmin conectado a la red:**
   - Lanza un contenedor con la imagen oficial de phpMyAdmin.
   - Conéctalo a la misma red personalizada.
   - Configura las variables de entorno necesarias para conectar a MySQL (host, usuario, contraseña).

4. **Comprobación:**
   - Accede a phpMyAdmin desde tu navegador.
   - Comprueba que puedes conectarte a MySQL.
   - Crea una base de datos de prueba y sube una captura de pantalla.

---

## Entrega

- Los comandos utilizados para crear la red, los volúmenes y lanzar ambos contenedores.

---

## Pistas

- Puedes usar los siguientes puertos:
  - MySQL: **3306**
  - phpMyAdmin: **8080** (o el que prefieras en tu host)
- El host MySQL para phpMyAdmin debe ser el nombre del contenedor de MySQL, ya que comparten la misma red personalizada.


---


