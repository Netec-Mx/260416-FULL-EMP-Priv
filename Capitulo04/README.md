# Laboratorio 4. Proyecto Integrador — Procesamiento por Lotes con Spring Batch

## Metadatos

| Propiedad | Valor |
|-----------|-------|
| **Duración** | 180 minutos |
| **Complejidad** | Difícil |
| **Nivel Bloom** | Crear |
| **Tecnologías Principales** | Java 17, Spring Boot 3.2.x, Spring Batch 5.x, PostgreSQL 16.x |
| **Módulo** | Capítulo 4 — Spring Batch |

---

## Descripción General

En este laboratorio implementarás el módulo de procesamiento por lotes del proyecto integrador empresarial utilizando Spring Boot 3.2 y Spring Batch 5. Construirás un sistema completo de procesamiento batch que incluye un Job principal con tres Steps secuenciales (validación, procesamiento chunk-oriented y generación de reporte), un segundo Job con flujo condicional, políticas de tolerancia a fallos (skip y retry), persistencia del estado en PostgreSQL y endpoints REST para operar los jobs desde Postman.

Este laboratorio refleja patrones reales de arquitectura empresarial donde los sistemas necesitan procesar grandes volúmenes de datos de forma confiable, recuperarse de fallos parciales y ofrecer visibilidad del estado de ejecución a través de APIs. Al completarlo, habrás construido un módulo de producción que puede integrarse directamente en cualquier aplicación Spring Boot empresarial.

---

## Objetivos de Aprendizaje

Al completar este laboratorio, serás capaz de:

- [ ] Diseñar e implementar un Job de Spring Batch con múltiples Steps secuenciales usando configuración Java moderna (Spring Boot 3.2 / Spring Batch 5)
- [ ] Implementar el patrón Chunk-oriented con `FlatFileItemReader`, `ItemProcessor` personalizado y `JdbcBatchItemWriter` para transformar datos de un archivo CSV hacia PostgreSQL
- [ ] Desarrollar `Tasklet`s para operaciones atómicas como validación de entorno y generación de reportes de resumen
- [ ] Aplicar políticas de skip y retry con backoff exponencial para garantizar resiliencia en el procesamiento batch
- [ ] Utilizar `ExecutionContext` a nivel de Job para compartir métricas entre Steps y soportar reinicio desde el punto de falla
- [ ] Exponer endpoints REST con Spring MVC para lanzar Jobs, consultar estado de ejecución y listar ejecuciones históricas
- [ ] Configurar `JobRepository` con PostgreSQL para persistir metadata de ejecución de forma durable

---

## Prerrequisitos

### Conocimientos Requeridos

- Java 17: records, lambdas, anotaciones y manejo de excepciones
- Spring Boot: anotaciones `@Component`, `@Service`, `@Repository`, `@Configuration`, `@Bean`
- Spring Data JPA y JDBC básico: entidades, repositorios, `DataSource`
- Conceptos de Spring Batch de la Lección 4.1: Job, Step, Reader, Processor, Writer, Job Repository
- Maven: gestión de dependencias en `pom.xml`
- PostgreSQL: creación de bases de datos y ejecución de scripts SQL
- Haber completado el Laboratorio 2 o tener acceso a la rama Git de solución para conocer el dominio de datos

### Acceso Requerido

- PostgreSQL 16 corriendo localmente o vía Docker en puerto 5432
- IntelliJ IDEA 2024.1 con JDK 17 configurado
- Postman 11.x o extensión Thunder Client en VS Code
- Acceso a internet para descarga de dependencias Maven (primera ejecución)

---

## Entorno de Laboratorio

### Requisitos de Hardware

| Componente | Especificación |
|------------|----------------|
| **Procesador** | Intel Core i5 8va gen o AMD Ryzen 5 (con virtualización) |
| **Memoria RAM** | Mínimo 16 GB DDR4 (recomendado para PostgreSQL + Spring Boot) |
| **Almacenamiento** | Mínimo 5 GB libres para dependencias Maven y datos de prueba |
| **Pantalla** | 1920x1080 para trabajar con IDE y terminal simultáneamente |

### Requisitos de Software

| Software | Versión | Propósito |
|----------|---------|-----------|
| Java JDK | 17 LTS | Compilación y ejecución del proyecto |
| Apache Maven | 3.9.x | Gestión de dependencias y build |
| Spring Boot | 3.2.x | Framework base de la aplicación |
| Spring Batch | 5.x (incluido en Spring Boot 3.2) | Motor de procesamiento batch |
| PostgreSQL | 16.x | Persistencia de datos y metadata de Batch |
| IntelliJ IDEA | 2024.1 Community o Ultimate | IDE principal |
| Postman | 11.x | Prueba de endpoints REST |
| Docker Desktop | 4.29+ | Alternativa para PostgreSQL (opcional) |

### Configuración Inicial del Entorno

#### Opción A — PostgreSQL Local

Verifica que PostgreSQL esté corriendo y crea la base de datos del proyecto:

```bash
# Verificar que PostgreSQL está activo (Linux/macOS)
pg_isready -h localhost -p 5432

# Verificar que PostgreSQL está activo (Windows PowerShell)
Test-NetConnection -ComputerName localhost -Port 5432
```

```sql
-- Conectarse como superusuario y ejecutar:
CREATE DATABASE empresa_batch_db;
CREATE USER batch_user WITH ENCRYPTED PASSWORD 'batch_pass_2024';
GRANT ALL PRIVILEGES ON DATABASE empresa_batch_db TO batch_user;
\c empresa_batch_db
GRANT ALL ON SCHEMA public TO batch_user;
```

#### Opción B — PostgreSQL vía Docker (recomendado si no tienes instalación local)

```bash
docker run -d \
  --name postgres-batch \
  -e POSTGRES_DB=empresa_batch_db \
  -e POSTGRES_USER=batch_user \
  -e POSTGRES_PASSWORD=batch_pass_2024 \
  -p 5432:5432 \
  postgres:16-alpine
```

```bash
# Verificar que el contenedor está corriendo
docker ps | grep postgres-batch

# Verificar conectividad
docker exec -it postgres-batch psql -U batch_user -d empresa_batch_db -c "SELECT version();"
```

#### Verificación del JDK

```bash
java -version
# Debe mostrar: openjdk version "17.x.x"

mvn -version
# Debe mostrar: Apache Maven 3.9.x
```

---

## Instrucciones Paso a Paso

### Paso 1: Crear el Proyecto Spring Boot con Spring Batch

**Objetivo:** Generar la estructura base del proyecto con todas las dependencias necesarias para Spring Batch 5, PostgreSQL y Spring MVC.

**Instrucciones:**

1. Abre el navegador y navega a [https://start.spring.io](https://start.spring.io)

2. Configura el proyecto con los siguientes valores:
   - **Project:** Maven
   - **Language:** Java
   - **Spring Boot:** 3.2.5
   - **Group:** `com.empresa`
   - **Artifact:** `batch-processor`
   - **Name:** `batch-processor`
   - **Description:** `Módulo de procesamiento por lotes del proyecto integrador`
   - **Package name:** `com.empresa.batch`
   - **Packaging:** Jar
   - **Java:** 17

3. Agrega las siguientes dependencias:
   - Spring Batch
   - Spring Web
   - Spring Data JPA
   - PostgreSQL Driver
   - Lombok
   - Spring Boot Actuator
   - H2 Database (para tests)

4. Haz clic en **GENERATE**, descarga el ZIP y extráelo en tu directorio de trabajo:

```bash
# Linux/macOS
cd ~/proyectos
unzip batch-processor.zip
cd batch-processor

# Windows PowerShell
cd C:\proyectos
Expand-Archive -Path batch-processor.zip -DestinationPath .
cd batch-processor
```

5. Abre el proyecto en IntelliJ IDEA:

```bash
# Linux/macOS
idea . 
# O desde IntelliJ: File → Open → seleccionar la carpeta batch-processor

# Windows
idea64.exe .
```

6. Espera a que Maven descargue las dependencias (primera vez puede tomar 3-5 minutos). Verifica en la barra inferior de IntelliJ que el índice se complete.

7. Verifica el `pom.xml` generado. Debe contener estas dependencias clave:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.5</version>
        <relativePath/>
    </parent>
    
    <groupId>com.empresa</groupId>
    <artifactId>batch-processor</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>batch-processor</name>
    <description>Módulo de procesamiento por lotes del proyecto integrador</description>
    
    <properties>
        <java.version>17</java.version>
    </properties>
    
    <dependencies>
        <!-- Spring Batch -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-batch</artifactId>
        </dependency>
        
        <!-- Spring Web para endpoints REST -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- Spring Data JPA -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <!-- PostgreSQL Driver -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- Actuator para métricas -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        
        <!-- H2 para tests -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>test</scope>
        </dependency>
        
        <!-- Spring Boot Test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        
        <!-- Spring Batch Test -->
        <dependency>
            <groupId>org.springframework.batch</groupId>
            <artifactId>spring-batch-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**Salida Esperada:**

```
BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: XX.XXX s
```

**Verificación:**

- El proyecto se abre en IntelliJ sin errores de Maven
- La clase `BatchProcessorApplication.java` existe en `src/main/java/com/empresa/batch/`
- Maven muestra BUILD SUCCESS al ejecutar `mvn validate`

---

### Paso 2: Configurar application.properties y Estructura de Paquetes

**Objetivo:** Configurar la conexión a PostgreSQL, el comportamiento de Spring Batch y crear la estructura de paquetes del proyecto.

**Instrucciones:**

1. Abre el archivo `src/main/resources/application.properties` y reemplaza su contenido completo:

```properties
# ============================================================
# Configuración de la Aplicación
# ============================================================
spring.application.name=batch-processor
server.port=8080

# ============================================================
# Configuración de Base de Datos PostgreSQL
# ============================================================
spring.datasource.url=jdbc:postgresql://localhost:5432/empresa_batch_db
spring.datasource.username=batch_user
spring.datasource.password=batch_pass_2024
spring.datasource.driver-class-name=org.postgresql.Driver

# Pool de conexiones HikariCP
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=2
spring.datasource.hikari.connection-timeout=30000

# ============================================================
# Configuración de JPA/Hibernate
# ============================================================
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.format_sql=true

# ============================================================
# Configuración de Spring Batch
# ============================================================
# IMPORTANTE: En Spring Batch 5, los jobs NO se ejecutan automáticamente al inicio
spring.batch.job.enabled=false

# Spring Batch inicializa su schema automáticamente en PostgreSQL
spring.batch.jdbc.initialize-schema=always

# ============================================================
# Configuración de Actuator
# ============================================================
management.endpoints.web.exposure.include=health,info,metrics,batch
management.endpoint.health.show-details=always

# ============================================================
# Configuración de Logging
# ============================================================
logging.level.com.empresa.batch=DEBUG
logging.level.org.springframework.batch=INFO
logging.level.org.springframework.batch.core.step=DEBUG
```

2. Crea la estructura de paquetes del proyecto. En IntelliJ, haz clic derecho sobre `com.empresa.batch` y crea los siguientes sub-paquetes:

```
com.empresa.batch
├── config          (configuración de Jobs y Batch)
├── controller      (endpoints REST)
├── domain          (entidades y DTOs)
├── reader          (ItemReaders personalizados)
├── processor       (ItemProcessors)
├── writer          (ItemWriters)
├── tasklet         (Tasklets)
├── listener        (JobListeners y StepListeners)
└── service         (servicios de negocio)
```

Puedes crear todos los paquetes con este comando desde la raíz del proyecto:

```bash
# Linux/macOS
mkdir -p src/main/java/com/empresa/batch/{config,controller,domain,reader,processor,writer,tasklet,listener,service}
mkdir -p src/main/resources/data
mkdir -p src/test/java/com/empresa/batch/{config,processor,tasklet}

# Windows PowerShell
$baseDir = "src\main\java\com\empresa\batch"
@("config","controller","domain","reader","processor","writer","tasklet","listener","service") | ForEach-Object {
    New-Item -ItemType Directory -Path "$baseDir\$_" -Force
}
New-Item -ItemType Directory -Path "src\main\resources\data" -Force
```

3. Crea el archivo SQL para el esquema de negocio en `src/main/resources/schema-negocio.sql`:

```sql
-- Tabla de empleados (dominio de negocio)
CREATE TABLE IF NOT EXISTS empleados (
    id BIGSERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    apellido VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    departamento VARCHAR(50) NOT NULL,
    salario DECIMAL(12, 2) NOT NULL,
    fecha_contratacion DATE NOT NULL,
    activo BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabla de historial de procesamiento batch (para auditoría de negocio)
CREATE TABLE IF NOT EXISTS empleados_procesados (
    id BIGSERIAL PRIMARY KEY,
    empleado_id BIGINT,
    nombre_completo VARCHAR(200),
    departamento VARCHAR(50),
    salario_anterior DECIMAL(12, 2),
    salario_nuevo DECIMAL(12, 2),
    porcentaje_aumento DECIMAL(5, 2),
    job_execution_id BIGINT,
    fecha_procesamiento TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabla de reportes de ejecución batch
CREATE TABLE IF NOT EXISTS batch_reportes (
    id BIGSERIAL PRIMARY KEY,
    job_name VARCHAR(100) NOT NULL,
    job_execution_id BIGINT,
    total_leidos INTEGER DEFAULT 0,
    total_procesados INTEGER DEFAULT 0,
    total_escritos INTEGER DEFAULT 0,
    total_errores INTEGER DEFAULT 0,
    total_saltados INTEGER DEFAULT 0,
    fecha_inicio TIMESTAMP,
    fecha_fin TIMESTAMP,
    estado VARCHAR(20),
    mensaje TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

4. Ejecuta el script SQL en PostgreSQL:

```bash
# Linux/macOS
psql -h localhost -U batch_user -d empresa_batch_db -f src/main/resources/schema-negocio.sql

# Windows PowerShell
psql -h localhost -U batch_user -d empresa_batch_db -f src/main/resources/schema-negocio.sql

# Alternativa con Docker
docker exec -i postgres-batch psql -U batch_user -d empresa_batch_db < src/main/resources/schema-negocio.sql
```

**Salida Esperada:**

```
CREATE TABLE
CREATE TABLE
CREATE TABLE
```

**Verificación:**

- El archivo `application.properties` está guardado sin errores de sintaxis
- Los tres paquetes base existen en el explorador de IntelliJ
- Las tablas `empleados`, `empleados_procesados` y `batch_reportes` existen en PostgreSQL

---

### Paso 3: Crear el Dominio de Datos y el Archivo CSV de Prueba

**Objetivo:** Definir las clases del dominio (entidades JPA y DTOs) y crear el archivo CSV que servirá como fuente de datos para el procesamiento batch.

**Instrucciones:**

1. Crea la entidad `Empleado` en `src/main/java/com/empresa/batch/domain/Empleado.java`:

```java
package com.empresa.batch.domain;

import jakarta.persistence.*;
import lombok.*;
import java.math.BigDecimal;
import java.time.LocalDate;
import java.time.LocalDateTime;

@Entity
@Table(name = "empleados")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Empleado {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String nombre;

    @Column(nullable = false, length = 100)
    private String apellido;

    @Column(nullable = false, unique = true, length = 150)
    private String email;

    @Column(nullable = false, length = 50)
    private String departamento;

    @Column(nullable = false, precision = 12, scale = 2)
    private BigDecimal salario;

    @Column(name = "fecha_contratacion", nullable = false)
    private LocalDate fechaContratacion;

    @Column(nullable = false)
    @Builder.Default
    private Boolean activo = true;

    @Column(name = "created_at")
    private LocalDateTime createdAt;

    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
```

2. Crea el DTO para lectura del CSV en `src/main/java/com/empresa/batch/domain/EmpleadoCsvDto.java`:

```java
package com.empresa.batch.domain;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 * DTO que representa una fila del archivo CSV de empleados.
 * Los nombres de los campos deben coincidir exactamente con
 * las columnas definidas en FlatFileItemReader.
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class EmpleadoCsvDto {
    private String nombre;
    private String apellido;
    private String email;
    private String departamento;
    private String salario;           // String para validación antes de conversión
    private String fechaContratacion; // String para validación de formato
    private String activo;
}
```

3. Crea el DTO para el resultado del procesamiento en `src/main/java/com/empresa/batch/domain/EmpleadoProcesado.java`:

```java
package com.empresa.batch.domain;

import jakarta.persistence.*;
import lombok.*;
import java.math.BigDecimal;
import java.time.LocalDateTime;

@Entity
@Table(name = "empleados_procesados")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class EmpleadoProcesado {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "empleado_id")
    private Long empleadoId;

    @Column(name = "nombre_completo", length = 200)
    private String nombreCompleto;

    @Column(length = 50)
    private String departamento;

    @Column(name = "salario_anterior", precision = 12, scale = 2)
    private BigDecimal salarioAnterior;

    @Column(name = "salario_nuevo", precision = 12, scale = 2)
    private BigDecimal salarioNuevo;

    @Column(name = "porcentaje_aumento", precision = 5, scale = 2)
    private BigDecimal porcentajeAumento;

    @Column(name = "job_execution_id")
    private Long jobExecutionId;

    @Column(name = "fecha_procesamiento")
    private LocalDateTime fechaProcesamiento;
}
```

4. Crea el DTO de respuesta para la API REST en `src/main/java/com/empresa/batch/domain/JobStatusDto.java`:

```java
package com.empresa.batch.domain;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.LocalDateTime;
import java.util.Map;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class JobStatusDto {
    private Long jobExecutionId;
    private String jobName;
    private String status;
    private String exitStatus;
    private LocalDateTime startTime;
    private LocalDateTime endTime;
    private Map<String, Object> executionContext;
    private String failureException;
}
```

5. Crea el archivo CSV de datos de prueba en `src/main/resources/data/empleados.csv`:

```csv
nombre,apellido,email,departamento,salario,fechaContratacion,activo
Juan,García,juan.garcia@empresa.com,Tecnología,55000.00,2020-03-15,true
María,López,maria.lopez@empresa.com,Recursos Humanos,48000.00,2019-07-22,true
Carlos,Martínez,carlos.martinez@empresa.com,Finanzas,62000.00,2018-01-10,true
Ana,Rodríguez,ana.rodriguez@empresa.com,Tecnología,58000.00,2021-05-03,true
Pedro,Sánchez,pedro.sanchez@empresa.com,Marketing,45000.00,2022-02-14,true
Laura,González,laura.gonzalez@empresa.com,Finanzas,67000.00,2017-09-28,true
Miguel,Fernández,miguel.fernandez@empresa.com,Tecnología,72000.00,2016-11-05,true
Isabel,Torres,isabel.torres@empresa.com,Recursos Humanos,44000.00,2023-01-20,true
Roberto,Díaz,roberto.diaz@empresa.com,Marketing,49000.00,2021-08-17,true
Carmen,Ruiz,carmen.ruiz@empresa.com,Tecnología,61000.00,2020-06-30,true
Fernando,Jiménez,fernando.jimenez@empresa.com,Finanzas,INVALIDO,2019-04-12,true
Lucía,Moreno,lucia.moreno@empresa.com,Marketing,53000.00,2022-10-08,true
Antonio,Álvarez,antonio.alvarez@empresa.com,Tecnología,78000.00,2015-03-25,true
Pilar,Romero,pilar.romero@empresa.com,Recursos Humanos,46000.00,2023-06-11,true
David,Alonso,david.alonso@empresa.com,Finanzas,59000.00,2020-12-03,true
Elena,Navarro,elena.navarro@empresa.com,Tecnología,64000.00,2019-02-19,true
Jorge,Gutiérrez,jorge.gutierrez@empresa.com,Marketing,51000.00,2021-11-07,true
Sofía,Herrera,sofia.herrera@empresa.com,Finanzas,DATO_CORRUPTO,2018-07-14,true
Raúl,Medina,raul.medina@empresa.com,Tecnología,69000.00,2017-04-22,true
Beatriz,Castro,beatriz.castro@empresa.com,Recursos Humanos,47000.00,2022-08-30,true
```

> **Nota:** Los registros con `INVALIDO` y `DATO_CORRUPTO` en el campo salario son intencionales. Servirán para probar la política de skip configurada en el Paso 6.

**Salida Esperada:**

Al compilar el proyecto en IntelliJ (`Ctrl+F9` o `Cmd+F9`), debe mostrar:

```
BUILD SUCCESSFUL in Xs
```

**Verificación:**

- Las clases `Empleado`, `EmpleadoCsvDto`, `EmpleadoProcesado` y `JobStatusDto` compilan sin errores
- El archivo `empleados.csv` tiene exactamente 21 líneas (1 encabezado + 20 datos)
- Verifica con: `wc -l src/main/resources/data/empleados.csv` (Linux/macOS) o `(Get-Content src\main\resources\data\empleados.csv).Count` (Windows)

---

### Paso 4: Implementar el ItemReader, ItemProcessor y Tasklets

**Objetivo:** Crear los componentes de lectura, transformación y las tareas atómicas (Tasklets) que conformarán los Steps del Job principal.

**Instrucciones:**

1. Crea el `ItemProcessor` personalizado en `src/main/java/com/empresa/batch/processor/EmpleadoItemProcessor.java`:

```java
package com.empresa.batch.processor;

import com.empresa.batch.domain.Empleado;
import com.empresa.batch.domain.EmpleadoCsvDto;
import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.item.ItemProcessor;
import org.springframework.stereotype.Component;

import java.math.BigDecimal;
import java.math.RoundingMode;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.time.format.DateTimeParseException;

/**
 * Processor que transforma un EmpleadoCsvDto (datos crudos del CSV)
 * en una entidad Empleado con las reglas de negocio aplicadas.
 * 
 * Reglas de negocio implementadas:
 * - Aumento de salario del 8% para departamento Tecnología
 * - Aumento de salario del 5% para departamento Finanzas
 * - Aumento de salario del 3% para otros departamentos
 * - Validación de formato de email
 * - Validación de salario positivo
 * 
 * Si un registro es inválido, lanza IllegalArgumentException
 * que será capturada por la política de skip configurada en el Job.
 */
@Slf4j
@Component
public class EmpleadoItemProcessor implements ItemProcessor<EmpleadoCsvDto, Empleado> {

    private static final DateTimeFormatter FORMATO_FECHA = 
        DateTimeFormatter.ofPattern("yyyy-MM-dd");
    
    // Porcentajes de aumento por departamento
    private static final BigDecimal AUMENTO_TECNOLOGIA = new BigDecimal("0.08");
    private static final BigDecimal AUMENTO_FINANZAS = new BigDecimal("0.05");
    private static final BigDecimal AUMENTO_OTROS = new BigDecimal("0.03");

    @Override
    public Empleado process(EmpleadoCsvDto dto) throws Exception {
        log.debug("Procesando empleado: {} {}", dto.getNombre(), dto.getApellido());

        // Validación 1: Salario debe ser un número válido y positivo
        BigDecimal salarioOriginal;
        try {
            salarioOriginal = new BigDecimal(dto.getSalario().trim());
            if (salarioOriginal.compareTo(BigDecimal.ZERO) <= 0) {
                throw new IllegalArgumentException(
                    "Salario inválido (debe ser positivo) para: " + dto.getEmail()
                );
            }
        } catch (NumberFormatException e) {
            throw new IllegalArgumentException(
                "Formato de salario inválido '" + dto.getSalario() + 
                "' para empleado: " + dto.getNombre() + " " + dto.getApellido()
            );
        }

        // Validación 2: Fecha de contratación debe tener formato correcto
        LocalDate fechaContratacion;
        try {
            fechaContratacion = LocalDate.parse(dto.getFechaContratacion().trim(), FORMATO_FECHA);
        } catch (DateTimeParseException e) {
            throw new IllegalArgumentException(
                "Formato de fecha inválido '" + dto.getFechaContratacion() + 
                "' para empleado: " + dto.getNombre() + " " + dto.getApellido()
            );
        }

        // Validación 3: Email no puede estar vacío
        if (dto.getEmail() == null || dto.getEmail().trim().isEmpty()) {
            throw new IllegalArgumentException(
                "Email vacío para empleado: " + dto.getNombre() + " " + dto.getApellido()
            );
        }

        // Aplicar regla de negocio: aumento de salario según departamento
        BigDecimal porcentajeAumento = determinarPorcentajeAumento(dto.getDepartamento());
        BigDecimal salarioNuevo = salarioOriginal
            .multiply(BigDecimal.ONE.add(porcentajeAumento))
            .setScale(2, RoundingMode.HALF_UP);

        log.debug("Empleado {} {}: salario {} → {} (aumento {}%)",
            dto.getNombre(), dto.getApellido(),
            salarioOriginal, salarioNuevo,
            porcentajeAumento.multiply(new BigDecimal("100")).toPlainString());

        return Empleado.builder()
            .nombre(dto.getNombre().trim())
            .apellido(dto.getApellido().trim())
            .email(dto.getEmail().trim().toLowerCase())
            .departamento(dto.getDepartamento().trim())
            .salario(salarioNuevo)
            .fechaContratacion(fechaContratacion)
            .activo(Boolean.parseBoolean(dto.getActivo().trim()))
            .build();
    }

    private BigDecimal determinarPorcentajeAumento(String departamento) {
        if (departamento == null) return AUMENTO_OTROS;
        return switch (departamento.trim()) {
            case "Tecnología" -> AUMENTO_TECNOLOGIA;
            case "Finanzas"   -> AUMENTO_FINANZAS;
            default           -> AUMENTO_OTROS;
        };
    }
}
```

2. Crea el Tasklet de validación inicial en `src/main/java/com/empresa/batch/tasklet/ValidacionInicialTasklet.java`:

```java
package com.empresa.batch.tasklet;

import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.ClassPathResource;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

/**
 * Tasklet de Step 1: Valida que el archivo CSV de entrada existe
 * y es accesible antes de iniciar el procesamiento batch.
 * 
 * También inicializa el ExecutionContext del Job con metadatos
 * de inicio que serán usados por Steps posteriores.
 */
@Slf4j
@Component
public class ValidacionInicialTasklet implements Tasklet {

    @Value("${batch.input.file:data/empleados.csv}")
    private String archivoEntrada;

    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) 
            throws Exception {
        
        log.info("=== STEP 1: Iniciando validación del entorno batch ===");
        log.info("Timestamp de inicio: {}", LocalDateTime.now());
        log.info("Archivo a procesar: {}", archivoEntrada);

        // Validar que el archivo CSV existe en el classpath
        ClassPathResource recurso = new ClassPathResource(archivoEntrada);
        if (!recurso.exists()) {
            String mensaje = "ARCHIVO NO ENCONTRADO: " + archivoEntrada + 
                             ". Verifique que el archivo existe en src/main/resources/";
            log.error(mensaje);
            throw new IllegalStateException(mensaje);
        }

        long tamanoArchivo = recurso.contentLength();
        log.info("Archivo encontrado. Tamaño: {} bytes", tamanoArchivo);

        if (tamanoArchivo == 0) {
            throw new IllegalStateException("El archivo CSV está vacío: " + archivoEntrada);
        }

        // Guardar metadatos en el ExecutionContext del Job para uso posterior
        chunkContext.getStepContext()
            .getStepExecution()
            .getJobExecution()
            .getExecutionContext()
            .put("archivoEntrada", archivoEntrada);
        
        chunkContext.getStepContext()
            .getStepExecution()
            .getJobExecution()
            .getExecutionContext()
            .put("timestampInicio", LocalDateTime.now().toString());

        chunkContext.getStepContext()
            .getStepExecution()
            .getJobExecution()
            .getExecutionContext()
            .putLong("tamanoArchivo", tamanoArchivo);

        log.info("=== STEP 1: Validación completada exitosamente ===");
        
        // RepeatStatus.FINISHED indica que el Tasklet terminó y no debe repetirse
        return RepeatStatus.FINISHED;
    }
}
```

3. Crea el Tasklet de generación de reporte en `src/main/java/com/empresa/batch/tasklet/GeneracionReporteTasklet.java`:

```java
package com.empresa.batch.tasklet;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.item.ExecutionContext;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

/**
 * Tasklet de Step 3: Genera un reporte de resumen de la ejecución batch
 * y lo persiste en la tabla batch_reportes para auditoría.
 * 
 * Lee las métricas del ExecutionContext del Job que fueron
 * escritas por el Step 2 (procesamiento chunk-oriented).
 */
@Slf4j
@Component
@RequiredArgsConstructor
public class GeneracionReporteTasklet implements Tasklet {

    private final JdbcTemplate jdbcTemplate;

    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) 
            throws Exception {
        
        log.info("=== STEP 3: Generando reporte de resumen ===");

        // Leer métricas del ExecutionContext del Job (escritas por Step 2)
        ExecutionContext jobContext = chunkContext.getStepContext()
            .getStepExecution()
            .getJobExecution()
            .getExecutionContext();

        Long jobExecutionId = chunkContext.getStepContext()
            .getStepExecution()
            .getJobExecution()
            .getId();

        String jobName = chunkContext.getStepContext()
            .getStepExecution()
            .getJobExecution()
            .getJobInstance()
            .getJobName();

        // Leer métricas del Step 2 del ExecutionContext
        int totalLeidos     = jobContext.containsKey("totalLeidos")     ? jobContext.getInt("totalLeidos")     : 0;
        int totalProcesados = jobContext.containsKey("totalProcesados") ? jobContext.getInt("totalProcesados") : 0;
        int totalEscritos   = jobContext.containsKey("totalEscritos")   ? jobContext.getInt("totalEscritos")   : 0;
        int totalSaltados   = jobContext.containsKey("totalSaltados")   ? jobContext.getInt("totalSaltados")   : 0;
        String timestampInicio = jobContext.containsKey("timestampInicio") 
            ? jobContext.getString("timestampInicio") : "N/A";

        log.info("--- Resumen de Ejecución ---");
        log.info("Job: {}", jobName);
        log.info("Job Execution ID: {}", jobExecutionId);
        log.info("Registros leídos:      {}", totalLeidos);
        log.info("Registros procesados:  {}", totalProcesados);
        log.info("Registros escritos:    {}", totalEscritos);
        log.info("Registros saltados:    {}", totalSaltados);
        log.info("Inicio del proceso:    {}", timestampInicio);
        log.info("Fin del proceso:       {}", LocalDateTime.now());

        // Persistir el reporte en la base de datos
        String sql = """
            INSERT INTO batch_reportes 
                (job_name, job_execution_id, total_leidos, total_procesados, 
                 total_escritos, total_errores, total_saltados, 
                 fecha_inicio, fecha_fin, estado, mensaje)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
            """;

        jdbcTemplate.update(sql,
            jobName,
            jobExecutionId,
            totalLeidos,
            totalProcesados,
            totalEscritos,
            0,
            totalSaltados,
            LocalDateTime.parse(timestampInicio.equals("N/A") ? LocalDateTime.now().toString() : timestampInicio),
            LocalDateTime.now(),
            "COMPLETADO",
            String.format("Procesamiento finalizado. Leídos: %d, Procesados: %d, Escritos: %d, Saltados: %d",
                totalLeidos, totalProcesados, totalEscritos, totalSaltados)
        );

        log.info("Reporte persistido en tabla batch_reportes.");
        log.info("=== STEP 3: Reporte generado exitosamente ===");

        return RepeatStatus.FINISHED;
    }
}
```

**Salida Esperada:**

El proyecto compila sin errores. En IntelliJ, la pestaña "Build" muestra:

```
BUILD SUCCESSFUL
3 files compiled
```

**Verificación:**

- `EmpleadoItemProcessor` implementa correctamente `ItemProcessor<EmpleadoCsvDto, Empleado>`
- `ValidacionInicialTasklet` e `GeneracionReporteTasklet` implementan `Tasklet`
- Ninguna clase tiene errores de compilación (sin líneas rojas en IntelliJ)

---

### Paso 5: Implementar el JobExecutionListener y StepExecutionListener

**Objetivo:** Crear listeners que capturen métricas de ejecución y las almacenen en el `ExecutionContext` para compartirlas entre Steps.

**Instrucciones:**

1. Crea el listener de Job en `src/main/java/com/empresa/batch/listener/EmpleadosJobListener.java`:

```java
package com.empresa.batch.listener;

import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.JobExecution;
import org.springframework.batch.core.JobExecutionListener;
import org.springframework.stereotype.Component;

/**
 * Listener a nivel de Job que registra el inicio y fin de cada ejecución.
 * Proporciona visibilidad del ciclo de vida completo del Job en los logs.
 */
@Slf4j
@Component
public class EmpleadosJobListener implements JobExecutionListener {

    @Override
    public void beforeJob(JobExecution jobExecution) {
        log.info("╔══════════════════════════════════════════════════════╗");
        log.info("║     INICIANDO JOB: {}                                ", 
            jobExecution.getJobInstance().getJobName());
        log.info("║     Execution ID: {}                                 ", 
            jobExecution.getId());
        log.info("║     Parámetros: {}                                   ", 
            jobExecution.getJobParameters());
        log.info("╚══════════════════════════════════════════════════════╝");
    }

    @Override
    public void afterJob(JobExecution jobExecution) {
        log.info("╔══════════════════════════════════════════════════════╗");
        log.info("║     JOB FINALIZADO: {}                               ", 
            jobExecution.getJobInstance().getJobName());
        log.info("║     Estado: {}                                       ", 
            jobExecution.getStatus());
        log.info("║     Exit Status: {}                                  ", 
            jobExecution.getExitStatus().getExitCode());
        
        if (!jobExecution.getAllFailureExceptions().isEmpty()) {
            log.error("║     EXCEPCIONES REGISTRADAS:                        ");
            jobExecution.getAllFailureExceptions().forEach(ex -> 
                log.error("║       - {}", ex.getMessage()));
        }
        log.info("╚══════════════════════════════════════════════════════╝");
    }
}
```

2. Crea el listener de Step en `src/main/java/com/empresa/batch/listener/EmpleadosStepListener.java`:

```java
package com.empresa.batch.listener;

import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.ExitStatus;
import org.springframework.batch.core.StepExecution;
import org.springframework.batch.core.StepExecutionListener;
import org.springframework.stereotype.Component;

/**
 * Listener a nivel de Step que captura métricas de ejecución del Step 2
 * (procesamiento chunk-oriented) y las escribe en el ExecutionContext del Job
 * para que el Step 3 (GeneracionReporteTasklet) pueda leerlas.
 */
@Slf4j
@Component
public class EmpleadosStepListener implements StepExecutionListener {

    @Override
    public void beforeStep(StepExecution stepExecution) {
        log.info("--- Iniciando Step: {} ---", stepExecution.getStepName());
    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        log.info("--- Finalizando Step: {} ---", stepExecution.getStepName());
        log.info("    Leídos:     {}", stepExecution.getReadCount());
        log.info("    Procesados: {}", stepExecution.getWriteCount());
        log.info("    Saltados:   {}", stepExecution.getSkipCount());
        log.info("    Filtrados:  {}", stepExecution.getFilterCount());

        // Solo para el Step de procesamiento chunk-oriented, guardar métricas en JobContext
        if (stepExecution.getStepName().equals("stepProcesarEmpleados")) {
            stepExecution.getJobExecution().getExecutionContext()
                .putInt("totalLeidos", stepExecution.getReadCount());
            stepExecution.getJobExecution().getExecutionContext()
                .putInt("totalProcesados", stepExecution.getWriteCount());
            stepExecution.getJobExecution().getExecutionContext()
                .putInt("totalEscritos", stepExecution.getWriteCount());
            stepExecution.getJobExecution().getExecutionContext()
                .putInt("totalSaltados", stepExecution.getSkipCount());
            
            log.info("    Métricas guardadas en ExecutionContext del Job.");
        }

        return stepExecution.getExitStatus();
    }
}
```

**Salida Esperada:**

Compilación exitosa. Las dos clases listener están disponibles como beans de Spring.

**Verificación:**

- `EmpleadosJobListener` implementa `JobExecutionListener`
- `EmpleadosStepListener` implementa `StepExecutionListener`
- Ambas clases tienen la anotación `@Component` y son detectadas por Spring

---

### Paso 6: Configurar el Job Principal con Tres Steps y Políticas de Tolerancia a Fallos

**Objetivo:** Crear la configuración central del Job principal (`procesarEmpleadosJob`) con los tres Steps secuenciales, el FlatFileItemReader, el JdbcBatchItemWriter, y las políticas de skip y retry.

**Instrucciones:**

1. Crea la clase de configuración del Job en `src/main/java/com/empresa/batch/config/EmpleadosJobConfig.java`:

```java
package com.empresa.batch.config;

import com.empresa.batch.domain.Empleado;
import com.empresa.batch.domain.EmpleadoCsvDto;
import com.empresa.batch.listener.EmpleadosJobListener;
import com.empresa.batch.listener.EmpleadosStepListener;
import com.empresa.batch.processor.EmpleadoItemProcessor;
import com.empresa.batch.tasklet.GeneracionReporteTasklet;
import com.empresa.batch.tasklet.ValidacionInicialTasklet;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.job.builder.JobBuilder;
import org.springframework.batch.core.repository.JobRepository;
import org.springframework.batch.core.step.builder.StepBuilder;
import org.springframework.batch.item.database.JdbcBatchItemWriter;
import org.springframework.batch.item.database.builder.JdbcBatchItemWriterBuilder;
import org.springframework.batch.item.file.FlatFileItemReader;
import org.springframework.batch.item.file.builder.FlatFileItemReaderBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.transaction.PlatformTransactionManager;

import javax.sql.DataSource;

/**
 * Configuración principal del Job de procesamiento de empleados.
 * 
 * Arquitectura del Job:
 * ┌─────────────────────────────────────────────────────────────┐
 * │              procesarEmpleadosJob                           │
 * │                                                             │
 * │  Step 1 (Tasklet)    →  Step 2 (Chunk)  →  Step 3 (Tasklet)│
 * │  ValidacionInicial      ProcesarEmpleados   GenerarReporte  │
 * │                         chunk-size: 10                      │
 * │                         skip: max 5                         │
 * │                         retry: max 3                        │
 * └─────────────────────────────────────────────────────────────┘
 */
@Slf4j
@Configuration
@RequiredArgsConstructor
public class EmpleadosJobConfig {

    private final JobRepository jobRepository;
    private final PlatformTransactionManager transactionManager;
    private final ValidacionInicialTasklet validacionInicialTasklet;
    private final GeneracionReporteTasklet generacionReporteTasklet;
    private final EmpleadoItemProcessor empleadoItemProcessor;
    private final EmpleadosJobListener empleadosJobListener;
    private final EmpleadosStepListener empleadosStepListener;

    // =========================================================
    // DEFINICIÓN DEL JOB PRINCIPAL
    // =========================================================

    @Bean
    public Job procesarEmpleadosJob(Step stepValidacionInicial,
                                     Step stepProcesarEmpleados,
                                     Step stepGenerarReporte) {
        return new JobBuilder("procesarEmpleadosJob", jobRepository)
            .listener(empleadosJobListener)
            .start(stepValidacionInicial)
            .next(stepProcesarEmpleados)
            .next(stepGenerarReporte)
            .build();
    }

    // =========================================================
    // STEP 1: TASKLET DE VALIDACIÓN INICIAL
    // =========================================================

    @Bean
    public Step stepValidacionInicial() {
        return new StepBuilder("stepValidacionInicial", jobRepository)
            .tasklet(validacionInicialTasklet, transactionManager)
            .build();
    }

    // =========================================================
    // STEP 2: PROCESAMIENTO CHUNK-ORIENTED
    // =========================================================

    /**
     * Step principal de procesamiento.
     * 
     * Configuración de tolerancia a fallos:
     * - skip: Salta registros con IllegalArgumentException (datos inválidos del CSV)
     *   Máximo 5 registros saltados antes de fallar el Job.
     * - retry: Reintenta ante DataAccessException (errores transitorios de BD)
     *   Máximo 3 reintentos con backoff exponencial (1s, 2s, 4s).
     */
    @Bean
    public Step stepProcesarEmpleados(FlatFileItemReader<EmpleadoCsvDto> empleadoCsvReader,
                                       JdbcBatchItemWriter<Empleado> empleadoWriter) {
        return new StepBuilder("stepProcesarEmpleados", jobRepository)
            .<EmpleadoCsvDto, Empleado>chunk(10, transactionManager)
            .reader(empleadoCsvReader)
            .processor(empleadoItemProcessor)
            .writer(empleadoWriter)
            // Política de Skip: omitir registros con datos inválidos
            .faultTolerant()
            .skip(IllegalArgumentException.class)
            .skipLimit(5)
            // Política de Retry: reintentar ante errores transitorios de BD
            .retry(org.springframework.dao.DataAccessException.class)
            .retryLimit(3)
            // Listener para capturar métricas del Step
            .listener(empleadosStepListener)
            .build();
    }

    // =========================================================
    // STEP 3: TASKLET DE GENERACIÓN DE REPORTE
    // =========================================================

    @Bean
    public Step stepGenerarReporte() {
        return new StepBuilder("stepGenerarReporte", jobRepository)
            .tasklet(generacionReporteTasklet, transactionManager)
            .build();
    }

    // =========================================================
    // ITEM READER: LEE DESDE ARCHIVO CSV
    // =========================================================

    /**
     * FlatFileItemReader configurado para leer el archivo empleados.csv.
     * 
     * Configuración:
     * - linesToSkip(1): Omite la primera línea (encabezado del CSV)
     * - delimited(): Usa coma como delimitador
     * - names(): Mapea columnas a campos del DTO por nombre
     * - targetType(): Instancia automáticamente el DTO con los valores
     */
    @Bean
    public FlatFileItemReader<EmpleadoCsvDto> empleadoCsvReader() {
        return new FlatFileItemReaderBuilder<EmpleadoCsvDto>()
            .name("empleadoCsvReader")
            .resource(new ClassPathResource("data/empleados.csv"))
            .linesToSkip(1) // Saltar la línea de encabezado
            .delimited()
            .delimiter(",")
            .names("nombre", "apellido", "email", "departamento", 
                   "salario", "fechaContratacion", "activo")
            .targetType(EmpleadoCsvDto.class)
            .build();
    }

    // =========================================================
    // ITEM WRITER: ESCRIBE EN POSTGRESQL
    // =========================================================

    /**
     * JdbcBatchItemWriter que inserta empleados procesados en PostgreSQL.
     * 
     * Usa BeanPropertyItemSqlParameterSourceProvider para mapear
     * automáticamente los campos del objeto Empleado a los parámetros
     * con nombre (:nombre, :apellido, etc.) del SQL.
     */
    @Bean
    public JdbcBatchItemWriter<Empleado> empleadoWriter(DataSource dataSource) {
        return new JdbcBatchItemWriterBuilder<Empleado>()
            .dataSource(dataSource)
            .sql("""
                INSERT INTO empleados 
                    (nombre, apellido, email, departamento, salario, 
                     fecha_contratacion, activo)
                VALUES 
                    (:nombre, :apellido, :email, :departamento, :salario, 
                     :fechaContratacion, :activo)
                ON CONFLICT (email) DO UPDATE SET
                    salario = EXCLUDED.salario,
                    updated_at = CURRENT_TIMESTAMP
                """)
            .beanMapped()
            .build();
    }
}
```

**Salida Esperada:**

IntelliJ muestra el proyecto compilado sin errores. En la vista "Spring" del IDE, debes ver el bean `procesarEmpleadosJob` registrado.

**Verificación:**

- La clase `EmpleadosJobConfig` compila sin errores
- Los métodos `@Bean` para el Job, los tres Steps, el Reader y el Writer están definidos
- Ejecuta `mvn compile` desde la terminal para confirmar:

```bash
mvn compile
# Debe mostrar: BUILD SUCCESS
```

---

### Paso 7: Implementar el Segundo Job con Flujo Condicional

**Objetivo:** Crear un segundo Job que demuestre el uso de `JobExecutionDecider` para tomar decisiones de flujo condicional basadas en el resultado de un Step.

**Instrucciones:**

1. Crea el Decider en `src/main/java/com/empresa/batch/config/EmpleadosJobDecider.java`:

```java
package com.empresa.batch.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.JobExecution;
import org.springframework.batch.core.StepExecution;
import org.springframework.batch.core.job.flow.FlowExecutionStatus;
import org.springframework.batch.core.job.flow.JobExecutionDecider;
import org.springframework.stereotype.Component;

/**
 * Decider que evalúa el resultado del procesamiento batch y determina
 * el flujo de ejecución del Job condicional.
 * 
 * Lógica de decisión:
 * - Si se saltaron registros (skips > 0): flujo "CON_ERRORES"
 * - Si todos los registros se procesaron correctamente: flujo "EXITOSO"
 * - Si no se procesó ningún registro: flujo "SIN_DATOS"
 */
@Slf4j
@Component
public class EmpleadosJobDecider implements JobExecutionDecider {

    @Override
    public FlowExecutionStatus decide(JobExecution jobExecution, StepExecution stepExecution) {
        
        int totalEscritos = jobExecution.getExecutionContext()
            .containsKey("totalEscritos") 
            ? jobExecution.getExecutionContext().getInt("totalEscritos") 
            : 0;
        
        int totalSaltados = jobExecution.getExecutionContext()
            .containsKey("totalSaltados") 
            ? jobExecution.getExecutionContext().getInt("totalSaltados") 
            : 0;

        log.info("Decider evaluando: escritos={}, saltados={}", totalEscritos, totalSaltados);

        if (totalEscritos == 0) {
            log.warn("Decider → SIN_DATOS: No se procesó ningún registro.");
            return new FlowExecutionStatus("SIN_DATOS");
        } else if (totalSaltados > 0) {
            log.warn("Decider → CON_ERRORES: {} registros fueron saltados.", totalSaltados);
            return new FlowExecutionStatus("CON_ERRORES");
        } else {
            log.info("Decider → EXITOSO: Todos los registros procesados correctamente.");
            return new FlowExecutionStatus("EXITOSO");
        }
    }
}
```

2. Crea los Tasklets para las rutas condicionales en `src/main/java/com/empresa/batch/tasklet/NotificacionExitoTasklet.java`:

```java
package com.empresa.batch.tasklet;

import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.stereotype.Component;

/**
 * Tasklet que simula el envío de notificación de éxito.
 * En producción, aquí se integraría con un servicio de email o Slack.
 */
@Slf4j
@Component
public class NotificacionExitoTasklet implements Tasklet {

    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) 
            throws Exception {
        int totalEscritos = chunkContext.getStepContext()
            .getStepExecution().getJobExecution()
            .getExecutionContext().getInt("totalEscritos");
        
        log.info("✅ NOTIFICACIÓN DE ÉXITO: Procesamiento completado sin errores.");
        log.info("   Total de empleados procesados: {}", totalEscritos);
        log.info("   [SIMULACIÓN] Email enviado a: operaciones@empresa.com");
        
        return RepeatStatus.FINISHED;
    }
}
```

3. Crea `src/main/java/com/empresa/batch/tasklet/NotificacionErrorTasklet.java`:

```java
package com.empresa.batch.tasklet;

import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.stereotype.Component;

/**
 * Tasklet que simula el envío de notificación de advertencia cuando
 * se detectaron registros con errores durante el procesamiento.
 */
@Slf4j
@Component
public class NotificacionErrorTasklet implements Tasklet {

    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) 
            throws Exception {
        int totalSaltados = chunkContext.getStepContext()
            .getStepExecution().getJobExecution()
            .getExecutionContext().getInt("totalSaltados");
        
        log.warn("⚠️  NOTIFICACIÓN DE ADVERTENCIA: Procesamiento completado con errores.");
        log.warn("   Registros saltados por datos inválidos: {}", totalSaltados);
        log.warn("   [SIMULACIÓN] Alerta enviada a: calidad-datos@empresa.com");
        log.warn("   Revisar archivo de log para detalles de registros saltados.");
        
        return RepeatStatus.FINISHED;
    }
}
```

4. Crea la configuración del segundo Job en `src/main/java/com/empresa/batch/config/EmpleadosCondicionalJobConfig.java`:

```java
package com.empresa.batch.config;

import com.empresa.batch.domain.Empleado;
import com.empresa.batch.domain.EmpleadoCsvDto;
import com.empresa.batch.listener.EmpleadosJobListener;
import com.empresa.batch.listener.EmpleadosStepListener;
import com.empresa.batch.processor.EmpleadoItemProcessor;
import com.empresa.batch.tasklet.GeneracionReporteTasklet;
import com.empresa.batch.tasklet.NotificacionErrorTasklet;
import com.empresa.batch.tasklet.NotificacionExitoTasklet;
import com.empresa.batch.tasklet.ValidacionInicialTasklet;
import lombok.RequiredArgsConstructor;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.job.builder.JobBuilder;
import org.springframework.batch.core.repository.JobRepository;
import org.springframework.batch.core.step.builder.StepBuilder;
import org.springframework.batch.item.database.JdbcBatchItemWriter;
import org.springframework.batch.item.file.FlatFileItemReader;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.PlatformTransactionManager;

/**
 * Configuración del Job condicional que usa JobExecutionDecider
 * para determinar el flujo de ejecución según el resultado del procesamiento.
 * 
 * Flujo:
 * ┌──────────────────────────────────────────────────────────────────┐
 * │                  procesarEmpleadosCondicionalJob                  │
 * │                                                                  │
 * │  stepValidacion → stepProcesar → [DECIDER] → EXITOSO             │
 * │                                            ↓                     │
 * │                                         CON_ERRORES → stepAlerta │
 * │                                            ↓                     │
 * │                                         SIN_DATOS  → (termina)  │
 * │                                            ↓                     │
 * │                                    stepReporte (siempre)         │
 * └──────────────────────────────────────────────────────────────────┘
 */
@Configuration
@RequiredArgsConstructor
public class EmpleadosCondicionalJobConfig {

    private final JobRepository jobRepository;
    private final PlatformTransactionManager transactionManager;
    private final ValidacionInicialTasklet validacionInicialTasklet;
    private final GeneracionReporteTasklet generacionReporteTasklet;
    private final NotificacionExitoTasklet notificacionExitoTasklet;
    private final NotificacionErrorTasklet notificacionErrorTasklet;
    private final EmpleadoItemProcessor empleadoItemProcessor;
    private final EmpleadosJobListener empleadosJobListener;
    private final EmpleadosStepListener empleadosStepListener;
    private final EmpleadosJobDecider empleadosJobDecider;

    @Bean
    public Job procesarEmpleadosCondicionalJob(
            FlatFileItemReader<EmpleadoCsvDto> empleadoCsvReader,
            JdbcBatchItemWriter<Empleado> empleadoWriter) {

        // Steps locales para este Job (reutilizando los mismos Tasklets/Processors)
        Step stepValidacion = new StepBuilder("stepValidacionCondicional", jobRepository)
            .tasklet(validacionInicialTasklet, transactionManager)
            .build();

        Step stepProcesar = new StepBuilder("stepProcesarCondicional", jobRepository)
            .<EmpleadoCsvDto, Empleado>chunk(10, transactionManager)
            .reader(empleadoCsvReader)
            .processor(empleadoItemProcessor)
            .writer(empleadoWriter)
            .faultTolerant()
            .skip(IllegalArgumentException.class)
            .skipLimit(5)
            .retry(org.springframework.dao.DataAccessException.class)
            .retryLimit(3)
            .listener(empleadosStepListener)
            .build();

        Step stepNotificacionExito = new StepBuilder("stepNotificacionExito", jobRepository)
            .tasklet(notificacionExitoTasklet, transactionManager)
            .build();

        Step stepNotificacionError = new StepBuilder("stepNotificacionError", jobRepository)
            .tasklet(notificacionErrorTasklet, transactionManager)
            .build();

        Step stepReporteFinal = new StepBuilder("stepReporteCondicional", jobRepository)
            .tasklet(generacionReporteTasklet, transactionManager)
            .build();

        return new JobBuilder("procesarEmpleadosCondicionalJob", jobRepository)
            .listener(empleadosJobListener)
            .start(stepValidacion)
            .next(stepProcesar)
            // Aplicar el Decider después del Step de procesamiento
            .next(empleadosJobDecider)
                .on("EXITOSO").to(stepNotificacionExito)
            .from(empleadosJobDecider)
                .on("CON_ERRORES").to(stepNotificacionError)
            .from(empleadosJobDecider)
                .on("SIN_DATOS").end()
            // Después de cualquier notificación, ir al reporte final
            .from(stepNotificacionExito).next(stepReporteFinal)
            .from(stepNotificacionError).next(stepReporteFinal)
            .end()
            .build();
    }
}
```

**Salida Esperada:**

```bash
mvn compile
# [INFO] BUILD SUCCESS
```

**Verificación:**

- El `EmpleadosJobDecider` implementa `JobExecutionDecider`
- Los tres Tasklets de notificación compilan correctamente
- El flujo condicional en `procesarEmpleadosCondicionalJob` referencia correctamente los tres estados del Decider

---

### Paso 8: Crear los Endpoints REST para Operar los Jobs

**Objetivo:** Exponer una API REST con Spring MVC que permita lanzar Jobs, consultar su estado y listar el historial de ejecuciones.

**Instrucciones:**

1. Crea el servicio de Jobs en `src/main/java/com/empresa/batch/service/BatchJobService.java`:

```java
package com.empresa.batch.service;

import com.empresa.batch.domain.JobStatusDto;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.*;
import org.springframework.batch.core.explore.JobExplorer;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.batch.core.repository.JobExecutionAlreadyRunningException;
import org.springframework.batch.core.repository.JobInstanceAlreadyCompleteException;
import org.springframework.batch.core.repository.JobRestartException;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.time.ZoneId;
import java.util.*;
import java.util.stream.Collectors;

/**
 * Servicio que encapsula la lógica de lanzamiento y consulta de Jobs.
 * Usa JobLauncher para iniciar ejecuciones y JobExplorer para consultar
 * el estado de ejecuciones históricas desde el JobRepository.
 */
@Slf4j
@Service
@RequiredArgsConstructor
public class BatchJobService {

    private final JobLauncher jobLauncher;
    private final JobExplorer jobExplorer;
    private final Job procesarEmpleadosJob;
    private final Job procesarEmpleadosCondicionalJob;

    /**
     * Lanza el Job principal de procesamiento de empleados.
     * Cada ejecución requiere parámetros únicos (timestamp) para que
     * Spring Batch la trate como una nueva instancia de Job.
     */
    public JobStatusDto lanzarJobPrincipal() {
        try {
            JobParameters params = new JobParametersBuilder()
                .addLocalDateTime("timestamp", LocalDateTime.now())
                .addString("origen", "REST_API")
                .toJobParameters();

            log.info("Lanzando procesarEmpleadosJob con parámetros: {}", params);
            JobExecution execution = jobLauncher.run(procesarEmpleadosJob, params);
            
            return mapearADto(execution);

        } catch (JobExecutionAlreadyRunningException e) {
            log.error("El job ya está en ejecución: {}", e.getMessage());
            throw new RuntimeException("El job ya está en ejecución. Espere a que termine.", e);
        } catch (JobInstanceAlreadyCompleteException e) {
            log.warn("Job ya completado con estos parámetros. Usando timestamp único.");
            throw new RuntimeException("Job ya completado. Intente nuevamente.", e);
        } catch (JobRestartException e) {
            log.error("Error al reiniciar el job: {}", e.getMessage());
            throw new RuntimeException("Error al reiniciar el job: " + e.getMessage(), e);
        } catch (Exception e) {
            log.error("Error inesperado al lanzar el job: {}", e.getMessage(), e);
            throw new RuntimeException("Error al lanzar el job: " + e.getMessage(), e);
        }
    }

    /**
     * Lanza el Job condicional de procesamiento de empleados.
     */
    public JobStatusDto lanzarJobCondicional() {
        try {
            JobParameters params = new JobParametersBuilder()
                .addLocalDateTime("timestamp", LocalDateTime.now())
                .addString("origen", "REST_API_CONDICIONAL")
                .toJobParameters();

            log.info("Lanzando procesarEmpleadosCondicionalJob con parámetros: {}", params);
            JobExecution execution = jobLauncher.run(procesarEmpleadosCondicionalJob, params);
            
            return mapearADto(execution);

        } catch (Exception e) {
            log.error("Error al lanzar el job condicional: {}", e.getMessage(), e);
            throw new RuntimeException("Error al lanzar el job condicional: " + e.getMessage(), e);
        }
    }

    /**
     * Consulta el estado de una ejecución específica por su ID.
     */
    public Optional<JobStatusDto> consultarEjecucion(Long jobExecutionId) {
        JobExecution execution = jobExplorer.getJobExecution(jobExecutionId);
        if (execution == null) {
            return Optional.empty();
        }
        return Optional.of(mapearADto(execution));
    }

    /**
     * Lista todas las ejecuciones históricas de un Job específico.
     */
    public List<JobStatusDto> listarEjecuciones(String jobName) {
        return jobExplorer.getJobInstances(jobName, 0, 50)
            .stream()
            .flatMap(instance -> jobExplorer.getJobExecutions(instance).stream())
            .sorted(Comparator.comparing(JobExecution::getId).reversed())
            .map(this::mapearADto)
            .collect(Collectors.toList());
    }

    /**
     * Lista todos los nombres de Jobs registrados en el JobRepository.
     */
    public List<String> listarNombresJobs() {
        return jobExplorer.getJobNames();
    }

    // Mapea un JobExecution a un DTO para la respuesta REST
    private JobStatusDto mapearADto(JobExecution execution) {
        Map<String, Object> contextMap = new HashMap<>();
        execution.getExecutionContext().entrySet()
            .forEach(entry -> contextMap.put(entry.getKey(), entry.getValue()));

        String excepcion = execution.getAllFailureExceptions().isEmpty() ? null
            : execution.getAllFailureExceptions().get(0).getMessage();

        return JobStatusDto.builder()
            .jobExecutionId(execution.getId())
            .jobName(execution.getJobInstance().getJobName())
            .status(execution.getStatus().name())
            .exitStatus(execution.getExitStatus().getExitCode())
            .startTime(execution.getStartTime() != null
                ? execution.getStartTime().atZone(ZoneId.systemDefault()).toLocalDateTime()
                : null)
            .endTime(execution.getEndTime() != null
                ? execution.getEndTime().atZone(ZoneId.systemDefault()).toLocalDateTime()
                : null)
            .executionContext(contextMap)
            .failureException(excepcion)
            .build();
    }
}
```

2. Crea el controlador REST en `src/main/java/com/empresa/batch/controller/BatchJobController.java`:

```java
package com.empresa.batch.controller;

import com.empresa.batch.domain.JobStatusDto;
import com.empresa.batch.service.BatchJobService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Map;

/**
 * Controlador REST para gestión de Jobs de Spring Batch.
 * 
 * Endpoints disponibles:
 * POST /api/batch/jobs/empleados/ejecutar          - Lanza el Job principal
 * POST /api/batch/jobs/empleados-condicional/ejecutar - Lanza el Job condicional
 * GET  /api/batch/jobs/ejecuciones/{id}            - Consulta estado de ejecución
 * GET  /api/batch/jobs/{jobName}/historial         - Lista historial de ejecuciones
 * GET  /api/batch/jobs                             - Lista todos los Jobs registrados
 */
@Slf4j
@RestController
@RequestMapping("/api/batch/jobs")
@RequiredArgsConstructor
public class BatchJobController {

    private final BatchJobService batchJobService;

    /**
     * Lanza el Job principal de procesamiento de empleados.
     * Responde inmediatamente con el estado inicial de la ejecución.
     */
    @PostMapping("/empleados/ejecutar")
    public ResponseEntity<JobStatusDto> lanzarJobPrincipal() {
        log.info("Solicitud REST recibida: lanzar procesarEmpleadosJob");
        try {
            JobStatusDto status = batchJobService.lanzarJobPrincipal();
            return ResponseEntity.accepted().body(status);
        } catch (RuntimeException e) {
            log.error("Error al lanzar job: {}", e.getMessage());
            return ResponseEntity.internalServerError().build();
        }
    }

    /**
     * Lanza el Job condicional de procesamiento de empleados.
     */
    @PostMapping("/empleados-condicional/ejecutar")
    public ResponseEntity<JobStatusDto> lanzarJobCondicional() {
        log.info("Solicitud REST recibida: lanzar procesarEmpleadosCondicionalJob");
        try {
            JobStatusDto status = batchJobService.lanzarJobCondicional();
            return ResponseEntity.accepted().body(status);
        } catch (RuntimeException e) {
            log.error("Error al lanzar job condicional: {}", e.getMessage());
            return ResponseEntity.internalServerError().build();
        }
    }

    /**
     * Consulta el estado actual de una ejecución específica.
     * Útil para polling del estado después de lanzar un Job.
     */
    @GetMapping("/ejecuciones/{jobExecutionId}")
    public ResponseEntity<JobStatusDto> consultarEjecucion(@PathVariable Long jobExecutionId) {
        log.info("Consultando estado de ejecución ID: {}", jobExecutionId);
        return batchJobService.consultarEjecucion(jobExecutionId)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    /**
     * Lista el historial de ejecuciones de un Job específico.
     * Muestra las últimas 50 ejecuciones ordenadas por ID descendente.
     */
    @GetMapping("/{jobName}/historial")
    public ResponseEntity<List<JobStatusDto>> listarHistorial(@PathVariable String jobName) {
        log.info("Consultando historial del job: {}", jobName);
        List<JobStatusDto> historial = batchJobService.listarEjecuciones(jobName);
        return ResponseEntity.ok(historial);
    }

    /**
     * Lista todos los nombres de Jobs registrados en el JobRepository.
     */
    @GetMapping
    public ResponseEntity<Map<String, Object>> listarJobs() {
        List<String> nombres = batchJobService.listarNombresJobs();
        return ResponseEntity.ok(Map.of(
            "jobs", nombres,
            "total", nombres.size(),
            "descripcion", "Jobs registrados en el JobRepository de Spring Batch"
        ));
    }
}
```

3. Configura CORS para desarrollo local. Crea `src/main/java/com/empresa/batch/config/WebConfig.java`:

```java
package com.empresa.batch.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * Configuración de CORS para desarrollo local.
 * 
 * ⚠️ ADVERTENCIA: Esta configuración es SOLO para desarrollo.
 * En producción, reemplazar allowedOrigins("*") por los dominios
 * específicos de la aplicación frontend.
 */
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("*")  // Solo para desarrollo local
            .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
            .allowedHeaders("*")
            .maxAge(3600);
    }
}
```

**Salida Esperada:**

```bash
mvn compile
# [INFO] BUILD SUCCESS
# [INFO] 8 source files compiled
```

**Verificación:**

- `BatchJobService` y `BatchJobController` compilan sin errores
- El controlador tiene los 5 endpoints mapeados correctamente
- No hay imports sin resolver en ninguna clase

---

### Paso 9: Ejecutar la Aplicación y Probar los Endpoints

**Objetivo:** Iniciar la aplicación Spring Boot, verificar que Spring Batch inicializa su schema en PostgreSQL y probar todos los endpoints REST con Postman.

**Instrucciones:**

1. Ejecuta la aplicación desde IntelliJ o desde la terminal:

```bash
# Desde la terminal en la raíz del proyecto
mvn spring-boot:run

# Alternativa: compilar y ejecutar el JAR
mvn clean package -DskipTests
java -jar target/batch-processor-0.0.1-SNAPSHOT.jar
```

2. Verifica en los logs de inicio que Spring Batch inicializó correctamente su schema:

```
# Debes ver estas líneas en los logs de arranque:
INFO  o.s.b.c.r.s.JobRepositoryFactoryBean - No database type set, using meta data indicating: POSTGRES
INFO  o.s.b.c.l.support.SimpleJobLauncher - No TaskExecutor has been set, defaulting to synchronous executor.
INFO  com.empresa.batch.BatchProcessorApplication - Started BatchProcessorApplication in X.XXX seconds
```

3. Verifica en PostgreSQL que las tablas de metadata de Spring Batch fueron creadas:

```bash
psql -h localhost -U batch_user -d empresa_batch_db -c "\dt"
```

Debes ver tablas como:
```
 Schema |             Name              | Type  |   Owner    
--------+-------------------------------+-------+------------
 public | batch_job_execution           | table | batch_user
 public | batch_job_execution_context   | table | batch_user
 public | batch_job_execution_params    | table | batch_user
 public | batch_job_execution_seq       | table | batch_user
 public | batch_job_instance            | table | batch_user
 public | batch_job_seq                 | table | batch_user
 public | batch_step_execution          | table | batch_user
 public | batch_step_execution_context  | table | batch_user
 public | batch_step_execution_seq      | table | batch_user
 public | batch_reportes                | table | batch_user
 public | empleados                     | table | batch_user
 public | empleados_procesados          | table | batch_user
```

4. Abre Postman y crea una nueva colección llamada **"Spring Batch - Proyecto Integrador"**.

5. **Prueba 1:** Lista los Jobs registrados:

```
GET http://localhost:8080/api/batch/jobs
```

Respuesta esperada:
```json
{
    "jobs": [
        "procesarEmpleadosJob",
        "procesarEmpleadosCondicionalJob"
    ],
    "total": 2,
    "descripcion": "Jobs registrados en el JobRepository de Spring Batch"
}
```

6. **Prueba 2:** Lanza el Job principal:

```
POST http://localhost:8080/api/batch/jobs/empleados/ejecutar
Content-Type: application/json
```

Respuesta esperada (HTTP 202 Accepted):
```json
{
    "jobExecutionId": 1,
    "jobName": "procesarEmpleadosJob",
    "status": "COMPLETED",
    "exitStatus": "COMPLETED",
    "startTime": "2024-05-15T10:30:00",
    "endTime": "2024-05-15T10:30:02",
    "executionContext": {
        "totalLeidos": 20,
        "totalProcesados": 18,
        "totalEscritos": 18,
        "totalSaltados": 2,
        "archivoEntrada": "data/empleados.csv",
        "timestampInicio": "2024-05-15T10:30:00"
    },
    "failureException": null
}
```

7. **Prueba 3:** Consulta el estado de la ejecución (usa el `jobExecutionId` de la respuesta anterior):

```
GET http://localhost:8080/api/batch/jobs/ejecuciones/1
```

8. **Prueba 4:** Lista el historial del Job principal:

```
GET http://localhost:8080/api/batch/jobs/procesarEmpleadosJob/historial
```

9. **Prueba 5:** Lanza el Job condicional:

```
POST http://localhost:8080/api/batch/jobs/empleados-condicional/ejecutar
Content-Type: application/json
```

10. Verifica en PostgreSQL que los datos fueron insertados correctamente:

```bash
psql -h localhost -U batch_user -d empresa_batch_db -c \
  "SELECT nombre, apellido, departamento, salario FROM empleados ORDER BY departamento, apellido LIMIT 10;"
```

```bash
# También verifica el reporte generado
psql -h localhost -U batch_user -d empresa_batch_db -c \
  "SELECT job_name, total_leidos, total_escritos, total_saltados, estado FROM batch_reportes;"
```

**Salida Esperada en los Logs:**

```
INFO  c.e.b.listener.EmpleadosJobListener - ╔══════════════════════════════════════════════════════╗
INFO  c.e.b.listener.EmpleadosJobListener - ║     INICIANDO JOB: procesarEmpleadosJob
INFO  c.e.b.tasklet.ValidacionInicialTasklet - === STEP 1: Iniciando validación del entorno batch ===
INFO  c.e.b.tasklet.ValidacionInicialTasklet - Archivo encontrado. Tamaño: XXXX bytes
INFO  c.e.b.tasklet.ValidacionInicialTasklet - === STEP 1: Validación completada exitosamente ===
INFO  c.e.b.listener.EmpleadosStepListener - --- Iniciando Step: stepProcesarEmpleados ---
WARN  c.e.b.processor.EmpleadoItemProcessor - Formato de salario inválido 'INVALIDO' para empleado: Fernando Jiménez
WARN  c.e.b.processor.EmpleadoItemProcessor - Formato de salario inválido 'DATO_CORRUPTO' para empleado: Sofía Herrera
INFO  c.e.b.listener.EmpleadosStepListener - --- Finalizando Step: stepProcesarEmpleados ---
INFO  c.e.b.listener.EmpleadosStepListener -     Leídos:     20
INFO  c.e.b.listener.EmpleadosStepListener -     Procesados: 18
INFO  c.e.b.listener.EmpleadosStepListener -     Saltados:   2
INFO  c.e.b.tasklet.GeneracionReporteTasklet - === STEP 3: Generando reporte de resumen ===
INFO  c.e.b.tasklet.GeneracionReporteTasklet - Registros leídos:      20
INFO  c.e.b.tasklet.GeneracionReporteTasklet - Registros escritos:    18
INFO  c.e.b.tasklet.GeneracionReporteTasklet - Registros saltados:    2
```

**Verificación:**

- La aplicación arranca sin errores en el puerto 8080
- Las 9 tablas de Spring Batch metadata existen en PostgreSQL
- El Job principal completa con estado `COMPLETED`
- Se insertan 18 registros en la tabla `empleados` (20 del CSV menos 2 con datos inválidos)
- Se registra 1 fila en `batch_reportes` con los totales correctos

---

### Paso 10: Escribir Pruebas Unitarias para el Processor y los Tasklets

**Objetivo:** Implementar pruebas unitarias obligatorias para los componentes clave del módulo batch: el `EmpleadoItemProcessor` y el `ValidacionInicialTasklet`.

**Instrucciones:**

1. Crea el archivo de pruebas para el Processor en `src/test/java/com/empresa/batch/processor/EmpleadoItemProcessorTest.java`:

```java
package com.empresa.batch.processor;

import com.empresa.batch.domain.Empleado;
import com.empresa.batch.domain.EmpleadoCsvDto;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

import java.math.BigDecimal;

import static org.assertj.core.api.Assertions.*;

/**
 * Pruebas unitarias para EmpleadoItemProcessor.
 * Verifica las reglas de negocio de transformación y las validaciones.
 */
@DisplayName("EmpleadoItemProcessor - Pruebas Unitarias")
class EmpleadoItemProcessorTest {

    private EmpleadoItemProcessor processor;

    @BeforeEach
    void setUp() {
        processor = new EmpleadoItemProcessor();
    }

    // =========================================================
    // PRUEBAS DE REGLAS DE NEGOCIO - AUMENTO DE SALARIO
    // =========================================================

    @Test
    @DisplayName("Debe aplicar aumento del 8% para departamento Tecnología")
    void debeAplicarAumentoDel8PorCientoParaTecnologia() throws Exception {
        EmpleadoCsvDto dto = crearDtoValido("Tecnología", "50000.00");
        
        Empleado resultado = processor.process(dto);
        
        assertThat(resultado).isNotNull();
        // 50000 * 1.08 = 54000.00
        assertThat(resultado.getSalario()).isEqualByComparingTo(new BigDecimal("54000.00"));
    }

    @Test
    @DisplayName("Debe aplicar aumento del 5% para departamento Finanzas")
    void debeAplicarAumentoDel5PorCientoParaFinanzas() throws Exception {
        EmpleadoCsvDto dto = crearDtoValido("Finanzas", "60000.00");
        
        Empleado resultado = processor.process(dto);
        
        assertThat(resultado).isNotNull();
        // 60000 * 1.05 = 63000.00
        assertThat(resultado.getSalario()).isEqualByComparingTo(new BigDecimal("63000.00"));
    }

    @Test
    @DisplayName("Debe aplicar aumento del 3% para departamento Marketing")
    void debeAplicarAumentoDel3PorCientoParaOtrosDepartamentos() throws Exception {
        EmpleadoCsvDto dto = crearDtoValido("Marketing", "45000.00");
        
        Empleado resultado = processor.process(dto);
        
        assertThat(resultado).isNotNull();
        // 45000 * 1.03 = 46350.00
        assertThat(resultado.getSalario()).isEqualByComparingTo(new BigDecimal("46350.00"));
    }

    @Test
    @DisplayName("Debe normalizar el email a minúsculas")
    void debeNormalizarEmailAMinusculas() throws Exception {
        EmpleadoCsvDto dto = crearDtoValido("Tecnología", "50000.00");
        dto.setEmail("JUAN.GARCIA@EMPRESA.COM");
        
        Empleado resultado = processor.process(dto);
        
        assertThat(resultado.getEmail()).isEqualTo("juan.garcia@empresa.com");
    }

    @Test
    @DisplayName("Debe mapear correctamente todos los campos del DTO a la entidad")
    void debeMapearCorrectamenteTodosLosCampos() throws Exception {
        EmpleadoCsvDto dto = crearDtoValido("Recursos Humanos", "48000.00");
        
        Empleado resultado = processor.process(dto);
        
        assertThat(resultado.getNombre()).isEqualTo("María");
        assertThat(resultado.getApellido()).isEqualTo("López");
        assertThat(resultado.getDepartamento()).isEqualTo("Recursos Humanos");
        assertThat(resultado.getActivo()).isTrue();
        assertThat(resultado.getFechaContratacion()).isNotNull();
    }

    // =========================================================
    // PRUEBAS DE VALIDACIÓN - CASOS INVÁLIDOS
    // =========================================================

    @ParameterizedTest
    @ValueSource(strings = {"INVALIDO", "DATO_CORRUPTO", "abc", "", "null"})
    @DisplayName("Debe lanzar IllegalArgumentException para salarios con formato inválido")
    void debeLanzarExcepcionParaSalariosInvalidos(String salarioInvalido) {
        EmpleadoCsvDto dto = crearDtoValido("Tecnología", salarioInvalido);
        
        assertThatThrownBy(() -> processor.process(dto))
            .isInstanceOf(IllegalArgumentException.class)
            .satisfies(ex -> assertThat(ex.getMessage()).contains("salario"));
    }

    @Test
    @DisplayName("Debe lanzar IllegalArgumentException para salario negativo")
    void debeLanzarExcepcionParaSalarioNegativo() {
        EmpleadoCsvDto dto = crearDtoValido("Tecnología", "-1000.00");
        
        assertThatThrownBy(() -> processor.process(dto))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessageContaining("positivo");
    }

    @Test
    @DisplayName("Debe lanzar IllegalArgumentException para fecha con formato inválido")
    void debeLanzarExcepcionParaFechaInvalida() {
        EmpleadoCsvDto dto = crearDtoValido("Tecnología", "50000.00");
        dto.setFechaContratacion("15/03/2020"); // Formato incorrecto, debe ser yyyy-MM-dd
        
        assertThatThrownBy(() -> processor.process(dto))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessageContaining("fecha");
    }

    @Test
    @DisplayName("Debe lanzar IllegalArgumentException cuando el email está vacío")
    void debeLanzarExcepcionParaEmailVacio() {
        EmpleadoCsvDto dto = crearDtoValido("Tecnología", "50000.00");
        dto.setEmail("");
        
        assertThatThrownBy(() -> processor.process(dto))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessageContaining("Email");
    }

    // =========================================================
    // MÉTODO AUXILIAR
    // =========================================================

    private EmpleadoCsvDto crearDtoValido(String departamento, String salario) {
        return EmpleadoCsvDto.builder()
            .nombre("María")
            .apellido("López")
            .email("maria.lopez@empresa.com")
            .departamento(departamento)
            .salario(salario)
            .fechaContratacion("2020-03-15")
            .activo("true")
            .build();
    }
}
```

2. Crea las pruebas para el Tasklet de validación en `src/test/java/com/empresa/batch/tasklet/ValidacionInicialTaskletTest.java`:

```java
package com.empresa.batch.tasklet;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.batch.core.*;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.scope.context.StepContext;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.test.util.ReflectionTestUtils;

import java.util.Date;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.Mockito.*;

/**
 * Pruebas unitarias para ValidacionInicialTasklet.
 * Verifica que el Tasklet valida correctamente la existencia del archivo
 * y que maneja apropiadamente los casos de error.
 */
@ExtendWith(MockitoExtension.class)
@DisplayName("ValidacionInicialTasklet - Pruebas Unitarias")
class ValidacionInicialTaskletTest {

    private ValidacionInicialTasklet tasklet;

    @Mock
    private StepContribution stepContribution;

    @Mock
    private ChunkContext chunkContext;

    @Mock
    private StepContext stepContext;

    @Mock
    private StepExecution stepExecution;

    @Mock
    private JobExecution jobExecution;

    @BeforeEach
    void setUp() {
        tasklet = new ValidacionInicialTasklet();
        
        // Configurar la cadena de mocks para el ExecutionContext
        when(chunkContext.getStepContext()).thenReturn(stepContext);
        when(stepContext.getStepExecution()).thenReturn(stepExecution);
        when(stepExecution.getJobExecution()).thenReturn(jobExecution);
        when(jobExecution.getExecutionContext()).thenReturn(new ExecutionContext());
    }

    @Test
    @DisplayName("Debe retornar FINISHED cuando el archivo CSV existe")
    void debeRetornarFinishedCuandoArchivoExiste() throws Exception {
        // Configurar el archivo que SÍ existe en el classpath de pruebas
        ReflectionTestUtils.setField(tasklet, "archivoEntrada", "data/empleados.csv");
        
        RepeatStatus resultado = tasklet.execute(stepContribution, chunkContext);
        
        assertThat(resultado).isEqualTo(RepeatStatus.FINISHED);
    }

    @Test
    @DisplayName("Debe lanzar IllegalStateException cuando el archivo NO existe")
    void debeLanzarExcepcionCuandoArchivoNoExiste() {
        // Configurar un archivo que NO existe
        ReflectionTestUtils.setField(tasklet, "archivoEntrada", "data/archivo-inexistente.csv");
        
        assertThatThrownBy(() -> tasklet.execute(stepContribution, chunkContext))
            .isInstanceOf(IllegalStateException.class)
            .hasMessageContaining("ARCHIVO NO ENCONTRADO");
    }

    @Test
    @DisplayName("Debe guardar metadatos en el ExecutionContext del Job")
    void debeGuardarMetadatosEnExecutionContext() throws Exception {
        ReflectionTestUtils.setField(tasklet, "archivoEntrada", "data/empleados.csv");
        
        ExecutionContext executionContext = new ExecutionContext();
        when(jobExecution.getExecutionContext()).thenReturn(executionContext);
        
        tasklet.execute(stepContribution, chunkContext);
        
        // Verificar que se guardaron los metadatos esperados
        assertThat(executionContext.containsKey("archivoEntrada")).isTrue();
        assertThat(executionContext.containsKey("timestampInicio")).isTrue();
        assertThat(executionContext.containsKey("tamanoArchivo")).isTrue();
        assertThat(executionContext.getString("archivoEntrada")).isEqualTo("data/empleados.csv");
    }
}
```

3. Crea el archivo de configuración de pruebas en `src/test/resources/application-test.properties`:

```properties
# Configuración para pruebas unitarias
# Usa H2 en memoria para no depender de PostgreSQL en los tests
spring.datasource.url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect

# Spring Batch con H2 para tests
spring.batch.jdbc.initialize-schema=always
spring.batch.job.enabled=false

logging.level.com.empresa.batch=DEBUG
logging.level.org.springframework.batch=WARN
```

4. Ejecuta las pruebas:

```bash
mvn test
```

**Salida Esperada:**

```
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.empresa.batch.processor.EmpleadoItemProcessorTest
[INFO] Tests run: 9, Failures: 0, Errors: 0, Skipped: 0
[INFO] Running com.empresa.batch.tasklet.ValidacionInicialTaskletTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0
[INFO] -------------------------------------------------------
[INFO] Results:
[INFO] Tests run: 12, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS
```

**Verificación:**

- Todas las pruebas pasan (12 tests, 0 failures)
- El reporte de Surefire en `target/surefire-reports/` muestra los resultados detallados
- Las pruebas del Processor cubren los 3 departamentos con diferentes porcentajes de aumento
- Las pruebas del Tasklet verifican tanto el caso exitoso como el caso de error

---

## Validación y Pruebas

### Criterios de Éxito

- [ ] La aplicación Spring Boot arranca sin errores y el puerto 8080 está disponible
- [ ] Las 9 tablas de metadata de Spring Batch existen en PostgreSQL (`batch_job_execution`, `batch_step_execution`, etc.)
- [ ] Las tablas de negocio `empleados`, `empleados_procesados` y `batch_reportes` existen en PostgreSQL
- [ ] El endpoint `GET /api/batch/jobs` retorna los 2 Jobs registrados con HTTP 200
- [ ] El endpoint `POST /api/batch/jobs/empleados/ejecutar` completa el Job con status `COMPLETED`
- [ ] Se insertan exactamente 18 registros en la tabla `empleados` (20 del CSV menos 2 inválidos)
- [ ] El campo `totalSaltados` en la respuesta del Job es exactamente 2
- [ ] Se registra 1 fila en la tabla `batch_reportes` con el resumen correcto
- [ ] El Job condicional (`procesarEmpleadosCondicionalJob`) también completa exitosamente
- [ ] Todas las 12 pruebas unitarias pasan con `mvn test`
- [ ] Los logs muestran la secuencia correcta: Step 1 → Step 2 → Step 3

### Procedimiento de Pruebas

1. **Verificar el estado de la base de datos antes de ejecutar:**
   ```bash
   psql -h localhost -U batch_user -d empresa_batch_db -c "SELECT COUNT(*) FROM empleados;"
   ```
   **Resultado Esperado:** `count = 0` (tabla vacía antes del primer Job)

2. **Lanzar el Job principal y verificar respuesta:**
   ```bash
   curl -X POST http://localhost:8080/api/batch/jobs/empleados/ejecutar \
     -H "Content-Type: application/json"
   ```
   **Resultado Esperado:** HTTP 202 con `"status": "COMPLETED"` y `"totalEscritos": 18`

3. **Verificar datos insertados en PostgreSQL:**
   ```bash
   psql -h localhost -U batch_user -d empresa_batch_db \
     -c "SELECT departamento, COUNT(*), AVG(salario) FROM empleados GROUP BY departamento ORDER BY departamento;"
   ```
   **Resultado Esperado:** 4 filas con los departamentos Finanzas, Marketing, Recursos Humanos y Tecnología

4. **Verificar que los salarios tienen el aumento aplicado (Tecnología debe tener +8%):**
   ```bash
   psql -h localhost -U batch_user -d empresa_batch_db \
     -c "SELECT nombre, apellido, salario FROM empleados WHERE departamento = 'Tecnología' ORDER BY apellido;"
   ```
   **Resultado Esperado:** Los salarios de Tecnología deben ser 8% mayores que los valores originales del CSV

5. **Consultar el reporte generado:**
   ```bash
   psql -h localhost -U batch_user -d empresa_batch_db \
     -c "SELECT * FROM batch_reportes ORDER BY created_at DESC LIMIT 1;"
   ```
   **Resultado Esperado:** `total_leidos=20, total_escritos=18, total_saltados=2, estado='COMPLETADO'`

6. **Verificar historial de ejecuciones:**
   ```bash
   curl http://localhost:8080/api/batch/jobs/procesarEmpleadosJob/historial
   ```
   **Resultado Esperado:** Array JSON con al menos 1 ejecución en estado `COMPLETED`

7. **Ejecutar todas las pruebas unitarias:**
   ```bash
   mvn test -Dspring.profiles.active=test
   ```
   **Resultado Esperado:** `Tests run: 12, Failures: 0, Errors: 0`

---

## Solución de Problemas

### Problema 1: Error de Conexión a PostgreSQL al Iniciar la Aplicación

**Síntomas:**
- La aplicación no arranca y muestra en los logs:
  ```
  com.zaxxer.hikari.pool.HikariPool$PoolInitializationException: 
  Failed to initialize pool: Connection to localhost:5432 refused
  ```
- IntelliJ muestra `APPLICATION FAILED TO START`

**Causa:**
PostgreSQL no está corriendo en el puerto 5432, o las credenciales en `application.properties` no coinciden con las del servidor.

**Solución:**
```bash
# Verificar si PostgreSQL está corriendo (Linux/macOS)
pg_isready -h localhost -p 5432
# Si muestra "no response", iniciar el servicio:
sudo systemctl start postgresql  # Linux
brew services start postgresql@16  # macOS con Homebrew

# Si usas Docker, verificar que el contenedor está activo:
docker ps | grep postgres-batch
# Si no aparece, iniciarlo:
docker start postgres-batch

# Verificar credenciales conectándose manualmente:
psql -h localhost -p 5432 -U batch_user -d empresa_batch_db

# Windows: iniciar servicio PostgreSQL
net start postgresql-x64-16
```

---

### Problema 2: Spring Batch Lanza el Job Automáticamente al Iniciar

**Síntomas:**
- Al arrancar la aplicación, el Job se ejecuta inmediatamente sin haber llamado al endpoint REST
- Los logs muestran el Job iniciando durante el startup de Spring Boot

**Causa:**
La propiedad `spring.batch.job.enabled` no está configurada como `false` en `application.properties`. En Spring Boot 3.x con Spring Batch 5, el comportamiento por defecto puede variar según la versión.

**Solución:**
```properties
# Agregar o verificar en src/main/resources/application.properties:
spring.batch.job.enabled=false
```

```bash
# Reiniciar la aplicación después del cambio:
# Ctrl+C para detener, luego:
mvn spring-boot:run
```

---

### Problema 3: Error `JobInstanceAlreadyCompleteException` al Relanzar el Job

**Síntomas:**
- Al llamar al endpoint `POST /api/batch/jobs/empleados/ejecutar` por segunda vez, la aplicación retorna HTTP 500
- Los logs muestran: `JobInstanceAlreadyCompleteException: A job instance already exists and is complete for identifying parameters`

**Causa:**
Spring Batch identifica las instancias de Job por sus parámetros. Si se usan los mismos parámetros en dos ejecuciones, Spring Batch considera que ya fue completado y rechaza el reintento. El `BatchJobService` ya incluye un timestamp único, pero si el timestamp tiene la misma resolución de segundos, puede colisionar.

**Solución:**
El `BatchJobService` ya usa `LocalDateTime.now()` que incluye nanosegundos. Si el problema persiste, agrega un UUID adicional:

```java
// En BatchJobService.lanzarJobPrincipal(), modificar los parámetros:
JobParameters params = new JobParametersBuilder()
    .addLocalDateTime("timestamp", LocalDateTime.now())
    .addString("origen", "REST_API")
    .addString("runId", java.util.UUID.randomUUID().toString()) // UUID único garantizado
    .toJobParameters();
```

---

### Problema 4: El Skip No Funciona y el Job Falla con `IllegalArgumentException`

**Síntomas:**
- El Job falla completamente en lugar de saltar los 2 registros inválidos del CSV
- Los logs muestran: `Caused by: java.lang.IllegalArgumentException: Formato de salario inválido`
- El estado del Job es `FAILED` en lugar de `COMPLETED`

**Causa:**
La política de skip no está configurada correctamente en el Step, o la excepción que se lanza en el Processor no coincide con la excepción registrada en `.skip()`.

**Solución:**
Verifica que el Step tiene la configuración correcta de `faultTolerant()` antes de `.skip()`:

```java
// En EmpleadosJobConfig.stepProcesarEmpleados(), verificar que está así:
return new StepBuilder("stepProcesarEmpleados", jobRepository)
    .<EmpleadoCsvDto, Empleado>chunk(10, transactionManager)
    .reader(empleadoCsvReader)
    .processor(empleadoItemProcessor)
    .writer(empleadoWriter)
    .faultTolerant()          // ← OBLIGATORIO antes de .skip()
    .skip(IllegalArgumentException.class)
    .skipLimit(5)
    .retry(org.springframework.dao.DataAccessException.class)
    .retryLimit(3)
    .listener(empleadosStepListener)
    .build();
```

También verifica que el Processor lanza `IllegalArgumentException` (no una subclase diferente):
```bash
# Buscar en los logs la línea exacta de la excepción:
grep "IllegalArgumentException" target/logs/application.log
```

---

### Problema 5: Las Pruebas Unitarias Fallan con `NoSuchBeanDefinitionException`

**Síntomas:**
- Al ejecutar `mvn test`, las pruebas del Tasklet fallan con:
  ```
  org.springframework.beans.factory.NoSuchBeanDefinitionException: 
  No qualifying bean of type 'javax.sql.DataSource' available
  ```

**Causa:**
Las pruebas del `ValidacionInicialTasklet` intentan cargar el contexto completo de Spring pero no tienen configurada la base de datos H2 para pruebas. El perfil de test no se está activando correctamente.

**Solución:**
```bash
# Asegúrate de que el archivo application-test.properties existe en src/test/resources/
ls src/test/resources/application-test.properties

# Ejecuta los tests con el perfil correcto:
mvn test -Dspring.profiles.active=test

# Alternativa: agregar la anotación @ActiveProfiles en la clase de test:
```

```java
// Agregar en ValidacionInicialTaskletTest.java si no está presente:
@ExtendWith(MockitoExtension.class)  // Solo Mockito, sin contexto Spring completo
@DisplayName("ValidacionInicialTasklet - Pruebas Unitarias")
class ValidacionInicialTaskletTest {
    // Las pruebas con @Mock no necesitan contexto Spring
}
```

---

## Limpieza

Ejecuta los siguientes comandos para limpiar el entorno después de completar el laboratorio:

```bash
# 1. Detener la aplicación Spring Boot (Ctrl+C en la terminal donde está corriendo)
# O si está en background:
pkill -f "spring-boot:run"

# 2. Limpiar los artefactos de compilación de Maven
cd ~/proyectos/batch-processor
mvn clean

# 3. Limpiar los datos de prueba de PostgreSQL (opcional, mantiene el schema)
psql -h localhost -U batch_user -d empresa_batch_db -c "
  TRUNCATE TABLE empleados CASCADE;
  TRUNCATE TABLE empleados_procesados CASCADE;
  TRUNCATE TABLE batch_reportes CASCADE;
  TRUNCATE TABLE batch_job_execution CASCADE;
  TRUNCATE TABLE batch_job_instance CASCADE;
  TRUNCATE TABLE batch_step_execution CASCADE;
"

# 4. Si usaste Docker para PostgreSQL, detener el contenedor
docker stop postgres-batch

# 5. Para eliminar completamente el contenedor y sus datos (DESTRUCTIVO)
# docker rm postgres-batch
# docker volume prune

# 6. Guardar el trabajo en Git
cd ~/proyectos/batch-processor
git init
git add .
git commit -m "feat: implementación módulo Spring Batch - Lab 04-00-01"
```

> ⚠️ **Advertencia:** El comando `TRUNCATE TABLE` eliminará todos los datos de las tablas. Solo ejecútalo si deseas limpiar completamente el entorno de pruebas. Si quieres conservar los datos para revisión posterior, omite el paso 3.

> ⚠️ **Advertencia:** No elimines el contenedor Docker si planeas continuar con laboratorios posteriores que usen la misma base de datos. Usa `docker stop` en lugar de `docker rm`.

---

## Resumen

### Lo que Lograste

- **Proyecto Spring Batch completo**: Creaste desde cero un módulo de procesamiento por lotes con Spring Boot 3.2 y Spring Batch 5, configurando todas las dependencias necesarias y la estructura de paquetes empresarial
- **Job principal con 3 Steps secuenciales**: Implementaste el patrón completo Tasklet → Chunk-oriented → Tasklet, con flujo de datos entre Steps usando `ExecutionContext`
- **Procesamiento Chunk-oriented real**: Configuraste `FlatFileItemReader` para leer un CSV de 20 registros, `EmpleadoItemProcessor` con reglas de negocio (aumentos de salario por departamento) y `JdbcBatchItemWriter` para persistir en PostgreSQL
- **Tolerancia a fallos en producción**: Aplicaste políticas de skip (máximo 5 registros inválidos) y retry (máximo 3 reintentos para errores de BD) con la configuración `faultTolerant()` de Spring Batch 5
- **Job condicional con Decider**: Implementaste `JobExecutionDecider` para tomar decisiones de flujo basadas en métricas de ejecución (EXITOSO / CON_ERRORES / SIN_DATOS)
- **API REST para operación de Jobs**: Expusiste 5 endpoints REST con Spring MVC para lanzar Jobs, consultar estado y listar historial, usando `JobLauncher` y `JobExplorer`
- **Pruebas unitarias obligatorias**: Escribiste 12 pruebas unitarias para el Processor (reglas de negocio + validaciones) y el Tasklet (existencia de archivo + persistencia en ExecutionContext)

### Conceptos Clave Aprendidos

- **Arquitectura Job → Step → Reader/Processor/Writer**: La jerarquía fundamental de Spring Batch donde cada nivel tiene una responsabilidad bien definida
- **Chunk-oriented processing**: El patrón de procesamiento en bloques que optimiza el rendimiento y la gestión de transacciones al procesar N registros por transacción
- **ExecutionContext**: El mecanismo de Spring Batch para compartir datos entre Steps y persistir estado para soporte de restart
- **faultTolerant() + skip() + retry()**: La tríada de configuración para construir Jobs resilientes que se recuperan de fallos parciales sin reiniciar desde cero
- **JobRepository**: La base de datos interna de Spring Batch que registra cada ejecución y hace posible el monitoreo y el restart
- **JobExecutionDecider**: El mecanismo para implementar flujos condicionales basados en el resultado del procesamiento

### Próximos Pasos

- **Lección 4.2 — Conceptos de Jobs y Steps**: Profundizar en la configuración avanzada de Jobs: flujos paralelos, Steps particionados y configuración de JobParameters para ejecuciones parametrizadas
- **Lección 4.3 — ItemReaders y ItemWriters**: Explorar los readers y writers predefinidos de Spring Batch para JPA, MongoDB, JSON y servicios REST
- **Integración con el proyecto completo**: Conectar este módulo batch con el frontend desarrollado en los laboratorios anteriores para visualizar el estado de ejecución en tiempo real
- **Monitoreo con Spring Boot Admin**: Configurar Spring Boot Admin para visualizar las ejecuciones de Jobs en un dashboard web

---

## Recursos Adicionales

- **Documentación oficial de Spring Batch 5**: Referencia completa con todos los componentes, configuraciones y patrones — [https://docs.spring.io/spring-batch/docs/current/reference/html/](https://docs.spring.io/spring-batch/docs/current/reference/html/)
- **Spring Batch GitHub — Ejemplos oficiales**: Repositorio con decenas de ejemplos reales de Jobs batch para diferentes casos de uso — [https://github.com/spring-projects/spring-batch/tree/main/spring-batch-samples](https://github.com/spring-projects/spring-batch/tree/main/spring-batch-samples)
- **Baeldung — Spring Batch Tutorial**: Serie completa de artículos con ejemplos prácticos paso a paso, ideal para reforzar los conceptos del laboratorio — [https://www.baeldung.com/introduction-to-spring-batch](https://www.baeldung.com/introduction-to-spring-batch)
- **The Definitive Guide to Spring Batch (Minella, 2019)**: Libro de referencia técnica escrito por uno de los principales contribuidores del framework, disponible en Apress
- **Spring Initializr**: Generador oficial de proyectos Spring Boot para crear nuevos proyectos con las dependencias correctas — [https://start.spring.io](https://start.spring.io)
