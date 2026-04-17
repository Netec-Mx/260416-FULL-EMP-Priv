# Laboratorio 7. Proyecto Integrador — Aplicación Full Stack con Docker

## 1. Metadatos

| Campo            | Detalle                                      |
|------------------|----------------------------------------------|
| **Duración**     | 80 minutos                                   |
| **Complejidad**  | Alta                                         |
| **Nivel Bloom**  | Crear                                        |
| **Módulo**       | 7 — Integración Full Stack Empresarial       |
| **Laboratorio**  | 07-00-01                                     |

---

## 2. Descripción General

En este laboratorio integrador, el participante une todos los componentes desarrollados a lo largo del curso en una aplicación full stack cohesiva y lista para producción. Se conectará el frontend construido con Lit al backend RESTful (Node.js/Express), se configurará CORS, se verificará la persistencia de datos en base de datos, y se contenerizará toda la solución mediante Docker y Docker Compose. El laboratorio concluye con la validación end-to-end del sistema completo y un repaso de los conceptos clave del módulo.

---

## 3. Objetivos de Aprendizaje

Al completar este laboratorio, el participante será capaz de:

- [ ] Configurar CORS en el backend Express y consumir la API REST desde componentes Lit usando `fetch` con manejo de estados de carga y error.
- [ ] Conectar el backend Node.js/Express a una base de datos (PostgreSQL o MongoDB) y verificar que las operaciones CRUD se persistan correctamente desde el frontend.
- [ ] Crear `Dockerfile` optimizados para el frontend (Nginx) y el backend (Node.js), y orquestar todos los servicios con `docker-compose.yml`.
- [ ] Aplicar buenas prácticas empresariales: variables de entorno con `.env`, `.dockerignore`, health checks y logging estructurado.
- [ ] Validar el funcionamiento integral del sistema mediante pruebas end-to-end con Postman y el navegador.

---

## 4. Prerrequisitos

### Conocimiento previo
- Laboratorio 2 completado: proyecto frontend funcional con al menos un componente Lit que liste y cree registros.
- Comprensión de métodos HTTP (GET, POST, PUT, DELETE) y códigos de estado (200, 201, 400, 404, 500).
- Backend Node.js/Express con endpoints CRUD disponibles (desarrollado en módulos anteriores).
- Familiaridad básica con Docker: imágenes, contenedores, puertos.

### Acceso y herramientas requeridas
- Docker Desktop 4.29+ instalado y en ejecución (`docker --version` y `docker compose version` sin errores).
- Node.js 20.x LTS y npm disponibles en terminal.
- Postman 11.x (o Bruno) para pruebas de API.
- Editor de código: VS Code 1.88+ o IntelliJ IDEA 2024.1.
- Repositorio Git del proyecto del curso con acceso de escritura.

---

## 5. Entorno del Laboratorio

### Hardware mínimo recomendado

| Recurso        | Mínimo              | Recomendado         |
|----------------|---------------------|---------------------|
| RAM            | 16 GB               | 32 GB               |
| Almacenamiento | 50 GB libres (SSD)  | 100 GB libres (SSD) |
| CPU            | i5 8va gen / Ryzen 5| i7 10ma gen / Ryzen 7|
| Pantalla       | 1920×1080           | Dual monitor        |

### Software y versiones

| Herramienta        | Versión      | Propósito                        |
|--------------------|--------------|----------------------------------|
| Node.js            | 20.x LTS     | Runtime backend                  |
| Lit                | 3.x          | Framework frontend               |
| Docker Engine      | 24.x         | Contenerización                  |
| Docker Compose     | 2.x          | Orquestación de servicios        |
| Nginx              | 1.25-alpine  | Servidor estático para frontend  |
| PostgreSQL         | 15.x         | Base de datos relacional         |
| MongoDB            | 7.0          | Base de datos NoSQL (alternativa)|
| Postman            | 11.x         | Pruebas de API                   |

### Verificación del entorno antes de comenzar

Ejecuta los siguientes comandos en tu terminal para confirmar que el entorno está listo:

```bash
# Verificar Node.js
node --version
# Esperado: v20.x.x

# Verificar npm
npm --version
# Esperado: 10.x.x

# Verificar Docker
docker --version
# Esperado: Docker version 24.x.x

# Verificar Docker Compose
docker compose version
# Esperado: Docker Compose version v2.x.x

# Verificar que Docker está corriendo
docker info | grep "Server Version"
```

### Estructura del proyecto al inicio del laboratorio

```
proyecto-fullstack/
├── frontend/          ← Proyecto Lit del Lab 2
│   ├── src/
│   ├── package.json
│   └── ...
├── backend/           ← API Node.js/Express de módulos anteriores
│   ├── src/
│   ├── package.json
│   └── ...
└── README.md
```

> **Checkpoint Git**: Si no tienes el proyecto de los laboratorios anteriores, clona la rama de referencia:
> ```bash
> git clone -b checkpoint/lab-06-complete https://github.com/<tu-org>/proyecto-fullstack.git
> cd proyecto-fullstack
> ```

---

## 6. Pasos del Laboratorio

### Paso 1: Configurar CORS en el Backend Express

**Objetivo**: Permitir que el frontend (corriendo en un puerto diferente) pueda consumir la API del backend sin errores de CORS.

#### Instrucciones

1. Abre una terminal y navega al directorio del backend:

   ```bash
   cd proyecto-fullstack/backend
   ```

2. Instala el paquete `cors` si no está ya en las dependencias:

   ```bash
   npm install cors
   ```

3. Abre el archivo principal del servidor (usualmente `src/app.js` o `src/index.js`) y agrega la configuración de CORS. Reemplaza o actualiza el bloque de configuración de middlewares:

   ```javascript
   // src/app.js
   const express = require('express');
   const cors = require('cors');
   const app = express();

   // ─── Configuración de CORS ────────────────────────────────────────────────
   const corsOptions = {
     // En desarrollo: permite el origen del servidor de desarrollo de Vite/Lit
     // En producción: reemplazar con el dominio real del frontend
     origin: process.env.FRONTEND_URL || 'http://localhost:5173',
     methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
     allowedHeaders: ['Content-Type', 'Authorization'],
     credentials: true
   };

   app.use(cors(corsOptions));
   app.use(express.json());

   // ─── Rutas ───────────────────────────────────────────────────────────────
   const employeeRoutes = require('./routes/employees');
   app.use('/api/employees', employeeRoutes);

   // ─── Manejo global de errores ─────────────────────────────────────────────
   app.use((err, req, res, next) => {
     console.error(`[ERROR] ${new Date().toISOString()} - ${err.message}`);
     res.status(err.status || 500).json({
       error: {
         code: err.code || 'INTERNAL_ERROR',
         message: err.message || 'Error interno del servidor'
       }
     });
   });

   module.exports = app;
   ```

4. Crea el archivo `.env` en la raíz del directorio `backend/` con las variables de entorno:

   ```bash
   # En la raíz de backend/
   touch .env     # Linux/macOS
   # En Windows PowerShell:
   # New-Item -Path ".env" -ItemType File
   ```

   Contenido del archivo `.env`:

   ```dotenv
   # backend/.env
   NODE_ENV=development
   PORT=3000
   FRONTEND_URL=http://localhost:5173

   # PostgreSQL (usar una u otra sección según tu stack)
   DB_HOST=localhost
   DB_PORT=5432
   DB_NAME=empresa_db
   DB_USER=postgres
   DB_PASSWORD=postgres123

   # MongoDB (alternativa)
   # MONGO_URI=mongodb://localhost:27017/empresa_db
   ```

5. Instala `dotenv` y cárgalo al inicio de tu archivo de entrada (`src/server.js` o `src/index.js`):

   ```bash
   npm install dotenv
   ```

   ```javascript
   // src/server.js (punto de entrada principal)
   require('dotenv').config();
   const app = require('./app');

   const PORT = process.env.PORT || 3000;

   app.listen(PORT, () => {
     console.log(`[INFO] Servidor iniciado en http://localhost:${PORT}`);
     console.log(`[INFO] Ambiente: ${process.env.NODE_ENV}`);
   });
   ```

6. Inicia el backend en modo desarrollo para verificar:

   ```bash
   npm run dev
   # o si no tienes script dev:
   node src/server.js
   ```

#### Resultado esperado

```
[INFO] Servidor iniciado en http://localhost:3000
[INFO] Ambiente: development
```

#### Verificación

Abre Postman y realiza una solicitud GET a `http://localhost:3000/api/employees`. Debes recibir un código `200 OK` con un array JSON. Verifica también que los headers de respuesta incluyan `Access-Control-Allow-Origin`.

---

### Paso 2: Conectar el Componente Lit a la API REST

**Objetivo**: Actualizar el componente Lit principal para consumir la API del backend con manejo correcto de estados de carga, error y datos.

#### Instrucciones

1. Navega al directorio del frontend:

   ```bash
   cd ../frontend
   ```

2. Crea un archivo de configuración de la API para centralizar la URL base. Esto facilita el cambio entre ambientes:

   ```bash
   # Crear archivo de configuración
   mkdir -p src/config
   touch src/config/api.js
   ```

   ```javascript
   // src/config/api.js
   // La variable de entorno se inyecta en tiempo de build (Vite)
   export const API_BASE_URL = import.meta.env.VITE_API_URL || 'http://localhost:3000/api';
   ```

3. Crea un archivo de utilidades para las llamadas HTTP con manejo robusto de errores:

   ```bash
   mkdir -p src/services
   touch src/services/api-client.js
   ```

   ```javascript
   // src/services/api-client.js
   import { API_BASE_URL } from '../config/api.js';

   /**
    * Cliente HTTP genérico con manejo de errores.
    * @param {string} endpoint - Ruta relativa, ej: '/employees'
    * @param {RequestInit} options - Opciones del fetch
    * @returns {Promise<any>} Datos de la respuesta JSON
    */
   async function request(endpoint, options = {}) {
     const url = `${API_BASE_URL}${endpoint}`;
     const defaultHeaders = {
       'Content-Type': 'application/json',
     };

     const config = {
       ...options,
       headers: { ...defaultHeaders, ...options.headers },
     };

     try {
       const response = await fetch(url, config);

       // Verificar errores HTTP (4xx, 5xx)
       if (!response.ok) {
         let errorMessage = `Error HTTP ${response.status}`;
         try {
           const errorBody = await response.json();
           errorMessage = errorBody?.error?.message || errorMessage;
         } catch {
           // El cuerpo no es JSON; usamos el mensaje por defecto
         }
         throw new Error(errorMessage);
       }

       // 204 No Content: no hay body que parsear
       if (response.status === 204) return null;

       return await response.json();

     } catch (networkError) {
       // Error de red (servidor caído, sin conexión, etc.)
       if (networkError.name === 'TypeError') {
         throw new Error('No se puede conectar al servidor. Verifique que el backend esté en ejecución.');
       }
       throw networkError;
     }
   }

   // ─── Métodos de conveniencia ──────────────────────────────────────────────
   export const apiClient = {
     get:    (endpoint)         => request(endpoint, { method: 'GET' }),
     post:   (endpoint, body)   => request(endpoint, { method: 'POST',   body: JSON.stringify(body) }),
     put:    (endpoint, body)   => request(endpoint, { method: 'PUT',    body: JSON.stringify(body) }),
     delete: (endpoint)         => request(endpoint, { method: 'DELETE' }),
   };
   ```

4. Actualiza el componente principal de lista de empleados (ajusta el nombre del archivo según tu proyecto del Lab 2):

   ```javascript
   // src/components/employee-list.js
   import { LitElement, html, css } from 'lit';
   import { apiClient } from '../services/api-client.js';

   export class EmployeeList extends LitElement {
     static properties = {
       employees: { type: Array },
       loading:   { type: Boolean },
       error:     { type: String },
     };

     static styles = css`
       :host { display: block; font-family: sans-serif; padding: 1rem; }
       .loading { color: #666; font-style: italic; }
       .error   { color: #c0392b; background: #fdecea; padding: 0.75rem; border-radius: 4px; }
       .retry-btn {
         margin-top: 0.5rem; padding: 0.4rem 1rem; cursor: pointer;
         background: #c0392b; color: white; border: none; border-radius: 4px;
       }
       table  { width: 100%; border-collapse: collapse; }
       th, td { padding: 0.6rem 1rem; border: 1px solid #ddd; text-align: left; }
       th     { background: #2c3e50; color: white; }
       tr:nth-child(even) { background: #f9f9f9; }
     `;

     constructor() {
       super();
       this.employees = [];
       this.loading = false;
       this.error = '';
     }

     connectedCallback() {
       super.connectedCallback();
       this.loadEmployees();
     }

     async loadEmployees() {
       this.loading = true;
       this.error = '';
       try {
         this.employees = await apiClient.get('/employees');
       } catch (err) {
         this.error = err.message;
       } finally {
         this.loading = false;
       }
     }

     async deleteEmployee(id) {
       if (!confirm('¿Eliminar este empleado?')) return;
       try {
         await apiClient.delete(`/employees/${id}`);
         // Actualizar la lista localmente sin recargar todo
         this.employees = this.employees.filter(e => e.id !== id);
       } catch (err) {
         this.error = `Error al eliminar: ${err.message}`;
       }
     }

     render() {
       if (this.loading) {
         return html`<p class="loading">⏳ Cargando empleados...</p>`;
       }

       if (this.error) {
         return html`
           <div class="error">
             <strong>⚠️ Error:</strong> ${this.error}
             <br />
             <button class="retry-btn" @click=${this.loadEmployees}>Reintentar</button>
           </div>
         `;
       }

       if (this.employees.length === 0) {
         return html`<p>No hay empleados registrados.</p>`;
       }

       return html`
         <h2>Lista de Empleados (${this.employees.length})</h2>
         <table>
           <thead>
             <tr><th>ID</th><th>Nombre</th><th>Departamento</th><th>Acciones</th></tr>
           </thead>
           <tbody>
             ${this.employees.map(emp => html`
               <tr>
                 <td>${emp.id}</td>
                 <td>${emp.name}</td>
                 <td>${emp.department}</td>
                 <td>
                   <button @click=${() => this.deleteEmployee(emp.id)}>Eliminar</button>
                 </td>
               </tr>
             `)}
           </tbody>
         </table>
       `;
     }
   }

   customElements.define('employee-list', EmployeeList);
   ```

5. Crea el archivo `.env` en la raíz del directorio `frontend/`:

   ```dotenv
   # frontend/.env
   VITE_API_URL=http://localhost:3000/api
   ```

6. Inicia el servidor de desarrollo del frontend:

   ```bash
   npm run dev
   ```

#### Resultado esperado

El servidor de desarrollo de Vite arranca en `http://localhost:5173`. Al abrir el navegador, el componente `<employee-list>` debe mostrar los datos provenientes del backend. La consola del navegador no debe mostrar errores CORS.

#### Verificación

1. Abre `http://localhost:5173` en el navegador.
2. Abre las DevTools (F12) → pestaña **Network**.
3. Recarga la página y verifica que la solicitud a `http://localhost:3000/api/employees` devuelve `200 OK`.
4. Detén el backend (`Ctrl+C`) y recarga la página: debe aparecer el mensaje de error con el botón "Reintentar".

---

### Paso 3: Crear los Dockerfiles para Frontend y Backend

**Objetivo**: Contenerizar el frontend y el backend con imágenes Docker optimizadas para producción.

#### Instrucciones

##### 3a. Dockerfile del Backend (Node.js/Express)

1. Crea el archivo `Dockerfile` en `backend/`:

   ```dockerfile
   # backend/Dockerfile
   # ─── Etapa 1: Dependencias ────────────────────────────────────────────────
   FROM node:20-alpine AS dependencies
   WORKDIR /app
   COPY package*.json ./
   # Instalar solo dependencias de producción
   RUN npm ci --only=production

   # ─── Etapa 2: Imagen final ────────────────────────────────────────────────
   FROM node:20-alpine AS production
   WORKDIR /app

   # Copiar dependencias ya instaladas
   COPY --from=dependencies /app/node_modules ./node_modules

   # Copiar código fuente
   COPY src/ ./src/
   COPY package.json ./

   # Usuario no-root para seguridad
   RUN addgroup -S appgroup && adduser -S appuser -G appgroup
   USER appuser

   EXPOSE 3000

   # Health check: verifica que el servidor responde
   HEALTHCHECK --interval=30s --timeout=10s --start-period=15s --retries=3 \
     CMD wget -qO- http://localhost:3000/health || exit 1

   CMD ["node", "src/server.js"]
   ```

2. Agrega el endpoint `/health` al backend si no existe. Abre `src/app.js`:

   ```javascript
   // Agregar antes de las rutas de negocio en src/app.js
   app.get('/health', (req, res) => {
     res.status(200).json({
       status: 'ok',
       timestamp: new Date().toISOString(),
       uptime: process.uptime()
     });
   });
   ```

3. Crea el archivo `.dockerignore` en `backend/`:

   ```
   # backend/.dockerignore
   node_modules
   .env
   .env.*
   npm-debug.log
   .git
   .gitignore
   README.md
   *.test.js
   coverage/
   ```

##### 3b. Dockerfile del Frontend (Lit + Vite + Nginx)

1. Crea el archivo `Dockerfile` en `frontend/`:

   ```dockerfile
   # frontend/Dockerfile
   # ─── Etapa 1: Build de producción ────────────────────────────────────────
   FROM node:20-alpine AS builder
   WORKDIR /app

   COPY package*.json ./
   RUN npm ci

   COPY . .

   # La URL de la API se inyecta en tiempo de build
   ARG VITE_API_URL=http://localhost:3000/api
   ENV VITE_API_URL=${VITE_API_URL}

   RUN npm run build

   # ─── Etapa 2: Servidor Nginx para archivos estáticos ─────────────────────
   FROM nginx:1.25-alpine AS production

   # Copiar el build generado por Vite
   COPY --from=builder /app/dist /usr/share/nginx/html

   # Configuración de Nginx para SPA (Single Page Application)
   COPY nginx.conf /etc/nginx/conf.d/default.conf

   EXPOSE 80

   HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
     CMD wget -qO- http://localhost:80/ || exit 1

   CMD ["nginx", "-g", "daemon off;"]
   ```

2. Crea el archivo de configuración de Nginx en `frontend/nginx.conf`:

   ```nginx
   # frontend/nginx.conf
   server {
       listen 80;
       server_name localhost;
       root /usr/share/nginx/html;
       index index.html;

       # Compresión gzip para mejor rendimiento
       gzip on;
       gzip_types text/plain text/css application/javascript application/json;

       # Manejo de rutas SPA: redirige todo al index.html
       location / {
           try_files $uri $uri/ /index.html;
       }

       # Cache para assets estáticos con hash en el nombre
       location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff2)$ {
           expires 1y;
           add_header Cache-Control "public, immutable";
       }

       # Deshabilitar logs de health check para no saturar logs
       location /nginx-health {
           access_log off;
           return 200 "healthy\n";
           add_header Content-Type text/plain;
       }
   }
   ```

3. Crea el archivo `.dockerignore` en `frontend/`:

   ```
   # frontend/.dockerignore
   node_modules
   dist
   .env
   .env.*
   .git
   .gitignore
   README.md
   *.test.js
   coverage/
   ```

#### Resultado esperado

Los archivos `Dockerfile`, `.dockerignore` y (para el frontend) `nginx.conf` están creados en sus respectivos directorios. No se producen errores al escribir los archivos.

#### Verificación

Verifica que los archivos existen:

```bash
# Desde la raíz del proyecto
ls backend/Dockerfile backend/.dockerignore
ls frontend/Dockerfile frontend/.dockerignore frontend/nginx.conf
```

---

### Paso 4: Crear el archivo `docker-compose.yml`

**Objetivo**: Definir y orquestar todos los servicios (frontend, backend, base de datos) con redes internas, volúmenes persistentes y variables de entorno correctamente configuradas.

#### Instrucciones

1. En la **raíz** del proyecto (`proyecto-fullstack/`), crea el archivo `docker-compose.yml`:

   ```yaml
   # docker-compose.yml
   # Orquestación completa: frontend + backend + base de datos
   version: '3.9'

   services:

     # ─── Base de datos PostgreSQL ──────────────────────────────────────────
     database:
       image: postgres:15-alpine
       container_name: empresa-db
       restart: unless-stopped
       environment:
         POSTGRES_DB:       ${DB_NAME:-empresa_db}
         POSTGRES_USER:     ${DB_USER:-postgres}
         POSTGRES_PASSWORD: ${DB_PASSWORD:-postgres123}
       volumes:
         - db-data:/var/lib/postgresql/data
         # Script de inicialización (se ejecuta solo la primera vez)
         - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
       networks:
         - backend-net
       healthcheck:
         test: ["CMD-SHELL", "pg_isready -U ${DB_USER:-postgres} -d ${DB_NAME:-empresa_db}"]
         interval: 10s
         timeout: 5s
         retries: 5
         start_period: 20s
       ports:
         # Exponer solo en desarrollo para acceso desde pgAdmin local
         - "5432:5432"

     # ─── Backend Node.js/Express ───────────────────────────────────────────
     backend:
       build:
         context: ./backend
         dockerfile: Dockerfile
       container_name: empresa-backend
       restart: unless-stopped
       environment:
         NODE_ENV:     production
         PORT:         3000
         FRONTEND_URL: http://localhost:80
         DB_HOST:      database          # Nombre del servicio en la red interna
         DB_PORT:      5432
         DB_NAME:      ${DB_NAME:-empresa_db}
         DB_USER:      ${DB_USER:-postgres}
         DB_PASSWORD:  ${DB_PASSWORD:-postgres123}
       networks:
         - backend-net
         - frontend-net
       depends_on:
         database:
           condition: service_healthy   # Esperar a que la DB esté lista
       healthcheck:
         test: ["CMD-SHELL", "wget -qO- http://localhost:3000/health || exit 1"]
         interval: 30s
         timeout: 10s
         retries: 3
         start_period: 15s
       ports:
         # Exponer solo en desarrollo para pruebas con Postman
         - "3000:3000"

     # ─── Frontend Lit + Nginx ──────────────────────────────────────────────
     frontend:
       build:
         context: ./frontend
         dockerfile: Dockerfile
         args:
           # En producción Docker, el backend es accesible por el host
           # En una red real, sería el dominio público del backend
           VITE_API_URL: http://localhost:3000/api
       container_name: empresa-frontend
       restart: unless-stopped
       networks:
         - frontend-net
       depends_on:
         backend:
           condition: service_healthy
       ports:
         - "80:80"
       healthcheck:
         test: ["CMD-SHELL", "wget -qO- http://localhost:80/nginx-health || exit 1"]
         interval: 30s
         timeout: 5s
         retries: 3
         start_period: 10s

   # ─── Redes ─────────────────────────────────────────────────────────────────
   networks:
     backend-net:
       name: empresa-backend-net
       driver: bridge
     frontend-net:
       name: empresa-frontend-net
       driver: bridge

   # ─── Volúmenes ─────────────────────────────────────────────────────────────
   volumes:
     db-data:
       name: empresa-db-data
   ```

2. Crea el archivo `.env` en la **raíz** del proyecto para las variables compartidas:

   ```dotenv
   # .env (raíz del proyecto)
   # Variables para Docker Compose
   DB_NAME=empresa_db
   DB_USER=postgres
   DB_PASSWORD=postgres123
   ```

3. Crea el directorio y el script de inicialización de la base de datos:

   ```bash
   mkdir -p database
   touch database/init.sql
   ```

   ```sql
   -- database/init.sql
   -- Script de inicialización ejecutado al crear el contenedor por primera vez

   CREATE TABLE IF NOT EXISTS employees (
     id          SERIAL PRIMARY KEY,
     name        VARCHAR(100) NOT NULL,
     email       VARCHAR(150) UNIQUE NOT NULL,
     department  VARCHAR(80),
     salary      NUMERIC(10, 2),
     created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );

   -- Datos de prueba
   INSERT INTO employees (name, email, department, salary) VALUES
     ('Ana García',    'ana.garcia@empresa.com',    'Ingeniería',  75000.00),
     ('Carlos López',  'carlos.lopez@empresa.com',  'Marketing',   65000.00),
     ('María Rodríguez','maria.rodriguez@empresa.com','RRHH',      60000.00)
   ON CONFLICT (email) DO NOTHING;
   ```

4. Agrega un archivo `.gitignore` en la raíz para proteger los secretos:

   ```
   # .gitignore (raíz)
   .env
   .env.*
   !.env.example
   ```

5. Crea un archivo `.env.example` como documentación de las variables requeridas:

   ```dotenv
   # .env.example — Copiar como .env y completar con valores reales
   DB_NAME=empresa_db
   DB_USER=postgres
   DB_PASSWORD=CAMBIAR_EN_PRODUCCION
   ```

#### Resultado esperado

La estructura del proyecto debe verse así:

```
proyecto-fullstack/
├── .env                    ← Variables de entorno (NO commitear)
├── .env.example            ← Plantilla de variables (sí commitear)
├── .gitignore
├── docker-compose.yml
├── database/
│   └── init.sql
├── frontend/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── nginx.conf
│   └── ...
└── backend/
    ├── Dockerfile
    ├── .dockerignore
    └── ...
```

#### Verificación

```bash
# Validar la sintaxis del docker-compose.yml sin ejecutarlo
docker compose config

# Debe imprimir la configuración expandida sin errores
```

---

### Paso 5: Construir y Levantar la Aplicación Completa

**Objetivo**: Construir las imágenes Docker y levantar todos los servicios orquestados, verificando que la comunicación entre contenedores funciona correctamente.

#### Instrucciones

1. Desde la **raíz** del proyecto, construye todas las imágenes (la primera vez tarda varios minutos):

   ```bash
   docker compose build --no-cache
   ```

   > **Nota de tiempo**: La construcción puede tardar 3–8 minutos dependiendo de la velocidad de internet y el hardware. El flag `--no-cache` garantiza una construcción limpia.

2. Levanta todos los servicios en modo detached (background):

   ```bash
   docker compose up -d
   ```

3. Monitorea el estado de los servicios y espera a que todos estén `healthy`:

   ```bash
   # Ver estado de los contenedores
   docker compose ps

   # Seguir los logs en tiempo real
   docker compose logs -f

   # Ver logs de un servicio específico
   docker compose logs -f backend
   ```

4. Verifica la salud de cada servicio individualmente:

   ```bash
   # Health check del backend
   curl http://localhost:3000/health
   # Esperado: {"status":"ok","timestamp":"...","uptime":...}

   # Health check del frontend
   curl http://localhost:80/nginx-health
   # Esperado: healthy

   # Verificar conexión a la base de datos (desde el contenedor backend)
   docker compose exec backend wget -qO- http://localhost:3000/api/employees
   ```

5. Abre el navegador en `http://localhost:80` y verifica que la aplicación carga correctamente.

#### Resultado esperado

```bash
$ docker compose ps
NAME                 IMAGE                     STATUS                    PORTS
empresa-db           postgres:15-alpine        Up (healthy)              0.0.0.0:5432->5432/tcp
empresa-backend      proyecto-backend          Up (healthy)              0.0.0.0:3000->3000/tcp
empresa-frontend     proyecto-frontend         Up (healthy)              0.0.0.0:80->80/tcp
```

#### Verificación

```bash
# Todos los servicios deben aparecer como "healthy"
docker compose ps --format "table {{.Name}}\t{{.Status}}"
```

---

### Paso 6: Validar el CRUD Completo End-to-End

**Objetivo**: Confirmar que las operaciones Create, Read, Update y Delete funcionan de forma completa desde el frontend hasta la base de datos, pasando por el backend.

#### Instrucciones

1. **READ — Leer empleados**: Abre `http://localhost:80` en el navegador. La lista de empleados debe mostrar los 3 registros del script `init.sql`.

2. **CREATE — Crear un empleado**: En Postman, crea una solicitud POST:

   ```
   POST http://localhost:3000/api/employees
   Content-Type: application/json

   {
     "name": "Luis Fernández",
     "email": "luis.fernandez@empresa.com",
     "department": "Tecnología",
     "salary": 80000.00
   }
   ```

   Respuesta esperada: `201 Created` con el objeto creado incluyendo el `id` asignado.

3. **Verificar persistencia**: Recarga la página en `http://localhost:80`. El nuevo empleado debe aparecer en la lista.

4. **UPDATE — Actualizar un empleado**: En Postman, actualiza el empleado recién creado (reemplaza `{id}` con el ID obtenido):

   ```
   PUT http://localhost:3000/api/employees/{id}
   Content-Type: application/json

   {
     "salary": 85000.00
   }
   ```

   Respuesta esperada: `200 OK` con el objeto actualizado.

5. **DELETE — Eliminar desde el frontend**: En la interfaz web, haz clic en "Eliminar" en el empleado "Luis Fernández". Confirma el diálogo. El empleado debe desaparecer de la lista sin recargar la página.

6. **Verificar persistencia del DELETE**: Recarga la página. El empleado eliminado no debe reaparecer.

7. **Probar manejo de errores**: En Postman, intenta crear un empleado con un email duplicado:

   ```
   POST http://localhost:3000/api/employees
   Content-Type: application/json

   {
     "name": "Duplicado",
     "email": "ana.garcia@empresa.com",
     "department": "Test",
     "salary": 0
   }
   ```

   Respuesta esperada: `409 Conflict` con un mensaje de error estructurado.

#### Resultado esperado

Todas las operaciones CRUD funcionan correctamente. Los datos persisten en el volumen Docker `empresa-db-data` y sobreviven reinicios del contenedor.

#### Verificación

```bash
# Conectar directamente a PostgreSQL y verificar los datos
docker compose exec database psql -U postgres -d empresa_db \
  -c "SELECT id, name, email, department FROM employees ORDER BY id;"
```

---

## 7. Validación y Pruebas

### Lista de verificación final

Completa esta lista antes de dar el laboratorio por terminado:

| # | Verificación | Comando / Acción | Resultado esperado |
|---|-------------|------------------|--------------------|
| 1 | Todos los contenedores `healthy` | `docker compose ps` | 3 servicios en estado `healthy` |
| 2 | Frontend accesible | Abrir `http://localhost:80` | Lista de empleados visible |
| 3 | API responde correctamente | `curl http://localhost:3000/health` | `{"status":"ok",...}` |
| 4 | CORS configurado | DevTools Network → solicitud a API | No hay errores CORS |
| 5 | CREATE funciona | POST en Postman | `201 Created` |
| 6 | DELETE funciona desde UI | Clic en botón Eliminar | Registro desaparece sin recargar |
| 7 | Datos persisten | Reiniciar contenedores y verificar | Datos siguen presentes |
| 8 | Variables de entorno | `docker compose exec backend env \| grep DB` | Variables de DB visibles |

### Prueba de persistencia de datos

```bash
# Detener y reiniciar los contenedores (sin eliminar volúmenes)
docker compose stop
docker compose start

# Esperar 30 segundos y verificar que los datos persisten
sleep 30
curl http://localhost:3000/api/employees | python3 -m json.tool
```

### Prueba de resiliencia

```bash
# Simular caída del backend y verificar el mensaje de error en el frontend
docker compose stop backend

# Recargar http://localhost:80 → debe mostrar mensaje de error con botón "Reintentar"

# Reiniciar el backend
docker compose start backend

# Esperar el health check (~30s) y probar el botón "Reintentar" en el frontend
```

---

## 8. Solución de Problemas

### Problema 1: Error CORS al consumir la API desde el frontend

**Síntoma**: En la consola del navegador aparece el error:
```
Access to fetch at 'http://localhost:3000/api/employees' from origin
'http://localhost:5173' has been blocked by CORS policy:
No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

**Causa**: El middleware `cors` no está configurado correctamente en el backend, o el origen del frontend no coincide con el valor en `FRONTEND_URL`. Esto ocurre frecuentemente cuando se cambia el puerto del servidor de desarrollo de Vite (por defecto 5173, pero puede cambiar si el puerto está ocupado).

**Solución**:

1. Verifica el puerto real en el que corre el frontend:
   ```bash
   # Observa la salida de npm run dev
   # Ejemplo: "Local: http://localhost:5174" (puerto diferente al esperado)
   ```

2. Actualiza el archivo `.env` del backend con el puerto correcto:
   ```dotenv
   FRONTEND_URL=http://localhost:5174
   ```

3. En desarrollo, como alternativa temporal, permite todos los orígenes:
   ```javascript
   // src/app.js — SOLO para desarrollo, nunca en producción
   app.use(cors({ origin: '*' }));
   ```

4. Reinicia el servidor backend:
   ```bash
   # En desarrollo local
   npm run dev

   # Con Docker
   docker compose restart backend
   ```

5. Recarga la página del frontend. El error CORS debe desaparecer.

---

### Problema 2: El contenedor del backend falla con `depends_on` pero la base de datos no está lista

**Síntoma**: Al ejecutar `docker compose up`, el backend arranca pero falla con un error similar a:
```
Error: connect ECONNREFUSED 127.0.0.1:5432
```
o
```
error: database "empresa_db" does not exist
```
El contenedor `empresa-backend` aparece como `Exited` o `Restarting`.

**Causa**: Docker Compose con `depends_on` solo espera a que el contenedor de la base de datos esté **iniciado**, no a que PostgreSQL esté **listo para aceptar conexiones**. PostgreSQL puede tardar 10–20 segundos en inicializarse completamente, especialmente en la primera ejecución cuando crea los archivos de datos.

**Solución**:

1. Verifica el estado del health check de la base de datos:
   ```bash
   docker compose ps database
   # Si aparece "(health: starting)", espera 30 segundos más
   ```

2. Confirma que el `docker-compose.yml` tiene la condición correcta en `depends_on`:
   ```yaml
   backend:
     depends_on:
       database:
         condition: service_healthy   # ← Esto es crítico
   ```
   Si solo dice `depends_on: [database]` sin la condición, cámbialo a la forma extendida mostrada arriba.

3. Agrega lógica de reintento en el backend para manejar la latencia de inicio. Instala `pg-retry` o implementa un loop de reconexión en la configuración de tu pool de conexiones:

   ```javascript
   // src/database/connection.js
   const { Pool } = require('pg');

   const pool = new Pool({
     host:     process.env.DB_HOST,
     port:     process.env.DB_PORT,
     database: process.env.DB_NAME,
     user:     process.env.DB_USER,
     password: process.env.DB_PASSWORD,
     // Reintentar conexiones automáticamente
     connectionTimeoutMillis: 5000,
     idleTimeoutMillis:       30000,
     max:                     10,
   });

   // Verificar la conexión al iniciar
   pool.connect((err, client, release) => {
     if (err) {
       console.error('[DB] Error al conectar:', err.message);
       process.exit(1); // Docker reiniciará el contenedor (restart: unless-stopped)
     }
     release();
     console.log('[DB] Conexión establecida correctamente');
   });

   module.exports = pool;
   ```

4. Reconstruye y levanta los servicios:
   ```bash
   docker compose down
   docker compose up -d
   docker compose logs -f backend
   ```

---

## 9. Limpieza

### Detener los servicios sin eliminar datos

```bash
# Detener contenedores (los volúmenes y datos se conservan)
docker compose stop

# Verificar que los contenedores están detenidos
docker compose ps
```

### Detener y eliminar contenedores (conservando volúmenes)

```bash
# Eliminar contenedores y redes, pero conservar el volumen de datos
docker compose down

# Los datos en 'empresa-db-data' persisten para la próxima sesión
docker volume ls | grep empresa
```

### Limpieza completa (incluyendo datos)

> ⚠️ **Advertencia**: Este comando elimina todos los datos de la base de datos. Úsalo solo al finalizar el laboratorio o cuando quieras empezar desde cero.

```bash
# Eliminar contenedores, redes Y volúmenes
docker compose down -v

# Verificar que el volumen fue eliminado
docker volume ls | grep empresa
# No debe aparecer ningún resultado
```

### Limpiar imágenes construidas (opcional)

```bash
# Eliminar las imágenes construidas para este proyecto
docker compose down --rmi local

# Limpiar imágenes sin usar (dangling)
docker image prune -f
```

### Comandos por sistema operativo

| Acción | Windows (PowerShell) | macOS / Linux (Bash) |
|--------|---------------------|----------------------|
| Detener servicios | `docker compose stop` | `docker compose stop` |
| Eliminar contenedores | `docker compose down` | `docker compose down` |
| Limpieza completa | `docker compose down -v` | `docker compose down -v` |
| Ver logs | `docker compose logs` | `docker compose logs` |

---

## 10. Resumen

### Lo que construiste en este laboratorio

En este laboratorio integrador completaste el ciclo completo de una aplicación empresarial full stack:

1. **Integración Frontend-Backend**: Configuraste CORS en Express y creaste un cliente HTTP centralizado en el frontend Lit que maneja estados de carga, error y datos de forma profesional, siguiendo el patrón cliente-servidor estudiado en la lección 7.1.

2. **Comunicación HTTP robusta**: Implementaste el patrón `fetch` + `async/await` con verificación explícita de `response.ok`, distinguiendo entre errores de red y errores HTTP, tal como se explicó en la lección.

3. **Contenerización completa**: Creaste Dockerfiles multi-stage optimizados para producción (Node.js + Nginx), garantizando imágenes ligeras y seguras.

4. **Orquestación con Docker Compose**: Definiste un stack completo con tres servicios, redes internas separadas, volúmenes persistentes y health checks que garantizan el orden correcto de inicio.

5. **Buenas prácticas empresariales**: Aplicaste variables de entorno con `.env`, `.dockerignore` para imágenes optimizadas, usuario no-root en los contenedores y logging estructurado.

### Conceptos clave reforzados

| Concepto | Aplicación en el laboratorio |
|----------|------------------------------|
| Modelo cliente-servidor | Frontend Lit → API Express → PostgreSQL |
| CORS | Middleware `cors` con origen específico por ambiente |
| Manejo de errores | `response.ok`, mensajes de error al usuario, botón de reintento |
| Variables de entorno | `.env` separado por capa (frontend, backend, raíz) |
| Docker multi-stage | Build de Vite + Nginx en una sola imagen optimizada |
| Health checks | Orquestación confiable con `condition: service_healthy` |
| Persistencia | Volumen Docker nombrado para datos de PostgreSQL |

### Recursos adicionales

- **Docker Compose Reference**: Documentación oficial de todas las opciones de `docker-compose.yml`. [https://docs.docker.com/compose/compose-file/](https://docs.docker.com/compose/compose-file/)
- **Nginx Configuration Guide**: Configuración de Nginx para aplicaciones SPA. [https://nginx.org/en/docs/](https://nginx.org/en/docs/)
- **MDN Fetch API**: Referencia completa de la Fetch API con ejemplos avanzados. [https://developer.mozilla.org/es/docs/Web/API/Fetch_API/Using_Fetch](https://developer.mozilla.org/es/docs/Web/API/Fetch_API/Using_Fetch)
- **Docker Best Practices for Node.js**: Guía oficial de Docker para aplicaciones Node.js. [https://docs.docker.com/language/nodejs/](https://docs.docker.com/language/nodejs/)
- **Rama de solución completa**: `git checkout solution/lab-07-complete` para obtener el código final de referencia.

---

> **Felicitaciones**: Has completado el Laboratorio 7 y el proyecto integrador del curso Full Stack Empresarial. Tu aplicación está contenerizada, las capas se comunican correctamente y el sistema está listo para un despliegue en entorno productivo.
