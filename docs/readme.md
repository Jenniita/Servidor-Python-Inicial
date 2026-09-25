# Servidor Python en Docker

Servidor HTTP mínimo hecho con Python que responde:

```text
Hola mundo con Python
```

La ruta específica de este servidor es diferente a la de los servidores Node y PHP:

```text
http://localhost:8083/python/hola-mundo
```

También se utiliza un puerto diferente: el puerto `8083` del equipo se conecta con el puerto `8000` del contenedor.

La ruta raíz también está disponible:

```text
http://localhost:8083/
```

## Requisitos

- Docker Desktop instalado y en ejecución.
- Una terminal situada en la carpeta raíz del proyecto:

```text
Servidor-Python-Inicial/
```

No es necesario instalar Python ni ninguna dependencia en el equipo para ejecutar el servidor con Docker.

## Archivos del proyecto

- `app.py`: crea el servidor HTTP y define las rutas `/` y `/python/hola-mundo`.
- `Dockerfile`: indica cómo construir la imagen con Python 3.12.
- `docker-compose.yml`: configura el contenedor y el mapeo de puertos `8083:8000`.
- `.dockerignore`: evita copiar archivos innecesarios a la imagen.

## Pasos realizados para crear el servidor

### 1. Crear el servidor Python

Se creó el archivo `app.py` usando únicamente módulos incluidos en Python. Por eso no hace falta instalar Flask ni ninguna otra dependencia.

El servidor utiliza `HTTPServer` y `BaseHTTPRequestHandler`:

- `HTTPServer` mantiene el servidor escuchando peticiones.
- `BaseHTTPRequestHandler` permite decidir qué respuesta enviar en cada petición.
- `HOST = "0.0.0.0"` permite recibir peticiones desde fuera del contenedor.
- `PORT = 8000` define el puerto interno del servidor.

Cuando se recibe una petición `GET` a `/` o a `/python/hola-mundo`, se devuelve el texto `Hola mundo con Python` con estado HTTP `200`. Para cualquier otra ruta se devuelve `Ruta no encontrada` con estado `404`.

### 2. Crear el Dockerfile

El archivo `Dockerfile` contiene las instrucciones para crear la imagen:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY app.py .
EXPOSE 8000
CMD ["python", "app.py"]
```

Esto significa lo siguiente:

1. Se utiliza Python 3.12 en una imagen ligera.
2. Se establece `/app` como carpeta de trabajo dentro del contenedor.
3. Se copia `app.py` al contenedor.
4. Se indica que el servidor utiliza el puerto `8000`.
5. Se ejecuta `python app.py` al iniciar el contenedor.

### 3. Configurar Docker Compose

Se creó `docker-compose.yml` para poder construir y arrancar el proyecto con un único servicio:

```yaml
services:
	python:
		build: .
		ports:
			- "8083:8000"
```

El formato `8083:8000` significa que:

- `8083` es el puerto disponible en el equipo.
- `8000` es el puerto donde escucha Python dentro del contenedor.

Se eligió el puerto `8083` para que sea diferente de los puertos utilizados por los servidores Node y PHP.

### 4. Añadir `.dockerignore`

Se creó `.dockerignore` para no enviar a Docker archivos innecesarios como `__pycache__`, archivos `.pyc` o la carpeta `.git` cuando se construye la imagen.

### 5. Construir y arrancar el contenedor

Desde la carpeta raíz del proyecto se ejecutó:

```powershell
docker compose up --build -d
```

La opción `--build` obliga a Docker a reconstruir la imagen con los cambios actuales. La opción `-d` deja el contenedor ejecutándose en segundo plano.

Para comprobar que está activo se puede ejecutar:

```powershell
docker compose ps
```

Debe aparecer un contenedor con un estado similar a `Up` y el puerto `0.0.0.0:8083->8000/tcp`.

### 6. Probar el servidor

Se comprobó la respuesta desde el propio contenedor usando las dos rutas:

```text
http://localhost:8083/
http://localhost:8083/python/hola-mundo
```

Las dos rutas devuelven:

```text
Hola mundo con Python
```

### 7. Corregir la ruta raíz

Inicialmente solo estaba definida `/python/hola-mundo`. Por eso al visitar `http://localhost:8083/` aparecía `Ruta no encontrada`.

Se modificó `app.py` para aceptar las dos rutas:

```python
if self.path in ("/", "/python/hola-mundo"):
```

Después de modificar el código fue necesario reconstruir el contenedor:

```powershell
docker compose down
docker compose up --build -d
```

Así Docker incorporó el nuevo código y la ruta raíz empezó a responder correctamente.

## Opción 1: levantar el servidor con Docker Compose

### 1. Abrir la terminal en la carpeta del proyecto

En PowerShell:

```powershell
cd "C:\ruta\a\Servidor-Python-Inicial"
```

Sustituye `C:\ruta\a` por la ubicación real del proyecto.

### 2. Construir la imagen y arrancar el contenedor

```powershell
docker compose up --build
```

La opción `--build` construye la imagen usando el `Dockerfile`. La primera vez puede tardar mientras Docker descarga la imagen base de Python.

Cuando aparezca el mensaje del servidor, el contenedor estará funcionando.

### 3. Probar la respuesta

Abre esta dirección en el navegador:

```text
http://localhost:8083/python/hola-mundo
```

También puedes abrir directamente la ruta raíz:

```text
http://localhost:8083/
```

También puedes probarla desde PowerShell:

```powershell
Invoke-WebRequest http://localhost:8083/python/hola-mundo
```

La respuesta esperada es:

```text
Hola mundo con Python
```

### 4. Detener el servidor

En la terminal donde se está ejecutando Docker Compose, pulsa `Ctrl+C`. Después elimina el contenedor con:

```powershell
docker compose down
```

## Opción 2: ejecutar el contenedor con Docker directamente

Desde la carpeta raíz del proyecto:

```powershell
docker build -t servidor-python-inicial .
docker run --name servidor-python -p 8083:8000 servidor-python-inicial
```

Después visita:

```text
http://localhost:8083/python/hola-mundo
```

Para detener y eliminar este contenedor:

```powershell
docker stop servidor-python
docker rm servidor-python
```

## Ejecutarlo sin Docker

Si Python 3 está instalado en el equipo, también se puede ejecutar directamente:

```powershell
python app.py
```

El servidor escuchará en el puerto `8000`, por lo que la URL local será:

```text
http://localhost:8000/python/hola-mundo
```

En Windows, si el comando anterior no existe, prueba:

```powershell
py app.py
```

## Rutas disponibles

| Método | Ruta                 | Respuesta                       |
| ------ | -------------------- | ------------------------------- |
| `GET`  | `/`                  | `200` y `Hola mundo con Python` |
| `GET`  | `/python/hola-mundo` | `200` y `Hola mundo con Python` |
| `GET`  | Cualquier otra ruta  | `404` y `Ruta no encontrada`    |
