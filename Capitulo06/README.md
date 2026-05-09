# Laboratorio 6. Proyecto Integrador — Contenerización y Gestión con Docker y GitHub

## Metadatos

| Propiedad | Valor |
|-----------|-------|
| **Duración** | 80 minutos |
| **Complejidad** | Intermedio |
| **Nivel Bloom** | Crear |

## Descripción General

En este laboratorio contenerizarás el proyecto integrador completo —frontend y backend— utilizando Docker con builds multi-etapa optimizados, y orquestarás todos los servicios mediante Docker Compose. Además, configurarás un repositorio GitHub profesional con una estructura de ramas bien definida, mensajes de commit atómicos y documentación completa en el README.

Este laboratorio representa el punto de integración final del curso: toma todo lo construido en los laboratorios anteriores (API REST con Spring Boot, frontend con Lit/Web Components, persistencia en PostgreSQL y MongoDB) y lo empaqueta en un stack listo para despliegue en cualquier entorno, eliminando el clásico problema de "funciona en mi máquina" que Docker fue diseñado para resolver.

## Objetivos de Aprendizaje

Al completar este laboratorio, serás capaz de:

- [ ] Crear un Dockerfile multi-etapa optimizado para el backend Spring Boot, logrando una imagen final menor a 200 MB
- [ ] Crear un Dockerfile para el frontend que use Node.js para el build y nginx:alpine para servir los estáticos
- [ ] Definir un archivo `compose.yml` con 5 servicios, redes personalizadas, volúmenes persistentes, variables de entorno y health checks
- [ ] Ejecutar y gestionar el stack completo usando comandos esenciales de Docker y Docker Compose
- [ ] Inicializar un repositorio Git con estructura de ramas profesional y publicarlo en GitHub con mínimo 10 commits atómicos documentados

## Prerrequisitos

### Conocimientos Requeridos

- Comprensión de los conceptos de imágenes y contenedores Docker (Lección 6.1)
- Conocimiento básico de línea de comandos (bash en macOS/Linux o PowerShell en Windows)
- Familiaridad con Spring Boot y la estructura de un proyecto Maven
- Conocimiento básico de Git: `init`, `add`, `commit`, `push`, `branch`
- Comprensión de variables de entorno y archivos de configuración en Spring Boot (`application.properties`)

### Acceso Requerido

- Cuenta de GitHub creada, verificada y con SSH key configurada
- Docker Desktop 4.25 o superior instalado y en ejecución
- Proyecto integrador (backend Spring Boot + frontend Lit) funcionando localmente desde los labs anteriores
- Acceso a internet para descargar imágenes base desde Docker Hub
- Puertos 80, 8080, 5432, 27017 y 5050 disponibles en la máquina local

## Entorno de Laboratorio

### Hardware Requirements

| Componente | Especificación |
|------------|----------------|
| RAM | Mínimo 16 GB (Docker necesitará ~4-6 GB para los 5 servicios) |
| Almacenamiento | Mínimo 10 GB libres para imágenes Docker y artefactos de build |
| CPU | 4 núcleos recomendados para builds paralelos |
| Red | Conexión estable para descargar imágenes base (~1.5 GB total) |

### Software Requirements

| Software | Versión | Propósito |
|----------|---------|-----------|
| Docker Desktop | 4.25+ | Motor de contenedores y Docker Compose |
| Git | 2.43+ | Control de versiones |
| JDK | 17 LTS | Verificación local del proyecto backend |
| Node.js | 20.x LTS | Verificación local del proyecto frontend |
| VS Code o IntelliJ | Cualquier versión reciente | Edición de Dockerfiles y YAML |

### Configuración Inicial

Antes de comenzar, verifica que el entorno esté listo:

```bash
# Verificar Docker está corriendo
docker run hello-world

# Verificar versión de Docker Compose
docker compose version

# Verificar Git
git --version

# Verificar que los puertos estén libres (Linux/macOS)
lsof -i :80 -i :8080 -i :5432 -i :27017 -i :5050

# Verificar que los puertos estén libres (Windows PowerShell)
netstat -ano | findstr "80 8080 5432 27017 5050"
```

Si `hello-world` se ejecuta correctamente, Docker está listo. Si algún puerto está ocupado, detén el servicio que lo usa antes de continuar.

## Instrucciones 

### Paso 1: Preparar la Estructura del Proyecto Integrador

**Objetivo:** Organizar el proyecto en una estructura de directorios clara que facilite la contenerización y la gestión con Git.

**Instrucciones:**

1. Abre una terminal y navega al directorio donde tienes tu proyecto integrador. Si vienes de los labs anteriores, la estructura debería ser similar a la siguiente. Crea los directorios que falten:

   ```bash
   # Navegar al directorio raíz del proyecto (ajusta la ruta según tu entorno)
   cd ~/proyectos/proyecto-integrador

   # Verificar la estructura actual
   ls -la

   # Si no existe, crear la estructura base
   mkdir -p backend frontend docker
   ```

2. Confirma que la estructura del proyecto tiene al menos estos elementos clave:

   ```bash
   # La estructura debe verse así:
   # proyecto-integrador/
   # ├── backend/          ← Proyecto Spring Boot (con pom.xml)
   # │   ├── pom.xml
   # │   └── src/
   # ├── frontend/         ← Proyecto Lit/Web Components (con package.json)
   # │   ├── package.json
   # │   └── src/
   # └── docker/           ← (Nuevo) Archivos auxiliares Docker

   # Verificar que el backend compila localmente
   cd backend
   ./mvnw clean package -DskipTests
   ls -la target/*.jar

   # Verificar que el frontend construye localmente
   cd ../frontend
   npm install
   npm run build
   ls -la dist/
   ```

3. Regresa al directorio raíz del proyecto:

   ```bash
   cd ..
   pwd
   # Debes ver algo como: /home/usuario/proyectos/proyecto-integrador
   ```

**Salida Esperada:**

```
BUILD SUCCESS
...
target/
  proyecto-integrador-0.0.1-SNAPSHOT.jar   (debe existir, ~50-80 MB)

dist/
  index.html
  assets/
```

**Verificación:**

- El archivo `.jar` del backend existe en `backend/target/`
- El directorio `dist/` del frontend contiene `index.html` y los assets compilados
- Estás posicionado en el directorio raíz del proyecto

---

### Paso 2: Crear el Dockerfile Multi-Etapa para el Backend

**Objetivo:** Construir un Dockerfile optimizado para el backend Spring Boot usando un build multi-etapa: una etapa para compilar con Maven y otra etapa liviana solo para ejecutar el JAR.

**Instrucciones:**

1. Crea el archivo `Dockerfile` dentro del directorio `backend/`:

   ```bash
   # Desde el directorio raíz del proyecto
   touch backend/Dockerfile
   ```

2. Abre `backend/Dockerfile` en tu editor y escribe el siguiente contenido:

   ```dockerfile
   # ============================================================
   # ETAPA 1: BUILD — Compilar el proyecto con Maven
   # Usamos la imagen oficial de Maven con JDK 17 (Temurin)
   # ============================================================
   FROM maven:3.9-eclipse-temurin-17 AS builder

   # Establecer directorio de trabajo dentro del contenedor
   WORKDIR /app

   # Copiar primero solo el pom.xml para aprovechar el cache de capas Docker.
   # Si el pom.xml no cambia, Docker reutiliza la capa de dependencias descargadas.
   COPY pom.xml .

   # Descargar todas las dependencias Maven (se cachean en esta capa)
   RUN mvn dependency:go-offline -B

   # Copiar el código fuente completo
   COPY src ./src

   # Compilar y empaquetar la aplicación, omitiendo tests
   # (los tests ya fueron ejecutados en el pipeline de CI)
   RUN mvn clean package -DskipTests -B

   # ============================================================
   # ETAPA 2: RUNTIME — Imagen final mínima para producción
   # Usamos JRE Alpine (sin JDK completo) para reducir tamaño
   # ============================================================
   FROM eclipse-temurin:17-jre-alpine AS runtime

   # Metadatos de la imagen (buena práctica)
   LABEL maintainer="equipo-desarrollo@empresa.com"
   LABEL version="1.0.0"
   LABEL description="Backend Spring Boot - Proyecto Integrador"

   # Crear un usuario no-root por seguridad (evitar correr como root)
   RUN addgroup -S appgroup && adduser -S appuser -G appgroup

   # Establecer directorio de trabajo
   WORKDIR /app

   # Copiar SOLO el JAR compilado desde la etapa builder
   # Esto descarta todas las herramientas de Maven y el código fuente
   COPY --from=builder /app/target/*.jar app.jar

   # Cambiar la propiedad del archivo al usuario no-root
   RUN chown appuser:appgroup app.jar

   # Usar el usuario no-root para ejecutar la aplicación
   USER appuser

   # Exponer el puerto en el que Spring Boot escucha
   EXPOSE 8080

   # Variables de entorno con valores por defecto
   ENV JAVA_OPTS="-Xms256m -Xmx512m"
   ENV SPRING_PROFILES_ACTIVE="docker"

   # Comando de inicio: ejecutar el JAR con las opciones de JVM configurables
   ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
   ```

3. Construye la imagen del backend para verificar que el Dockerfile es correcto:

   ```bash
   # Desde el directorio raíz del proyecto
   docker build -t proyecto-integrador/backend:1.0.0 ./backend

   # Verificar el tamaño de la imagen generada
   docker images proyecto-integrador/backend:1.0.0
   ```

**Salida Esperada:**

```
[+] Building 120.5s (14/14) FINISHED
 => [builder 1/6] FROM maven:3.9-eclipse-temurin-17
 => [builder 4/6] RUN mvn dependency:go-offline -B
 => [builder 6/6] RUN mvn clean package -DskipTests -B
 => [runtime 3/4] COPY --from=builder /app/target/*.jar app.jar
 => exporting to image

REPOSITORY                         TAG       SIZE
proyecto-integrador/backend        1.0.0     ~180MB   ← Debe ser < 200MB
```

**Verificación:**

- El build termina con `FINISHED` sin errores
- El tamaño de la imagen es menor a 200 MB (objetivo del laboratorio)
- La imagen aparece en `docker images`

---

### Paso 3: Crear el Dockerfile para el Frontend

**Objetivo:** Construir un Dockerfile multi-etapa para el frontend: Node.js para compilar los assets con Vite y nginx:alpine para servirlos de manera eficiente.

**Instrucciones:**

1. Crea el archivo `Dockerfile` dentro del directorio `frontend/`:

   ```bash
   touch frontend/Dockerfile
   ```

2. Crea también un archivo de configuración para nginx que manejará el enrutamiento del SPA (Single Page Application):

   ```bash
   mkdir -p frontend/nginx
   touch frontend/nginx/default.conf
   ```

3. Escribe la configuración de nginx en `frontend/nginx/default.conf`:

   ```nginx
   server {
       listen 80;
       server_name localhost;
       root /usr/share/nginx/html;
       index index.html;

       # Compresión gzip para assets estáticos
       gzip on;
       gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

       # Configuración para SPA: redirigir todas las rutas al index.html
       location / {
           try_files $uri $uri/ /index.html;
       }

       # Cache para assets estáticos (JS, CSS, imágenes)
       location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
           expires 1y;
           add_header Cache-Control "public, immutable";
       }

       # Proxy para el backend API (evita CORS en producción)
       location /api/ {
           proxy_pass http://backend:8080/api/;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       }
   }
   ```

4. Escribe el Dockerfile en `frontend/Dockerfile`:

   ```dockerfile
   # ============================================================
   # ETAPA 1: BUILD — Compilar el frontend con Node.js y Vite
   # ============================================================
   FROM node:20-alpine AS builder

   LABEL stage="frontend-builder"

   # Establecer directorio de trabajo
   WORKDIR /app

   # Copiar primero los archivos de dependencias para aprovechar el cache
   COPY package.json package-lock.json ./

   # Instalar dependencias (--ci para builds reproducibles)
   RUN npm ci

   # Copiar el código fuente del frontend
   COPY . .

   # Compilar el proyecto para producción
   RUN npm run build

   # ============================================================
   # ETAPA 2: RUNTIME — Servir los estáticos con nginx Alpine
   # ============================================================
   FROM nginx:alpine AS runtime

   LABEL maintainer="equipo-desarrollo@empresa.com"
   LABEL version="1.0.0"
   LABEL description="Frontend Lit/Web Components - Proyecto Integrador"

   # Eliminar la configuración default de nginx
   RUN rm /etc/nginx/conf.d/default.conf

   # Copiar nuestra configuración personalizada de nginx
   COPY nginx/default.conf /etc/nginx/conf.d/default.conf

   # Copiar los assets compilados desde la etapa builder
   COPY --from=builder /app/dist /usr/share/nginx/html

   # Exponer el puerto 80
   EXPOSE 80

   # nginx se inicia automáticamente con el CMD de la imagen base
   # CMD ["nginx", "-g", "daemon off;"]  ← ya está definido en la imagen base
   ```

5. Construye la imagen del frontend para verificar:

   ```bash
   # Desde el directorio raíz del proyecto
   docker build -t proyecto-integrador/frontend:1.0.0 ./frontend

   # Verificar tamaño
   docker images proyecto-integrador/frontend:1.0.0
   ```

**Salida Esperada:**

```
[+] Building 45.3s (12/12) FINISHED
 => [builder 3/5] RUN npm ci
 => [builder 5/5] RUN npm run build
 => [runtime 3/3] COPY --from=builder /app/dist /usr/share/nginx/html

REPOSITORY                          TAG       SIZE
proyecto-integrador/frontend        1.0.0     ~25MB   ← Muy pequeña gracias a Alpine
```

**Verificación:**

- Ambas imágenes (backend y frontend) están presentes en `docker images`
- El frontend es significativamente más pequeño que el backend
- No hay errores de compilación en ninguna de las dos etapas

---

### Paso 4: Crear el Archivo .env con Variables de Entorno

**Objetivo:** Centralizar todas las configuraciones sensibles y variables de entorno en un archivo `.env` que Docker Compose utilizará, siguiendo la buena práctica de no hardcodear credenciales en los archivos de configuración.

**Instrucciones:**

1. Crea el archivo `.env` en el directorio raíz del proyecto:

   ```bash
   # Desde el directorio raíz del proyecto
   touch .env
   ```

2. Escribe el siguiente contenido en `.env`:

   ```dotenv
   # ============================================================
   # VARIABLES DE ENTORNO - PROYECTO INTEGRADOR
   # ¡IMPORTANTE! No commitear este archivo a Git con credenciales reales.
   # Usar .env.example como plantilla para el repositorio.
   # ============================================================

   # --- Configuración del Proyecto ---
   COMPOSE_PROJECT_NAME=proyecto-integrador

   # --- PostgreSQL ---
   POSTGRES_DB=inventario_db
   POSTGRES_USER=app_user
   POSTGRES_PASSWORD=App_S3cur3_P@ss2024
   POSTGRES_PORT=5432

   # --- MongoDB ---
   MONGO_INITDB_ROOT_USERNAME=mongo_admin
   MONGO_INITDB_ROOT_PASSWORD=Mongo_S3cur3_P@ss2024
   MONGO_INITDB_DATABASE=inventario_nosql
   MONGO_PORT=27017

   # --- Backend Spring Boot ---
   BACKEND_PORT=8080
   SPRING_PROFILES_ACTIVE=docker
   JWT_SECRET=mi-secreto-jwt-muy-largo-y-seguro-para-produccion-2024
   JAVA_OPTS=-Xms256m -Xmx512m

   # --- Frontend ---
   FRONTEND_PORT=80

   # --- pgAdmin ---
   PGADMIN_DEFAULT_EMAIL=admin@empresa.com
   PGADMIN_DEFAULT_PASSWORD=PgAdmin_P@ss2024
   PGADMIN_PORT=5050
   ```

3. Crea también el archivo `.env.example` (versión sin credenciales reales para el repositorio):

   ```bash
   touch .env.example
   ```

4. Escribe el contenido de `.env.example`:

   ```dotenv
   # ============================================================
   # PLANTILLA DE VARIABLES DE ENTORNO
   # Copia este archivo como .env y completa los valores
   # cp .env.example .env
   # ============================================================

   COMPOSE_PROJECT_NAME=proyecto-integrador

   # PostgreSQL
   POSTGRES_DB=inventario_db
   POSTGRES_USER=tu_usuario_postgres
   POSTGRES_PASSWORD=tu_password_seguro
   POSTGRES_PORT=5432

   # MongoDB
   MONGO_INITDB_ROOT_USERNAME=tu_usuario_mongo
   MONGO_INITDB_ROOT_PASSWORD=tu_password_seguro
   MONGO_INITDB_DATABASE=inventario_nosql
   MONGO_PORT=27017

   # Backend
   BACKEND_PORT=8080
   SPRING_PROFILES_ACTIVE=docker
   JWT_SECRET=tu-secreto-jwt-muy-largo-y-seguro
   JAVA_OPTS=-Xms256m -Xmx512m

   # Frontend
   FRONTEND_PORT=80

   # pgAdmin
   PGADMIN_DEFAULT_EMAIL=admin@empresa.com
   PGADMIN_DEFAULT_PASSWORD=tu_password_pgadmin
   PGADMIN_PORT=5050
   ```

**Salida Esperada:**

```bash
ls -la .env .env.example
# -rw-r--r--  1 usuario  staff  856 Jan 15 10:30 .env
# -rw-r--r--  1 usuario  staff  742 Jan 15 10:30 .env.example
```

**Verificación:**

- Ambos archivos existen en el directorio raíz
- `.env` contiene valores reales (no se commiteará)
- `.env.example` contiene solo placeholders (sí se commiteará)

---

### Paso 5: Crear el Perfil de Configuración Docker para Spring Boot

**Objetivo:** Agregar un perfil de configuración específico para el entorno Docker en el backend Spring Boot, que use los nombres de los servicios de Docker Compose como hostnames en lugar de `localhost`.

**Instrucciones:**

1. Crea el archivo de configuración para el perfil Docker en el backend:

   ```bash
   touch backend/src/main/resources/application-docker.properties
   ```

2. Escribe la configuración en `application-docker.properties`:

   ```properties
   # ============================================================
   # CONFIGURACIÓN SPRING BOOT - PERFIL DOCKER
   # Los hostnames corresponden a los nombres de servicios en compose.yml
   # ============================================================

   # --- Servidor ---
   server.port=8080

   # --- PostgreSQL (el hostname 'postgres' es el nombre del servicio en Compose) ---
   spring.datasource.url=jdbc:postgresql://postgres:5432/${POSTGRES_DB:inventario_db}
   spring.datasource.username=${POSTGRES_USER:app_user}
   spring.datasource.password=${POSTGRES_PASSWORD:changeme}
   spring.datasource.driver-class-name=org.postgresql.Driver

   # --- JPA/Hibernate ---
   spring.jpa.hibernate.ddl-auto=update
   spring.jpa.show-sql=false
   spring.jpa.properties.hibernate.format_sql=false

   # --- MongoDB (el hostname 'mongodb' es el nombre del servicio en Compose) ---
   spring.data.mongodb.uri=mongodb://${MONGO_INITDB_ROOT_USERNAME:mongo_admin}:${MONGO_INITDB_ROOT_PASSWORD:changeme}@mongodb:27017/${MONGO_INITDB_DATABASE:inventario_nosql}?authSource=admin

   # --- Logging ---
   logging.level.root=INFO
   logging.level.com.empresa.proyecto=DEBUG

   # --- Actuator (para health checks) ---
   management.endpoints.web.exposure.include=health,info,metrics
   management.endpoint.health.show-details=always
   ```

**Salida Esperada:**

```bash
ls backend/src/main/resources/
# application.properties
# application-docker.properties   ← El nuevo archivo
```

**Verificación:**

- El archivo `application-docker.properties` existe en el directorio `resources`
- Los hostnames `postgres` y `mongodb` coincidirán con los nombres de servicios que definiremos en `compose.yml`

---

### Paso 6: Crear el Archivo Docker Compose

**Objetivo:** Definir todos los servicios del stack completo en un archivo `compose.yml`, incluyendo redes personalizadas, volúmenes persistentes, health checks y dependencias entre servicios.

**Instrucciones:**

1. Crea el archivo `compose.yml` en el directorio raíz del proyecto:

   ```bash
   touch compose.yml
   ```

2. Escribe el siguiente contenido completo en `compose.yml`:

   ```yaml
   # ============================================================
   # DOCKER COMPOSE - PROYECTO INTEGRADOR
   # Stack completo: Frontend + Backend + PostgreSQL + MongoDB + pgAdmin
   # Versión: Docker Compose v2 (no requiere campo 'version')
   # ============================================================

   services:

     # ----------------------------------------------------------
     # SERVICIO 1: PostgreSQL - Base de datos relacional
     # ----------------------------------------------------------
     postgres:
       image: postgres:16-alpine
       container_name: pi-postgres
       restart: unless-stopped
       environment:
         POSTGRES_DB: ${POSTGRES_DB}
         POSTGRES_USER: ${POSTGRES_USER}
         POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
       ports:
         - "${POSTGRES_PORT:-5432}:5432"
       volumes:
         - postgres_data:/var/lib/postgresql/data
         - ./docker/init-postgres.sql:/docker-entrypoint-initdb.d/init.sql:ro
       networks:
         - backend-network
       healthcheck:
         test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
         interval: 10s
         timeout: 5s
         retries: 5
         start_period: 30s

     # ----------------------------------------------------------
     # SERVICIO 2: MongoDB - Base de datos NoSQL
     # ----------------------------------------------------------
     mongodb:
       image: mongo:7.0
       container_name: pi-mongodb
       restart: unless-stopped
       environment:
         MONGO_INITDB_ROOT_USERNAME: ${MONGO_INITDB_ROOT_USERNAME}
         MONGO_INITDB_ROOT_PASSWORD: ${MONGO_INITDB_ROOT_PASSWORD}
         MONGO_INITDB_DATABASE: ${MONGO_INITDB_DATABASE}
       ports:
         - "${MONGO_PORT:-27017}:27017"
       volumes:
         - mongodb_data:/data/db
       networks:
         - backend-network
       healthcheck:
         test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
         interval: 10s
         timeout: 5s
         retries: 5
         start_period: 30s

     # ----------------------------------------------------------
     # SERVICIO 3: Backend Spring Boot
     # ----------------------------------------------------------
     backend:
       build:
         context: ./backend
         dockerfile: Dockerfile
         target: runtime
       image: proyecto-integrador/backend:1.0.0
       container_name: pi-backend
       restart: unless-stopped
       environment:
         SPRING_PROFILES_ACTIVE: ${SPRING_PROFILES_ACTIVE:-docker}
         POSTGRES_DB: ${POSTGRES_DB}
         POSTGRES_USER: ${POSTGRES_USER}
         POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
         MONGO_INITDB_ROOT_USERNAME: ${MONGO_INITDB_ROOT_USERNAME}
         MONGO_INITDB_ROOT_PASSWORD: ${MONGO_INITDB_ROOT_PASSWORD}
         MONGO_INITDB_DATABASE: ${MONGO_INITDB_DATABASE}
         JWT_SECRET: ${JWT_SECRET}
         JAVA_OPTS: ${JAVA_OPTS:--Xms256m -Xmx512m}
       ports:
         - "${BACKEND_PORT:-8080}:8080"
       networks:
         - backend-network
         - frontend-network
       depends_on:
         postgres:
           condition: service_healthy
         mongodb:
           condition: service_healthy
       healthcheck:
         test: ["CMD-SHELL", "wget -qO- http://localhost:8080/actuator/health || exit 1"]
         interval: 15s
         timeout: 10s
         retries: 5
         start_period: 60s

     # ----------------------------------------------------------
     # SERVICIO 4: Frontend Lit/Web Components (nginx)
     # ----------------------------------------------------------
     frontend:
       build:
         context: ./frontend
         dockerfile: Dockerfile
         target: runtime
       image: proyecto-integrador/frontend:1.0.0
       container_name: pi-frontend
       restart: unless-stopped
       ports:
         - "${FRONTEND_PORT:-80}:80"
       networks:
         - frontend-network
       depends_on:
         backend:
           condition: service_healthy

     # ----------------------------------------------------------
     # SERVICIO 5: pgAdmin - Administración visual de PostgreSQL
     # ----------------------------------------------------------
     pgadmin:
       image: dpage/pgadmin4:8
       container_name: pi-pgadmin
       restart: unless-stopped
       environment:
         PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL}
         PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD}
         PGADMIN_CONFIG_SERVER_MODE: "False"
       ports:
         - "${PGADMIN_PORT:-5050}:80"
       volumes:
         - pgadmin_data:/var/lib/pgadmin
       networks:
         - backend-network
       depends_on:
         postgres:
           condition: service_healthy

   # ============================================================
   # VOLÚMENES PERSISTENTES
   # Los datos sobreviven aunque los contenedores sean eliminados
   # ============================================================
   volumes:
     postgres_data:
       name: pi-postgres-data
     mongodb_data:
       name: pi-mongodb-data
     pgadmin_data:
       name: pi-pgadmin-data

   # ============================================================
   # REDES PERSONALIZADAS
   # Segmentación: backend-network (DB + API) y frontend-network (API + UI)
   # ============================================================
   networks:
     backend-network:
       name: pi-backend-net
       driver: bridge
     frontend-network:
       name: pi-frontend-net
       driver: bridge
   ```

3. Crea el script de inicialización de PostgreSQL:

   ```bash
   mkdir -p docker
   touch docker/init-postgres.sql
   ```

4. Escribe el contenido de `docker/init-postgres.sql`:

   ```sql
   -- Script de inicialización de PostgreSQL
   -- Se ejecuta automáticamente la primera vez que el contenedor arranca

   -- Crear extensiones útiles
   CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
   CREATE EXTENSION IF NOT EXISTS "pg_trgm";

   -- Mensaje de confirmación
   DO $$
   BEGIN
     RAISE NOTICE 'Base de datos % inicializada correctamente', current_database();
   END $$;
   ```

**Salida Esperada:**

```bash
ls -la compose.yml docker/init-postgres.sql
# -rw-r--r--  compose.yml
# -rw-r--r--  docker/init-postgres.sql
```

**Verificación:**

- El archivo `compose.yml` existe en el directorio raíz
- El script SQL de inicialización existe en `docker/`
- La sintaxis YAML es correcta (puedes verificar con `docker compose config`)

---

### Paso 7: Construir y Ejecutar el Stack Completo

**Objetivo:** Construir todas las imágenes y levantar el stack completo con Docker Compose, verificando que todos los servicios arrancan correctamente y los health checks pasan.

**Instrucciones:**

1. Valida la sintaxis del archivo `compose.yml` antes de ejecutar:

   ```bash
   # Desde el directorio raíz del proyecto
   docker compose config
   # Debe mostrar la configuración completa sin errores
   ```

2. Construye todas las imágenes del proyecto:

   ```bash
   # Construir todas las imágenes definidas en compose.yml
   # --no-cache fuerza la reconstrucción completa (útil para la primera vez)
   docker compose build --no-cache

   # Verificar las imágenes construidas
   docker images | grep proyecto-integrador
   ```

3. Levanta todos los servicios en modo detached (segundo plano):

   ```bash
   # Levantar el stack completo
   docker compose up -d

   # Observar el progreso del arranque (Ctrl+C para salir del seguimiento)
   docker compose logs -f
   ```

4. Verifica el estado de todos los servicios:

   ```bash
   # Ver el estado de todos los contenedores
   docker compose ps

   # Ver los logs de un servicio específico (útil para depurar)
   docker compose logs backend --tail=50
   docker compose logs postgres --tail=20
   ```

5. Prueba que los servicios responden correctamente:

   ```bash
   # Probar el backend (health check de Spring Actuator)
   curl http://localhost:8080/actuator/health

   # Probar el frontend (debe retornar HTML)
   curl -I http://localhost:80

   # Probar pgAdmin (debe retornar código 200)
   curl -I http://localhost:5050
   ```

6. Explora los comandos esenciales de gestión:

   ```bash
   # Ejecutar un comando dentro de un contenedor en ejecución
   docker compose exec postgres psql -U app_user -d inventario_db -c "\dt"

   # Ver el uso de recursos de los contenedores
   docker stats --no-stream

   # Ver los volúmenes creados
   docker volume ls | grep pi-

   # Ver las redes creadas
   docker network ls | grep pi-
   ```

**Salida Esperada:**

```
NAME              IMAGE                              STATUS                   PORTS
pi-backend        proyecto-integrador/backend:1.0.0  Up 2 minutes (healthy)   0.0.0.0:8080->8080/tcp
pi-frontend       proyecto-integrador/frontend:1.0.0 Up 1 minute              0.0.0.0:80->80/tcp
pi-mongodb        mongo:7.0                          Up 3 minutes (healthy)   0.0.0.0:27017->27017/tcp
pi-pgadmin        dpage/pgadmin4:8                   Up 2 minutes             0.0.0.0:5050->80/tcp
pi-postgres       postgres:16-alpine                 Up 3 minutes (healthy)   0.0.0.0:5432->5432/tcp

# Respuesta de health check:
{"status":"UP","components":{"db":{"status":"UP"},"mongo":{"status":"UP"}}}
```

**Verificación:**

- Todos los servicios muestran estado `Up` y `(healthy)`
- El backend responde en `http://localhost:8080/actuator/health` con `{"status":"UP"}`
- El frontend carga en `http://localhost:80`
- pgAdmin está disponible en `http://localhost:5050`

---

### Paso 8: Configurar el Repositorio Git y Publicar en GitHub

**Objetivo:** Inicializar el repositorio Git con estructura de ramas profesional, configurar `.gitignore`, escribir el README y publicar en GitHub con mínimo 10 commits atómicos bien documentados.

**Instrucciones:**

1. Inicializa el repositorio Git (si no existe ya):

   ```bash
   # Desde el directorio raíz del proyecto
   git init
   git config user.name "Tu Nombre"
   git config user.email "tu.email@empresa.com"
   ```

2. Crea el archivo `.gitignore` completo:

   ```bash
   touch .gitignore
   ```

   Escribe el siguiente contenido en `.gitignore`:

   ```gitignore
   # ============================================================
   # .gitignore - Proyecto Integrador Full Stack
   # ============================================================

   # --- Variables de entorno (NUNCA commitear credenciales) ---
   .env
   .env.local
   .env.production

   # --- Java / Maven ---
   backend/target/
   backend/.mvn/wrapper/maven-wrapper.jar
   *.class
   *.jar
   *.war
   *.ear
   *.log
   .classpath
   .project
   .settings/
   backend/.idea/
   *.iml

   # --- Node.js / NPM ---
   frontend/node_modules/
   frontend/dist/
   frontend/.vite/
   npm-debug.log*
   yarn-debug.log*
   yarn-error.log*

   # --- IDEs ---
   .idea/
   .vscode/settings.json
   *.swp
   *.swo
   .DS_Store
   Thumbs.db

   # --- Docker ---
   # No ignorar Dockerfiles ni compose.yml
   # Sí ignorar datos locales de Docker si los hubiera

   # --- OS ---
   .DS_Store
   .DS_Store?
   ._*
   .Spotlight-V100
   .Trashes
   ehthumbs.db
   Desktop.ini
   ```

3. Crea el README.md principal del proyecto:

   ```bash
   touch README.md
   ```

   Escribe el siguiente contenido en `README.md`:

   ````markdown
   # 🏢 Proyecto Integrador — Sistema de Gestión de Inventario

   Stack empresarial full stack con Spring Boot, Lit/Web Components, PostgreSQL, MongoDB y Docker.

   ## 🏗️ Arquitectura

   ```
   ┌─────────────────────────────────────────────────────────┐
   │                    DOCKER NETWORK                        │
   │                                                          │
   │  ┌──────────┐    ┌──────────┐    ┌──────────────────┐  │
   │  │ Frontend │───▶│ Backend  │───▶│   PostgreSQL 16  │  │
   │  │  :80     │    │  :8080   │    │      :5432       │  │
   │  │ nginx    │    │Spring    │    └──────────────────┘  │
   │  │ Lit/WC   │    │Boot 3.2  │    ┌──────────────────┐  │
   │  └──────────┘    └──────────┘───▶│   MongoDB 7.0    │  │
   │                                  │      :27017      │  │
   │  ┌──────────┐                    └──────────────────┘  │
   │  │ pgAdmin  │                                           │
   │  │  :5050   │                                           │
   │  └──────────┘                                           │
   └─────────────────────────────────────────────────────────┘
   ```

   ## 📋 Prerrequisitos

   - Docker Desktop 4.25+
   - Git 2.43+
   - Puertos disponibles: 80, 8080, 5432, 27017, 5050

   ## 🚀 Instrucciones de Ejecución

   ```bash
   # 1. Clonar el repositorio
   git clone git@github.com:tu-usuario/proyecto-integrador.git
   cd proyecto-integrador

   # 2. Configurar variables de entorno
   cp .env.example .env
   # Editar .env con tus credenciales

   # 3. Construir y levantar el stack completo
   docker compose up -d --build

   # 4. Verificar que todos los servicios estén saludables
   docker compose ps
   ```

   ## 🌐 URLs de Acceso

   | Servicio | URL | Credenciales |
   |----------|-----|--------------|
   | Frontend | http://localhost:80 | N/A |
   | Backend API | http://localhost:8080/api | Ver .env |
   | API Health | http://localhost:8080/actuator/health | N/A |
   | pgAdmin | http://localhost:5050 | Ver .env |

   ## 🛑 Detener el Stack

   ```bash
   # Detener sin eliminar datos
   docker compose stop

   # Detener y eliminar contenedores (datos persisten en volúmenes)
   docker compose down

   # Detener y eliminar TODO incluyendo volúmenes (¡borra datos!)
   docker compose down -v
   ```

   ## 🌿 Estructura de Ramas

   | Rama | Propósito |
   |------|-----------|
   | `main` | Código estable para producción |
   | `develop` | Integración de features |
   | `feature/*` | Desarrollo de nuevas funcionalidades |

   ## 🛠️ Stack Tecnológico

   - **Frontend**: Lit 3.x, Web Components, HTML5, CSS3
   - **Backend**: Spring Boot 3.2, Java 17, Spring Data JPA, Spring Data MongoDB
   - **Bases de datos**: PostgreSQL 16, MongoDB 7.0
   - **Infraestructura**: Docker, Docker Compose, nginx
   - **Testing**: JUnit 5, Mockito, @web/test-runner

   ## 📁 Estructura del Proyecto

   ```
   proyecto-integrador/
   ├── backend/                 # API REST Spring Boot
   │   ├── src/
   │   ├── Dockerfile           # Multi-stage build
   │   └── pom.xml
   ├── frontend/                # SPA Lit/Web Components
   │   ├── src/
   │   ├── nginx/
   │   ├── Dockerfile           # Multi-stage build
   │   └── package.json
   ├── docker/                  # Scripts auxiliares Docker
   │   └── init-postgres.sql
   ├── compose.yml              # Orquestación completa
   ├── .env.example             # Plantilla de variables de entorno
   └── README.md
   ```
   ````

4. Realiza el primer commit y configura la rama `main`:

   ```bash
   # Agregar todos los archivos al staging area
   git add .gitignore README.md .env.example

   # Primer commit
   git commit -m "chore: inicializar repositorio con .gitignore y README"

   # Renombrar la rama principal a 'main' (convención moderna)
   git branch -M main
   ```

5. Realiza commits atómicos para cada componente del proyecto:

   ```bash
   # Commit 2: Archivos de configuración Docker de base de datos
   git add docker/init-postgres.sql
   git commit -m "chore(docker): agregar script de inicialización de PostgreSQL"

   # Commit 3: Configuración de perfil Docker para Spring Boot
   git add backend/src/main/resources/application-docker.properties
   git commit -m "feat(backend): agregar perfil de configuración para entorno Docker"

   # Commit 4: Dockerfile del backend
   git add backend/Dockerfile
   git commit -m "feat(docker): crear Dockerfile multi-stage para backend Spring Boot"

   # Commit 5: Configuración nginx del frontend
   git add frontend/nginx/
   git commit -m "feat(frontend): agregar configuración nginx para SPA con proxy al backend"

   # Commit 6: Dockerfile del frontend
   git add frontend/Dockerfile
   git commit -m "feat(docker): crear Dockerfile multi-stage para frontend con nginx"

   # Commit 7: Archivo Docker Compose
   git add compose.yml
   git commit -m "feat(docker): definir stack completo con Docker Compose (5 servicios)"

   # Verificar el historial de commits hasta ahora
   git log --oneline
   ```

6. Crea la rama `develop` y una rama `feature` de ejemplo:

   ```bash
   # Crear y cambiar a la rama develop
   git checkout -b develop

   # Crear una rama feature para documentación adicional
   git checkout -b feature/documentacion-api

   # Crear un archivo de documentación de la API
   cat > docker/COMANDOS-DOCKER.md << 'EOF'
   # Comandos Docker Esenciales — Referencia Rápida

   ## Gestión del Stack
   ```bash
   docker compose up -d              # Levantar todos los servicios
   docker compose down               # Detener y eliminar contenedores
   docker compose restart backend    # Reiniciar un servicio específico
   docker compose logs -f backend    # Ver logs en tiempo real
   docker compose ps                 # Estado de todos los servicios
   ```

   ## Gestión de Imágenes
   ```bash
   docker images                     # Listar imágenes locales
   docker build -t nombre:tag .      # Construir imagen
   docker rmi nombre:tag             # Eliminar imagen
   docker image prune                # Limpiar imágenes sin usar
   ```

   ## Gestión de Contenedores
   ```bash
   docker compose exec backend bash  # Entrar al contenedor backend
   docker compose exec postgres psql -U app_user -d inventario_db
   docker stats                      # Uso de recursos en tiempo real
   ```

   ## Gestión de Volúmenes y Redes
   ```bash
   docker volume ls                  # Listar volúmenes
   docker network ls                 # Listar redes
   docker volume inspect pi-postgres-data  # Inspeccionar volumen
   ```
   EOF

   git add docker/COMANDOS-DOCKER.md
   git commit -m "docs: agregar referencia rápida de comandos Docker esenciales"

   # Merge de la feature a develop
   git checkout develop
   git merge feature/documentacion-api --no-ff -m "feat: merge documentación de comandos Docker"

   # Merge de develop a main (simulando un release)
   git checkout main
   git merge develop --no-ff -m "release: v1.0.0 - stack completo contenerizado"

   # Crear un tag de versión
   git tag -a v1.0.0 -m "Version 1.0.0: Stack completo con Docker Compose"

   # Ver el historial completo
   git log --oneline --graph --all
   ```

7. Crea el repositorio en GitHub y publica el código:

   ```bash
   # Opción A: Usando SSH (recomendado si tienes SSH key configurada)
   git remote add origin git@github.com:TU_USUARIO/proyecto-integrador.git

   # Opción B: Usando HTTPS
   # git remote add origin https://github.com/TU_USUARIO/proyecto-integrador.git

   # Publicar todas las ramas y tags
   git push -u origin main
   git push origin develop
   git push origin --tags

   # Verificar que el remote está configurado correctamente
   git remote -v
   ```

**Salida Esperada:**

```bash
git log --oneline
# a1b2c3d (HEAD -> main, tag: v1.0.0, origin/main) release: v1.0.0 - stack completo contenerizado
# e4f5g6h feat: merge documentación de comandos Docker
# h7i8j9k docs: agregar referencia rápida de comandos Docker esenciales
# k0l1m2n feat(docker): definir stack completo con Docker Compose (5 servicios)
# n3o4p5q feat(docker): crear Dockerfile multi-stage para frontend con nginx
# q6r7s8t feat(frontend): agregar configuración nginx para SPA con proxy al backend
# t9u0v1w feat(docker): crear Dockerfile multi-stage para backend Spring Boot
# w2x3y4z feat(backend): agregar perfil de configuración para entorno Docker
# z5a6b7c chore(docker): agregar script de inicialización de PostgreSQL
# c8d9e0f chore: inicializar repositorio con .gitignore y README

git log --oneline | wc -l
# 10   ← Mínimo 10 commits requeridos ✓
```

**Verificación:**

- `git log --oneline` muestra al menos 10 commits
- El repositorio está publicado en GitHub y visible en el navegador
- Las ramas `main` y `develop` existen tanto local como remotamente
- El tag `v1.0.0` está publicado en GitHub
- El `.env` NO aparece en el repositorio remoto

---

## Validación y Pruebas

### Criterios de Éxito

- [ ] La imagen del backend pesa menos de 200 MB (`docker images | grep backend`)
- [ ] La imagen del frontend pesa menos de 50 MB (`docker images | grep frontend`)
- [ ] Todos los 5 servicios muestran estado `Up (healthy)` en `docker compose ps`
- [ ] El endpoint `http://localhost:8080/actuator/health` retorna `{"status":"UP"}`
- [ ] El frontend carga correctamente en `http://localhost:80`
- [ ] pgAdmin es accesible en `http://localhost:5050`
- [ ] Los volúmenes `pi-postgres-data` y `pi-mongodb-data` existen y tienen datos
- [ ] El repositorio GitHub tiene al menos 10 commits atómicos
- [ ] El archivo `.env` NO está en el repositorio GitHub
- [ ] El README.md en GitHub muestra la arquitectura y las instrucciones de ejecución

### Procedimiento de Pruebas

1. **Verificar imágenes y tamaños:**
   ```bash
   docker images | grep proyecto-integrador
   ```
   **Resultado Esperado:** Backend < 200 MB, Frontend < 50 MB

2. **Verificar estado del stack:**
   ```bash
   docker compose ps
   ```
   **Resultado Esperado:** Los 5 servicios en estado `Up (healthy)`

3. **Probar el health check del backend:**
   ```bash
   curl -s http://localhost:8080/actuator/health | python3 -m json.tool
   ```
   **Resultado Esperado:**
   ```json
   {
     "status": "UP",
     "components": {
       "db": { "status": "UP" },
       "mongo": { "status": "UP" }
     }
   }
   ```

4. **Verificar conectividad con PostgreSQL:**
   ```bash
   docker compose exec postgres psql -U app_user -d inventario_db -c "SELECT version();"
   ```
   **Resultado Esperado:** Versión de PostgreSQL 16.x

5. **Verificar conectividad con MongoDB:**
   ```bash
   docker compose exec mongodb mongosh -u mongo_admin -p Mongo_S3cur3_P@ss2024 --authenticationDatabase admin --eval "db.adminCommand('ping')"
   ```
   **Resultado Esperado:** `{ ok: 1 }`

6. **Verificar persistencia de datos (reiniciar y verificar):**
   ```bash
   # Detener y reiniciar el stack
   docker compose down
   docker compose up -d

   # Los datos deben seguir existiendo (gracias a los volúmenes)
   docker volume ls | grep pi-
   ```
   **Resultado Esperado:** Los volúmenes `pi-postgres-data` y `pi-mongodb-data` persisten

7. **Verificar el repositorio Git:**
   ```bash
   git log --oneline | wc -l
   git branch -a
   git tag
   ```
   **Resultado Esperado:** ≥ 10 commits, ramas `main` y `develop`, tag `v1.0.0`

## Solución de Problemas

### Issue 1: El Backend No Puede Conectarse a PostgreSQL

**Síntomas:**
- `docker compose ps` muestra el backend en estado `Up (unhealthy)` o reiniciando
- Los logs muestran `Connection refused` o `could not connect to server`

**Causa:**
El backend intenta conectarse a PostgreSQL antes de que el health check de la base de datos pase. Puede ocurrir si PostgreSQL tarda más de lo esperado en iniciar, o si las credenciales en el `.env` no coinciden con las que PostgreSQL usó al crear la base de datos.

**Solución:**
```bash
# Verificar los logs de PostgreSQL
docker compose logs postgres --tail=30

# Verificar que el health check pasa
docker inspect pi-postgres --format='{{.State.Health.Status}}'

# Si el contenedor de postgres ya existe con credenciales diferentes,
# eliminar el volumen y recrearlo (¡esto borra todos los datos!)
docker compose down
docker volume rm pi-postgres-data
docker compose up -d

# Verificar que las variables de entorno se leen correctamente
docker compose exec backend env | grep POSTGRES
```

---

### Issue 2: Puerto Ya en Uso (Address Already in Use)

**Síntomas:**
- Error al ejecutar `docker compose up`: `Bind for 0.0.0.0:8080 failed: port is already allocated`
- Uno o más servicios no arrancan

**Causa:**
Un proceso local (Spring Boot corriendo fuera de Docker, PostgreSQL instalado localmente, etc.) está usando el mismo puerto que el contenedor intenta exponer.

**Solución:**
```bash
# Identificar qué proceso usa el puerto (Linux/macOS)
lsof -i :8080
kill -9 <PID>

# Identificar qué proceso usa el puerto (Windows PowerShell)
netstat -ano | findstr :8080
Stop-Process -Id <PID>

# Alternativa: cambiar el puerto en .env sin modificar compose.yml
# Editar .env y cambiar:
# BACKEND_PORT=8081
# POSTGRES_PORT=5433

# Luego reiniciar
docker compose up -d
```

---

### Issue 3: Build del Backend Falla — JAR No Encontrado

**Síntomas:**
- Error en el `docker build` del backend: `failed to solve: failed to read dockerfile`
- O error: `COPY failed: file not found in build context or excluded by .dockerignore: target/*.jar`

**Causa:**
El Dockerfile usa `COPY --from=builder /app/target/*.jar` pero el glob `*.jar` no encuentra el archivo, o Maven falló silenciosamente durante el build.

**Solución:**
```bash
# Verificar que Maven compila correctamente dentro del contenedor
docker build --target builder -t backend-debug ./backend
docker run --rm backend-debug ls -la /app/target/

# Si el JAR no existe, revisar los logs de Maven
docker build --progress=plain --no-cache -t proyecto-integrador/backend:debug ./backend 2>&1 | grep -A5 "BUILD"

# Verificar que el nombre del JAR en pom.xml no tiene caracteres especiales
grep -A3 "<build>" backend/pom.xml

# Si el proyecto tiene un finalName diferente, ajustar el Dockerfile:
# COPY --from=builder /app/target/mi-proyecto-especifico.jar app.jar
```

---

### Issue 4: Health Check de MongoDB Falla

**Síntomas:**
- MongoDB muestra `Up (unhealthy)` en `docker compose ps`
- Los logs muestran que MongoDB está corriendo pero el health check falla

**Causa:**
En versiones recientes de MongoDB 7.x, el comando `mongosh` reemplazó a `mongo`. Si la imagen no incluye `mongosh`, el health check falla.

**Solución:**
```bash
# Verificar qué herramientas están disponibles en el contenedor
docker compose exec mongodb which mongosh
docker compose exec mongodb which mongo

# Si mongosh no está disponible, usar una alternativa en compose.yml:
# healthcheck:
#   test: ["CMD", "mongo", "--eval", "db.adminCommand('ping')"]

# O usar la verificación de puerto TCP:
# healthcheck:
#   test: ["CMD-SHELL", "echo 'db.runCommand({ping:1})' | mongosh localhost:27017/test --quiet"]
#   interval: 10s
#   timeout: 5s
#   retries: 5

# Reiniciar el servicio después de modificar compose.yml
docker compose up -d --force-recreate mongodb
```

---

### Issue 5: El Frontend Muestra Pantalla en Blanco

**Síntomas:**
- `http://localhost:80` carga pero muestra una página en blanco
- La consola del navegador muestra errores 404 para los assets JS/CSS

**Causa:**
El directorio `dist/` del frontend no se generó correctamente durante el build de Docker, o la ruta de los assets en `vite.config.js` no coincide con la configuración de nginx.

**Solución:**
```bash
# Verificar que el build de Vite generó archivos correctamente
docker build --target builder -t frontend-debug ./frontend
docker run --rm frontend-debug ls -la /app/dist/

# Verificar que nginx está sirviendo los archivos correctamente
docker compose exec frontend ls -la /usr/share/nginx/html/

# Verificar la configuración de nginx
docker compose exec frontend cat /etc/nginx/conf.d/default.conf

# Si el proyecto usa un base path diferente en vite.config.js,
# asegurarse de que coincida:
# En vite.config.js: base: '/'
# En nginx: root /usr/share/nginx/html;

# Reconstruir la imagen del frontend
docker compose build --no-cache frontend
docker compose up -d frontend
```

## Limpieza

```bash
# ============================================================
# LIMPIEZA COMPLETA DEL LABORATORIO
# ============================================================

# Opción 1: Detener el stack pero conservar los datos (volúmenes)
# Útil si quieres continuar trabajando después
docker compose stop

# Opción 2: Detener y eliminar contenedores (datos persisten en volúmenes)
docker compose down

# Opción 3: Limpieza completa (elimina contenedores, redes Y volúmenes)
# ¡ADVERTENCIA: Esto elimina todos los datos de PostgreSQL y MongoDB!
docker compose down -v

# Eliminar las imágenes construidas en este laboratorio
docker rmi proyecto-integrador/backend:1.0.0
docker rmi proyecto-integrador/frontend:1.0.0

# Eliminar imágenes intermedias (dangling images)
docker image prune -f

# Verificar que los recursos fueron eliminados
docker compose ps
docker images | grep proyecto-integrador
docker volume ls | grep pi-
docker network ls | grep pi-
```

> ⚠️ **Advertencia:** El comando `docker compose down -v` eliminará permanentemente todos los datos almacenados en los volúmenes de PostgreSQL y MongoDB. Solo úsalo si quieres un entorno completamente limpio. Para simplemente detener los servicios sin perder datos, usa `docker compose stop`.

> ⚠️ **Nota sobre Git:** No elimines el repositorio local ni el remoto en GitHub. El código publicado en GitHub es el entregable principal de este laboratorio y será necesario para el Lab 7 (simulador de examen y revisión final).

## Resumen

### Lo que Lograste

- **Dockerfiles multi-etapa** para backend (Spring Boot) y frontend (Lit/nginx), optimizando el tamaño de las imágenes finales al separar las herramientas de build del runtime
- **Stack completo orquestado** con Docker Compose: 5 servicios (frontend, backend, PostgreSQL, MongoDB, pgAdmin) con redes segmentadas, volúmenes persistentes y health checks
- **Configuración segura** de variables de entorno con archivo `.env` (excluido de Git) y `.env.example` como plantilla pública
- **Repositorio GitHub profesional** con estructura de ramas (`main`, `develop`, `feature/*`), `.gitignore` completo, README documentado y mínimo 10 commits atómicos con mensajes descriptivos
- **Flujo de trabajo Git** completo: init, branch, commit, merge, tag y push a repositorio remoto

### Conceptos Clave Aprendidos

- Los **builds multi-etapa** de Docker permiten usar imágenes pesadas (Maven, Node.js) solo para compilar y producir una imagen final mínima con solo el runtime necesario
- Los **health checks** en Docker Compose garantizan que los servicios dependientes (como el backend) no arranquen hasta que sus dependencias (bases de datos) estén realmente listas para aceptar conexiones
- Las **redes personalizadas** en Docker Compose proporcionan segmentación y resolución de nombres por servicio (el backend puede conectarse a `postgres:5432` en lugar de `localhost:5432`)
- Los **volúmenes nombrados** garantizan que los datos persisten entre reinicios de contenedores
- Los **commits atómicos** con mensajes descriptivos (usando convención `tipo(scope): descripción`) facilitan el entendimiento del historial y la revisión de código en equipos

### Próximos Pasos

- Explorar la publicación de imágenes en **Docker Hub** (`docker push`) para distribuir el stack a otros miembros del equipo
- Investigar **Docker Compose profiles** para gestionar diferentes configuraciones (desarrollo vs. producción) en el mismo `compose.yml`
- Profundizar en **GitHub Actions** para automatizar el build y push de imágenes Docker en cada commit a `main` (CI/CD pipeline)
- Revisar las mejores prácticas de seguridad en Docker: usuarios no-root (ya implementado), escaneo de vulnerabilidades con `docker scout` y gestión de secretos con Docker Secrets

## Recursos Adicionales

- **Documentación oficial de Docker Compose** — Referencia completa de la sintaxis de `compose.yml`, incluyendo todos los campos disponibles: [https://docs.docker.com/compose/compose-file/](https://docs.docker.com/compose/compose-file/)
- **Mejores prácticas para escribir Dockerfiles** — Guía oficial con técnicas de optimización de capas, builds multi-etapa y seguridad: [https://docs.docker.com/develop/develop-images/dockerfile_best-practices/](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- **Convención de Commits (Conventional Commits)** — Especificación estándar para mensajes de commit atómicos y descriptivos usada en este laboratorio: [https://www.conventionalcommits.org/es/v1.0.0/](https://www.conventionalcommits.org/es/v1.0.0/)
- **Play with Docker** — Entorno interactivo en el navegador para practicar Docker sin instalación local, útil para experimentar con los comandos aprendidos: [https://labs.play-with-docker.com](https://labs.play-with-docker.com)
- **GitHub Docs: SSH Keys** — Guía para configurar autenticación SSH con GitHub, necesaria para el `git push` seguro: [https://docs.github.com/es/authentication/connecting-to-github-with-ssh](https://docs.github.com/es/authentication/connecting-to-github-with-ssh)
- **Docker Scout** — Herramienta de análisis de vulnerabilidades en imágenes Docker, siguiente paso en seguridad de contenedores: [https://docs.docker.com/scout/](https://docs.docker.com/scout/)
