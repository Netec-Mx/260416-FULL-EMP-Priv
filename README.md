# Full Stack Empresarial

Este curso intensivo tiene como objetivo desarrollar en los participantes las habilidades necesarias para diseñar, construir e integrar aplicaciones modernas de manera integral, abarcando desde la capa de presentación hasta el procesamiento de datos y su despliegue.

A lo largo del programa, se abordan fundamentos sólidos de programación, construcción de interfaces basadas en componentes, desarrollo de servicios backend estructurados, procesamiento de datos por lotes, manejo de bases de datos y prácticas de integración y despliegue.

El enfoque del curso es práctico y progresivo, permitiendo que los participantes consoliden conocimientos clave mediante ejercicios guiados, implementación de soluciones y escenarios similares a los del entorno profesional.



## Lista de laboratorios

### Capítulo 1

- [Laboratorio 1. JavaScript](Capitulo01/README.md#laboratorio-1-javascript)

En este laboratorio aplicarás las características fundamentales del JavaScript moderno (ES6+) para resolver problemas prácticos del mundo empresarial, implementarás una jerarquía de clases orientada a objetos que modela un sistema de empleados, explorarás conceptos equivalentes en Java 17 comparando ambos lenguajes, y configurarás un repositorio Git local conectado a GitHub que servirá como base para todos los laboratorios del curso.

Este laboratorio es el punto de partida del curso y establece las competencias técnicas esenciales que se utilizarán de forma acumulativa en los laboratorios posteriores. Al finalizar, tendrás un proyecto organizado con módulos ES, clases, operaciones asíncronas y control de versiones profesional.

- Duración estimada: 220 min

<br/>

### Capítulo 2

- [Laboratorio 2. Proyecto Integrado usando tecnologías frontend](Capitulo02/README.md#laboratorio-2-proyecto-integrado-usando-tecnologías-frontend)

En este laboratorio construirás una **aplicación web de página única (SPA)** que funciona como catálogo interactivo de productos empresariales. Comenzarás desde cero con HTML5 semántico y CSS3 avanzado, añadirás interactividad con JavaScript ES6+, refactorizarás la interfaz hacia Web Components nativos y finalmente migrarás los componentes a **Lit 3.x** con propiedades reactivas y Shadow DOM. Al finalizar, tendrás un proyecto frontend modular, probado con Vitest y listo para integrarse con el backend del Laboratorio 7.

- Duración estimada: 260 min

<br/>

### Capítulo 3

- [Laboratorio 3. Proyecto integrador](Capitulo03/README.md#laboratorio-3-proyecto-integrador)

En este laboratorio construirás el backend empresarial del proyecto integrador del curso: un sistema de gestión de inventario de productos. Partiendo desde cero con Spring Initializr, crearás un proyecto Spring Boot 3.2 correctamente estructurado con Maven, implementarás una API REST completa con arquitectura en capas (Controller → Service → Repository → Model), aplicarás los patrones de diseño Facade y DTO para desacoplar la lógica de negocio de la capa de presentación, y escribirás pruebas unitarias con JUnit 5 y Mockito que validen el comportamiento de tus componentes.

Este laboratorio representa el núcleo técnico del curso: todo lo que construyas aquí será la base sobre la que los laboratorios posteriores agregarán persistencia real con PostgreSQL y MongoDB, procesamiento por lotes con Spring Batch, y el frontend con Web Components y Lit. Es fundamental que lo completes con rigor, especialmente las pruebas unitarias, que son un entregable obligatorio.


- Duración estimada: 170 min


<br/>

### Capítulo 4

- [Laboratorio 4. Proyecto integrador](Capitulo04/README.md#laboratorio-4-proyecto-integrador)


En este laboratorio implementarás el módulo de procesamiento por lotes del proyecto integrador empresarial utilizando Spring Boot 3.2 y Spring Batch 5. Construirás un sistema completo de procesamiento batch que incluye un Job principal con tres Steps secuenciales (validación, procesamiento chunk-oriented y generación de reporte), un segundo Job con flujo condicional, políticas de tolerancia a fallos (skip y retry), persistencia del estado en PostgreSQL y endpoints REST para operar los jobs desde Postman.

Este laboratorio refleja patrones reales de arquitectura empresarial donde los sistemas necesitan procesar grandes volúmenes de datos de forma confiable, recuperarse de fallos parciales y ofrecer visibilidad del estado de ejecución a través de APIs. Al completarlo, habrás construido un módulo de producción que puede integrarse directamente en cualquier aplicación Spring Boot empresarial.

  - Duración estimada: 180 min

<br/>

### Capítulo 5

- [Laboratorio 5. Proyecto integrador](Capitulo05/README.md#laboratorio-5-proyecto-integrador)

En este laboratorio implementarás la capa de persistencia completa del proyecto integrador empresarial, trabajando simultáneamente con dos paradigmas de almacenamiento: el modelo relacional en PostgreSQL y el modelo documental en MongoDB. Diseñarás un esquema relacional normalizado con mínimo 5 tablas relacionadas, ejecutarás consultas SQL de complejidad creciente incluyendo JOINs, subconsultas y window functions, e implementarás transacciones explícitas con control ACID. En la parte NoSQL, modelarás colecciones MongoDB para datos no estructurados, ejecutarás operaciones CRUD con operadores avanzados y construirás pipelines de agregación. Al finalizar, integrarás ambas bases de datos en el proyecto Spring Boot demostrando la coexistencia de Spring Data JPA y Spring Data MongoDB en una misma aplicación.

Este laboratorio refleja un escenario real de arquitectura empresarial donde los sistemas modernos combinan bases de datos relacionales para transacciones críticas con bases de datos documentales para datos flexibles y logs de actividad.


- Duración estimada: 120 min

### Capítulo 6

- [Laboratorio 6. Proyecto integrador](Capitulo06/README.md#laboratorio-6-proyecto-integrador)


En este laboratorio contenerizarás el proyecto integrador completo —frontend y backend— utilizando Docker con builds multi-etapa optimizados, y orquestarás todos los servicios mediante Docker Compose. Además, configurarás un repositorio GitHub profesional con una estructura de ramas bien definida, mensajes de commit atómicos y documentación completa en el README.

Este laboratorio representa el punto de integración final del curso: toma todo lo construido en los laboratorios anteriores (API REST con Spring Boot, frontend con Lit/Web Components, persistencia en PostgreSQL y MongoDB) y lo empaqueta en un stack listo para despliegue en cualquier entorno, eliminando el clásico problema de "funciona en mi máquina" que Docker fue diseñado para resolver.

- Duración estimada: 80 min

<br/>

### Capítulo 7

- [Laboratorio 7. Proyecto integrador](Capitulo07/README.md#laboratorio-7-proyecto-integrador)


En este laboratorio integrador, el participante une todos los componentes desarrollados a lo largo del curso en una aplicación full stack cohesiva y lista para producción. Se conectará el frontend construido con Lit al backend RESTful (Node.js/Express), se configurará CORS, se verificará la persistencia de datos en base de datos, y se contenerizará toda la solución mediante Docker y Docker Compose. El laboratorio concluye con la validación end-to-end del sistema completo y un repaso de los conceptos clave del módulo.

- Duración estimada: 80 min

---



