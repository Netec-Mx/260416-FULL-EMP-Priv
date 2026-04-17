# Laboratorio 3. Proyecto Integrador — Backend Empresarial con Spring Boot y Maven

## Metadatos

| Propiedad | Valor |
|-----------|-------|
| **Duración** | 170 minutos |
| **Complejidad** | Difícil |
| **Nivel Bloom** | Crear |
| **Tecnologías** | Java 17, Spring Boot 3.2, Maven, Lombok, JUnit 5, Mockito, MockMvc, H2, Jackson |

---

## Descripción General

En este laboratorio construirás el backend empresarial del proyecto integrador del curso: un sistema de gestión de inventario de productos. Partiendo desde cero con Spring Initializr, crearás un proyecto Spring Boot 3.2 correctamente estructurado con Maven, implementarás una API REST completa con arquitectura en capas (Controller → Service → Repository → Model), aplicarás los patrones de diseño Facade y DTO para desacoplar la lógica de negocio de la capa de presentación, y escribirás pruebas unitarias con JUnit 5 y Mockito que validen el comportamiento de tus componentes.

Este laboratorio representa el núcleo técnico del curso: todo lo que construyas aquí será la base sobre la que los laboratorios posteriores agregarán persistencia real con PostgreSQL y MongoDB, procesamiento por lotes con Spring Batch, y el frontend con Web Components y Lit. Es fundamental que lo completes con rigor, especialmente las pruebas unitarias, que son un entregable obligatorio.

---

## Objetivos de Aprendizaje

Al completar este laboratorio, serás capaz de:

- [ ] Generar y configurar un proyecto Spring Boot 3.2 con Maven usando Spring Initializr, incluyendo dependencias correctas y estructura de paquetes por capas
- [ ] Implementar una API REST completa con operaciones CRUD usando los verbos HTTP correctos (GET, POST, PUT, DELETE), códigos de estado apropiados y serialización JSON con Jackson
- [ ] Aplicar el patrón DTO con conversión manual para separar el modelo de dominio de la API pública, controlando qué información se expone
- [ ] Implementar el patrón Facade para encapsular y simplificar la lógica de negocio compleja desde la perspectiva del controlador
- [ ] Configurar manejo global de errores con `@ControllerAdvice` y respuestas de error estandarizadas
- [ ] Escribir pruebas unitarias con JUnit 5 y Mockito para la capa de servicio, y pruebas de controlador con MockMvc

---

## Prerrequisitos

### Conocimientos Requeridos

- Programación orientada a objetos en Java (clases, herencia, interfaces, anotaciones)
- Conceptos básicos de HTTP: métodos (GET, POST, PUT, DELETE), códigos de estado (200, 201, 404, 400, 500) y headers
- Comprensión de la estructura Maven estudiada en la Lección 3.1 (pom.xml, estructura de directorios, ciclo de vida)
- Laboratorio 1 completado: Java instalado, Git configurado, cuenta GitHub activa
- Familiaridad básica con JSON como formato de intercambio de datos

### Acceso Requerido

- JDK 17 instalado y variable `JAVA_HOME` configurada
- Maven 3.9.x instalado o uso del Maven Wrapper (`mvnw`) que se genera con el proyecto
- IntelliJ IDEA 2024.1 con plugin Lombok habilitado
- Postman 11.x, Thunder Client (extensión VS Code) o `curl` disponible para pruebas de API
- Conexión a internet para descarga de dependencias Maven (primera vez)
- Cuenta GitHub para repositorio del proyecto integrador

---

## Entorno de Laboratorio

### Requisitos de Hardware

| Componente | Especificación |
|------------|----------------|
| Procesador | Intel Core i5 8va gen o AMD Ryzen 5 (o superior) |
| Memoria RAM | Mínimo 8 GB (recomendado 16 GB con IntelliJ abierto) |
| Almacenamiento | Mínimo 5 GB libres para dependencias Maven y proyecto |
| Pantalla | Resolución 1920x1080 recomendada para IntelliJ + terminal |

### Requisitos de Software

| Software | Versión | Propósito |
|----------|---------|-----------|
| Java Development Kit | 17 LTS | Compilación y ejecución del proyecto Spring Boot |
| Apache Maven | 3.9.x | Gestión de dependencias y ciclo de vida del proyecto |
| IntelliJ IDEA | 2024.1 Community o Ultimate | IDE principal para desarrollo Java |
| Spring Boot | 3.2.x | Framework principal del backend |
| Postman / Thunder Client | 11.x / cualquier versión | Pruebas manuales de la API REST |
| Git | 2.44 o superior | Control de versiones del proyecto integrador |

### Configuración Inicial

Verifica que tu entorno esté correctamente configurado antes de comenzar:

```bash
# Verificar Java 17
java -version
# Esperado: openjdk version "17.x.x" o similar

# Verificar Maven
mvn -version
# Esperado: Apache Maven 3.9.x

# Verificar Git
git --version
# Esperado: git version 2.44.x o superior
```

Si Maven no está instalado globalmente, no te preocupes: el proyecto generado con Spring Initializr incluye el Maven Wrapper (`mvnw` / `mvnw.cmd`) que funciona sin instalación global.

---

## Instrucciones Paso a Paso

### Paso 1: Generar el Proyecto con Spring Initializr y Configurar el Repositorio Git

**Objetivo:** Crear el proyecto Spring Boot base con todas las dependencias necesarias, verificar la estructura de directorios Maven y conectarlo a un repositorio GitHub.

**Instrucciones:**

1. Abre tu navegador y navega a [https://start.spring.io](https://start.spring.io)

2. Configura el proyecto con los siguientes valores:
   - **Project:** Maven
   - **Language:** Java
   - **Spring Boot:** 3.2.x (selecciona la versión estable más reciente de la serie 3.2)
   - **Group:** `com.empresa.inventario`
   - **Artifact:** `inventario-api`
   - **Name:** `inventario-api`
   - **Description:** `Sistema de gestión de inventario - Proyecto Integrador`
   - **Package name:** `com.empresa.inventario`
   - **Packaging:** Jar
   - **Java:** 17

3. Agrega las siguientes dependencias haciendo clic en "ADD DEPENDENCIES":
   - **Spring Web** (Spring MVC, servidor Tomcat embebido)
   - **Spring Data JPA** (acceso a datos con Hibernate)
   - **H2 Database** (base de datos en memoria para desarrollo y pruebas)
   - **Lombok** (reducción de código boilerplate)
   - **Validation** (Bean Validation con Hibernate Validator)

4. Haz clic en **"GENERATE"** para descargar el archivo ZIP.

5. Descomprime el archivo en tu directorio de proyectos. Ejemplo:

   ```bash
   # En Linux/macOS
   cd ~/proyectos
   unzip inventario-api.zip
   cd inventario-api

   # En Windows (PowerShell)
   cd C:\proyectos
   Expand-Archive -Path inventario-api.zip -DestinationPath .
   cd inventario-api
   ```

6. Verifica la estructura de directorios generada:

   ```bash
   # En Linux/macOS
   find . -type f -name "*.java" -o -name "pom.xml" | head -20

   # En Windows (PowerShell)
   Get-ChildItem -Recurse -Include "*.java","pom.xml" | Select-Object FullName | Select-Object -First 20
   ```

7. Inicializa el repositorio Git y haz el primer commit:

   ```bash
   git init
   git add .
   git commit -m "feat: inicializar proyecto Spring Boot con Spring Initializr"
   ```

8. Crea un repositorio en GitHub llamado `inventario-api` (sin README, sin .gitignore) y conecta el repositorio local:

   ```bash
   git remote add origin https://github.com/TU_USUARIO/inventario-api.git
   git branch -M main
   git push -u origin main
   ```

9. Abre el proyecto en IntelliJ IDEA:
   - **File → Open** → selecciona la carpeta `inventario-api`
   - IntelliJ detectará automáticamente el proyecto Maven
   - Espera a que finalice la indexación y descarga de dependencias (puede tomar 2-5 minutos la primera vez)
   - Verifica que el plugin Lombok esté activo: **File → Settings → Plugins** → busca "Lombok" → debe aparecer como instalado y habilitado

**Salida Esperada:**

```
[INFO] BUILD SUCCESS
[INFO] Total time: 3.456 s
```

Al ejecutar `mvn validate` en la terminal del proyecto, Maven debe validar el `pom.xml` sin errores.

**Verificación:**

- [ ] El directorio `src/main/java/com/empresa/inventario/` existe con la clase `InventarioApiApplication.java`
- [ ] El directorio `src/test/java/com/empresa/inventario/` existe con la clase `InventarioApiApplicationTests.java`
- [ ] El archivo `pom.xml` contiene las dependencias: `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `h2`, `lombok`, `spring-boot-starter-validation`
- [ ] El repositorio GitHub tiene el primer commit visible

---

### Paso 2: Revisar y Enriquecer el pom.xml

**Objetivo:** Comprender la estructura del `pom.xml` generado, agregar configuraciones importantes y familiarizarse con el POM padre de Spring Boot.

**Instrucciones:**

1. Abre el archivo `pom.xml` en IntelliJ IDEA. Examina su estructura y localiza los siguientes elementos:
   - El bloque `<parent>` que hereda de `spring-boot-starter-parent`
   - El bloque `<properties>` con la versión de Java
   - El bloque `<dependencies>` con las dependencias seleccionadas
   - El bloque `<build>` con el plugin `spring-boot-maven-plugin`

2. Reemplaza el contenido completo del `pom.xml` con la versión enriquecida siguiente. Presta atención a los comentarios explicativos:

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                                https://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>

       <!-- POM Padre de Spring Boot: gestiona versiones de todas las dependencias -->
       <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>3.2.5</version>
           <relativePath/>
       </parent>

       <!-- Coordenadas GAV del proyecto -->
       <groupId>com.empresa.inventario</groupId>
       <artifactId>inventario-api</artifactId>
       <version>1.0.0-SNAPSHOT</version>
       <packaging>jar</packaging>

       <name>inventario-api</name>
       <description>Sistema de gestión de inventario - Proyecto Integrador del Curso</description>

       <!-- Propiedades centralizadas -->
       <properties>
           <java.version>17</java.version>
           <maven.compiler.source>17</maven.compiler.source>
           <maven.compiler.target>17</maven.compiler.target>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
           <!-- Versión del proyecto para referencia interna -->
           <app.version>1.0.0</app.version>
       </properties>

       <dependencies>
           <!-- ===== DEPENDENCIAS DE PRODUCCIÓN ===== -->

           <!-- Spring Web: REST controllers, Tomcat embebido, Jackson -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>

           <!-- Spring Data JPA: acceso a datos con Hibernate -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-jpa</artifactId>
           </dependency>

           <!-- Bean Validation: @NotNull, @Size, @Min, etc. -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-validation</artifactId>
           </dependency>

           <!-- Lombok: reduce código boilerplate (@Getter, @Builder, @Data) -->
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <optional>true</optional>
           </dependency>

           <!-- ===== DEPENDENCIAS DE BASE DE DATOS ===== -->

           <!-- H2: base de datos en memoria para desarrollo y pruebas -->
           <dependency>
               <groupId>com.h2database</groupId>
               <artifactId>h2</artifactId>
               <scope>runtime</scope>
           </dependency>

           <!-- ===== DEPENDENCIAS DE PRUEBAS ===== -->

           <!-- Spring Boot Test: incluye JUnit 5, Mockito, MockMvc, AssertJ -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <scope>test</scope>
           </dependency>
       </dependencies>

       <build>
           <plugins>
               <!-- Plugin de Spring Boot: genera JAR ejecutable con Tomcat embebido -->
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
                   <configuration>
                       <!-- Excluir Lombok del JAR final (solo necesario en compilación) -->
                       <excludes>
                           <exclude>
                               <groupId>org.projectlombok</groupId>
                               <artifactId>lombok</artifactId>
                           </exclude>
                       </excludes>
                   </configuration>
               </plugin>

               <!-- Plugin de Surefire: ejecución de pruebas unitarias -->
               <plugin>
                   <groupId>org.apache.maven.plugins</groupId>
                   <artifactId>maven-surefire-plugin</artifactId>
                   <version>3.1.2</version>
               </plugin>
           </plugins>
       </build>

   </project>
   ```

3. Guarda el archivo. IntelliJ mostrará una notificación para recargar el proyecto Maven. Haz clic en **"Load Maven Changes"** (o el ícono de recarga que aparece en la esquina superior derecha del editor).

4. Verifica que el proyecto compile correctamente:

   ```bash
   # En Linux/macOS
   ./mvnw clean compile

   # En Windows (PowerShell)
   .\mvnw.cmd clean compile
   ```

5. Haz commit del `pom.xml` actualizado:

   ```bash
   git add pom.xml
   git commit -m "build: enriquecer pom.xml con configuracion de plugins y comentarios"
   ```

**Salida Esperada:**

```
[INFO] --- maven-compiler-plugin:3.11.0:compile (default-compile) @ inventario-api ---
[INFO] Changes detected - recompiling the module! :source
[INFO] Compiling 1 source file with javac [debug release 17] to target/classes
[INFO] BUILD SUCCESS
```

**Verificación:**

- [ ] `mvn clean compile` termina con `BUILD SUCCESS`
- [ ] El directorio `target/classes/` fue creado con el archivo `.class` de la clase principal
- [ ] No hay errores de compilación en IntelliJ (sin líneas rojas en el código)

---

### Paso 3: Crear la Estructura de Paquetes y el Modelo de Dominio

**Objetivo:** Establecer la arquitectura en capas del proyecto creando los paquetes necesarios e implementando la entidad `Producto` con Lombok y JPA.

**Instrucciones:**

1. Crea la siguiente estructura de paquetes dentro de `src/main/java/com/empresa/inventario/`. En IntelliJ, haz clic derecho sobre el paquete base → **New → Package**:

   ```
   com.empresa.inventario/
   ├── controller/        ← Capa de presentación (endpoints REST)
   ├── service/           ← Capa de negocio (lógica de aplicación)
   │   └── impl/          ← Implementaciones de los servicios
   ├── repository/        ← Capa de acceso a datos (Spring Data JPA)
   ├── model/             ← Entidades JPA (modelo de dominio)
   ├── dto/               ← Data Transfer Objects (objetos de transferencia)
   │   ├── request/       ← DTOs para peticiones entrantes
   │   └── response/      ← DTOs para respuestas salientes
   ├── facade/            ← Patrón Facade (orquestación de servicios)
   ├── exception/         ← Excepciones personalizadas y manejador global
   └── config/            ← Configuraciones de Spring (CORS, Jackson, etc.)
   ```

   Puedes crear todos los paquetes ejecutando el siguiente script alternativo desde la terminal del proyecto:

   ```bash
   # En Linux/macOS
   BASE="src/main/java/com/empresa/inventario"
   mkdir -p $BASE/controller
   mkdir -p $BASE/service/impl
   mkdir -p $BASE/repository
   mkdir -p $BASE/model
   mkdir -p $BASE/dto/request
   mkdir -p $BASE/dto/response
   mkdir -p $BASE/facade
   mkdir -p $BASE/exception
   mkdir -p $BASE/config

   # En Windows (PowerShell)
   $BASE = "src\main\java\com\empresa\inventario"
   New-Item -ItemType Directory -Force -Path "$BASE\controller"
   New-Item -ItemType Directory -Force -Path "$BASE\service\impl"
   New-Item -ItemType Directory -Force -Path "$BASE\repository"
   New-Item -ItemType Directory -Force -Path "$BASE\model"
   New-Item -ItemType Directory -Force -Path "$BASE\dto\request"
   New-Item -ItemType Directory -Force -Path "$BASE\dto\response"
   New-Item -ItemType Directory -Force -Path "$BASE\facade"
   New-Item -ItemType Directory -Force -Path "$BASE\exception"
   New-Item -ItemType Directory -Force -Path "$BASE\config"
   ```

2. Crea la entidad JPA `Producto`. En IntelliJ, haz clic derecho sobre el paquete `model` → **New → Java Class** → nombre: `Producto`:

   ```java
   package com.empresa.inventario.model;

   import jakarta.persistence.*;
   import jakarta.validation.constraints.*;
   import lombok.*;
   import java.math.BigDecimal;
   import java.time.LocalDateTime;

   /**
    * Entidad JPA que representa un producto en el inventario.
    * Usa Lombok para eliminar código boilerplate (getters, setters, constructores).
    */
   @Entity
   @Table(name = "productos")
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   @Builder
   public class Producto {

       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Long id;

       @NotBlank(message = "El nombre del producto no puede estar vacío")
       @Size(min = 2, max = 100, message = "El nombre debe tener entre 2 y 100 caracteres")
       @Column(nullable = false, length = 100)
       private String nombre;

       @Size(max = 500, message = "La descripción no puede superar 500 caracteres")
       @Column(length = 500)
       private String descripcion;

       @NotNull(message = "El precio es obligatorio")
       @DecimalMin(value = "0.0", inclusive = false, message = "El precio debe ser mayor a cero")
       @Column(nullable = false, precision = 10, scale = 2)
       private BigDecimal precio;

       @NotNull(message = "El stock es obligatorio")
       @Min(value = 0, message = "El stock no puede ser negativo")
       @Column(nullable = false)
       private Integer stock;

       @NotBlank(message = "La categoría es obligatoria")
       @Column(nullable = false, length = 50)
       private String categoria;

       @Column(nullable = false)
       @Builder.Default
       private Boolean activo = true;

       @Column(name = "fecha_creacion", nullable = false, updatable = false)
       private LocalDateTime fechaCreacion;

       @Column(name = "fecha_actualizacion")
       private LocalDateTime fechaActualizacion;

       /**
        * Se ejecuta automáticamente antes de persistir la entidad por primera vez.
        */
       @PrePersist
       protected void onCreate() {
           fechaCreacion = LocalDateTime.now();
           fechaActualizacion = LocalDateTime.now();
       }

       /**
        * Se ejecuta automáticamente antes de cada actualización.
        */
       @PreUpdate
       protected void onUpdate() {
           fechaActualizacion = LocalDateTime.now();
       }
   }
   ```

3. Crea el enum `CategoriaProducto` en el paquete `model` para tipificar las categorías:

   ```java
   package com.empresa.inventario.model;

   /**
    * Categorías válidas para clasificar productos en el inventario.
    */
   public enum CategoriaProducto {
       ELECTRONICA,
       ROPA,
       ALIMENTOS,
       HOGAR,
       DEPORTES,
       LIBROS,
       OTROS
   }
   ```

4. Configura la base de datos H2 en `src/main/resources/application.properties`:

   ```properties
   # ===== CONFIGURACIÓN DE LA APLICACIÓN =====
   spring.application.name=inventario-api
   server.port=8080

   # ===== BASE DE DATOS H2 (EN MEMORIA) =====
   spring.datasource.url=jdbc:h2:mem:inventariodb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
   spring.datasource.driver-class-name=org.h2.Driver
   spring.datasource.username=sa
   spring.datasource.password=

   # ===== JPA / HIBERNATE =====
   spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
   # create-drop: crea el esquema al iniciar y lo elimina al cerrar (ideal para desarrollo)
   spring.jpa.hibernate.ddl-auto=create-drop
   spring.jpa.show-sql=true
   spring.jpa.properties.hibernate.format_sql=true

   # ===== CONSOLA H2 (solo para desarrollo) =====
   spring.h2.console.enabled=true
   spring.h2.console.path=/h2-console

   # ===== JACKSON (SERIALIZACIÓN JSON) =====
   # Ignorar propiedades desconocidas al deserializar JSON
   spring.jackson.deserialization.fail-on-unknown-properties=false
   # Formato de fechas ISO-8601
   spring.jackson.serialization.write-dates-as-timestamps=false

   # ===== LOGGING =====
   logging.level.com.empresa.inventario=DEBUG
   logging.level.org.springframework.web=INFO
   ```

5. Haz commit de los cambios:

   ```bash
   git add src/
   git commit -m "feat: agregar modelo de dominio Producto con JPA y Lombok, configurar H2"
   ```

**Salida Esperada:**

Al compilar con `./mvnw compile`, no debe haber errores. La clase `Producto` debe mostrar en IntelliJ los métodos generados por Lombok (getters, setters, builder) en la vista de estructura del archivo.

**Verificación:**

- [ ] La clase `Producto` compila sin errores y Lombok genera los métodos (verifica en IntelliJ: View → Tool Windows → Structure)
- [ ] El archivo `application.properties` está en `src/main/resources/`
- [ ] La estructura de paquetes está completa con todos los subdirectorios creados

---

### Paso 4: Implementar el Repositorio y la Capa de Servicio

**Objetivo:** Crear el repositorio Spring Data JPA para acceso a datos y la interfaz de servicio con su implementación, separando correctamente las responsabilidades de cada capa.

**Instrucciones:**

1. Crea la interfaz `ProductoRepository` en el paquete `repository`:

   ```java
   package com.empresa.inventario.repository;

   import com.empresa.inventario.model.Producto;
   import org.springframework.data.jpa.repository.JpaRepository;
   import org.springframework.data.jpa.repository.Query;
   import org.springframework.data.repository.query.Param;
   import org.springframework.stereotype.Repository;

   import java.math.BigDecimal;
   import java.util.List;
   import java.util.Optional;

   /**
    * Repositorio Spring Data JPA para la entidad Producto.
    * Spring genera automáticamente la implementación en tiempo de ejecución.
    */
   @Repository
   public interface ProductoRepository extends JpaRepository<Producto, Long> {

       /**
        * Busca productos por categoría (solo activos).
        */
       List<Producto> findByCategoriaAndActivoTrue(String categoria);

       /**
        * Busca productos cuyo nombre contenga el texto dado (insensible a mayúsculas).
        */
       List<Producto> findByNombreContainingIgnoreCaseAndActivoTrue(String nombre);

       /**
        * Busca productos con stock por debajo del umbral especificado.
        */
       @Query("SELECT p FROM Producto p WHERE p.stock < :umbral AND p.activo = true")
       List<Producto> findProductosBajoStock(@Param("umbral") Integer umbral);

       /**
        * Verifica si existe un producto activo con el nombre dado.
        */
       boolean existsByNombreIgnoreCaseAndActivoTrue(String nombre);

       /**
        * Cuenta productos activos por categoría.
        */
       @Query("SELECT COUNT(p) FROM Producto p WHERE p.categoria = :categoria AND p.activo = true")
       long countByCategoriaAndActivo(@Param("categoria") String categoria);

       /**
        * Obtiene solo productos activos.
        */
       List<Producto> findByActivoTrue();
   }
   ```

2. Crea la interfaz `ProductoService` en el paquete `service`:

   ```java
   package com.empresa.inventario.service;

   import com.empresa.inventario.model.Producto;

   import java.util.List;
   import java.util.Optional;

   /**
    * Contrato de la capa de negocio para la gestión de productos.
    * Definir una interfaz permite desacoplar la implementación y facilita el testing con Mockito.
    */
   public interface ProductoService {

       /**
        * Obtiene todos los productos activos del inventario.
        */
       List<Producto> obtenerTodos();

       /**
        * Busca un producto por su ID.
        * @throws com.empresa.inventario.exception.ProductoNotFoundException si no existe
        */
       Producto obtenerPorId(Long id);

       /**
        * Crea un nuevo producto en el inventario.
        * @throws com.empresa.inventario.exception.ProductoDuplicadoException si ya existe uno con el mismo nombre
        */
       Producto crear(Producto producto);

       /**
        * Actualiza un producto existente.
        * @throws com.empresa.inventario.exception.ProductoNotFoundException si no existe
        */
       Producto actualizar(Long id, Producto producto);

       /**
        * Realiza una baja lógica del producto (activo = false).
        * @throws com.empresa.inventario.exception.ProductoNotFoundException si no existe
        */
       void eliminar(Long id);

       /**
        * Busca productos por categoría.
        */
       List<Producto> buscarPorCategoria(String categoria);

       /**
        * Busca productos cuyo nombre contenga el texto dado.
        */
       List<Producto> buscarPorNombre(String nombre);

       /**
        * Obtiene productos con stock por debajo del umbral.
        */
       List<Producto> obtenerBajoStock(Integer umbral);
   }
   ```

3. Crea las excepciones personalizadas en el paquete `exception`:

   ```java
   // Archivo: exception/ProductoNotFoundException.java
   package com.empresa.inventario.exception;

   public class ProductoNotFoundException extends RuntimeException {
       public ProductoNotFoundException(Long id) {
           super("Producto no encontrado con ID: " + id);
       }
       public ProductoNotFoundException(String mensaje) {
           super(mensaje);
       }
   }
   ```

   ```java
   // Archivo: exception/ProductoDuplicadoException.java
   package com.empresa.inventario.exception;

   public class ProductoDuplicadoException extends RuntimeException {
       public ProductoDuplicadoException(String nombre) {
           super("Ya existe un producto activo con el nombre: " + nombre);
       }
   }
   ```

4. Crea la implementación `ProductoServiceImpl` en el paquete `service/impl`:

   ```java
   package com.empresa.inventario.service.impl;

   import com.empresa.inventario.exception.ProductoDuplicadoException;
   import com.empresa.inventario.exception.ProductoNotFoundException;
   import com.empresa.inventario.model.Producto;
   import com.empresa.inventario.repository.ProductoRepository;
   import com.empresa.inventario.service.ProductoService;
   import lombok.RequiredArgsConstructor;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.stereotype.Service;
   import org.springframework.transaction.annotation.Transactional;

   import java.util.List;

   /**
    * Implementación de la lógica de negocio para gestión de productos.
    *
    * @Service: marca esta clase como componente de servicio Spring
    * @Transactional: gestiona transacciones de base de datos automáticamente
    * @RequiredArgsConstructor: Lombok genera constructor con todos los campos final
    * @Slf4j: Lombok genera un logger estático (log.info(), log.error(), etc.)
    */
   @Service
   @Transactional
   @RequiredArgsConstructor
   @Slf4j
   public class ProductoServiceImpl implements ProductoService {

       private final ProductoRepository productoRepository;

       @Override
       @Transactional(readOnly = true)
       public List<Producto> obtenerTodos() {
           log.debug("Obteniendo todos los productos activos");
           return productoRepository.findByActivoTrue();
       }

       @Override
       @Transactional(readOnly = true)
       public Producto obtenerPorId(Long id) {
           log.debug("Buscando producto con ID: {}", id);
           return productoRepository.findById(id)
                   .filter(Producto::getActivo)
                   .orElseThrow(() -> new ProductoNotFoundException(id));
       }

       @Override
       public Producto crear(Producto producto) {
           log.info("Creando nuevo producto: {}", producto.getNombre());

           if (productoRepository.existsByNombreIgnoreCaseAndActivoTrue(producto.getNombre())) {
               throw new ProductoDuplicadoException(producto.getNombre());
           }

           Producto productoGuardado = productoRepository.save(producto);
           log.info("Producto creado exitosamente con ID: {}", productoGuardado.getId());
           return productoGuardado;
       }

       @Override
       public Producto actualizar(Long id, Producto productoActualizado) {
           log.info("Actualizando producto con ID: {}", id);

           Producto productoExistente = obtenerPorId(id);

           // Verificar nombre duplicado solo si cambió
           if (!productoExistente.getNombre().equalsIgnoreCase(productoActualizado.getNombre())
                   && productoRepository.existsByNombreIgnoreCaseAndActivoTrue(productoActualizado.getNombre())) {
               throw new ProductoDuplicadoException(productoActualizado.getNombre());
           }

           // Actualizar campos del producto existente
           productoExistente.setNombre(productoActualizado.getNombre());
           productoExistente.setDescripcion(productoActualizado.getDescripcion());
           productoExistente.setPrecio(productoActualizado.getPrecio());
           productoExistente.setStock(productoActualizado.getStock());
           productoExistente.setCategoria(productoActualizado.getCategoria());

           return productoRepository.save(productoExistente);
       }

       @Override
       public void eliminar(Long id) {
           log.info("Eliminando (baja lógica) producto con ID: {}", id);
           Producto producto = obtenerPorId(id);
           producto.setActivo(false);
           productoRepository.save(producto);
           log.info("Producto con ID: {} desactivado exitosamente", id);
       }

       @Override
       @Transactional(readOnly = true)
       public List<Producto> buscarPorCategoria(String categoria) {
           log.debug("Buscando productos por categoría: {}", categoria);
           return productoRepository.findByCategoriaAndActivoTrue(categoria);
       }

       @Override
       @Transactional(readOnly = true)
       public List<Producto> buscarPorNombre(String nombre) {
           log.debug("Buscando productos por nombre: {}", nombre);
           return productoRepository.findByNombreContainingIgnoreCaseAndActivoTrue(nombre);
       }

       @Override
       @Transactional(readOnly = true)
       public List<Producto> obtenerBajoStock(Integer umbral) {
           log.debug("Buscando productos con stock menor a: {}", umbral);
           return productoRepository.findProductosBajoStock(umbral);
       }
   }
   ```

5. Haz commit de la capa de servicio:

   ```bash
   git add src/
   git commit -m "feat: implementar capa de repositorio y servicio con logica de negocio"
   ```

**Salida Esperada:**

```
[INFO] BUILD SUCCESS
```

Al ejecutar `./mvnw compile`, el proyecto debe compilar sin errores.

**Verificación:**

- [ ] `ProductoRepository` extiende `JpaRepository<Producto, Long>`
- [ ] `ProductoService` es una interfaz con los 8 métodos definidos
- [ ] `ProductoServiceImpl` implementa todos los métodos de `ProductoService`
- [ ] Las excepciones `ProductoNotFoundException` y `ProductoDuplicadoException` existen en el paquete `exception`
- [ ] El proyecto compila con `./mvnw compile` sin errores

---

### Paso 5: Implementar DTOs y el Patrón Facade

**Objetivo:** Crear los DTOs para controlar qué información se expone en la API y el Facade para encapsular la lógica de orquestación, separando la capa de presentación de la lógica de negocio.

**Instrucciones:**

1. Crea el DTO de request `CrearProductoRequest` en `dto/request/`:

   ```java
   package com.empresa.inventario.dto.request;

   import jakarta.validation.constraints.*;
   import lombok.*;
   import java.math.BigDecimal;

   /**
    * DTO para recibir datos de creación de un producto.
    * Solo contiene los campos que el cliente PUEDE enviar (no incluye id, fechas, activo).
    * Las anotaciones de validación garantizan que los datos sean correctos antes de procesar.
    */
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   @Builder
   public class CrearProductoRequest {

       @NotBlank(message = "El nombre es obligatorio")
       @Size(min = 2, max = 100, message = "El nombre debe tener entre 2 y 100 caracteres")
       private String nombre;

       @Size(max = 500, message = "La descripción no puede superar 500 caracteres")
       private String descripcion;

       @NotNull(message = "El precio es obligatorio")
       @DecimalMin(value = "0.01", message = "El precio debe ser mayor a cero")
       private BigDecimal precio;

       @NotNull(message = "El stock es obligatorio")
       @Min(value = 0, message = "El stock no puede ser negativo")
       private Integer stock;

       @NotBlank(message = "La categoría es obligatoria")
       private String categoria;
   }
   ```

2. Crea el DTO de request `ActualizarProductoRequest` en `dto/request/`:

   ```java
   package com.empresa.inventario.dto.request;

   import jakarta.validation.constraints.*;
   import lombok.*;
   import java.math.BigDecimal;

   /**
    * DTO para recibir datos de actualización de un producto.
    * Similar a CrearProductoRequest pero todos los campos son opcionales
    * para soportar actualizaciones parciales.
    */
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   @Builder
   public class ActualizarProductoRequest {

       @Size(min = 2, max = 100, message = "El nombre debe tener entre 2 y 100 caracteres")
       private String nombre;

       @Size(max = 500, message = "La descripción no puede superar 500 caracteres")
       private String descripcion;

       @DecimalMin(value = "0.01", message = "El precio debe ser mayor a cero")
       private BigDecimal precio;

       @Min(value = 0, message = "El stock no puede ser negativo")
       private Integer stock;

       private String categoria;
   }
   ```

3. Crea el DTO de response `ProductoResponse` en `dto/response/`:

   ```java
   package com.empresa.inventario.dto.response;

   import lombok.*;
   import java.math.BigDecimal;
   import java.time.LocalDateTime;

   /**
    * DTO de respuesta para exponer datos de un producto.
    * Controla EXACTAMENTE qué información se envía al cliente.
    * Nota: no incluye el campo 'activo' (detalle interno del sistema).
    */
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   @Builder
   public class ProductoResponse {

       private Long id;
       private String nombre;
       private String descripcion;
       private BigDecimal precio;
       private Integer stock;
       private String categoria;
       private LocalDateTime fechaCreacion;
       private LocalDateTime fechaActualizacion;

       /**
        * Campo calculado: indica si el producto tiene stock disponible.
        * Este dato no existe en la entidad pero es útil para el cliente.
        */
       public boolean isDisponible() {
           return stock != null && stock > 0;
       }
   }
   ```

4. Crea el DTO `ErrorResponse` en `dto/response/` para respuestas de error estandarizadas:

   ```java
   package com.empresa.inventario.dto.response;

   import lombok.*;
   import java.time.LocalDateTime;
   import java.util.List;

   /**
    * DTO estándar para respuestas de error de la API.
    * Garantiza un formato consistente en todos los errores.
    */
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   @Builder
   public class ErrorResponse {

       private int status;
       private String error;
       private String mensaje;
       private String path;
       private LocalDateTime timestamp;
       private List<String> detalles;
   }
   ```

5. Crea el mapper `ProductoMapper` en el paquete `dto/` para convertir entre entidades y DTOs:

   ```java
   package com.empresa.inventario.dto;

   import com.empresa.inventario.dto.request.ActualizarProductoRequest;
   import com.empresa.inventario.dto.request.CrearProductoRequest;
   import com.empresa.inventario.dto.response.ProductoResponse;
   import com.empresa.inventario.model.Producto;
   import org.springframework.stereotype.Component;

   import java.util.List;
   import java.util.stream.Collectors;

   /**
    * Componente responsable de convertir entre entidades Producto y DTOs.
    * Centralizar las conversiones aquí evita código duplicado y facilita cambios futuros.
    */
   @Component
   public class ProductoMapper {

       /**
        * Convierte una entidad Producto a su DTO de respuesta.
        */
       public ProductoResponse toResponse(Producto producto) {
           if (producto == null) return null;

           return ProductoResponse.builder()
                   .id(producto.getId())
                   .nombre(producto.getNombre())
                   .descripcion(producto.getDescripcion())
                   .precio(producto.getPrecio())
                   .stock(producto.getStock())
                   .categoria(producto.getCategoria())
                   .fechaCreacion(producto.getFechaCreacion())
                   .fechaActualizacion(producto.getFechaActualizacion())
                   .build();
       }

       /**
        * Convierte una lista de entidades a una lista de DTOs de respuesta.
        */
       public List<ProductoResponse> toResponseList(List<Producto> productos) {
           return productos.stream()
                   .map(this::toResponse)
                   .collect(Collectors.toList());
       }

       /**
        * Convierte un DTO de creación a una entidad Producto.
        */
       public Producto toEntity(CrearProductoRequest request) {
           if (request == null) return null;

           return Producto.builder()
                   .nombre(request.getNombre())
                   .descripcion(request.getDescripcion())
                   .precio(request.getPrecio())
                   .stock(request.getStock())
                   .categoria(request.getCategoria())
                   .activo(true)
                   .build();
       }

       /**
        * Actualiza una entidad existente con los datos del DTO de actualización.
        * Solo actualiza los campos que no son null (actualización parcial).
        */
       public void updateEntityFromRequest(ActualizarProductoRequest request, Producto producto) {
           if (request == null || producto == null) return;

           if (request.getNombre() != null) producto.setNombre(request.getNombre());
           if (request.getDescripcion() != null) producto.setDescripcion(request.getDescripcion());
           if (request.getPrecio() != null) producto.setPrecio(request.getPrecio());
           if (request.getStock() != null) producto.setStock(request.getStock());
           if (request.getCategoria() != null) producto.setCategoria(request.getCategoria());
       }
   }
   ```

6. Crea el `ProductoFacade` en el paquete `facade/`:

   ```java
   package com.empresa.inventario.facade;

   import com.empresa.inventario.dto.ProductoMapper;
   import com.empresa.inventario.dto.request.ActualizarProductoRequest;
   import com.empresa.inventario.dto.request.CrearProductoRequest;
   import com.empresa.inventario.dto.response.ProductoResponse;
   import com.empresa.inventario.model.Producto;
   import com.empresa.inventario.service.ProductoService;
   import lombok.RequiredArgsConstructor;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.stereotype.Component;

   import java.util.List;

   /**
    * Facade para la gestión de productos.
    *
    * PATRÓN FACADE: simplifica la interfaz de la capa de negocio para el controlador.
    * El controlador solo habla con el Facade, que a su vez coordina el servicio y el mapper.
    * Esto desacopla el controlador de los detalles de implementación del servicio.
    *
    * Beneficios:
    * - El controlador no conoce la entidad Producto (solo trabaja con DTOs)
    * - Si la lógica de negocio cambia, solo se modifica aquí, no en el controlador
    * - Facilita el testing del controlador (se mockea el Facade, no múltiples servicios)
    */
   @Component
   @RequiredArgsConstructor
   @Slf4j
   public class ProductoFacade {

       private final ProductoService productoService;
       private final ProductoMapper productoMapper;

       public List<ProductoResponse> obtenerTodos() {
           List<Producto> productos = productoService.obtenerTodos();
           return productoMapper.toResponseList(productos);
       }

       public ProductoResponse obtenerPorId(Long id) {
           Producto producto = productoService.obtenerPorId(id);
           return productoMapper.toResponse(producto);
       }

       public ProductoResponse crear(CrearProductoRequest request) {
           log.info("Facade: procesando creación de producto '{}'", request.getNombre());
           Producto producto = productoMapper.toEntity(request);
           Producto creado = productoService.crear(producto);
           return productoMapper.toResponse(creado);
       }

       public ProductoResponse actualizar(Long id, ActualizarProductoRequest request) {
           log.info("Facade: procesando actualización de producto ID: {}", id);
           // Obtenemos la entidad actual
           Producto productoExistente = productoService.obtenerPorId(id);
           // Aplicamos los cambios del request sobre la entidad existente
           productoMapper.updateEntityFromRequest(request, productoExistente);
           // Guardamos la entidad actualizada
           Producto actualizado = productoService.actualizar(id, productoExistente);
           return productoMapper.toResponse(actualizado);
       }

       public void eliminar(Long id) {
           log.info("Facade: procesando eliminación de producto ID: {}", id);
           productoService.eliminar(id);
       }

       public List<ProductoResponse> buscarPorCategoria(String categoria) {
           return productoMapper.toResponseList(productoService.buscarPorCategoria(categoria));
       }

       public List<ProductoResponse> buscarPorNombre(String nombre) {
           return productoMapper.toResponseList(productoService.buscarPorNombre(nombre));
       }

       public List<ProductoResponse> obtenerBajoStock(Integer umbral) {
           return productoMapper.toResponseList(productoService.obtenerBajoStock(umbral));
       }
   }
   ```

7. Haz commit de DTOs y Facade:

   ```bash
   git add src/
   git commit -m "feat: implementar DTOs, ProductoMapper y patron Facade"
   ```

**Salida Esperada:**

```
[INFO] BUILD SUCCESS
```

**Verificación:**

- [ ] Los DTOs `CrearProductoRequest`, `ActualizarProductoRequest`, `ProductoResponse` y `ErrorResponse` existen en sus paquetes correctos
- [ ] `ProductoMapper` tiene los 4 métodos de conversión
- [ ] `ProductoFacade` inyecta `ProductoService` y `ProductoMapper` mediante constructor
- [ ] El proyecto compila sin errores

---

### Paso 6: Implementar el Controlador REST y el Manejador Global de Errores

**Objetivo:** Crear el controlador REST con todos los endpoints CRUD usando los verbos HTTP correctos y códigos de estado apropiados, e implementar el manejo global de errores con `@ControllerAdvice`.

**Instrucciones:**

1. Crea el manejador global de errores `GlobalExceptionHandler` en el paquete `exception/`:

   ```java
   package com.empresa.inventario.exception;

   import com.empresa.inventario.dto.response.ErrorResponse;
   import jakarta.servlet.http.HttpServletRequest;
   import jakarta.validation.ConstraintViolationException;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.http.HttpStatus;
   import org.springframework.http.ResponseEntity;
   import org.springframework.validation.FieldError;
   import org.springframework.web.bind.MethodArgumentNotValidException;
   import org.springframework.web.bind.annotation.ExceptionHandler;
   import org.springframework.web.bind.annotation.RestControllerAdvice;

   import java.time.LocalDateTime;
   import java.util.List;
   import java.util.stream.Collectors;

   /**
    * Manejador global de excepciones para toda la API.
    *
    * @RestControllerAdvice: intercepta excepciones de todos los controladores
    * y devuelve respuestas de error en formato JSON estandarizado.
    */
   @RestControllerAdvice
   @Slf4j
   public class GlobalExceptionHandler {

       /**
        * Maneja el caso en que un producto no existe (404 Not Found).
        */
       @ExceptionHandler(ProductoNotFoundException.class)
       public ResponseEntity<ErrorResponse> handleProductoNotFound(
               ProductoNotFoundException ex, HttpServletRequest request) {

           log.warn("Producto no encontrado: {}", ex.getMessage());

           ErrorResponse error = ErrorResponse.builder()
                   .status(HttpStatus.NOT_FOUND.value())
                   .error("Recurso No Encontrado")
                   .mensaje(ex.getMessage())
                   .path(request.getRequestURI())
                   .timestamp(LocalDateTime.now())
                   .build();

           return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
       }

       /**
        * Maneja el caso de producto duplicado (409 Conflict).
        */
       @ExceptionHandler(ProductoDuplicadoException.class)
       public ResponseEntity<ErrorResponse> handleProductoDuplicado(
               ProductoDuplicadoException ex, HttpServletRequest request) {

           log.warn("Intento de crear producto duplicado: {}", ex.getMessage());

           ErrorResponse error = ErrorResponse.builder()
                   .status(HttpStatus.CONFLICT.value())
                   .error("Conflicto de Datos")
                   .mensaje(ex.getMessage())
                   .path(request.getRequestURI())
                   .timestamp(LocalDateTime.now())
                   .build();

           return ResponseEntity.status(HttpStatus.CONFLICT).body(error);
       }

       /**
        * Maneja errores de validación de @RequestBody (400 Bad Request).
        * Se activa cuando falla @Valid en los parámetros del controlador.
        */
       @ExceptionHandler(MethodArgumentNotValidException.class)
       public ResponseEntity<ErrorResponse> handleValidationErrors(
               MethodArgumentNotValidException ex, HttpServletRequest request) {

           List<String> detalles = ex.getBindingResult().getFieldErrors()
                   .stream()
                   .map(FieldError::getDefaultMessage)
                   .collect(Collectors.toList());

           log.warn("Errores de validación: {}", detalles);

           ErrorResponse error = ErrorResponse.builder()
                   .status(HttpStatus.BAD_REQUEST.value())
                   .error("Error de Validación")
                   .mensaje("Los datos enviados no son válidos")
                   .path(request.getRequestURI())
                   .timestamp(LocalDateTime.now())
                   .detalles(detalles)
                   .build();

           return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
       }

       /**
        * Maneja cualquier excepción no controlada (500 Internal Server Error).
        */
       @ExceptionHandler(Exception.class)
       public ResponseEntity<ErrorResponse> handleGenericException(
               Exception ex, HttpServletRequest request) {

           log.error("Error inesperado en la API: {}", ex.getMessage(), ex);

           ErrorResponse error = ErrorResponse.builder()
                   .status(HttpStatus.INTERNAL_SERVER_ERROR.value())
                   .error("Error Interno del Servidor")
                   .mensaje("Ocurrió un error inesperado. Por favor contacte al administrador.")
                   .path(request.getRequestURI())
                   .timestamp(LocalDateTime.now())
                   .build();

           return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
       }
   }
   ```

2. Crea la configuración CORS en el paquete `config/`:

   ```java
   package com.empresa.inventario.config;

   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.web.cors.CorsConfiguration;
   import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
   import org.springframework.web.filter.CorsFilter;

   /**
    * Configuración CORS para la API.
    *
    * IMPORTANTE: La configuración actual (allowedOrigins: "*") es SOLO para desarrollo local.
    * En producción se debe restringir a los dominios específicos del frontend:
    *   config.setAllowedOrigins(List.of("https://miapp.empresa.com"));
    */
   @Configuration
   public class CorsConfig {

       @Bean
       public CorsFilter corsFilter() {
           CorsConfiguration config = new CorsConfiguration();

           // DESARROLLO: permite cualquier origen (NO usar en producción)
           config.addAllowedOriginPattern("*");
           config.addAllowedMethod("*");
           config.addAllowedHeader("*");

           // Tiempo de caché para preflight requests (en segundos)
           config.setMaxAge(3600L);

           UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
           source.registerCorsConfiguration("/api/**", config);

           return new CorsFilter(source);
       }
   }
   ```

3. Crea el controlador principal `ProductoController` en el paquete `controller/`:

   ```java
   package com.empresa.inventario.controller;

   import com.empresa.inventario.dto.request.ActualizarProductoRequest;
   import com.empresa.inventario.dto.request.CrearProductoRequest;
   import com.empresa.inventario.dto.response.ProductoResponse;
   import com.empresa.inventario.facade.ProductoFacade;
   import jakarta.validation.Valid;
   import jakarta.validation.constraints.Min;
   import lombok.RequiredArgsConstructor;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.http.HttpStatus;
   import org.springframework.http.ResponseEntity;
   import org.springframework.web.bind.annotation.*;

   import java.util.List;

   /**
    * Controlador REST para la gestión de productos del inventario.
    *
    * Convenciones de la API:
    * - GET    /api/v1/productos          → Obtener todos los productos
    * - GET    /api/v1/productos/{id}     → Obtener producto por ID
    * - POST   /api/v1/productos          → Crear nuevo producto (201 Created)
    * - PUT    /api/v1/productos/{id}     → Actualizar producto completo
    * - DELETE /api/v1/productos/{id}     → Eliminar producto (baja lógica)
    * - GET    /api/v1/productos/buscar   → Buscar por nombre o categoría
    * - GET    /api/v1/productos/bajo-stock → Productos con stock crítico
    */
   @RestController
   @RequestMapping("/api/v1/productos")
   @RequiredArgsConstructor
   @Slf4j
   public class ProductoController {

       private final ProductoFacade productoFacade;

       /**
        * GET /api/v1/productos
        * Obtiene todos los productos activos del inventario.
        * Código 200 OK con lista (puede estar vacía).
        */
       @GetMapping
       public ResponseEntity<List<ProductoResponse>> obtenerTodos() {
           log.info("GET /api/v1/productos - Solicitando todos los productos");
           List<ProductoResponse> productos = productoFacade.obtenerTodos();
           return ResponseEntity.ok(productos);
       }

       /**
        * GET /api/v1/productos/{id}
        * Obtiene un producto específico por su ID.
        * Código 200 OK si existe, 404 Not Found si no existe.
        */
       @GetMapping("/{id}")
       public ResponseEntity<ProductoResponse> obtenerPorId(@PathVariable Long id) {
           log.info("GET /api/v1/productos/{} - Solicitando producto por ID", id);
           ProductoResponse producto = productoFacade.obtenerPorId(id);
           return ResponseEntity.ok(producto);
       }

       /**
        * POST /api/v1/productos
        * Crea un nuevo producto en el inventario.
        * Código 201 Created con el producto creado en el body.
        * @Valid activa las validaciones definidas en CrearProductoRequest.
        */
       @PostMapping
       public ResponseEntity<ProductoResponse> crear(
               @Valid @RequestBody CrearProductoRequest request) {
           log.info("POST /api/v1/productos - Creando producto: {}", request.getNombre());
           ProductoResponse creado = productoFacade.crear(request);
           return ResponseEntity.status(HttpStatus.CREATED).body(creado);
       }

       /**
        * PUT /api/v1/productos/{id}
        * Actualiza un producto existente.
        * Código 200 OK con el producto actualizado.
        */
       @PutMapping("/{id}")
       public ResponseEntity<ProductoResponse> actualizar(
               @PathVariable Long id,
               @Valid @RequestBody ActualizarProductoRequest request) {
           log.info("PUT /api/v1/productos/{} - Actualizando producto", id);
           ProductoResponse actualizado = productoFacade.actualizar(id, request);
           return ResponseEntity.ok(actualizado);
       }

       /**
        * DELETE /api/v1/productos/{id}
        * Realiza baja lógica del producto (activo = false).
        * Código 204 No Content (éxito sin body de respuesta).
        */
       @DeleteMapping("/{id}")
       public ResponseEntity<Void> eliminar(@PathVariable Long id) {
           log.info("DELETE /api/v1/productos/{} - Eliminando producto", id);
           productoFacade.eliminar(id);
           return ResponseEntity.noContent().build();
       }

       /**
        * GET /api/v1/productos/buscar?nombre=laptop
        * GET /api/v1/productos/buscar?categoria=ELECTRONICA
        * Busca productos por nombre o categoría (parámetros opcionales).
        */
       @GetMapping("/buscar")
       public ResponseEntity<List<ProductoResponse>> buscar(
               @RequestParam(required = false) String nombre,
               @RequestParam(required = false) String categoria) {

           log.info("GET /api/v1/productos/buscar - nombre={}, categoria={}", nombre, categoria);

           if (nombre != null && !nombre.isBlank()) {
               return ResponseEntity.ok(productoFacade.buscarPorNombre(nombre));
           }
           if (categoria != null && !categoria.isBlank()) {
               return ResponseEntity.ok(productoFacade.buscarPorCategoria(categoria));
           }

           // Si no se pasan parámetros, devuelve todos
           return ResponseEntity.ok(productoFacade.obtenerTodos());
       }

       /**
        * GET /api/v1/productos/bajo-stock?umbral=10
        * Obtiene productos con stock por debajo del umbral especificado.
        * Útil para alertas de reabastecimiento.
        */
       @GetMapping("/bajo-stock")
       public ResponseEntity<List<ProductoResponse>> obtenerBajoStock(
               @RequestParam(defaultValue = "10") @Min(1) Integer umbral) {
           log.info("GET /api/v1/productos/bajo-stock - umbral={}", umbral);
           return ResponseEntity.ok(productoFacade.obtenerBajoStock(umbral));
       }
   }
   ```

4. Crea un `DataInitializer` para cargar datos de prueba al iniciar la aplicación. Esto facilita las pruebas manuales con Postman:

   ```java
   package com.empresa.inventario.config;

   import com.empresa.inventario.model.Producto;
   import com.empresa.inventario.repository.ProductoRepository;
   import lombok.RequiredArgsConstructor;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.boot.CommandLineRunner;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;

   import java.math.BigDecimal;

   /**
    * Inicializa la base de datos H2 con datos de prueba al arrancar la aplicación.
    * Solo activo en el perfil de desarrollo.
    */
   @Configuration
   @RequiredArgsConstructor
   @Slf4j
   public class DataInitializer {

       @Bean
       public CommandLineRunner initData(ProductoRepository productoRepository) {
           return args -> {
               log.info("Inicializando datos de prueba...");

               productoRepository.save(Producto.builder()
                       .nombre("Laptop Dell XPS 15")
                       .descripcion("Laptop profesional con procesador Intel Core i7")
                       .precio(new BigDecimal("1299.99"))
                       .stock(15)
                       .categoria("ELECTRONICA")
                       .activo(true)
                       .build());

               productoRepository.save(Producto.builder()
                       .nombre("Teclado Mecánico Logitech")
                       .descripcion("Teclado gaming con switches Cherry MX Red")
                       .precio(new BigDecimal("89.99"))
                       .stock(8)
                       .categoria("ELECTRONICA")
                       .activo(true)
                       .build());

               productoRepository.save(Producto.builder()
                       .nombre("Silla Ergonómica Herman Miller")
                       .descripcion("Silla de oficina con soporte lumbar ajustable")
                       .precio(new BigDecimal("899.00"))
                       .stock(3)
                       .categoria("HOGAR")
                       .activo(true)
                       .build());

               productoRepository.save(Producto.builder()
                       .nombre("Clean Code - Robert C. Martin")
                       .descripcion("Libro sobre buenas prácticas de programación")
                       .precio(new BigDecimal("45.50"))
                       .stock(20)
                       .categoria("LIBROS")
                       .activo(true)
                       .build());

               productoRepository.save(Producto.builder()
                       .nombre("Monitor 4K LG UltraWide")
                       .descripcion("Monitor 34 pulgadas resolución 3440x1440")
                       .precio(new BigDecimal("549.99"))
                       .stock(5)
                       .categoria("ELECTRONICA")
                       .activo(true)
                       .build());

               log.info("Datos de prueba cargados: {} productos", productoRepository.count());
           };
       }
   }
   ```

5. Ejecuta la aplicación para verificar que arranca correctamente:

   ```bash
   # En Linux/macOS
   ./mvnw spring-boot:run

   # En Windows (PowerShell)
   .\mvnw.cmd spring-boot:run
   ```

6. En otra terminal, prueba los endpoints con `curl`:

   ```bash
   # Obtener todos los productos
   curl -s http://localhost:8080/api/v1/productos | python3 -m json.tool

   # Obtener producto por ID
   curl -s http://localhost:8080/api/v1/productos/1 | python3 -m json.tool

   # Crear un nuevo producto
   curl -s -X POST http://localhost:8080/api/v1/productos \
     -H "Content-Type: application/json" \
     -d '{"nombre":"Mouse Inalámbrico","descripcion":"Mouse ergonómico Logitech MX Master","precio":79.99,"stock":25,"categoria":"ELECTRONICA"}' \
     | python3 -m json.tool

   # Buscar por categoría
   curl -s "http://localhost:8080/api/v1/productos/buscar?categoria=ELECTRONICA" | python3 -m json.tool

   # Obtener productos bajo stock (umbral 10)
   curl -s "http://localhost:8080/api/v1/productos/bajo-stock?umbral=10" | python3 -m json.tool
   ```

   En Windows (PowerShell) sin `python3`:
   ```powershell
   # Obtener todos los productos
   Invoke-RestMethod -Uri "http://localhost:8080/api/v1/productos" -Method GET | ConvertTo-Json -Depth 5

   # Crear un nuevo producto
   $body = '{"nombre":"Mouse Inalámbrico","descripcion":"Mouse ergonómico","precio":79.99,"stock":25,"categoria":"ELECTRONICA"}'
   Invoke-RestMethod -Uri "http://localhost:8080/api/v1/productos" -Method POST -Body $body -ContentType "application/json"
   ```

7. Haz commit del controlador y configuraciones:

   ```bash
   # Detén la aplicación con Ctrl+C primero
   git add src/
   git commit -m "feat: implementar controlador REST, manejo global de errores y datos iniciales"
   git push origin main
   ```

**Salida Esperada:**

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v3.2.5)

...
Started InventarioApiApplication in 2.345 seconds
Datos de prueba cargados: 5 productos
```

**Verificación:**

- [ ] La aplicación arranca sin errores en el puerto 8080
- [ ] `GET http://localhost:8080/api/v1/productos` devuelve una lista con 5 productos en JSON
- [ ] `POST` con datos válidos devuelve código 201 y el producto creado
- [ ] `POST` con nombre duplicado devuelve código 409 con mensaje de error
- [ ] `POST` con datos inválidos (nombre vacío) devuelve código 400 con detalles de validación
- [ ] `GET http://localhost:8080/api/v1/productos/999` devuelve código 404

---

### Paso 7: Escribir Pruebas Unitarias para la Capa de Servicio con JUnit 5 y Mockito

**Objetivo:** Implementar pruebas unitarias completas para `ProductoServiceImpl` usando Mockito para simular el repositorio, cubriendo los escenarios de éxito y de error.

> ⚠️ **Importante:** Las pruebas unitarias son un entregable obligatorio de este laboratorio. No las omitas. Dedica el tiempo necesario a comprenderlas y ejecutarlas.

**Instrucciones:**

1. Crea la clase de prueba `ProductoServiceImplTest` en `src/test/java/com/empresa/inventario/service/`:

   ```java
   package com.empresa.inventario.service;

   import com.empresa.inventario.exception.ProductoDuplicadoException;
   import com.empresa.inventario.exception.ProductoNotFoundException;
   import com.empresa.inventario.model.Producto;
   import com.empresa.inventario.repository.ProductoRepository;
   import com.empresa.inventario.service.impl.ProductoServiceImpl;
   import org.junit.jupiter.api.BeforeEach;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Nested;
   import org.junit.jupiter.api.Test;
   import org.junit.jupiter.api.extension.ExtendWith;
   import org.mockito.InjectMocks;
   import org.mockito.Mock;
   import org.mockito.junit.jupiter.MockitoExtension;

   import java.math.BigDecimal;
   import java.util.Arrays;
   import java.util.Collections;
   import java.util.List;
   import java.util.Optional;

   import static org.assertj.core.api.Assertions.*;
   import static org.mockito.ArgumentMatchers.*;
   import static org.mockito.Mockito.*;

   /**
    * Pruebas unitarias para ProductoServiceImpl.
    *
    * @ExtendWith(MockitoExtension.class): integra Mockito con JUnit 5
    * @Mock: crea un mock del repositorio (no toca la base de datos real)
    * @InjectMocks: crea la instancia real del servicio inyectando los mocks
    *
    * Patrón AAA (Arrange-Act-Assert):
    * - Arrange: preparar datos y configurar mocks
    * - Act: ejecutar el método bajo prueba
    * - Assert: verificar el resultado esperado
    */
   @ExtendWith(MockitoExtension.class)
   @DisplayName("Pruebas Unitarias: ProductoServiceImpl")
   class ProductoServiceImplTest {

       @Mock
       private ProductoRepository productoRepository;

       @InjectMocks
       private ProductoServiceImpl productoService;

       // Datos de prueba reutilizables
       private Producto productoActivo;
       private Producto productoInactivo;

       @BeforeEach
       void setUp() {
           productoActivo = Producto.builder()
                   .id(1L)
                   .nombre("Laptop Dell XPS")
                   .descripcion("Laptop profesional")
                   .precio(new BigDecimal("1299.99"))
                   .stock(10)
                   .categoria("ELECTRONICA")
                   .activo(true)
                   .build();

           productoInactivo = Producto.builder()
                   .id(2L)
                   .nombre("Producto Eliminado")
                   .precio(new BigDecimal("50.00"))
                   .stock(5)
                   .categoria("OTROS")
                   .activo(false)
                   .build();
       }

       // ==================== TESTS: obtenerTodos() ====================

       @Nested
       @DisplayName("obtenerTodos()")
       class ObtenerTodosTests {

           @Test
           @DisplayName("Debe retornar lista de productos activos cuando existen productos")
           void debeRetornarListaDeProductosActivos() {
               // Arrange
               List<Producto> productosActivos = Arrays.asList(productoActivo);
               when(productoRepository.findByActivoTrue()).thenReturn(productosActivos);

               // Act
               List<Producto> resultado = productoService.obtenerTodos();

               // Assert
               assertThat(resultado).isNotNull();
               assertThat(resultado).hasSize(1);
               assertThat(resultado.get(0).getNombre()).isEqualTo("Laptop Dell XPS");
               verify(productoRepository, times(1)).findByActivoTrue();
           }

           @Test
           @DisplayName("Debe retornar lista vacía cuando no hay productos activos")
           void debeRetornarListaVaciaCuandoNoHayProductos() {
               // Arrange
               when(productoRepository.findByActivoTrue()).thenReturn(Collections.emptyList());

               // Act
               List<Producto> resultado = productoService.obtenerTodos();

               // Assert
               assertThat(resultado).isNotNull();
               assertThat(resultado).isEmpty();
           }
       }

       // ==================== TESTS: obtenerPorId() ====================

       @Nested
       @DisplayName("obtenerPorId()")
       class ObtenerPorIdTests {

           @Test
           @DisplayName("Debe retornar el producto cuando existe y está activo")
           void debeRetornarProductoCuandoExisteYEstaActivo() {
               // Arrange
               when(productoRepository.findById(1L)).thenReturn(Optional.of(productoActivo));

               // Act
               Producto resultado = productoService.obtenerPorId(1L);

               // Assert
               assertThat(resultado).isNotNull();
               assertThat(resultado.getId()).isEqualTo(1L);
               assertThat(resultado.getNombre()).isEqualTo("Laptop Dell XPS");
               assertThat(resultado.getActivo()).isTrue();
           }

           @Test
           @DisplayName("Debe lanzar ProductoNotFoundException cuando el producto no existe")
           void debeLanzarExcepcionCuandoProductoNoExiste() {
               // Arrange
               when(productoRepository.findById(999L)).thenReturn(Optional.empty());

               // Act & Assert
               assertThatThrownBy(() -> productoService.obtenerPorId(999L))
                       .isInstanceOf(ProductoNotFoundException.class)
                       .hasMessageContaining("999");
           }

           @Test
           @DisplayName("Debe lanzar ProductoNotFoundException cuando el producto está inactivo")
           void debeLanzarExcepcionCuandoProductoEstaInactivo() {
               // Arrange
               when(productoRepository.findById(2L)).thenReturn(Optional.of(productoInactivo));

               // Act & Assert
               assertThatThrownBy(() -> productoService.obtenerPorId(2L))
                       .isInstanceOf(ProductoNotFoundException.class);
           }
       }

       // ==================== TESTS: crear() ====================

       @Nested
       @DisplayName("crear()")
       class CrearTests {

           @Test
           @DisplayName("Debe crear el producto exitosamente cuando el nombre no está duplicado")
           void debeCrearProductoExitosamente() {
               // Arrange
               Producto nuevoProducto = Producto.builder()
                       .nombre("Mouse Logitech")
                       .precio(new BigDecimal("49.99"))
                       .stock(20)
                       .categoria("ELECTRONICA")
                       .activo(true)
                       .build();

               Producto productoGuardado = Producto.builder()
                       .id(3L)
                       .nombre("Mouse Logitech")
                       .precio(new BigDecimal("49.99"))
                       .stock(20)
                       .categoria("ELECTRONICA")
                       .activo(true)
                       .build();

               when(productoRepository.existsByNombreIgnoreCaseAndActivoTrue("Mouse Logitech"))
                       .thenReturn(false);
               when(productoRepository.save(nuevoProducto)).thenReturn(productoGuardado);

               // Act
               Producto resultado = productoService.crear(nuevoProducto);

               // Assert
               assertThat(resultado).isNotNull();
               assertThat(resultado.getId()).isEqualTo(3L);
               assertThat(resultado.getNombre()).isEqualTo("Mouse Logitech");
               verify(productoRepository, times(1)).save(nuevoProducto);
           }

           @Test
           @DisplayName("Debe lanzar ProductoDuplicadoException cuando ya existe un producto con el mismo nombre")
           void debeLanzarExcepcionCuandoNombreEsDuplicado() {
               // Arrange
               Producto productoConNombreDuplicado = Producto.builder()
                       .nombre("Laptop Dell XPS")
                       .precio(new BigDecimal("999.00"))
                       .stock(5)
                       .categoria("ELECTRONICA")
                       .build();

               when(productoRepository.existsByNombreIgnoreCaseAndActivoTrue("Laptop Dell XPS"))
                       .thenReturn(true);

               // Act & Assert
               assertThatThrownBy(() -> productoService.crear(productoConNombreDuplicado))
                       .isInstanceOf(ProductoDuplicadoException.class)
                       .hasMessageContaining("Laptop Dell XPS");

               // Verificar que save NUNCA fue llamado cuando hay duplicado
               verify(productoRepository, never()).save(any(Producto.class));
           }
       }

       // ==================== TESTS: eliminar() ====================

       @Nested
       @DisplayName("eliminar()")
       class EliminarTests {

           @Test
           @DisplayName("Debe realizar baja lógica del producto cuando existe")
           void debeRealizarBajaLogicaDelProducto() {
               // Arrange
               when(productoRepository.findById(1L)).thenReturn(Optional.of(productoActivo));
               when(productoRepository.save(any(Producto.class))).thenReturn(productoActivo);

               // Act
               productoService.eliminar(1L);

               // Assert: verificar que el producto fue guardado con activo=false
               verify(productoRepository, times(1)).save(argThat(p -> !p.getActivo()));
           }

           @Test
           @DisplayName("Debe lanzar ProductoNotFoundException al intentar eliminar un producto inexistente")
           void debeLanzarExcepcionAlEliminarProductoInexistente() {
               // Arrange
               when(productoRepository.findById(999L)).thenReturn(Optional.empty());

               // Act & Assert
               assertThatThrownBy(() -> productoService.eliminar(999L))
                       .isInstanceOf(ProductoNotFoundException.class);

               verify(productoRepository, never()).save(any());
           }
       }

       // ==================== TESTS: buscarPorCategoria() ====================

       @Nested
       @DisplayName("buscarPorCategoria()")
       class BuscarPorCategoriaTests {

           @Test
           @DisplayName("Debe retornar productos de la categoría especificada")
           void debeRetornarProductosDeLaCategoria() {
               // Arrange
               List<Producto> electronicos = Arrays.asList(productoActivo);
               when(productoRepository.findByCategoriaAndActivoTrue("ELECTRONICA"))
                       .thenReturn(electronicos);

               // Act
               List<Producto> resultado = productoService.buscarPorCategoria("ELECTRONICA");

               // Assert
               assertThat(resultado).hasSize(1);
               assertThat(resultado.get(0).getCategoria()).isEqualTo("ELECTRONICA");
           }

           @Test
           @DisplayName("Debe retornar lista vacía cuando no hay productos en la categoría")
           void debeRetornarListaVaciaCuandoNoHayProductosEnCategoria() {
               // Arrange
               when(productoRepository.findByCategoriaAndActivoTrue("DEPORTES"))
                       .thenReturn(Collections.emptyList());

               // Act
               List<Producto> resultado = productoService.buscarPorCategoria("DEPORTES");

               // Assert
               assertThat(resultado).isEmpty();
           }
       }
   }
   ```

2. Ejecuta las pruebas unitarias:

   ```bash
   # En Linux/macOS
   ./mvnw test

   # En Windows (PowerShell)
   .\mvnw.cmd test
   ```

3. Verifica el reporte de pruebas en la terminal. Debes ver algo similar a:

   ```
   [INFO] Tests run: 10, Failures: 0, Errors: 0, Skipped: 0
   ```

4. También puedes ejecutar las pruebas desde IntelliJ haciendo clic derecho sobre la clase `ProductoServiceImplTest` → **Run 'ProductoServiceImplTest'**. Verás el árbol de pruebas con los nombres descriptivos definidos con `@DisplayName`.

5. Haz commit de las pruebas:

   ```bash
   git add src/test/
   git commit -m "test: agregar pruebas unitarias para ProductoServiceImpl con JUnit 5 y Mockito"
   ```

**Salida Esperada:**

```
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.empresa.inventario.service.ProductoServiceImplTest
[INFO] Tests run: 10, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.856 s
[INFO] -------------------------------------------------------
[INFO] BUILD SUCCESS
```

**Verificación:**

- [ ] Los 10 tests pasan exitosamente (0 Failures, 0 Errors)
- [ ] En IntelliJ, todos los tests aparecen en verde en el árbol de resultados
- [ ] El reporte en `target/surefire-reports/` contiene el archivo XML con los resultados

---

### Paso 8: Escribir Pruebas de Controlador con MockMvc

**Objetivo:** Implementar pruebas de integración ligera para el controlador REST usando MockMvc, verificando que los endpoints respondan correctamente a diferentes escenarios.

**Instrucciones:**

1. Crea la clase de prueba `ProductoControllerTest` en `src/test/java/com/empresa/inventario/controller/`:

   ```java
   package com.empresa.inventario.controller;

   import com.empresa.inventario.dto.request.CrearProductoRequest;
   import com.empresa.inventario.dto.response.ProductoResponse;
   import com.empresa.inventario.exception.ProductoNotFoundException;
   import com.empresa.inventario.facade.ProductoFacade;
   import com.fasterxml.jackson.databind.ObjectMapper;
   import org.junit.jupiter.api.BeforeEach;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Nested;
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
   import org.springframework.boot.test.mock.mockito.MockBean;
   import org.springframework.http.MediaType;
   import org.springframework.test.web.servlet.MockMvc;

   import java.math.BigDecimal;
   import java.time.LocalDateTime;
   import java.util.Arrays;
   import java.util.Collections;
   import java.util.List;

   import static org.mockito.ArgumentMatchers.*;
   import static org.mockito.Mockito.*;
   import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
   import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

   /**
    * Pruebas del controlador REST usando MockMvc.
    *
    * @WebMvcTest(ProductoController.class): levanta solo la capa web (sin base de datos)
    * @MockBean: crea un mock de Spring del Facade (reemplaza el bean real en el contexto)
    *
    * MockMvc permite simular peticiones HTTP y verificar respuestas
    * sin necesidad de levantar un servidor real.
    */
   @WebMvcTest(ProductoController.class)
   @DisplayName("Pruebas de Controlador: ProductoController")
   class ProductoControllerTest {

       @Autowired
       private MockMvc mockMvc;

       @MockBean
       private ProductoFacade productoFacade;

       @Autowired
       private ObjectMapper objectMapper;

       private ProductoResponse productoResponse1;
       private ProductoResponse productoResponse2;

       @BeforeEach
       void setUp() {
           productoResponse1 = ProductoResponse.builder()
                   .id(1L)
                   .nombre("Laptop Dell XPS")
                   .descripcion("Laptop profesional")
                   .precio(new BigDecimal("1299.99"))
                   .stock(10)
                   .categoria("ELECTRONICA")
                   .fechaCreacion(LocalDateTime.now())
                   .build();

           productoResponse2 = ProductoResponse.builder()
                   .id(2L)
                   .nombre("Teclado Mecánico")
                   .precio(new BigDecimal("89.99"))
                   .stock(5)
                   .categoria("ELECTRONICA")
                   .fechaCreacion(LocalDateTime.now())
                   .build();
       }

       // ==================== TESTS: GET /api/v1/productos ====================

       @Nested
       @DisplayName("GET /api/v1/productos")
       class GetTodosTests {

           @Test
           @DisplayName("Debe retornar 200 OK con lista de productos")
           void debeRetornar200ConListaDeProductos() throws Exception {
               // Arrange
               List<ProductoResponse> productos = Arrays.asList(productoResponse1, productoResponse2);
               when(productoFacade.obtenerTodos()).thenReturn(productos);

               // Act & Assert
               mockMvc.perform(get("/api/v1/productos")
                               .accept(MediaType.APPLICATION_JSON))
                       .andDo(print())
                       .andExpect(status().isOk())
                       .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                       .andExpect(jsonPath("$.length()").value(2))
                       .andExpect(jsonPath("$[0].id").value(1))
                       .andExpect(jsonPath("$[0].nombre").value("Laptop Dell XPS"))
                       .andExpect(jsonPath("$[1].nombre").value("Teclado Mecánico"));
           }

           @Test
           @DisplayName("Debe retornar 200 OK con lista vacía cuando no hay productos")
           void debeRetornar200ConListaVacia() throws Exception {
               // Arrange
               when(productoFacade.obtenerTodos()).thenReturn(Collections.emptyList());

               // Act & Assert
               mockMvc.perform(get("/api/v1/productos"))
                       .andExpect(status().isOk())
                       .andExpect(jsonPath("$.length()").value(0));
           }
       }

       // ==================== TESTS: GET /api/v1/productos/{id} ====================

       @Nested
       @DisplayName("GET /api/v1/productos/{id}")
       class GetPorIdTests {

           @Test
           @DisplayName("Debe retornar 200 OK con el producto cuando existe")
           void debeRetornar200CuandoProductoExiste() throws Exception {
               // Arrange
               when(productoFacade.obtenerPorId(1L)).thenReturn(productoResponse1);

               // Act & Assert
               mockMvc.perform(get("/api/v1/productos/1"))
                       .andExpect(status().isOk())
                       .andExpect(jsonPath("$.id").value(1))
                       .andExpect(jsonPath("$.nombre").value("Laptop Dell XPS"))
                       .andExpect(jsonPath("$.precio").value(1299.99));
           }

           @Test
           @DisplayName("Debe retornar 404 Not Found cuando el producto no existe")
           void debeRetornar404CuandoProductoNoExiste() throws Exception {
               // Arrange
               when(productoFacade.obtenerPorId(999L))
                       .thenThrow(new ProductoNotFoundException(999L));

               // Act & Assert
               mockMvc.perform(get("/api/v1/productos/999"))
                       .andExpect(status().isNotFound())
                       .andExpect(jsonPath("$.status").value(404))
                       .andExpect(jsonPath("$.mensaje").exists());
           }
       }

       // ==================== TESTS: POST /api/v1/productos ====================

       @Nested
       @DisplayName("POST /api/v1/productos")
       class PostCrearTests {

           @Test
           @DisplayName("Debe retornar 201 Created con el producto creado cuando los datos son válidos")
           void debeRetornar201CuandoDatosSonValidos() throws Exception {
               // Arrange
               CrearProductoRequest request = CrearProductoRequest.builder()
                       .nombre("Monitor 4K")
                       .descripcion("Monitor de alta resolución")
                       .precio(new BigDecimal("549.99"))
                       .stock(7)
                       .categoria("ELECTRONICA")
                       .build();

               ProductoResponse responseCreado = ProductoResponse.builder()
                       .id(3L)
                       .nombre("Monitor 4K")
                       .precio(new BigDecimal("549.99"))
                       .stock(7)
                       .categoria("ELECTRONICA")
                       .build();

               when(productoFacade.crear(any(CrearProductoRequest.class))).thenReturn(responseCreado);

               // Act & Assert
               mockMvc.perform(post("/api/v1/productos")
                               .contentType(MediaType.APPLICATION_JSON)
                               .content(objectMapper.writeValueAsString(request)))
                       .andExpect(status().isCreated())
                       .andExpect(jsonPath("$.id").value(3))
                       .andExpect(jsonPath("$.nombre").value("Monitor 4K"));
           }

           @Test
           @DisplayName("Debe retornar 400 Bad Request cuando el nombre está vacío")
           void debeRetornar400CuandoNombreEstaVacio() throws Exception {
               // Arrange: request con nombre vacío (inválido)
               CrearProductoRequest requestInvalido = CrearProductoRequest.builder()
                       .nombre("")  // Inválido: @NotBlank
                       .precio(new BigDecimal("100.00"))
                       .stock(5)
                       .categoria("ELECTRONICA")
                       .build();

               // Act & Assert
               mockMvc.perform(post("/api/v1/productos")
                               .contentType(MediaType.APPLICATION_JSON)
                               .content(objectMapper.writeValueAsString(requestInvalido)))
                       .andExpect(status().isBadRequest())
                       .andExpect(jsonPath("$.status").value(400))
                       .andExpect(jsonPath("$.detalles").isArray());
           }

           @Test
           @DisplayName("Debe retornar 400 Bad Request cuando el precio es negativo")
           void debeRetornar400CuandoPrecioEsNegativo() throws Exception {
               // Arrange
               CrearProductoRequest requestInvalido = CrearProductoRequest.builder()
                       .nombre("Producto Test")
                       .precio(new BigDecimal("-10.00"))  // Inválido: @DecimalMin("0.01")
                       .stock(5)
                       .categoria("ELECTRONICA")
                       .build();

               // Act & Assert
               mockMvc.perform(post("/api/v1/productos")
                               .contentType(MediaType.APPLICATION_JSON)
                               .content(objectMapper.writeValueAsString(requestInvalido)))
                       .andExpect(status().isBadRequest());
           }
       }

       // ==================== TESTS: DELETE /api/v1/productos/{id} ====================

       @Nested
       @DisplayName("DELETE /api/v1/productos/{id}")
       class DeleteTests {

           @Test
           @DisplayName("Debe retornar 204 No Content al eliminar un producto existente")
           void debeRetornar204AlEliminarProductoExistente() throws Exception {
               // Arrange
               doNothing().when(productoFacade).eliminar(1L);

               // Act & Assert
               mockMvc.perform(delete("/api/v1/productos/1"))
                       .andExpect(status().isNoContent());

               verify(productoFacade, times(1)).eliminar(1L);
           }

           @Test
           @DisplayName("Debe retornar 404 al intentar eliminar un producto inexistente")
           void debeRetornar404AlEliminarProductoInexistente() throws Exception {
               // Arrange
               doThrow(new ProductoNotFoundException(999L)).when(productoFacade).eliminar(999L);

               // Act & Assert
               mockMvc.perform(delete("/api/v1/productos/999"))
                       .andExpect(status().isNotFound());
           }
       }
   }
   ```

2. Ejecuta todas las pruebas (servicio + controlador):

   ```bash
   # En Linux/macOS
   ./mvnw test

   # En Windows (PowerShell)
   .\mvnw.cmd test
   ```

3. Verifica que todas las pruebas pasen:

   ```bash
   # Ver resumen de pruebas
   ./mvnw test | grep -E "Tests run:|BUILD"
   ```

4. Haz el commit final de las pruebas del controlador:

   ```bash
   git add src/test/
   git commit -m "test: agregar pruebas de controlador con MockMvc para ProductoController"
   git push origin main
   ```

**Salida Esperada:**

```
[INFO] Tests run: 10, Failures: 0, Errors: 0, Skipped: 0 -- ProductoServiceImplTest
[INFO] Tests run: 9, Failures: 0, Errors: 0, Skipped: 0  -- ProductoControllerTest
[INFO] Results:
[INFO] Tests run: 19, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS
```

**Verificación:**

- [ ] Los 19 tests (10 de servicio + 9 de controlador) pasan exitosamente
- [ ] Los tests de controlador usan `@WebMvcTest` (no levantan contexto completo)
- [ ] Los tests de controlador verifican códigos de estado HTTP correctos (200, 201, 204, 400, 404)
- [ ] `BUILD SUCCESS` al final de `./mvnw test`

---

### Paso 9: Pruebas Manuales con Postman y Empaquetado Final

**Objetivo:** Realizar pruebas manuales completas de la API con Postman, verificar todos los endpoints y generar el artefacto JAR ejecutable con Maven.

**Instrucciones:**

1. Inicia la aplicación:

   ```bash
   # En Linux/macOS
   ./mvnw spring-boot:run

   # En Windows (PowerShell)
   .\mvnw.cmd spring-boot:run
   ```

2. Abre Postman (o Thunder Client en VS Code) y realiza las siguientes pruebas. Crea una colección llamada **"Inventario API"** con las siguientes requests:

   **Request 1: Obtener todos los productos**
   - Method: `GET`
   - URL: `http://localhost:8080/api/v1/productos`
   - Expected: Status 200, array JSON con 5 productos

   **Request 2: Obtener producto por ID (existente)**
   - Method: `GET`
   - URL: `http://localhost:8080/api/v1/productos/1`
   - Expected: Status 200, objeto JSON del producto

   **Request 3: Obtener producto por ID (inexistente)**
   - Method: `GET`
   - URL: `http://localhost:8080/api/v1/productos/9999`
   - Expected: Status 404, JSON con campo `mensaje`

   **Request 4: Crear producto válido**
   - Method: `POST`
   - URL: `http://localhost:8080/api/v1/productos`
   - Headers: `Content-Type: application/json`
   - Body (raw JSON):
     ```json
     {
       "nombre": "Auriculares Sony WH-1000XM5",
       "descripcion": "Auriculares inalámbricos con cancelación de ruido",
       "precio": 349.99,
       "stock": 12,
       "categoria": "ELECTRONICA"
     }
     ```
   - Expected: Status 201, JSON con `id` asignado

   **Request 5: Crear producto con nombre duplicado**
   - Method: `POST`
   - URL: `http://localhost:8080/api/v1/productos`
   - Body: mismo JSON que Request 4
   - Expected: Status 409, JSON con mensaje de conflicto

   **Request 6: Crear producto con datos inválidos**
   - Method: `POST`
   - URL: `http://localhost:8080/api/v1/productos`
   - Body:
     ```json
     {
       "nombre": "",
       "precio": -5.00,
       "stock": -1,
       "categoria": ""
     }
     ```
   - Expected: Status 400, JSON con array `detalles` que lista los errores de validación

   **Request 7: Actualizar producto**
   - Method: `PUT`
   - URL: `http://localhost:8080/api/v1/productos/1`
   - Body:
     ```json
     {
       "precio": 1199.99,
       "stock": 20
     }
     ```
   - Expected: Status 200, JSON con precio y stock actualizados

   **Request 8: Buscar por categoría**
   - Method: `GET`
   - URL: `http://localhost:8080/api/v1/productos/buscar?categoria=ELECTRONICA`
   - Expected: Status 200, array con productos de categoría ELECTRONICA

   **Request 9: Obtener bajo stock**
   - Method: `GET`
   - URL: `http://localhost:8080/api/v1/productos/bajo-stock?umbral=10`
   - Expected: Status 200, array con productos cuyo stock < 10

   **Request 10: Eliminar producto**
   - Method: `DELETE`
   - URL: `http://localhost:8080/api/v1/productos/4`
   - Expected: Status 204 (sin body)

   **Request 11: Verificar eliminación lógica**
   - Method: `GET`
   - URL: `http://localhost:8080/api/v1/productos/4`
   - Expected: Status 404 (el producto fue desactivado, ya no es visible)

3. Accede a la consola H2 para inspeccionar los datos directamente en la base de datos:
   - Abre el navegador en: `http://localhost:8080/h2-console`
   - JDBC URL: `jdbc:h2:mem:inventariodb`
   - Username: `sa`
   - Password: (vacío)
   - Ejecuta: `SELECT * FROM PRODUCTOS;`

4. Detén la aplicación (`Ctrl+C`) y genera el artefacto JAR ejecutable:

   ```bash
   # En Linux/macOS
   ./mvnw clean package

   # En Windows (PowerShell)
   .\mvnw.cmd clean package
   ```

5. Verifica que el JAR fue generado correctamente:

   ```bash
   # En Linux/macOS
   ls -lh target/*.jar

   # En Windows (PowerShell)
   Get-ChildItem target\*.jar | Select-Object Name, Length
   ```

6. Ejecuta el JAR directamente para verificar que es autocontenido:

   ```bash
   java -jar target/inventario-api-1.0.0-SNAPSHOT.jar
   ```

7. Haz el commit y push final:

   ```bash
   # Detén la aplicación (Ctrl+C)
   git add .
   git commit -m "docs: agregar coleccion postman y completar laboratorio 3"
   git push origin main
   ```

**Salida Esperada:**

```
[INFO] Building jar: /ruta/al/proyecto/target/inventario-api-1.0.0-SNAPSHOT.jar
[INFO] BUILD SUCCESS
```

El archivo JAR debe tener un tamaño aproximado de 18-25 MB (incluye Tomcat embebido y todas las dependencias).

**Verificación:**

- [ ] Todas las 11 requests de Postman devuelven los códigos de estado esperados
- [ ] La consola H2 muestra los registros en la tabla `PRODUCTOS`
- [ ] El archivo `target/inventario-api-1.0.0-SNAPSHOT.jar` existe y tiene más de 15 MB
- [ ] `java -jar target/inventario-api-1.0.0-SNAPSHOT.jar` inicia la aplicación correctamente
- [ ] El repositorio GitHub tiene todos los commits del laboratorio

---

## Validación y Pruebas

### Criterios de Éxito

- [ ] El proyecto compila con `./mvnw clean compile` sin errores ni warnings críticos
- [ ] Los 19 tests automatizados pasan con `./mvnw test` (0 Failures, 0 Errors)
- [ ] La aplicación arranca correctamente con `./mvnw spring-boot:run` en el puerto 8080
- [ ] El endpoint `GET /api/v1/productos` devuelve los 5 productos iniciales en formato JSON
- [ ] El endpoint `POST /api/v1/productos` devuelve 201 con datos válidos y 400 con datos inválidos
- [ ] El endpoint `DELETE /api/v1/productos/{id}` realiza baja lógica (producto no aparece en GET posterior)
- [ ] Los errores devuelven respuestas JSON estandarizadas con el formato `ErrorResponse`
- [ ] El patrón Facade está implementado: el controlador solo interactúa con `ProductoFacade`, no con `ProductoService` directamente
- [ ] Los DTOs separan correctamente el modelo de dominio de la API (la entidad `Producto` nunca se expone directamente)
- [ ] El JAR ejecutable se genera con `./mvnw clean package`
- [ ] El repositorio GitHub contiene todos los commits del laboratorio

### Procedimiento de Pruebas

1. Ejecutar suite completa de pruebas automatizadas:
   ```bash
   ./mvnw clean test
   ```
   **Resultado Esperado:** `Tests run: 19, Failures: 0, Errors: 0, Skipped: 0`

2. Verificar arranque de la aplicación:
   ```bash
   ./mvnw spring-boot:run &
   sleep 10
   curl -s http://localhost:8080/api/v1/productos | python3 -m json.tool
   ```
   **Resultado Esperado:** Array JSON con 5 productos

3. Verificar manejo de errores:
   ```bash
   curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/api/v1/productos/9999
   ```
   **Resultado Esperado:** `404`

4. Verificar validaciones:
   ```bash
   curl -s -X POST http://localhost:8080/api/v1/productos \
     -H "Content-Type: application/json" \
     -d '{"nombre":"","precio":-1,"stock":-5,"categoria":""}' \
     | python3 -m json.tool
   ```
   **Resultado Esperado:** Status 400 con array `detalles` con mensajes de validación

5. Verificar empaquetado:
   ```bash
   ./mvnw clean package -DskipTests
   ls -lh target/inventario-api-1.0.0-SNAPSHOT.jar
   ```
   **Resultado Esperado:** Archivo JAR de más de 15 MB

---

## Solución de Problemas

### Problema 1: Error "Lombok annotations not processed" en IntelliJ

**Síntomas:**
- IntelliJ muestra errores en clases con `@Data`, `@Builder` o `@Slf4j`
- El mensaje de error dice `cannot find symbol: method getNombre()`
- El código compila con Maven desde la terminal pero no en IntelliJ

**Causa:**
El plugin de Lombok en IntelliJ no está habilitado o el procesamiento de anotaciones (Annotation Processing) está desactivado.

**Solución:**
```
1. File → Settings (Ctrl+Alt+S)
2. Plugins → buscar "Lombok" → verificar que esté instalado y habilitado
3. Build, Execution, Deployment → Compiler → Annotation Processors
4. Marcar "Enable annotation processing"
5. Reiniciar IntelliJ: File → Invalidate Caches → Invalidate and Restart
```

---

### Problema 2: Error al arrancar - "Port 8080 was already in use"

**Síntomas:**
- La aplicación no arranca
- El log muestra: `Web server failed to start. Port 8080 was already in use`

**Causa:**
Otra instancia de la aplicación (u otro proceso) ya está usando el puerto 8080.

**Solución:**
```bash
# En Linux/macOS: encontrar y matar el proceso en el puerto 8080
lsof -ti:8080 | xargs kill -9

# En Windows (PowerShell): encontrar y matar el proceso
netstat -ano | findstr :8080
# Anotar el PID de la última columna y ejecutar:
taskkill /PID <PID_ENCONTRADO> /F

# Alternativa: cambiar el puerto en application.properties
# server.port=8081
```

---

### Problema 3: Tests fallan con "No qualifying bean of type ProductoFacade"

**Síntomas:**
- Los tests de `ProductoControllerTest` fallan con error de contexto Spring
- Mensaje: `No qualifying bean of type 'com.empresa.inventario.facade.ProductoFacade'`

**Causa:**
`@WebMvcTest` solo carga la capa web. Si el `ProductoFacade` no está declarado como `@MockBean` en el test, Spring no lo encuentra.

**Solución:**
Verifica que la clase `ProductoControllerTest` tenga la anotación `@MockBean` sobre el campo `productoFacade`:

```java
// Esto DEBE estar presente en ProductoControllerTest
@MockBean
private ProductoFacade productoFacade;
```

Si el error persiste, verifica que `ProductoFacade` tenga la anotación `@Component` en la clase.

---

### Problema 4: Error de compilación "package jakarta.persistence does not exist"

**Síntomas:**
- Error de compilación en la clase `Producto`
- Mensaje: `package jakarta.persistence does not exist`

**Causa:**
Se está usando Spring Boot 2.x (que usa `javax.persistence`) en lugar de Spring Boot 3.x (que usa `jakarta.persistence`). O el `pom.xml` tiene una versión incorrecta.

**Solución:**
```bash
# Verificar la versión de Spring Boot en pom.xml
grep -A2 "spring-boot-starter-parent" pom.xml
# Debe mostrar versión 3.2.x

# Si es 2.x, actualizar a 3.2.5 en pom.xml y recargar Maven en IntelliJ
```

En el `pom.xml`, asegúrate de que el parent sea:
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.5</version>
</parent>
```

---

### Problema 5: H2 Console no accesible en `http://localhost:8080/h2-console`

**Síntomas:**
- El navegador muestra 404 o la página no carga
- La aplicación está corriendo correctamente

**Causa:**
La consola H2 no está habilitada en `application.properties` o Spring Security está bloqueando el acceso.

**Solución:**
Verifica que `application.properties` contenga:
```properties
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

Si el problema persiste con Spring Security (si lo agregaste), añade esta configuración en `config/`:
```java
// Solo si tienes Spring Security en el proyecto
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.csrf(csrf -> csrf.disable())
        .headers(headers -> headers.frameOptions(frame -> frame.disable()))
        .authorizeHttpRequests(auth -> auth.anyRequest().permitAll());
    return http.build();
}
```

---

## Limpieza

Al finalizar el laboratorio, detén todos los procesos y limpia los artefactos generados:

```bash
# 1. Detener la aplicación Spring Boot (si está corriendo)
# Presiona Ctrl+C en la terminal donde está ejecutándose

# 2. Limpiar artefactos Maven generados (directorio target/)
./mvnw clean
# En Windows: .\mvnw.cmd clean

# 3. Verificar que target/ fue eliminado
ls target/
# Debe mostrar: "No such file or directory" (Linux/macOS)
# En Windows: Get-ChildItem target\ (debe dar error si fue eliminado)

# 4. Verificar el estado final del repositorio Git
git status
# Debe mostrar: "nothing to commit, working tree clean"

git log --oneline -10
# Debe mostrar todos los commits del laboratorio
```

> ⚠️ **Advertencia:** No elimines el directorio del proyecto. El código generado en este laboratorio es la base del proyecto integrador que se usará en los laboratorios 4, 5, 6 y 7. Asegúrate de que el último estado esté commiteado y pusheado a GitHub antes de cerrar.

> ⚠️ **Nota sobre la base de datos H2:** La base de datos H2 en memoria se destruye automáticamente al detener la aplicación (configuración `create-drop`). Esto es intencional para el entorno de desarrollo. En el Laboratorio 4 se migrará a PostgreSQL con persistencia real.

---

## Resumen

### Lo que Lograste

- **Proyecto Spring Boot estructurado:** Generaste y configuraste un proyecto Spring Boot 3.2 con Maven desde Spring Initializr, estableciendo la estructura de paquetes por capas que seguirá el proyecto integrador en todos los laboratorios siguientes.

- **API REST completa:** Implementaste 7 endpoints REST con los verbos HTTP correctos (GET, POST, PUT, DELETE), códigos de estado apropiados (200, 201, 204, 400, 404, 409) y serialización JSON automática con Jackson.

- **Patrón DTO implementado:** Creaste DTOs de request (`CrearProductoRequest`, `ActualizarProductoRequest`) y response (`ProductoResponse`) que separan el modelo de dominio de la API pública, con un `ProductoMapper` centralizado para las conversiones.

- **Patrón Facade aplicado:** El `ProductoFacade` encapsula la orquestación entre el servicio y el mapper, simplificando el controlador y desacoplando las capas.

- **Manejo global de errores:** El `GlobalExceptionHandler` con `@RestControllerAdvice` centraliza el manejo de excepciones y garantiza respuestas de error consistentes en toda la API.

- **Pruebas unitarias obligatorias:** Escribiste 19 tests automatizados: 10 pruebas unitarias para `ProductoServiceImpl` con Mockito (simulando el repositorio sin base de datos) y 9 pruebas de controlador con MockMvc (verificando endpoints sin servidor real).

### Conceptos Clave

- **Maven y la estructura de directorios:** La separación `src/main/java` (código de producción) vs `src/test/java` (código de pruebas) garantiza que las clases de test nunca lleguen al artefacto final. El directorio `target/` se regenera con `mvn clean package` y nunca va a Git.

- **Convención sobre configuración en Spring Boot:** Spring Boot detecta automáticamente componentes (`@Service`, `@Repository`, `@RestController`) y configura Hibernate, Jackson y Tomcat sin XML de configuración.

- **Patrón DTO:** Los DTOs protegen el modelo de dominio de cambios en la API y viceversa. Si la entidad `Producto` cambia internamente, la API puede mantenerse estable modificando solo el mapper.

- **Patrón Facade:** Reduce el acoplamiento entre el controlador y múltiples servicios. El controlador solo conoce el Facade; si la lógica de negocio se vuelve más compleja, el controlador no cambia.

- **Testing con Mockito:** `@Mock` crea objetos simulados que no ejecutan código real. `when(...).thenReturn(...)` define el comportamiento esperado. `verify(...)` confirma que los métodos fueron llamados correctamente.

### Próximos Pasos

- **Laboratorio 4:** Migrar la base de datos de H2 en memoria a PostgreSQL con Docker, configurar Spring Data JPA con una base de datos real, y agregar transacciones ACID.
- **Laboratorio 5:** Implementar Spring Batch para procesamiento masivo de productos (importación desde CSV), con manejo de errores, reintentos y monitoreo de jobs.
- **Laboratorio 6:** Desarrollar el frontend con Web Components y Lit que consuma la API REST construida en este laboratorio.

---

## Recursos Adicionales

- [Spring Boot Reference Documentation](https://docs.spring.io/spring-boot/docs/3.2.x/reference/html/) — Documentación oficial de Spring Boot 3.2, incluyendo configuración de propiedades y autoconfiguración
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/) — Referencia completa de Spring Data JPA, métodos de query derivados y JPQL
- [Maven Getting Started Guide](https://maven.apache.org/guides/getting-started/index.html) — Guía oficial de Apache Maven con ejemplos de ciclo de vida y configuración de plugins
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/) — Documentación completa de JUnit 5 con anotaciones, aserciones y extensiones
- [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html) — Referencia de la API de Mockito con ejemplos de mocking, stubbing y verificación
- [MockMvc Reference](https://docs.spring.io/spring-framework/reference/testing/spring-mvc-test-framework.html) — Guía de Spring MVC Test Framework para pruebas de controladores REST
- [Lombok Features](https://projectlombok.org/features/) — Catálogo completo de anotaciones Lombok con ejemplos del código que generan
- [Bean Validation 3.0 Specification](https://jakarta.ee/specifications/bean-validation/3.0/) — Especificación completa de Jakarta Bean Validation con todas las restricciones disponibles
