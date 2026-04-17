# Lab 06-00-01: Laboratorio 6. Proyecto Integrador — Contenerización Full Stack con Docker

## 1. Metadatos

| Campo            | Detalle                                      |
|------------------|----------------------------------------------|
| **Duración**     | 80 minutos                                   |
| **Complejidad**  | Alta                                         |
| **Nivel Bloom**  | Crear (Create)                               |
| **Módulo**       | 6 — Contenerización y Despliegue con Docker  |
| **Temas**        | 6.1 al 6.5                                   |

---

## 2. Descripción General

En este laboratorio integrador construirás y contenerizarás una aplicación full stack compuesta por tres servicios: un frontend estático servido por Nginx, una API REST en Node.js/Express y una base de datos PostgreSQL. Partirás de un proyecto base, crearás Dockerfiles optimizados para cada servicio, orquestarás el stack completo con Docker Compose y verificarás el funcionamiento end-to-end mediante Postman. Al finalizar, publicarás las imágenes en Docker Hub y versionarás el código en GitHub, consolidando el ciclo completo de contenerización empresarial.

---

## 3. Objetivos de Aprendizaje

Al completar este laboratorio serás capaz de:

- [ ] Construir una aplicación full stack simple (frontend + API REST Node.js + PostgreSQL) lista para contenerización
- [ ] Crear Dockerfiles optimizados aplicando buenas prácticas como imágenes base ligeras y capas de caché
- [ ] Definir y ejecutar un entorno multi-contenedor con Docker Compose orquestando redes y volúmenes
- [ ] Verificar la comunicación entre contenedores y validar el funcionamiento end-to-end con Postman
- [ ] Publicar imágenes en Docker Hub y versionar el proyecto en GitHub

---

## 4. Prerrequisitos

### Conocimientos Previos
- Haber revisado los contenidos teóricos de los temas 6.1 al 6.5 del Módulo 6
- Comprensión de la diferencia entre contenedores y máquinas virtuales
- Conocimientos básicos de línea de comandos (navegar directorios, crear archivos, ejecutar comandos)
- Familiaridad con conceptos de API REST y Node.js

### Acceso y Cuentas Requeridas
- Docker Desktop instalado, en ejecución y verificado con `docker run hello-world`
- Cuenta activa en **Docker Hub** y sesión iniciada con `docker login`
- Cuenta activa en **GitHub** con Git configurado (nombre de usuario y correo)
- Node.js 20 LTS instalado y verificado con `node --version`
- Visual Studio Code con la extensión **Docker** de Microsoft activa
- Postman 11.x instalado y funcional

---

## 5. Entorno de Laboratorio

### Requisitos de Hardware

| Recurso       | Mínimo                        | Recomendado                   |
|---------------|-------------------------------|-------------------------------|
| RAM           | 16 GB DDR4                    | 32 GB                         |
| Almacenamiento | 10 GB libres para este lab   | 20 GB libres                  |
| CPU           | Intel i5 8va gen / Ryzen 5    | Intel i7 / Ryzen 7            |
| Pantalla      | 1920×1080                     | Dual monitor                  |

### Software Requerido

| Herramienta        | Versión       | Verificación                    |
|--------------------|---------------|---------------------------------|
| Docker Desktop     | 4.29+         | `docker --version`              |
| Docker Compose     | v2 (incluido) | `docker compose version`        |
| Node.js            | 20.x LTS      | `node --version`                |
| Git                | 2.44+         | `git --version`                 |
| Visual Studio Code | 1.88+         | Abrir y verificar extensión Docker |
| Postman            | 11.x          | Abrir y crear workspace         |

### Verificación del Entorno Inicial

Antes de comenzar, ejecuta estos comandos en tu terminal para confirmar que el entorno está listo:

```bash
# Verificar Docker
docker --version
docker compose version
docker run hello-world

# Verificar Node.js y npm
node --version
npm --version

# Verificar Git y configuración
git --version
git config --global user.name
git config --global user.email

# Verificar sesión en Docker Hub (si no has hecho login)
docker login
```

> **Punto de control:** Todos los comandos anteriores deben ejecutarse sin errores antes de continuar. Si `docker run hello-world` falla, reinicia Docker Desktop y espera 30 segundos.

---

## 6. Instrucciones Paso a Paso

### Paso 1: Crear la Estructura del Proyecto Base

**Objetivo:** Establecer la estructura de directorios del proyecto y crear los archivos de código fuente de los tres servicios.

#### Instrucciones

**1.1** Crea el directorio raíz del proyecto y navega hacia él:

```bash
# Windows (PowerShell)
mkdir $env:USERPROFILE\Documents\lab06-docker-fullstack
cd $env:USERPROFILE\Documents\lab06-docker-fullstack

# macOS / Linux
mkdir ~/Documents/lab06-docker-fullstack
cd ~/Documents/lab06-docker-fullstack
```

**1.2** Crea la estructura de directorios completa:

```bash
# Windows (PowerShell)
mkdir frontend, backend, postgres-init

# macOS / Linux
mkdir -p frontend backend postgres-init
```

**1.3** Crea el archivo HTML del frontend. Abre VS Code (`code .`) y crea el archivo `frontend/index.html` con el siguiente contenido:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Gestión de Tareas — Lab Docker</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: 'Segoe UI', sans-serif;
      background: #f0f2f5;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 2rem;
    }
    header {
      background: #2563eb;
      color: white;
      width: 100%;
      max-width: 700px;
      padding: 1.5rem 2rem;
      border-radius: 12px 12px 0 0;
      text-align: center;
    }
    header h1 { font-size: 1.6rem; }
    header p { font-size: 0.9rem; opacity: 0.85; margin-top: 0.3rem; }
    .container {
      background: white;
      width: 100%;
      max-width: 700px;
      padding: 2rem;
      border-radius: 0 0 12px 12px;
      box-shadow: 0 4px 20px rgba(0,0,0,0.08);
    }
    .form-group { display: flex; gap: 0.75rem; margin-bottom: 1.5rem; }
    .form-group input {
      flex: 1;
      padding: 0.75rem 1rem;
      border: 1.5px solid #d1d5db;
      border-radius: 8px;
      font-size: 1rem;
      outline: none;
      transition: border-color 0.2s;
    }
    .form-group input:focus { border-color: #2563eb; }
    .form-group button {
      padding: 0.75rem 1.5rem;
      background: #2563eb;
      color: white;
      border: none;
      border-radius: 8px;
      font-size: 1rem;
      cursor: pointer;
      transition: background 0.2s;
    }
    .form-group button:hover { background: #1d4ed8; }
    #status {
      font-size: 0.85rem;
      margin-bottom: 1rem;
      padding: 0.5rem 0.75rem;
      border-radius: 6px;
      display: none;
    }
    #status.success { background: #d1fae5; color: #065f46; display: block; }
    #status.error   { background: #fee2e2; color: #991b1b; display: block; }
    ul#task-list { list-style: none; }
    ul#task-list li {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 0.85rem 1rem;
      border-bottom: 1px solid #f3f4f6;
      font-size: 0.95rem;
    }
    ul#task-list li:last-child { border-bottom: none; }
    .delete-btn {
      background: #ef4444;
      color: white;
      border: none;
      padding: 0.3rem 0.7rem;
      border-radius: 6px;
      cursor: pointer;
      font-size: 0.8rem;
    }
    .delete-btn:hover { background: #dc2626; }
    .empty-msg { text-align: center; color: #9ca3af; padding: 2rem 0; }
    footer {
      margin-top: 1.5rem;
      font-size: 0.78rem;
      color: #9ca3af;
      text-align: center;
    }
  </style>
</head>
<body>
  <header>
    <h1>📋 Gestión de Tareas</h1>
    <p>Aplicación Full Stack — Docker Lab 06</p>
  </header>
  <div class="container">
    <div class="form-group">
      <input type="text" id="task-input" placeholder="Escribe una nueva tarea..." />
      <button onclick="addTask()">Agregar</button>
    </div>
    <div id="status"></div>
    <ul id="task-list"></ul>
    <p class="empty-msg" id="empty-msg">No hay tareas aún. ¡Agrega la primera!</p>
  </div>
  <footer>Contenedor: Nginx Alpine | API: Node.js + Express | DB: PostgreSQL 16</footer>

  <script>
    const API_URL = '/api/tasks';

    async function loadTasks() {
      try {
        const res = await fetch(API_URL);
        const tasks = await res.json();
        renderTasks(tasks);
      } catch (e) {
        showStatus('Error al conectar con la API. Verifica que los contenedores estén activos.', 'error');
      }
    }

    function renderTasks(tasks) {
      const list = document.getElementById('task-list');
      const emptyMsg = document.getElementById('empty-msg');
      list.innerHTML = '';
      if (tasks.length === 0) {
        emptyMsg.style.display = 'block';
        return;
      }
      emptyMsg.style.display = 'none';
      tasks.forEach(task => {
        const li = document.createElement('li');
        li.innerHTML = `
          <span>${task.title}</span>
          <button class="delete-btn" onclick="deleteTask(${task.id})">Eliminar</button>
        `;
        list.appendChild(li);
      });
    }

    async function addTask() {
      const input = document.getElementById('task-input');
      const title = input.value.trim();
      if (!title) { showStatus('El título de la tarea no puede estar vacío.', 'error'); return; }
      try {
        const res = await fetch(API_URL, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ title })
        });
        if (!res.ok) throw new Error();
        input.value = '';
        showStatus('Tarea agregada exitosamente.', 'success');
        loadTasks();
      } catch (e) {
        showStatus('Error al agregar la tarea.', 'error');
      }
    }

    async function deleteTask(id) {
      try {
        await fetch(`${API_URL}/${id}`, { method: 'DELETE' });
        showStatus('Tarea eliminada.', 'success');
        loadTasks();
      } catch (e) {
        showStatus('Error al eliminar la tarea.', 'error');
      }
    }

    function showStatus(msg, type) {
      const el = document.getElementById('status');
      el.textContent = msg;
      el.className = type;
      setTimeout(() => { el.className = ''; el.style.display = 'none'; }, 3000);
    }

    loadTasks();
  </script>
</body>
</html>
```

**1.4** Crea el archivo de configuración de Nginx `frontend/nginx.conf`:

```nginx
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    # Proxy inverso: redirige /api/* hacia el servicio de backend
    location /api/ {
        proxy_pass http://api:3000/api/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

**1.5** Inicializa el proyecto Node.js para el backend:

```bash
cd backend
npm init -y
npm install express pg cors dotenv
cd ..
```

**1.6** Crea el archivo principal de la API `backend/index.js`:

```javascript
require('dotenv').config();
const express = require('express');
const cors = require('cors');
const { Pool } = require('pg');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(cors());
app.use(express.json());

// Configuración del pool de conexiones a PostgreSQL
const pool = new Pool({
  host:     process.env.DB_HOST     || 'localhost',
  port:     parseInt(process.env.DB_PORT) || 5432,
  database: process.env.DB_NAME     || 'tasksdb',
  user:     process.env.DB_USER     || 'taskuser',
  password: process.env.DB_PASSWORD || 'taskpass',
});

// Función para reintentar la conexión a la base de datos
async function waitForDB(retries = 10, delay = 3000) {
  for (let i = 0; i < retries; i++) {
    try {
      const client = await pool.connect();
      console.log('✅ Conexión a PostgreSQL establecida.');
      client.release();
      return;
    } catch (err) {
      console.log(`⏳ Esperando DB... intento ${i + 1}/${retries}`);
      await new Promise(res => setTimeout(res, delay));
    }
  }
  throw new Error('❌ No se pudo conectar a PostgreSQL después de varios intentos.');
}

// ── Endpoints CRUD ──────────────────────────────────────────────

// GET /api/tasks — Obtener todas las tareas
app.get('/api/tasks', async (req, res) => {
  try {
    const result = await pool.query('SELECT * FROM tasks ORDER BY created_at DESC');
    res.json(result.rows);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Error al obtener tareas' });
  }
});

// POST /api/tasks — Crear una nueva tarea
app.post('/api/tasks', async (req, res) => {
  const { title } = req.body;
  if (!title || title.trim() === '') {
    return res.status(400).json({ error: 'El campo title es requerido' });
  }
  try {
    const result = await pool.query(
      'INSERT INTO tasks (title) VALUES ($1) RETURNING *',
      [title.trim()]
    );
    res.status(201).json(result.rows[0]);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Error al crear la tarea' });
  }
});

// DELETE /api/tasks/:id — Eliminar una tarea por ID
app.delete('/api/tasks/:id', async (req, res) => {
  const { id } = req.params;
  try {
    const result = await pool.query('DELETE FROM tasks WHERE id = $1 RETURNING *', [id]);
    if (result.rowCount === 0) {
      return res.status(404).json({ error: 'Tarea no encontrada' });
    }
    res.json({ message: 'Tarea eliminada', task: result.rows[0] });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Error al eliminar la tarea' });
  }
});

// GET /api/health — Endpoint de salud para verificar que la API responde
app.get('/api/health', (req, res) => {
  res.json({ status: 'ok', service: 'tasks-api', timestamp: new Date().toISOString() });
});

// ── Inicio del servidor ─────────────────────────────────────────
waitForDB()
  .then(() => {
    app.listen(PORT, () => {
      console.log(`🚀 API escuchando en http://localhost:${PORT}`);
    });
  })
  .catch(err => {
    console.error(err.message);
    process.exit(1);
  });
```

**1.7** Crea el script SQL de inicialización de la base de datos `postgres-init/01-init.sql`:

```sql
-- Crear la base de datos (ya existe por variable de entorno POSTGRES_DB)
-- Este script se ejecuta automáticamente al iniciar el contenedor de PostgreSQL

CREATE TABLE IF NOT EXISTS tasks (
    id         SERIAL PRIMARY KEY,
    title      VARCHAR(255) NOT NULL,
    completed  BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Insertar datos de prueba
INSERT INTO tasks (title) VALUES
    ('Revisar documentación de Docker'),
    ('Crear Dockerfile para el backend'),
    ('Configurar Docker Compose'),
    ('Publicar imagen en Docker Hub'),
    ('Versionar proyecto en GitHub');

-- Confirmar creación
SELECT 'Tabla tasks creada con ' || COUNT(*) || ' registros de prueba.' AS resultado
FROM tasks;
```

**1.8** Crea el archivo `.env` para el backend (solo para desarrollo local, **no** para Docker):

```bash
# backend/.env  — Solo para pruebas locales sin Docker
PORT=3000
DB_HOST=localhost
DB_PORT=5432
DB_NAME=tasksdb
DB_USER=taskuser
DB_PASSWORD=taskpass
```

**Salida esperada:** La estructura del proyecto debe verse así:

```
lab06-docker-fullstack/
├── frontend/
│   ├── index.html
│   └── nginx.conf
├── backend/
│   ├── index.js
│   ├── package.json
│   ├── package-lock.json
│   ├── node_modules/
│   └── .env
└── postgres-init/
    └── 01-init.sql
```

**Verificación:**

```bash
# Verificar que la estructura es correcta
# Windows (PowerShell)
Get-ChildItem -Recurse -Depth 2 | Where-Object { !$_.PSIsContainer }

# macOS / Linux
find . -not -path '*/node_modules/*' -type f
```

---

### Paso 2: Crear los Dockerfiles Optimizados

**Objetivo:** Escribir Dockerfiles para el frontend y el backend aplicando buenas prácticas de capas de caché e imágenes base ligeras.

#### Instrucciones

**2.1** Crea el archivo `frontend/Dockerfile`:

```dockerfile
# ── Frontend Dockerfile ──────────────────────────────────────────
# Imagen base: Nginx Alpine (extremadamente ligera, ~23 MB)
FROM nginx:1.25-alpine

# Metadatos de la imagen
LABEL maintainer="lab06-docker"
LABEL description="Frontend estático servido por Nginx para la app de tareas"

# Copiar la configuración personalizada de Nginx
# Esto reemplaza el archivo de configuración por defecto
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copiar los archivos estáticos del frontend
COPY index.html /usr/share/nginx/html/index.html

# Exponer el puerto 80 (documentación; Docker Compose lo mapea)
EXPOSE 80

# El comando por defecto de la imagen nginx:alpine ya inicia Nginx
# No es necesario sobreescribir CMD
```

**2.2** Crea el archivo `backend/Dockerfile`:

```dockerfile
# ── Backend Dockerfile ───────────────────────────────────────────
# Etapa 1: Instalación de dependencias
# Usamos node:20-alpine para una imagen base ligera (~180 MB vs ~1 GB de node:20)
FROM node:20-alpine AS dependencies

# Establecer directorio de trabajo dentro del contenedor
WORKDIR /app

# BUENA PRÁCTICA: Copiar SOLO package.json y package-lock.json primero.
# Docker cachea esta capa. Si el código fuente cambia pero las dependencias no,
# Docker reutilizará la capa de node_modules sin reinstalar.
COPY package.json package-lock.json ./

# Instalar solo dependencias de producción (excluye devDependencies)
RUN npm ci --only=production

# ──────────────────────────────────────────────────────────────────
# Etapa 2: Imagen final de producción
FROM node:20-alpine AS production

LABEL maintainer="lab06-docker"
LABEL description="API REST de tareas — Node.js 20 + Express + PostgreSQL"

# Crear usuario no-root por seguridad (buena práctica)
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copiar node_modules desde la etapa de dependencias
COPY --from=dependencies /app/node_modules ./node_modules

# Copiar el código fuente de la aplicación
COPY index.js ./

# Cambiar la propiedad de los archivos al usuario no-root
RUN chown -R appuser:appgroup /app

# Ejecutar como usuario no-root
USER appuser

# Exponer el puerto de la API
EXPOSE 3000

# Comando para iniciar la aplicación
CMD ["node", "index.js"]
```

**2.3** Crea el archivo `.dockerignore` en el directorio `backend/` para excluir archivos innecesarios de la imagen:

```
# backend/.dockerignore
node_modules
npm-debug.log
.env
.git
.gitignore
*.md
README*
```

**2.4** Crea también un `.dockerignore` en el directorio raíz del proyecto:

```
# .dockerignore (raíz)
**/node_modules
**/.env
.git
.gitignore
*.md
```

**Salida esperada:** Cuatro nuevos archivos creados:
- `frontend/Dockerfile`
- `backend/Dockerfile`
- `backend/.dockerignore`
- `.dockerignore`

**Verificación:** Revisa la sintaxis de los Dockerfiles con VS Code (la extensión Docker mostrará resaltado de sintaxis y advertencias). No debe haber líneas subrayadas en rojo.

---

### Paso 3: Configurar Docker Compose

**Objetivo:** Definir el archivo `docker-compose.yml` que orquesta los tres servicios, sus redes internas y volúmenes persistentes.

#### Instrucciones

**3.1** En el directorio raíz del proyecto, crea el archivo `.env` para Docker Compose:

```bash
# .env (raíz del proyecto — usado por Docker Compose)
# Variables de la base de datos
POSTGRES_DB=tasksdb
POSTGRES_USER=taskuser
POSTGRES_PASSWORD=taskpass

# Puerto expuesto del frontend en el host
FRONTEND_PORT=8080

# Puerto expuesto de la API en el host (útil para pruebas directas con Postman)
API_PORT=3000
```

**3.2** Crea el archivo `docker-compose.yml` en el directorio raíz:

```yaml
# docker-compose.yml
# Docker Compose v2 — Stack completo: Frontend + API + PostgreSQL
# Versión omitida intencionalmente (obsoleta en Compose v2)

services:

  # ── Servicio 1: Base de datos PostgreSQL ─────────────────────────
  db:
    image: postgres:16-alpine          # Imagen oficial Alpine (ligera)
    container_name: tasks-db
    restart: unless-stopped            # Reiniciar automáticamente salvo detención manual
    environment:
      POSTGRES_DB:       ${POSTGRES_DB}
      POSTGRES_USER:     ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      # Volumen nombrado para persistencia de datos entre reinicios
      - postgres_data:/var/lib/postgresql/data
      # Script de inicialización: se ejecuta solo la primera vez
      - ./postgres-init:/docker-entrypoint-initdb.d:ro
    networks:
      - backend-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s

  # ── Servicio 2: API REST Node.js ──────────────────────────────────
  api:
    build:
      context: ./backend              # Directorio donde está el Dockerfile
      dockerfile: Dockerfile
    container_name: tasks-api
    restart: unless-stopped
    environment:
      PORT:        3000
      DB_HOST:     db                 # Nombre del servicio en la red Docker
      DB_PORT:     5432
      DB_NAME:     ${POSTGRES_DB}
      DB_USER:     ${POSTGRES_USER}
      DB_PASSWORD: ${POSTGRES_PASSWORD}
    ports:
      - "${API_PORT}:3000"            # Exponer para pruebas directas con Postman
    networks:
      - backend-net
      - frontend-net
    depends_on:
      db:
        condition: service_healthy    # Espera a que PostgreSQL esté listo

  # ── Servicio 3: Frontend Nginx ────────────────────────────────────
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: tasks-frontend
    restart: unless-stopped
    ports:
      - "${FRONTEND_PORT}:80"         # Acceso desde el navegador: localhost:8080
    networks:
      - frontend-net
    depends_on:
      - api

# ── Redes ─────────────────────────────────────────────────────────
networks:
  backend-net:
    driver: bridge
    name: tasks-backend-network
  frontend-net:
    driver: bridge
    name: tasks-frontend-network

# ── Volúmenes ─────────────────────────────────────────────────────
volumes:
  postgres_data:
    name: tasks-postgres-data
```

**3.3** Verifica la validez del archivo `docker-compose.yml`:

```bash
docker compose config
```

**Salida esperada:** Docker Compose mostrará la configuración expandida sin errores. Deberías ver los tres servicios (`db`, `api`, `frontend`) con todas sus propiedades resueltas.

**Verificación:**

```bash
# La salida debe incluir las tres secciones de servicios sin mensajes de error
docker compose config | grep "container_name"
# Debe mostrar:
#   container_name: tasks-db
#   container_name: tasks-api
#   container_name: tasks-frontend
```

---

### Paso 4: Construir y Ejecutar el Stack Completo

**Objetivo:** Construir las imágenes Docker y levantar todos los servicios con un único comando, verificando que la comunicación entre contenedores funciona correctamente.

#### Instrucciones

**4.1** Desde el directorio raíz del proyecto, construye todas las imágenes:

```bash
docker compose build --no-cache
```

> El flag `--no-cache` garantiza una construcción limpia. En iteraciones posteriores puedes omitirlo para aprovechar la caché.

**Salida esperada:** Verás la construcción de las imágenes `frontend` y `api`. La imagen de PostgreSQL se descarga directamente de Docker Hub sin construcción local. El proceso tarda aproximadamente 2-4 minutos dependiendo de la velocidad de internet.

```
[+] Building 45.2s (14/14) FINISHED
 => [api] FROM docker.io/library/node:20-alpine
 => [api dependencies 3/3] RUN npm ci --only=production
 => [api production 4/5] COPY index.js ./
 => [frontend] FROM docker.io/library/nginx:1.25-alpine
 => [frontend 2/3] COPY nginx.conf /etc/nginx/conf.d/default.conf
```

**4.2** Levanta todos los servicios en modo detached (segundo plano):

```bash
docker compose up -d
```

**Salida esperada:**

```
[+] Running 4/4
 ✔ Volume "tasks-postgres-data"      Created
 ✔ Container tasks-db                Started
 ✔ Container tasks-api               Started
 ✔ Container tasks-frontend          Started
```

**4.3** Verifica el estado de los contenedores:

```bash
docker compose ps
```

**Salida esperada:** Los tres contenedores deben estar en estado `running`. El contenedor `tasks-db` debe mostrar `healthy` después de ~20 segundos.

```
NAME              IMAGE                        STATUS                   PORTS
tasks-db          postgres:16-alpine           Up (healthy)             5432/tcp
tasks-api         lab06-docker-fullstack-api   Up                       0.0.0.0:3000->3000/tcp
tasks-frontend    lab06-docker-fullstack-...   Up                       0.0.0.0:8080->80/tcp
```

**4.4** Revisa los logs de cada servicio para detectar posibles errores:

```bash
# Ver logs de todos los servicios
docker compose logs

# Ver logs solo de la API (más útil para debugging)
docker compose logs api

# Seguir los logs en tiempo real
docker compose logs -f api
```

**Salida esperada en los logs de la API:**

```
tasks-api  | ⏳ Esperando DB... intento 1/10
tasks-api  | ✅ Conexión a PostgreSQL establecida.
tasks-api  | 🚀 API escuchando en http://localhost:3000
```

**4.5** Prueba el endpoint de salud directamente desde la terminal:

```bash
# Windows (PowerShell)
Invoke-WebRequest -Uri http://localhost:3000/api/health | Select-Object -ExpandProperty Content

# macOS / Linux
curl http://localhost:3000/api/health
```

**Salida esperada:**

```json
{"status":"ok","service":"tasks-api","timestamp":"2024-10-15T14:30:00.000Z"}
```

**Verificación:** Abre tu navegador y navega a `http://localhost:8080`. Deberías ver la interfaz de Gestión de Tareas con 5 tareas precargadas desde el script SQL de inicialización.

---

### Paso 5: Pruebas Funcionales con Postman

**Objetivo:** Validar todos los endpoints de la API REST mediante pruebas funcionales documentadas en Postman.

#### Instrucciones

**5.1** Abre Postman y crea una nueva colección llamada **"Lab 06 — Tasks API"**.

**5.2** Configura una variable de colección:
- Variable: `baseUrl`
- Valor inicial: `http://localhost:3000`

**5.3** Crea y ejecuta las siguientes peticiones en orden:

**Request 1: Verificar salud de la API**
- Método: `GET`
- URL: `{{baseUrl}}/api/health`
- Clic en **Send**

Resultado esperado: `200 OK` con body `{"status":"ok",...}`

**Request 2: Obtener todas las tareas**
- Método: `GET`
- URL: `{{baseUrl}}/api/tasks`
- Clic en **Send**

Resultado esperado: `200 OK` con array de 5 tareas (las del script SQL)

**Request 3: Crear una nueva tarea**
- Método: `POST`
- URL: `{{baseUrl}}/api/tasks`
- Headers: `Content-Type: application/json`
- Body (raw JSON):
```json
{
  "title": "Mi primera tarea creada via Postman"
}
```
- Clic en **Send**

Resultado esperado: `201 Created` con el objeto de la tarea creada incluyendo `id` y `created_at`

**Request 4: Verificar que la tarea fue creada**
- Método: `GET`
- URL: `{{baseUrl}}/api/tasks`
- Clic en **Send**

Resultado esperado: `200 OK` con 6 tareas (las 5 originales + la nueva)

**Request 5: Eliminar una tarea**
- Método: `DELETE`
- URL: `{{baseUrl}}/api/tasks/1`
- Clic en **Send**

Resultado esperado: `200 OK` con mensaje `{"message":"Tarea eliminada",...}`

**Request 6: Validar error de campo vacío**
- Método: `POST`
- URL: `{{baseUrl}}/api/tasks`
- Headers: `Content-Type: application/json`
- Body: `{"title": ""}`
- Clic en **Send**

Resultado esperado: `400 Bad Request` con `{"error":"El campo title es requerido"}`

**5.4** Prueba la aplicación desde el navegador:
1. Navega a `http://localhost:8080`
2. Escribe una tarea en el campo de texto y haz clic en **Agregar**
3. Verifica que la tarea aparece en la lista
4. Haz clic en **Eliminar** en cualquier tarea y confirma que desaparece

**Verificación:** Todos los requests de Postman deben retornar los códigos HTTP esperados. La interfaz web debe mostrar los cambios en tiempo real al agregar y eliminar tareas.

---

### Paso 6: Publicar Imágenes en Docker Hub

**Objetivo:** Etiquetar y publicar las imágenes construidas en Docker Hub para distribución.

#### Instrucciones

**6.1** Asegúrate de estar autenticado en Docker Hub:

```bash
docker login
# Ingresa tu usuario y contraseña de Docker Hub cuando se solicite
```

**6.2** Identifica el nombre de usuario de Docker Hub que usarás (reemplaza `TU_USUARIO` en todos los comandos siguientes):

```bash
# Verificar usuario actual
docker info | grep Username
```

**6.3** Etiqueta la imagen del backend con tu nombre de usuario de Docker Hub:

```bash
# Formato: docker tag <imagen-local> <usuario-dockerhub>/<nombre-imagen>:<tag>
docker tag lab06-docker-fullstack-api:latest TU_USUARIO/tasks-api:1.0.0
docker tag lab06-docker-fullstack-api:latest TU_USUARIO/tasks-api:latest
```

**6.4** Etiqueta la imagen del frontend:

```bash
docker tag lab06-docker-fullstack-frontend:latest TU_USUARIO/tasks-frontend:1.0.0
docker tag lab06-docker-fullstack-frontend:latest TU_USUARIO/tasks-frontend:latest
```

**6.5** Verifica las imágenes etiquetadas:

```bash
docker images | grep TU_USUARIO
```

**Salida esperada:**

```
TU_USUARIO/tasks-api        1.0.0     abc123def456   2 minutes ago   180MB
TU_USUARIO/tasks-api        latest    abc123def456   2 minutes ago   180MB
TU_USUARIO/tasks-frontend   1.0.0     def456ghi789   2 minutes ago   45MB
TU_USUARIO/tasks-frontend   latest    def456ghi789   2 minutes ago   45MB
```

**6.6** Publica las imágenes en Docker Hub:

```bash
docker push TU_USUARIO/tasks-api:1.0.0
docker push TU_USUARIO/tasks-api:latest
docker push TU_USUARIO/tasks-frontend:1.0.0
docker push TU_USUARIO/tasks-frontend:latest
```

**Salida esperada:** Cada push mostrará las capas siendo subidas. Las capas de la imagen base (node:20-alpine, nginx:alpine) ya existen en Docker Hub y se indicarán como `Layer already exists`.

**6.7** Verifica la publicación en Docker Hub:
1. Abre `https://hub.docker.com` en tu navegador
2. Navega a tu perfil → **Repositories**
3. Debes ver `tasks-api` y `tasks-frontend` con el tag `1.0.0`

**Verificación:**

```bash
# Simular descarga desde Docker Hub (como lo haría otro desarrollador)
docker pull TU_USUARIO/tasks-api:1.0.0
```

---

### Paso 7: Versionar el Proyecto en GitHub

**Objetivo:** Inicializar un repositorio Git local, configurar los archivos de exclusión apropiados y publicar el proyecto en GitHub.

#### Instrucciones

**7.1** Crea el archivo `.gitignore` en el directorio raíz del proyecto:

```
# .gitignore
node_modules/
.env
*.log
.DS_Store
Thumbs.db
dist/
build/
```

**7.2** Inicializa el repositorio Git local:

```bash
git init
git branch -M main
```

**7.3** Agrega todos los archivos al área de staging:

```bash
git add .
git status
```

**Salida esperada:** Debes ver todos los archivos del proyecto listados en verde como "new file". El directorio `node_modules/` y el archivo `.env` NO deben aparecer (están en `.gitignore`).

**7.4** Realiza el primer commit:

```bash
git commit -m "feat: proyecto integrador docker fullstack - lab06

- Frontend estático servido por Nginx Alpine
- API REST Node.js/Express con endpoints CRUD de tareas
- PostgreSQL 16 con script de inicialización SQL
- Dockerfiles optimizados con multi-stage build para API
- Docker Compose con redes y volúmenes persistentes
- Configuración de proxy inverso en Nginx"
```

**7.5** Crea un nuevo repositorio en GitHub:
1. Abre `https://github.com/new`
2. Nombre del repositorio: `lab06-docker-fullstack`
3. Descripción: `Proyecto integrador Docker - Aplicación full stack contenerizada`
4. Visibilidad: **Public**
5. **NO** inicialices con README (el repositorio debe estar vacío)
6. Haz clic en **Create repository**

**7.6** Conecta el repositorio local con GitHub y publica:

```bash
# Reemplaza TU_USUARIO con tu nombre de usuario de GitHub
git remote add origin https://github.com/TU_USUARIO/lab06-docker-fullstack.git
git push -u origin main
```

**7.7** Crea un archivo `README.md` básico y haz un segundo commit:

```bash
cat > README.md << 'EOF'
# Lab 06 — Proyecto Integrador Docker Full Stack

Aplicación de gestión de tareas contenerizada con Docker.

## Servicios
- **Frontend**: Nginx Alpine sirviendo HTML/CSS/JS estático
- **API**: Node.js 20 + Express — CRUD de tareas
- **DB**: PostgreSQL 16 Alpine con persistencia en volumen Docker

## Inicio rápido
```bash
docker compose up -d
```
Accede a: http://localhost:8080

## Imágenes en Docker Hub
- `TU_USUARIO/tasks-api:1.0.0`
- `TU_USUARIO/tasks-frontend:1.0.0`
EOF

git add README.md
git commit -m "docs: agregar README con instrucciones de inicio"
git push
```

**Verificación:**
- Abre `https://github.com/TU_USUARIO/lab06-docker-fullstack` en tu navegador
- Debes ver todos los archivos del proyecto excepto `node_modules` y `.env`
- El `README.md` debe renderizarse correctamente en la página principal del repositorio

---

## 7. Validación y Pruebas Finales

Ejecuta esta secuencia de validación completa para confirmar que el laboratorio está correctamente implementado:

```bash
# ── 1. Verificar que los tres contenedores están en ejecución ─────
docker compose ps
# Resultado esperado: tasks-db (healthy), tasks-api (running), tasks-frontend (running)

# ── 2. Verificar las redes Docker creadas ────────────────────────
docker network ls | grep tasks
# Resultado esperado:
#   tasks-backend-network    bridge
#   tasks-frontend-network   bridge

# ── 3. Verificar el volumen de datos persistente ─────────────────
docker volume ls | grep tasks
# Resultado esperado: tasks-postgres-data

# ── 4. Verificar comunicación interna entre contenedores ─────────
docker exec tasks-api ping -c 2 db
# Resultado esperado: 2 packets transmitted, 2 received

# ── 5. Verificar que la API responde ─────────────────────────────
# Windows
Invoke-WebRequest http://localhost:3000/api/tasks | Select-Object StatusCode
# macOS / Linux
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/api/tasks
# Resultado esperado: 200

# ── 6. Verificar que el frontend es accesible ─────────────────────
# Windows
Invoke-WebRequest http://localhost:8080 | Select-Object StatusCode
# macOS / Linux
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080
# Resultado esperado: 200

# ── 7. Verificar persistencia de datos ───────────────────────────
# Detener y volver a levantar el stack
docker compose down
docker compose up -d
# Esperar 15 segundos y verificar que los datos persisten
curl http://localhost:3000/api/tasks
# Resultado esperado: Las tareas creadas anteriormente siguen existiendo

# ── 8. Verificar imágenes en Docker Hub ──────────────────────────
docker search TU_USUARIO/tasks-api
# Resultado esperado: La imagen aparece en los resultados

# ── 9. Verificar repositorio en GitHub ───────────────────────────
git log --oneline
# Resultado esperado: Al menos 2 commits visibles
git remote -v
# Resultado esperado: origin apuntando a github.com/TU_USUARIO/lab06-docker-fullstack
```

### Lista de Verificación Final

| Criterio                                              | Estado |
|-------------------------------------------------------|--------|
| Los 3 contenedores están en ejecución (`docker compose ps`) | ☐ |
| El frontend es accesible en `http://localhost:8080`   | ☐ |
| La API responde en `http://localhost:3000/api/health` | ☐ |
| Se pueden crear tareas desde la interfaz web          | ☐ |
| Se pueden eliminar tareas desde la interfaz web       | ☐ |
| Los datos persisten después de `docker compose down/up` | ☐ |
| Las imágenes están publicadas en Docker Hub           | ☐ |
| El código está versionado en GitHub con 2+ commits    | ☐ |

---

## 8. Resolución de Problemas

### Problema 1: La API no puede conectarse a PostgreSQL

**Síntoma:** Los logs de la API muestran repetidamente `⏳ Esperando DB... intento N/10` y eventualmente `❌ No se pudo conectar a PostgreSQL después de varios intentos.` El contenedor `tasks-api` entra en estado `Exited`.

**Causa:** El contenedor de PostgreSQL tarda más de lo esperado en inicializarse (especialmente en la primera ejecución cuando debe crear la base de datos y ejecutar el script de inicialización), y la API agota sus reintentos antes de que la DB esté lista. Esto puede ocurrir en equipos con hardware más lento o con discos HDD.

**Solución:**

```bash
# Paso 1: Verificar el estado de salud de la DB
docker compose ps db
# Si muestra "starting" en lugar de "healthy", espera 30 segundos más

# Paso 2: Revisar los logs de PostgreSQL para confirmar que inició correctamente
docker compose logs db | tail -20
# Busca la línea: "database system is ready to accept connections"

# Paso 3: Si la API ya falló, reiniciarla manualmente
docker compose restart api

# Paso 4: Si el problema persiste, aumentar el tiempo de reintento en index.js
# Modifica la línea: async function waitForDB(retries = 10, delay = 3000)
# Cambia a:          async function waitForDB(retries = 20, delay = 5000)
# Luego reconstruye:
docker compose build api
docker compose up -d api

# Paso 5: Alternativa — iniciar servicios en orden con pausa manual
docker compose up -d db
# Esperar 30 segundos
docker compose up -d api
docker compose up -d frontend
```

---

### Problema 2: El navegador muestra "Bad Gateway" al acceder al frontend

**Síntoma:** Al abrir `http://localhost:8080`, Nginx muestra un error `502 Bad Gateway` en lugar de la interfaz de la aplicación. Las peticiones a `http://localhost:3000/api/tasks` sí funcionan directamente.

**Causa:** La configuración del proxy inverso en `nginx.conf` no puede resolver el nombre de host `api` porque el contenedor del frontend no está en la misma red Docker que el contenedor de la API. Esto ocurre cuando el archivo `docker-compose.yml` fue modificado incorrectamente o cuando se levantaron los contenedores de forma manual sin Docker Compose (perdiendo la configuración de redes).

**Solución:**

```bash
# Paso 1: Verificar que ambos contenedores están en la red frontend-net
docker network inspect tasks-frontend-network
# Busca en "Containers" que aparezcan tanto tasks-api como tasks-frontend

# Paso 2: Si falta algún contenedor en la red, verificar el docker-compose.yml
# Confirmar que el servicio "api" tiene la red "frontend-net" en su sección networks:
#   networks:
#     - backend-net
#     - frontend-net   <-- Esta línea debe estar presente

# Paso 3: Reconstruir el stack completo desde cero
docker compose down
docker compose up -d

# Paso 4: Verificar la conectividad desde el contenedor frontend hacia la API
docker exec tasks-frontend wget -qO- http://api:3000/api/health
# Resultado esperado: {"status":"ok",...}
# Si falla: confirma que el nombre del servicio en docker-compose.yml es "api"
# y que nginx.conf usa "proxy_pass http://api:3000/api/;"

# Paso 5: Si modificaste nginx.conf, reconstruye solo el frontend
docker compose build frontend
docker compose up -d frontend
```

---

## 9. Limpieza del Entorno

Una vez completado el laboratorio, ejecuta los siguientes comandos para liberar recursos del sistema:

```bash
# Detener y eliminar los contenedores y redes (conserva el volumen de datos)
docker compose down

# Para eliminar también el volumen de datos (limpieza completa)
docker compose down -v

# Eliminar las imágenes construidas localmente (opcional)
docker rmi lab06-docker-fullstack-api:latest
docker rmi lab06-docker-fullstack-frontend:latest

# Eliminar imágenes etiquetadas para Docker Hub (opcional)
docker rmi TU_USUARIO/tasks-api:1.0.0 TU_USUARIO/tasks-api:latest
docker rmi TU_USUARIO/tasks-frontend:1.0.0 TU_USUARIO/tasks-frontend:latest

# Limpiar imágenes intermedias y caché de construcción (libera espacio en disco)
docker builder prune -f

# Verificar que los contenedores fueron eliminados
docker compose ps
# Resultado esperado: lista vacía o mensaje "no containers"

# Verificar recursos liberados
docker system df
```

> **Nota:** Las imágenes publicadas en Docker Hub y el código en GitHub permanecen disponibles. Solo se eliminan los recursos locales.

---

## 10. Resumen y Reflexión

### Conceptos Aplicados

En este laboratorio implementaste el ciclo completo de contenerización de una aplicación full stack empresarial:

| Concepto                         | Aplicación en el Lab                                              |
|----------------------------------|-------------------------------------------------------------------|
| **Imagen Docker**                | Creaste imágenes para frontend (Nginx) y backend (Node.js)        |
| **Dockerfile optimizado**        | Multi-stage build en backend; imágenes Alpine ligeras             |
| **Capas de caché**               | `package.json` copiado antes que el código fuente                 |
| **Docker Compose**               | Orquestación de 3 servicios con un solo comando                   |
| **Redes Docker**                 | Red `backend-net` (DB↔API) y `frontend-net` (API↔Frontend)        |
| **Volúmenes persistentes**       | `tasks-postgres-data` sobrevive reinicios del contenedor          |
| **Variables de entorno**         | `.env` centraliza configuración sensible                          |
| **Health checks**                | PostgreSQL reporta estado `healthy` antes de iniciar la API       |
| **Proxy inverso**                | Nginx enruta `/api/*` hacia el contenedor de la API               |
| **Docker Hub**                   | Imágenes publicadas y versionadas con tag `1.0.0`                 |
| **GitHub**                       | Código versionado con commits descriptivos                        |

### Flujo Tradicional vs. Flujo con Docker

| Aspecto                        | Sin Docker                                  | Con Docker                                    |
|--------------------------------|---------------------------------------------|-----------------------------------------------|
| Configuración de entorno       | Manual, horas o días                        | `docker compose up -d`, segundos              |
| Onboarding de nuevo integrante | Instalar Node, PostgreSQL, configurar paths | Clonar repo + `docker compose up -d`          |
| "Funciona en mi máquina"       | Problema frecuente                          | Eliminado: mismo contenedor en todos lados    |
| Despliegue a producción        | Scripts manuales, propenso a errores        | Misma imagen testeada en dev                  |
| Escalabilidad                  | Configuración manual de servidores          | `docker compose scale api=3`                  |
| Versiones de software          | Conflictos entre proyectos                  | Cada contenedor tiene sus propias versiones   |

### Próximos Pasos

Este laboratorio sienta las bases para:
- **Kubernetes**: orquestación de contenedores a escala empresarial
- **CI/CD con GitHub Actions**: construir y publicar imágenes automáticamente en cada push
- **Docker Swarm**: alta disponibilidad con múltiples réplicas
- **Integración con Spring Boot**: contenerizar los servicios Java desarrollados en laboratorios anteriores

### Recursos Adicionales

- [Documentación oficial de Docker Compose](https://docs.docker.com/compose/)
- [Referencia de Dockerfile](https://docs.docker.com/engine/reference/builder/)
- [Mejores prácticas para escribir Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker Hub — Repositorio oficial de imágenes](https://hub.docker.com)
- [Play with Docker — Practicar sin instalación](https://labs.play-with-docker.com)
- [Documentación de PostgreSQL en Docker](https://hub.docker.com/_/postgres)
- [Documentación de Nginx en Docker](https://hub.docker.com/_/nginx)

---
