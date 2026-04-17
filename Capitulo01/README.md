# JavaScript Moderno, POO, Java y Git

## Metadatos

| Propiedad | Valor |
|-----------|-------|
| **Duración** | 220 minutos |
| **Complejidad** | Intermedio |
| **Nivel Bloom** | Aplicar |
| **Tecnologías** | JavaScript ES6+, Node.js, Java 17, Git, GitHub, VS Code |

---

## Descripción General

En este laboratorio aplicarás las características fundamentales del JavaScript moderno (ES6+) para resolver problemas prácticos del mundo empresarial, implementarás una jerarquía de clases orientada a objetos que modela un sistema de empleados, explorarás conceptos equivalentes en Java 17 comparando ambos lenguajes, y configurarás un repositorio Git local conectado a GitHub que servirá como base para todos los laboratorios del curso.

Este laboratorio es el punto de partida del curso y establece las competencias técnicas esenciales que se utilizarán de forma acumulativa en los laboratorios posteriores. Al finalizar, tendrás un proyecto organizado con módulos ES, clases, operaciones asíncronas y control de versiones profesional.

---

## Objetivos de Aprendizaje

Al completar este laboratorio, serás capaz de:

- [ ] Aplicar `let`, `const`, arrow functions, desestructuración, spread/rest y template literals en código funcional real
- [ ] Implementar Promises y `async/await` para simular y manejar operaciones asíncronas con manejo de errores
- [ ] Organizar código JavaScript en módulos ES usando `import` y `export` nombrados y por defecto
- [ ] Diseñar e implementar una jerarquía de clases JavaScript con herencia, encapsulamiento y polimorfismo
- [ ] Contrastar estructuras de datos y paradigmas entre JavaScript y Java 17 mediante ejemplos paralelos
- [ ] Configurar un repositorio Git local con ramas, commits descriptivos y push a GitHub

---

## Prerrequisitos

### Conocimientos Requeridos

- Conocimientos básicos de programación: variables, condicionales, bucles y funciones en cualquier lenguaje
- Comprensión de lógica de programación y estructuras de control (if/else, for, while)
- Manejo básico de línea de comandos: navegar directorios, crear archivos, ejecutar comandos
- Lectura del material teórico de la Lección 1.1: JavaScript Moderno

### Acceso y Herramientas Requeridas

- Node.js 20.x LTS instalado y accesible desde terminal (`node --version`)
- JDK 17 instalado con `JAVA_HOME` configurado (`java --version`)
- Git 2.44+ instalado (`git --version`)
- Cuenta activa en GitHub con acceso a crear repositorios
- VS Code 1.88+ instalado con extensión **JavaScript (ES6) code snippets**
- Conexión a Internet para crear repositorio remoto en GitHub

---

## Entorno de Laboratorio

### Requisitos de Hardware

| Componente | Especificación Mínima |
|------------|----------------------|
| Procesador | Intel Core i5 8va gen / AMD Ryzen 5 o superior |
| Memoria RAM | 8 GB DDR4 (16 GB recomendado) |
| Almacenamiento | 2 GB libres en disco SSD |
| Pantalla | Resolución 1366x768 mínimo (1920x1080 recomendado) |
| Conexión a Internet | 5 Mbps o superior |

### Requisitos de Software

| Software | Versión | Propósito |
|----------|---------|-----------|
| Node.js | 20.x LTS | Ejecutar scripts JavaScript en terminal |
| JDK | 17 LTS | Compilar y ejecutar código Java |
| Git | 2.44+ | Control de versiones local y remoto |
| VS Code | 1.88+ | Editor principal para JavaScript |
| Terminal | Bash / PowerShell / Zsh | Ejecutar comandos del laboratorio |

### Configuración Inicial

Verifica que todas las herramientas estén instaladas correctamente antes de comenzar:

```bash
# Verificar Node.js
node --version
# Resultado esperado: v20.x.x

# Verificar npm
npm --version
# Resultado esperado: 10.x.x

# Verificar Java
java --version
# Resultado esperado: openjdk 17.x.x

# Verificar javac (compilador)
javac --version
# Resultado esperado: javac 17.x.x

# Verificar Git
git --version
# Resultado esperado: git version 2.44.x
```

Si alguna herramienta no responde correctamente, detente y resuelve la instalación antes de continuar.

---

## Instrucciones Paso a Paso

### Paso 1: Crear la Estructura del Proyecto

**Objetivo:** Establecer la estructura de directorios del proyecto que se usará durante todo el laboratorio y configurar el entorno de Node.js.

**Instrucciones:**

1. Abre una terminal (PowerShell en Windows, Terminal en macOS/Linux) y crea el directorio raíz del proyecto:

   ```bash
   # En Windows (PowerShell)
   mkdir C:\curso-fullstack\lab-01
   cd C:\curso-fullstack\lab-01

   # En macOS / Linux
   mkdir -p ~/curso-fullstack/lab-01
   cd ~/curso-fullstack/lab-01
   ```

2. Crea la estructura de subdirectorios del proyecto:

   ```bash
   # En Windows (PowerShell)
   mkdir src
   mkdir src\js-moderno
   mkdir src\poo
   mkdir src\java-comparacion
   New-Item src\js-moderno\.gitkeep -ItemType File
   New-Item src\poo\.gitkeep -ItemType File
   New-Item src\java-comparacion\.gitkeep -ItemType File

   # En macOS / Linux
   mkdir -p src/js-moderno src/poo src/java-comparacion
   touch src/js-moderno/.gitkeep src/poo/.gitkeep src/java-comparacion/.gitkeep
   ```

3. Inicializa el proyecto Node.js:

   ```bash
   npm init -y
   ```

4. Abre el archivo `package.json` generado y agrega la configuración de módulos ES. Edita el archivo con VS Code:

   ```bash
   code package.json
   ```

   Reemplaza el contenido completo con:

   ```json
   {
     "name": "lab-01-javascript-moderno",
     "version": "1.0.0",
     "description": "Laboratorio 1: JavaScript Moderno, POO, Java y Git",
     "type": "module",
     "scripts": {
       "js-moderno": "node src/js-moderno/index.js",
       "poo": "node src/poo/index.js",
       "test": "echo \"Sin tests configurados aún\""
     },
     "keywords": ["javascript", "es6", "poo", "git"],
     "author": "Participante del Curso",
     "license": "ISC"
   }
   ```

   > ⚠️ **Importante:** La línea `"type": "module"` es esencial. Le indica a Node.js que todos los archivos `.js` del proyecto usan la sintaxis de módulos ES (`import`/`export`) en lugar de CommonJS (`require`).

5. Abre VS Code en el directorio del proyecto:

   ```bash
   code .
   ```

**Resultado Esperado:**

```
lab-01/
├── package.json
└── src/
    ├── js-moderno/
    │   └── .gitkeep
    ├── poo/
    │   └── .gitkeep
    └── java-comparacion/
        └── .gitkeep
```

**Verificación:**

- Ejecuta `cat package.json` (macOS/Linux) o `type package.json` (Windows) y confirma que aparece `"type": "module"`.
- Confirma que la estructura de directorios existe con `ls -la src/` (macOS/Linux) o `dir src\` (Windows).

---

### Paso 2: Aplicar `let`, `const`, Arrow Functions y Template Literals

**Objetivo:** Escribir código JavaScript ES6+ funcional que resuelva un problema empresarial real utilizando las características modernas de declaración de variables, funciones flecha y cadenas de texto enriquecidas.

**Instrucciones:**

1. Crea el archivo principal de esta sección:

   ```bash
   # En macOS / Linux
   touch src/js-moderno/variables-y-funciones.js

   # En Windows (PowerShell)
   New-Item src\js-moderno\variables-y-funciones.js -ItemType File
   ```

2. Abre el archivo en VS Code y escribe el siguiente código completo:

   ```javascript
   // src/js-moderno/variables-y-funciones.js
   // ============================================================
   // PARTE 1: let, const y alcance de bloque
   // ============================================================

   console.log("=== SECCIÓN 1: let, const y alcance de bloque ===\n");

   // const para valores que no cambian
   const IMPUESTO = 0.16;
   const MONEDA = "MXN";

   // let para valores que pueden cambiar
   let inventarioDisponible = 150;

   // Demostración de alcance de bloque con let
   for (let i = 0; i < 3; i++) {
     const mensajeIteracion = `Iteración ${i} de 3`; // const dentro del bloque
     console.log(mensajeIteracion);
   }
   // 'i' y 'mensajeIteracion' NO son accesibles aquí

   // const con objetos: la referencia es constante, no el contenido
   const producto = {
     id: 1,
     nombre: "Laptop Empresarial",
     precio: 15000
   };

   producto.precio = 14500; // ✅ Permitido: modificamos propiedad
   console.log(`\nProducto actualizado: ${producto.nombre} - $${producto.precio} ${MONEDA}`);

   // ============================================================
   // PARTE 2: Arrow Functions
   // ============================================================

   console.log("\n=== SECCIÓN 2: Arrow Functions ===\n");

   // Arrow function con retorno implícito (una expresión)
   const calcularImpuesto = (precio) => precio * IMPUESTO;

   // Arrow function con cuerpo de bloque
   const calcularPrecioFinal = (precio, descuentoPorcentaje = 0) => {
     const descuento = precio * (descuentoPorcentaje / 100);
     const precioConDescuento = precio - descuento;
     const impuesto = calcularImpuesto(precioConDescuento);
     return precioConDescuento + impuesto;
   };

   // Arrow functions con métodos de array
   const precios = [15000, 8500, 3200, 25000, 1100];

   const preciosFiltrados = precios.filter(p => p > 5000);
   const preciosConImpuesto = preciosFiltrados.map(p => calcularPrecioFinal(p));
   const totalInventario = preciosConImpuesto.reduce((suma, p) => suma + p, 0);

   console.log("Precios originales:", precios);
   console.log("Precios mayores a $5,000:", preciosFiltrados);
   console.log("Precios con impuesto:", preciosConImpuesto.map(p => p.toFixed(2)));
   console.log(`Total de inventario (con impuesto): $${totalInventario.toFixed(2)} ${MONEDA}`);

   // ============================================================
   // PARTE 3: Template Literals
   // ============================================================

   console.log("\n=== SECCIÓN 3: Template Literals ===\n");

   const generarReporteProducto = (prod, descuento = 0) => {
     const precioFinal = calcularPrecioFinal(prod.precio, descuento);
     const ahorros = prod.precio - (prod.precio * (1 - descuento / 100));

     // Template literal multilínea con expresiones
     return `
   ┌─────────────────────────────────────────┐
   │ REPORTE DE PRODUCTO                     │
   ├─────────────────────────────────────────┤
   │ ID:              ${String(prod.id).padEnd(24)}│
   │ Nombre:          ${prod.nombre.substring(0, 23).padEnd(24)}│
   │ Precio base:     $${String(prod.precio.toFixed(2)).padEnd(23)}│
   │ Descuento:       ${String(descuento + "%").padEnd(24)}│
   │ Ahorro:          $${String(ahorros.toFixed(2)).padEnd(23)}│
   │ Precio final:    $${String(precioFinal.toFixed(2)).padEnd(23)}│
   └─────────────────────────────────────────┘`;
   };

   console.log(generarReporteProducto(producto, 10));

   // Exportar funciones para uso en otros módulos
   export { calcularImpuesto, calcularPrecioFinal, generarReporteProducto };
   ```

3. Ejecuta el archivo para verificar que funciona correctamente:

   ```bash
   node src/js-moderno/variables-y-funciones.js
   ```

**Resultado Esperado:**

```
=== SECCIÓN 1: let, const y alcance de bloque ===

Iteración 0 de 3
Iteración 1 de 3
Iteración 2 de 3

Producto actualizado: Laptop Empresarial - $14500 MXN

=== SECCIÓN 2: Arrow Functions ===

Precios originales: [ 15000, 8500, 3200, 25000, 1100 ]
Precios mayores a $5,000: [ 15000, 8500, 25000 ]
Precios con impuesto: [ '17400.00', '9860.00', '29000.00' ]
Total de inventario (con impuesto): $56260.00 MXN

=== SECCIÓN 3: Template Literals ===

┌─────────────────────────────────────────┐
│ REPORTE DE PRODUCTO                     │
├─────────────────────────────────────────┤
│ ID:              1                       │
│ Nombre:          Laptop Empresarial      │
│ Precio base:     $14500.00               │
│ Descuento:       10%                     │
│ Ahorro:          $1450.00                │
│ Precio final:    $15138.00               │
└─────────────────────────────────────────┘
```

**Verificación:**

- Confirma que no hay errores de `SyntaxError` en la consola.
- Verifica que los cálculos de impuesto (16%) son correctos: $15,000 con 10% de descuento = $13,500 + $2,160 impuesto = $15,660. Los valores pueden variar ligeramente según el precio base actualizado en el código.
- Confirma que el reporte multilínea se muestra formateado correctamente.

---

### Paso 3: Implementar Desestructuración, Spread y Rest

**Objetivo:** Aplicar los operadores de desestructuración, spread y rest para manipular objetos y arrays de forma expresiva en el contexto de un sistema de inventario empresarial.

**Instrucciones:**

1. Crea el archivo para esta sección:

   ```bash
   # En macOS / Linux
   touch src/js-moderno/destructuring-spread-rest.js

   # En Windows (PowerShell)
   New-Item src\js-moderno\destructuring-spread-rest.js -ItemType File
   ```

2. Escribe el siguiente código en el archivo:

   ```javascript
   // src/js-moderno/destructuring-spread-rest.js
   // ============================================================
   // PARTE 1: Desestructuración de Objetos
   // ============================================================

   console.log("=== SECCIÓN 1: Desestructuración de Objetos ===\n");

   const empleado = {
     id: 101,
     nombre: "María González",
     departamento: "Tecnología",
     salario: 45000,
     contacto: {
       email: "maria.gonzalez@empresa.com",
       telefono: "555-0100"
     },
     habilidades: ["JavaScript", "React", "Node.js"]
   };

   // Desestructuración básica
   const { id, nombre, departamento } = empleado;
   console.log(`Empleado #${id}: ${nombre} - ${departamento}`);

   // Desestructuración con alias
   const { salario: sueldoMensual, departamento: area } = empleado;
   console.log(`Área: ${area} | Sueldo: $${sueldoMensual}`);

   // Desestructuración anidada
   const { contacto: { email }, nombre: nombreEmpleado } = empleado;
   console.log(`Contacto de ${nombreEmpleado}: ${email}`);

   // Desestructuración con valores por defecto
   const { puesto = "Sin asignar", nivel = 1 } = empleado;
   console.log(`Puesto: ${puesto} | Nivel: ${nivel}`);

   // ============================================================
   // PARTE 2: Desestructuración de Arrays
   // ============================================================

   console.log("\n=== SECCIÓN 2: Desestructuración de Arrays ===\n");

   const { habilidades } = empleado;
   const [habilidadPrincipal, habilidadSecundaria, ...otrasHabilidades] = habilidades;

   console.log(`Habilidad principal: ${habilidadPrincipal}`);
   console.log(`Habilidad secundaria: ${habilidadSecundaria}`);
   console.log(`Otras habilidades: ${otrasHabilidades.join(", ")}`);

   // Intercambiar variables sin variable temporal
   let valorA = 100;
   let valorB = 200;
   console.log(`\nAntes del intercambio: A=${valorA}, B=${valorB}`);
   [valorA, valorB] = [valorB, valorA];
   console.log(`Después del intercambio: A=${valorA}, B=${valorB}`);

   // ============================================================
   // PARTE 3: Operador Spread
   // ============================================================

   console.log("\n=== SECCIÓN 3: Operador Spread ===\n");

   const equipoDesarrollo = ["María", "Carlos", "Ana"];
   const equipoQA = ["Pedro", "Laura"];
   const equipoCompleto = [...equipoDesarrollo, ...equipoQA, "Roberto"];

   console.log("Equipo de Desarrollo:", equipoDesarrollo);
   console.log("Equipo de QA:", equipoQA);
   console.log("Equipo Completo:", equipoCompleto);

   // Spread para copiar y actualizar objetos (inmutabilidad)
   const empleadoActualizado = {
     ...empleado,
     salario: 50000,
     puesto: "Senior Developer",
     fechaActualizacion: new Date().toISOString().split("T")[0]
   };

   console.log(`\nSalario original: $${empleado.salario}`);
   console.log(`Salario actualizado: $${empleadoActualizado.salario}`);
   console.log(`Nuevo puesto: ${empleadoActualizado.puesto}`);
   console.log(`Fecha de actualización: ${empleadoActualizado.fechaActualizacion}`);

   // ============================================================
   // PARTE 4: Operador Rest en Funciones
   // ============================================================

   console.log("\n=== SECCIÓN 4: Operador Rest ===\n");

   // Rest para capturar argumentos variables
   const calcularBonificacion = (empleadoNombre, salarioBase, ...factores) => {
     const sumFactores = factores.reduce((suma, f) => suma + f, 0);
     const promedio = sumFactores / factores.length;
     const bonificacion = salarioBase * (promedio / 100);
     return {
       empleado: empleadoNombre,
       factores,
       promedioPorcentaje: promedio.toFixed(2),
       bonificacion: bonificacion.toFixed(2)
     };
   };

   const resultado = calcularBonificacion("María González", 50000, 15, 10, 20, 5);
   console.log("Cálculo de bonificación:");
   console.log(`  Empleado: ${resultado.empleado}`);
   console.log(`  Factores evaluados: [${resultado.factores.join(", ")}]`);
   console.log(`  Porcentaje promedio: ${resultado.promedioPorcentaje}%`);
   console.log(`  Bonificación: $${resultado.bonificacion}`);

   // Desestructuración en parámetros de función
   const formatearEmpleado = ({ nombre, departamento, salario, puesto = "Empleado" }) => {
     return `[${puesto}] ${nombre} | ${departamento} | $${salario.toLocaleString("es-MX")}`;
   };

   console.log("\nFormato de empleado:");
   console.log(formatearEmpleado(empleadoActualizado));

   export { calcularBonificacion, formatearEmpleado };
   ```

3. Ejecuta el archivo:

   ```bash
   node src/js-moderno/destructuring-spread-rest.js
   ```

**Resultado Esperado:**

```
=== SECCIÓN 1: Desestructuración de Objetos ===

Empleado #101: María González - Tecnología
Área: Tecnología | Sueldo: $45000
Contacto de María González: maria.gonzalez@empresa.com
Puesto: Sin asignar | Nivel: 1

=== SECCIÓN 2: Desestructuración de Arrays ===

Habilidad principal: JavaScript
Habilidad secundaria: React
Otras habilidades: Node.js

Antes del intercambio: A=100, B=200
Después del intercambio: A=200, B=100

=== SECCIÓN 3: Operador Spread ===

Equipo de Desarrollo: [ 'María', 'Carlos', 'Ana' ]
Equipo de QA: [ 'Pedro', 'Laura' ]
Equipo Completo: [ 'María', 'Carlos', 'Ana', 'Pedro', 'Laura', 'Roberto' ]

Salario original: $45000
Salario actualizado: $50000
Nuevo puesto: Senior Developer
Fecha de actualización: 2024-xx-xx

=== SECCIÓN 4: Operador Rest ===

Cálculo de bonificación:
  Empleado: María González
  Factores evaluados: [15, 10, 20, 5]
  Porcentaje promedio: 12.50%
  Bonificación: $6250.00

Formato de empleado:
[Senior Developer] María González | Tecnología | $50,000
```

**Verificación:**

- Confirma que el intercambio de variables `A` y `B` funciona correctamente sin variable temporal.
- Verifica que el objeto `empleado` original no fue modificado (el salario sigue siendo $45,000 mientras que `empleadoActualizado` tiene $50,000).
- Comprueba que el cálculo de bonificación con múltiples factores es correcto: (15+10+20+5)/4 = 12.5%.

---

### Paso 4: Implementar Promesas y `async/await`

**Objetivo:** Crear funciones asíncronas que simulen operaciones de base de datos y API REST, implementando manejo de errores robusto con `try/catch` y ejecución paralela con `Promise.all`.

**Instrucciones:**

1. Crea el archivo para manejo asíncrono:

   ```bash
   # En macOS / Linux
   touch src/js-moderno/async-await.js

   # En Windows (PowerShell)
   New-Item src\js-moderno\async-await.js -ItemType File
   ```

2. Escribe el siguiente código:

   ```javascript
   // src/js-moderno/async-await.js
   // ============================================================
   // Simulación de base de datos en memoria
   // ============================================================

   const baseDeDatos = {
     empleados: [
       { id: 1, nombre: "Ana Torres", departamentoId: 10, activo: true },
       { id: 2, nombre: "Luis Ramírez", departamentoId: 20, activo: true },
       { id: 3, nombre: "Carmen López", departamentoId: 10, activo: false },
       { id: 4, nombre: "Roberto Silva", departamentoId: 30, activo: true }
     ],
     departamentos: [
       { id: 10, nombre: "Tecnología", presupuesto: 500000 },
       { id: 20, nombre: "Recursos Humanos", presupuesto: 200000 },
       { id: 30, nombre: "Finanzas", presupuesto: 350000 }
     ]
   };

   // ============================================================
   // PARTE 1: Funciones que devuelven Promesas
   // ============================================================

   console.log("=== SECCIÓN 1: Promesas básicas ===\n");

   // Simula una consulta a base de datos con latencia
   const buscarEmpleadoPorId = (id) => {
     return new Promise((resolve, reject) => {
       setTimeout(() => {
         const empleado = baseDeDatos.empleados.find(e => e.id === id);
         if (empleado) {
           resolve(empleado);
         } else {
           reject(new Error(`Empleado con ID ${id} no encontrado`));
         }
       }, 200); // Simula 200ms de latencia de red/BD
     });
   };

   const buscarDepartamentoPorId = (id) => {
     return new Promise((resolve, reject) => {
       setTimeout(() => {
         const departamento = baseDeDatos.departamentos.find(d => d.id === id);
         if (departamento) {
           resolve(departamento);
         } else {
           reject(new Error(`Departamento con ID ${id} no encontrado`));
         }
       }, 150);
     });
   };

   // Consumir promesa con .then() / .catch()
   buscarEmpleadoPorId(1)
     .then(empleado => {
       console.log("✅ Empleado encontrado (con .then):", empleado.nombre);
     })
     .catch(error => {
       console.error("❌ Error:", error.message);
     });

   // ============================================================
   // PARTE 2: async/await con manejo de errores
   // ============================================================

   console.log("=== SECCIÓN 2: async/await ===\n");

   const obtenerEmpleadoConDepartamento = async (empleadoId) => {
     try {
       // Operaciones secuenciales (la segunda depende de la primera)
       const empleado = await buscarEmpleadoPorId(empleadoId);
       const departamento = await buscarDepartamentoPorId(empleado.departamentoId);

       return {
         ...empleado,
         departamento: departamento.nombre,
         presupuestoDepartamento: departamento.presupuesto
       };
     } catch (error) {
       console.error(`❌ Error al obtener empleado ${empleadoId}:`, error.message);
       throw error; // Propagar el error para que el llamador lo maneje
     }
   };

   const mostrarEmpleadoConDepartamento = async (id) => {
     try {
       const resultado = await obtenerEmpleadoConDepartamento(id);
       console.log(`✅ Empleado completo:`);
       console.log(`   Nombre: ${resultado.nombre}`);
       console.log(`   Departamento: ${resultado.departamento}`);
       console.log(`   Presupuesto del área: $${resultado.presupuestoDepartamento.toLocaleString("es-MX")}`);
       console.log(`   Estado: ${resultado.activo ? "Activo" : "Inactivo"}`);
     } catch (error) {
       console.error(`No se pudo mostrar el empleado ${id}`);
     }
   };

   // ============================================================
   // PARTE 3: Promise.all para operaciones paralelas
   // ============================================================

   const cargarDashboard = async () => {
     console.log("\n=== SECCIÓN 3: Promise.all (ejecución paralela) ===\n");
     console.log("⏳ Cargando datos del dashboard en paralelo...");

     const inicio = Date.now();

     try {
       // Ejecutar múltiples consultas independientes en paralelo
       const [empleado1, empleado2, empleado4] = await Promise.all([
         obtenerEmpleadoConDepartamento(1),
         obtenerEmpleadoConDepartamento(2),
         obtenerEmpleadoConDepartamento(4)
       ]);

       const tiempoTotal = Date.now() - inicio;

       console.log(`✅ Dashboard cargado en ${tiempoTotal}ms (paralelo)`);
       console.log("\nEmpleados activos:");
       [empleado1, empleado2, empleado4].forEach(emp => {
         if (emp.activo) {
           console.log(`  - ${emp.nombre} (${emp.departamento})`);
         }
       });

     } catch (error) {
       console.error("❌ Error al cargar el dashboard:", error.message);
     }
   };

   // ============================================================
   // PARTE 4: Manejo de errores con Promise.allSettled
   // ============================================================

   const cargarConManejoDeErrores = async () => {
     console.log("\n=== SECCIÓN 4: Promise.allSettled (tolerante a errores) ===\n");

     // IDs donde el 99 no existe (causará un error)
     const idsABuscar = [1, 99, 4];

     const resultados = await Promise.allSettled(
       idsABuscar.map(id => obtenerEmpleadoConDepartamento(id))
     );

     resultados.forEach((resultado, index) => {
       const id = idsABuscar[index];
       if (resultado.status === "fulfilled") {
         console.log(`✅ ID ${id}: ${resultado.value.nombre} - ${resultado.value.departamento}`);
       } else {
         console.log(`❌ ID ${id}: ${resultado.reason.message}`);
       }
     });
   };

   // Ejecutar todas las demostraciones en secuencia
   const ejecutarDemostraciones = async () => {
     await mostrarEmpleadoConDepartamento(2);
     await cargarDashboard();
     await cargarConManejoDeErrores();
     console.log("\n✅ Todas las demostraciones completadas");
   };

   ejecutarDemostraciones();

   export { buscarEmpleadoPorId, buscarDepartamentoPorId, obtenerEmpleadoConDepartamento };
   ```

3. Ejecuta el archivo:

   ```bash
   node src/js-moderno/async-await.js
   ```

**Resultado Esperado:**

```
=== SECCIÓN 1: Promesas básicas ===

=== SECCIÓN 2: async/await ===

✅ Empleado encontrado (con .then): Ana Torres
✅ Empleado completo:
   Nombre: Luis Ramírez
   Departamento: Recursos Humanos
   Presupuesto del área: $200,000
   Estado: Activo

=== SECCIÓN 3: Promise.all (ejecución paralela) ===

⏳ Cargando datos del dashboard en paralelo...
✅ Dashboard cargado en ~350ms (paralelo)

Empleados activos:
  - Ana Torres (Tecnología)
  - Luis Ramírez (Recursos Humanos)
  - Roberto Silva (Finanzas)

=== SECCIÓN 4: Promise.allSettled (tolerante a errores) ===

✅ ID 1: Ana Torres - Tecnología
❌ ID 99: Empleado con ID 99 no encontrado
✅ ID 4: Roberto Silva - Finanzas

✅ Todas las demostraciones completadas
```

**Verificación:**

- Confirma que el tiempo de `Promise.all` es aproximadamente 350ms (no 600ms que sería secuencial), demostrando la ejecución paralela.
- Verifica que `Promise.allSettled` no falla aunque uno de los IDs no exista (a diferencia de `Promise.all` que fallaría completamente).
- Comprueba que el manejo de errores con `try/catch` funciona correctamente para el ID inexistente (99).

---

### Paso 5: Organizar Código con Módulos ES

**Objetivo:** Crear un sistema de módulos ES bien organizado que integre las funciones creadas en los pasos anteriores, demostrando exportaciones nombradas, exportaciones por defecto e importaciones selectivas.

**Instrucciones:**

1. Crea el módulo de utilidades compartidas:

   ```bash
   # En macOS / Linux
   touch src/js-moderno/utilidades.js

   # En Windows (PowerShell)
   New-Item src\js-moderno\utilidades.js -ItemType File
   ```

2. Escribe el contenido del módulo de utilidades:

   ```javascript
   // src/js-moderno/utilidades.js
   // Módulo de utilidades compartidas - exportaciones nombradas

   // Constantes exportadas
   export const MONEDA = "MXN";
   export const IMPUESTO_DEFAULT = 0.16;
   export const VERSION_APP = "1.0.0";

   // Funciones de formato
   export const formatearMoneda = (cantidad, moneda = MONEDA) => {
     return new Intl.NumberFormat("es-MX", {
       style: "currency",
       currency: moneda
     }).format(cantidad);
   };

   export const formatearFecha = (fecha = new Date()) => {
     return new Intl.DateTimeFormat("es-MX", {
       year: "numeric",
       month: "long",
       day: "numeric"
     }).format(fecha);
   };

   // Función de validación
   export const validarEmail = (email) => {
     const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
     return regex.test(email);
   };

   export const validarSalario = (salario) => {
     return typeof salario === "number" && salario > 0 && salario <= 1000000;
   };

   // Función de logging con timestamp
   export const log = (nivel, mensaje, datos = null) => {
     const timestamp = new Date().toISOString();
     const niveles = { INFO: "ℹ️", WARN: "⚠️", ERROR: "❌", SUCCESS: "✅" };
     const icono = niveles[nivel] || "📝";
     console.log(`[${timestamp}] ${icono} [${nivel}] ${mensaje}`);
     if (datos) console.log("  Datos:", datos);
   };
   ```

3. Crea el módulo del servicio de empleados:

   ```bash
   # En macOS / Linux
   touch src/js-moderno/ServicioEmpleados.js

   # En Windows (PowerShell)
   New-Item src\js-moderno\ServicioEmpleados.js -ItemType File
   ```

4. Escribe el servicio con exportación por defecto:

   ```javascript
   // src/js-moderno/ServicioEmpleados.js
   // Módulo con exportación por defecto (clase de servicio)

   import { validarEmail, validarSalario, log, formatearMoneda } from "./utilidades.js";

   // Datos simulados (en producción vendría de una API o BD)
   const empleadosDB = [
     { id: 1, nombre: "Ana Torres", email: "ana@empresa.com", salario: 45000, departamento: "Tecnología" },
     { id: 2, nombre: "Luis Ramírez", email: "luis@empresa.com", salario: 38000, departamento: "RRHH" },
     { id: 3, nombre: "Carmen López", email: "carmen@empresa.com", salario: 52000, departamento: "Tecnología" },
     { id: 4, nombre: "Roberto Silva", email: "roberto@empresa.com", salario: 61000, departamento: "Finanzas" }
   ];

   class ServicioEmpleados {
     #empleados; // Campo privado (ES2022)
     #siguienteId;

     constructor() {
       this.#empleados = [...empleadosDB]; // Copia para no mutar el original
       this.#siguienteId = Math.max(...this.#empleados.map(e => e.id)) + 1;
       log("INFO", "ServicioEmpleados inicializado", { totalEmpleados: this.#empleados.length });
     }

     obtenerTodos() {
       return [...this.#empleados]; // Retorna copia para evitar mutaciones externas
     }

     buscarPorId(id) {
       const empleado = this.#empleados.find(e => e.id === id);
       if (!empleado) throw new Error(`Empleado con ID ${id} no encontrado`);
       return { ...empleado };
     }

     buscarPorDepartamento(departamento) {
       return this.#empleados
         .filter(e => e.departamento.toLowerCase() === departamento.toLowerCase())
         .map(e => ({ ...e }));
     }

     agregar(datosEmpleado) {
       const { nombre, email, salario, departamento } = datosEmpleado;

       // Validaciones
       if (!nombre || nombre.trim().length < 2) {
         throw new Error("El nombre debe tener al menos 2 caracteres");
       }
       if (!validarEmail(email)) {
         throw new Error(`Email inválido: ${email}`);
       }
       if (!validarSalario(salario)) {
         throw new Error(`Salario inválido: ${salario}`);
       }
       if (this.#empleados.some(e => e.email === email)) {
         throw new Error(`Ya existe un empleado con el email: ${email}`);
       }

       const nuevoEmpleado = {
         id: this.#siguienteId++,
         nombre: nombre.trim(),
         email: email.toLowerCase(),
         salario,
         departamento: departamento || "Sin asignar"
       };

       this.#empleados.push(nuevoEmpleado);
       log("SUCCESS", `Empleado agregado: ${nuevoEmpleado.nombre}`, { id: nuevoEmpleado.id });
       return { ...nuevoEmpleado };
     }

     obtenerEstadisticas() {
       const empleados = this.#empleados;
       const salarios = empleados.map(e => e.salario);
       const total = salarios.reduce((sum, s) => sum + s, 0);
       const promedio = total / empleados.length;
       const maximo = Math.max(...salarios);
       const minimo = Math.min(...salarios);

       // Agrupar por departamento
       const porDepartamento = empleados.reduce((grupos, emp) => {
         const dept = emp.departamento;
         if (!grupos[dept]) grupos[dept] = [];
         grupos[dept].push(emp);
         return grupos;
       }, {});

       return {
         totalEmpleados: empleados.length,
         salarioPromedio: promedio,
         salarioMaximo: maximo,
         salarioMinimo: minimo,
         masaLaboral: total,
         porDepartamento: Object.entries(porDepartamento).map(([dept, emps]) => ({
           departamento: dept,
           cantidad: emps.length,
           salarioPromedio: emps.reduce((s, e) => s + e.salario, 0) / emps.length
         }))
       };
     }
   }

   // Exportación por defecto
   export default ServicioEmpleados;
   ```

5. Crea el archivo `index.js` que integra todos los módulos:

   ```bash
   # En macOS / Linux
   touch src/js-moderno/index.js

   # En Windows (PowerShell)
   New-Item src\js-moderno\index.js -ItemType File
   ```

6. Escribe el archivo principal:

   ```javascript
   // src/js-moderno/index.js
   // Archivo principal que integra todos los módulos

   // Importación por defecto (clase)
   import ServicioEmpleados from "./ServicioEmpleados.js";

   // Importaciones nombradas (funciones y constantes)
   import { formatearMoneda, formatearFecha, log, VERSION_APP } from "./utilidades.js";

   // Importar funciones de módulos anteriores
   import { calcularPrecioFinal } from "./variables-y-funciones.js";

   console.log("╔══════════════════════════════════════════════════╗");
   console.log(`║  Sistema de Gestión Empresarial v${VERSION_APP}        ║`);
   console.log(`║  Fecha: ${formatearFecha()}  ║`);
   console.log("╚══════════════════════════════════════════════════╝\n");

   // Inicializar el servicio
   const servicio = new ServicioEmpleados();

   // Mostrar todos los empleados
   console.log("\n📋 LISTA DE EMPLEADOS:");
   console.log("─".repeat(60));
   servicio.obtenerTodos().forEach(emp => {
     console.log(`  [${emp.id}] ${emp.nombre.padEnd(20)} | ${emp.departamento.padEnd(15)} | ${formatearMoneda(emp.salario)}`);
   });

   // Agregar un nuevo empleado
   console.log("\n➕ AGREGAR NUEVO EMPLEADO:");
   console.log("─".repeat(60));
   try {
     const nuevoEmp = servicio.agregar({
       nombre: "Patricia Mendoza",
       email: "patricia.mendoza@empresa.com",
       salario: 48000,
       departamento: "Tecnología"
     });
     console.log(`  Empleado creado: ${nuevoEmp.nombre} (ID: ${nuevoEmp.id})`);

     // Intentar agregar con email duplicado (debe lanzar error)
     servicio.agregar({
       nombre: "Otro Usuario",
       email: "patricia.mendoza@empresa.com", // Email duplicado
       salario: 30000
     });
   } catch (error) {
     log("WARN", `Validación fallida: ${error.message}`);
   }

   // Buscar por departamento
   console.log("\n🔍 EMPLEADOS DE TECNOLOGÍA:");
   console.log("─".repeat(60));
   const tecnologia = servicio.buscarPorDepartamento("Tecnología");
   tecnologia.forEach(emp => {
     console.log(`  - ${emp.nombre} | ${formatearMoneda(emp.salario)}`);
   });

   // Mostrar estadísticas
   console.log("\n📊 ESTADÍSTICAS GENERALES:");
   console.log("─".repeat(60));
   const stats = servicio.obtenerEstadisticas();
   console.log(`  Total de empleados:  ${stats.totalEmpleados}`);
   console.log(`  Salario promedio:    ${formatearMoneda(stats.salarioPromedio)}`);
   console.log(`  Salario máximo:      ${formatearMoneda(stats.salarioMaximo)}`);
   console.log(`  Salario mínimo:      ${formatearMoneda(stats.salarioMinimo)}`);
   console.log(`  Masa laboral total:  ${formatearMoneda(stats.masaLaboral)}`);

   console.log("\n📈 ESTADÍSTICAS POR DEPARTAMENTO:");
   console.log("─".repeat(60));
   stats.porDepartamento.forEach(({ departamento, cantidad, salarioPromedio }) => {
     console.log(`  ${departamento.padEnd(20)} | ${String(cantidad).padStart(2)} empleados | Promedio: ${formatearMoneda(salarioPromedio)}`);
   });

   // Demostrar uso de función importada de otro módulo
   console.log("\n💰 CÁLCULO DE PRECIO CON IMPUESTO (módulo externo):");
   console.log("─".repeat(60));
   const precioEjemplo = calcularPrecioFinal(10000, 15);
   console.log(`  Precio base: ${formatearMoneda(10000)} con 15% descuento`);
   console.log(`  Precio final (con IVA 16%): ${formatearMoneda(precioEjemplo)}`);
   ```

7. Ejecuta el archivo principal:

   ```bash
   npm run js-moderno
   ```

**Resultado Esperado:**

```
╔══════════════════════════════════════════════════╗
║  Sistema de Gestión Empresarial v1.0.0        ║
║  Fecha: [fecha actual en español]  ║
╚══════════════════════════════════════════════════╝

[timestamp] ℹ️ [INFO] ServicioEmpleados inicializado { totalEmpleados: 4 }

📋 LISTA DE EMPLEADOS:
────────────────────────────────────────────────────────────
  [1] Ana Torres            | Tecnología      | $45,000.00
  [2] Luis Ramírez          | RRHH            | $38,000.00
  [3] Carmen López          | Tecnología      | $52,000.00
  [4] Roberto Silva         | Finanzas        | $61,000.00

➕ AGREGAR NUEVO EMPLEADO:
────────────────────────────────────────────────────────────
[timestamp] ✅ [SUCCESS] Empleado agregado: Patricia Mendoza { id: 5 }
  Empleado creado: Patricia Mendoza (ID: 5)
[timestamp] ⚠️ [WARN] Validación fallida: Ya existe un empleado con el email: patricia.mendoza@empresa.com

🔍 EMPLEADOS DE TECNOLOGÍA:
────────────────────────────────────────────────────────────
  - Ana Torres | $45,000.00
  - Carmen López | $52,000.00
  - Patricia Mendoza | $48,000.00

📊 ESTADÍSTICAS GENERALES:
────────────────────────────────────────────────────────────
  Total de empleados:  5
  Salario promedio:    $48,800.00
  Salario máximo:      $61,000.00
  Salario mínimo:      $38,000.00
  Masa laboral total:  $244,000.00
```

**Verificación:**

- Confirma que la importación por defecto (`ServicioEmpleados`) y las importaciones nombradas (`formatearMoneda`, etc.) funcionan correctamente.
- Verifica que el error de email duplicado es capturado y manejado sin detener la ejecución del programa.
- Comprueba que las estadísticas incluyen al nuevo empleado (Patricia Mendoza) en el conteo.

---

### Paso 6: Implementar POO con Clases JavaScript

**Objetivo:** Diseñar e implementar una jerarquía de clases que modele un sistema de empleados empresarial, aplicando herencia, encapsulamiento con campos privados, polimorfismo y abstracción.

**Instrucciones:**

1. Crea los archivos de la jerarquía de clases:

   ```bash
   # En macOS / Linux
   touch src/poo/Persona.js
   touch src/poo/Empleado.js
   touch src/poo/EmpleadoTecnico.js
   touch src/poo/Gerente.js
   touch src/poo/index.js

   # En Windows (PowerShell)
   New-Item src\poo\Persona.js -ItemType File
   New-Item src\poo\Empleado.js -ItemType File
   New-Item src\poo\EmpleadoTecnico.js -ItemType File
   New-Item src\poo\Gerente.js -ItemType File
   New-Item src\poo\index.js -ItemType File
   ```

2. Escribe la clase base `Persona`:

   ```javascript
   // src/poo/Persona.js
   // Clase base - representa a cualquier persona en el sistema

   export class Persona {
     // Campos privados (encapsulamiento)
     #nombre;
     #email;
     #fechaNacimiento;

     constructor(nombre, email, fechaNacimiento) {
       if (new.target === Persona) {
         throw new Error("Persona es una clase abstracta y no puede instanciarse directamente");
       }
       this.#nombre = this.#validarNombre(nombre);
       this.#email = this.#validarEmail(email);
       this.#fechaNacimiento = new Date(fechaNacimiento);
     }

     // Métodos privados de validación
     #validarNombre(nombre) {
       if (!nombre || nombre.trim().length < 2) {
         throw new Error("El nombre debe tener al menos 2 caracteres");
       }
       return nombre.trim();
     }

     #validarEmail(email) {
       const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
       if (!regex.test(email)) {
         throw new Error(`Email inválido: ${email}`);
       }
       return email.toLowerCase();
     }

     // Getters (acceso controlado a campos privados)
     get nombre() { return this.#nombre; }
     get email() { return this.#email; }
     get fechaNacimiento() { return new Date(this.#fechaNacimiento); }

     get edad() {
       const hoy = new Date();
       const nacimiento = this.#fechaNacimiento;
       let edad = hoy.getFullYear() - nacimiento.getFullYear();
       const mes = hoy.getMonth() - nacimiento.getMonth();
       if (mes < 0 || (mes === 0 && hoy.getDate() < nacimiento.getDate())) {
         edad--;
       }
       return edad;
     }

     // Setter con validación
     set email(nuevoEmail) {
       this.#email = this.#validarEmail(nuevoEmail);
     }

     // Método que las subclases DEBEN sobrescribir (polimorfismo)
     obtenerDescripcion() {
       throw new Error("Las subclases deben implementar obtenerDescripcion()");
     }

     // Método común para todas las personas
     toString() {
       return `${this.constructor.name}[${this.#nombre}, ${this.#email}, edad: ${this.edad}]`;
     }
   }
   ```

3. Escribe la clase `Empleado` que extiende `Persona`:

   ```javascript
   // src/poo/Empleado.js
   import { Persona } from "./Persona.js";

   export class Empleado extends Persona {
     #salario;
     #departamento;
     #fechaContratacion;
     static #contadorEmpleados = 0; // Campo estático privado

     constructor(nombre, email, fechaNacimiento, salario, departamento) {
       super(nombre, email, fechaNacimiento); // Llamada al constructor padre
       this.#salario = this.#validarSalario(salario);
       this.#departamento = departamento || "Sin asignar";
       this.#fechaContratacion = new Date();
       this.id = ++Empleado.#contadorEmpleados;
     }

     #validarSalario(salario) {
       if (typeof salario !== "number" || salario <= 0) {
         throw new Error(`Salario inválido: ${salario}. Debe ser un número positivo.`);
       }
       return salario;
     }

     // Getters
     get salario() { return this.#salario; }
     get departamento() { return this.#departamento; }
     get fechaContratacion() { return new Date(this.#fechaContratacion); }
     get antiguedad() {
       const hoy = new Date();
       const contratacion = this.#fechaContratacion;
       const diffMs = hoy - contratacion;
       const diffDias = Math.floor(diffMs / (1000 * 60 * 60 * 24));
       return diffDias;
     }

     // Setter con validación
     set salario(nuevoSalario) {
       if (nuevoSalario < this.#salario) {
         throw new Error("No se puede reducir el salario de un empleado");
       }
       this.#salario = this.#validarSalario(nuevoSalario);
     }

     // Método estático
     static obtenerTotalEmpleados() {
       return Empleado.#contadorEmpleados;
     }

     // Implementación del método abstracto (polimorfismo)
     obtenerDescripcion() {
       return `Empleado: ${this.nombre} | Depto: ${this.#departamento} | Salario: $${this.#salario.toLocaleString("es-MX")}`;
     }

     // Método para calcular bono (puede sobrescribirse)
     calcularBono() {
       return this.#salario * 0.05; // 5% por defecto
     }

     toJSON() {
       return {
         id: this.id,
         nombre: this.nombre,
         email: this.email,
         edad: this.edad,
         salario: this.#salario,
         departamento: this.#departamento,
         antiguedadDias: this.antiguedad,
         tipo: this.constructor.name
       };
     }
   }
   ```

4. Escribe la clase `EmpleadoTecnico` que extiende `Empleado`:

   ```javascript
   // src/poo/EmpleadoTecnico.js
   import { Empleado } from "./Empleado.js";

   export class EmpleadoTecnico extends Empleado {
     #tecnologias;
     #nivelSeniority;
     static NIVELES = ["Junior", "Semi-Senior", "Senior", "Lead", "Principal"];

     constructor(nombre, email, fechaNacimiento, salario, tecnologias, nivelSeniority = "Junior") {
       super(nombre, email, fechaNacimiento, salario, "Tecnología");
       this.#tecnologias = Array.isArray(tecnologias) ? [...tecnologias] : [tecnologias];
       this.#nivelSeniority = this.#validarNivel(nivelSeniority);
     }

     #validarNivel(nivel) {
       if (!EmpleadoTecnico.NIVELES.includes(nivel)) {
         throw new Error(`Nivel inválido: ${nivel}. Opciones: ${EmpleadoTecnico.NIVELES.join(", ")}`);
       }
       return nivel;
     }

     get tecnologias() { return [...this.#tecnologias]; }
     get nivelSeniority() { return this.#nivelSeniority; }

     agregarTecnologia(tecnologia) {
       if (!this.#tecnologias.includes(tecnologia)) {
         this.#tecnologias.push(tecnologia);
       }
     }

     promover() {
       const indiceActual = EmpleadoTecnico.NIVELES.indexOf(this.#nivelSeniority);
       if (indiceActual < EmpleadoTecnico.NIVELES.length - 1) {
         this.#nivelSeniority = EmpleadoTecnico.NIVELES[indiceActual + 1];
         const incremento = this.salario * 0.15;
         this.salario = this.salario + incremento;
         return `${this.nombre} promovido a ${this.#nivelSeniority}. Nuevo salario: $${this.salario.toLocaleString("es-MX")}`;
       }
       return `${this.nombre} ya está en el nivel máximo (${this.#nivelSeniority})`;
     }

     // Sobrescribir (polimorfismo)
     obtenerDescripcion() {
       return `[${this.#nivelSeniority}] ${this.nombre} | Tecnologías: ${this.#tecnologias.join(", ")} | Salario: $${this.salario.toLocaleString("es-MX")}`;
     }

     calcularBono() {
       const multiplicadores = { "Junior": 0.05, "Semi-Senior": 0.08, "Senior": 0.12, "Lead": 0.18, "Principal": 0.25 };
       return this.salario * (multiplicadores[this.#nivelSeniority] || 0.05);
     }
   }
   ```

5. Escribe la clase `Gerente`:

   ```javascript
   // src/poo/Gerente.js
   import { Empleado } from "./Empleado.js";

   export class Gerente extends Empleado {
     #equipo;
     #presupuestoGestion;

     constructor(nombre, email, fechaNacimiento, salario, departamento, presupuestoGestion) {
       super(nombre, email, fechaNacimiento, salario, departamento);
       this.#equipo = [];
       this.#presupuestoGestion = presupuestoGestion || 0;
     }

     get equipo() { return [...this.#equipo]; }
     get tamanoEquipo() { return this.#equipo.length; }
     get presupuestoGestion() { return this.#presupuestoGestion; }

     agregarMiembro(empleado) {
       if (!(empleado instanceof Empleado)) {
         throw new Error("Solo se pueden agregar instancias de Empleado al equipo");
       }
       if (this.#equipo.some(e => e.id === empleado.id)) {
         throw new Error(`${empleado.nombre} ya está en el equipo`);
       }
       this.#equipo.push(empleado);
     }

     obtenerMasaLaboral() {
       return this.#equipo.reduce((total, emp) => total + emp.salario, 0);
     }

     // Polimorfismo: sobrescribir método de la clase padre
     obtenerDescripcion() {
       return `[GERENTE] ${this.nombre} | ${this.departamento} | Equipo: ${this.#equipo.length} personas | Salario: $${this.salario.toLocaleString("es-MX")}`;
     }

     calcularBono() {
       // Gerentes reciben bono base + porcentaje por tamaño de equipo
       const bonoBase = this.salario * 0.20;
       const bonoEquipo = this.#equipo.length * 500;
       return bonoBase + bonoEquipo;
     }
   }
   ```

6. Escribe el archivo `index.js` de POO:

   ```javascript
   // src/poo/index.js
   import { Empleado } from "./Empleado.js";
   import { EmpleadoTecnico } from "./EmpleadoTecnico.js";
   import { Gerente } from "./Gerente.js";

   console.log("╔══════════════════════════════════════════════════╗");
   console.log("║         DEMOSTRACIÓN DE POO EN JAVASCRIPT        ║");
   console.log("╚══════════════════════════════════════════════════╝\n");

   // ============================================================
   // Crear instancias de diferentes tipos
   // ============================================================
   const dev1 = new EmpleadoTecnico(
     "Ana Torres", "ana@empresa.com", "1992-05-15",
     45000, ["JavaScript", "React", "Node.js"], "Senior"
   );

   const dev2 = new EmpleadoTecnico(
     "Carlos Vega", "carlos@empresa.com", "1998-11-20",
     32000, ["Python", "Django"], "Junior"
   );

   const gerente = new Gerente(
     "María Directora", "maria.directora@empresa.com", "1985-03-10",
     85000, "Tecnología", 200000
   );

   // Agregar miembros al equipo del gerente
   gerente.agregarMiembro(dev1);
   gerente.agregarMiembro(dev2);

   // ============================================================
   // Demostrar polimorfismo
   // ============================================================
   console.log("📋 POLIMORFISMO - obtenerDescripcion():");
   console.log("─".repeat(60));
   const personas = [dev1, dev2, gerente];
   personas.forEach(p => {
     console.log("  " + p.obtenerDescripcion());
   });

   // ============================================================
   // Demostrar cálculo de bonos (polimorfismo)
   // ============================================================
   console.log("\n💰 CÁLCULO DE BONOS (polimorfismo en calcularBono):");
   console.log("─".repeat(60));
   personas.forEach(p => {
     const bono = p.calcularBono();
     console.log(`  ${p.nombre.padEnd(20)} | Tipo: ${p.constructor.name.padEnd(16)} | Bono: $${bono.toLocaleString("es-MX", { minimumFractionDigits: 2 })}`);
   });

   // ============================================================
   // Demostrar encapsulamiento
   // ============================================================
   console.log("\n🔒 ENCAPSULAMIENTO - Validaciones:");
   console.log("─".repeat(60));

   try {
     dev1.salario = 20000; // Intentar reducir salario
   } catch (error) {
     console.log(`  ✅ Error esperado al reducir salario: ${error.message}`);
   }

   try {
     dev1.email = "email-invalido"; // Email inválido
   } catch (error) {
     console.log(`  ✅ Error esperado con email inválido: ${error.message}`);
   }

   // Actualización válida
   dev1.email = "ana.torres@empresa.com";
   console.log(`  ✅ Email actualizado correctamente: ${dev1.email}`);

   // ============================================================
   // Demostrar herencia e instanceof
   // ============================================================
   console.log("\n🔗 HERENCIA - instanceof:");
   console.log("─".repeat(60));
   console.log(`  dev1 instanceof EmpleadoTecnico: ${dev1 instanceof EmpleadoTecnico}`);
   console.log(`  dev1 instanceof Empleado:        ${dev1 instanceof Empleado}`);
   console.log(`  gerente instanceof Gerente:      ${gerente instanceof Gerente}`);
   console.log(`  gerente instanceof Empleado:     ${gerente instanceof Empleado}`);

   // ============================================================
   // Promover empleado técnico
   // ============================================================
   console.log("\n⬆️  PROMOCIÓN DE EMPLEADO:");
   console.log("─".repeat(60));
   console.log(`  Estado actual: ${dev2.obtenerDescripcion()}`);
   const resultadoPromocion = dev2.promover();
   console.log(`  ${resultadoPromocion}`);

   // ============================================================
   // Estadísticas del equipo
   // ============================================================
   console.log("\n📊 ESTADÍSTICAS DEL EQUIPO:");
   console.log("─".repeat(60));
   console.log(`  Gerente: ${gerente.nombre}`);
   console.log(`  Tamaño del equipo: ${gerente.tamanoEquipo} personas`);
   console.log(`  Masa laboral del equipo: $${gerente.obtenerMasaLaboral().toLocaleString("es-MX")}`);
   console.log(`  Total de empleados creados: ${Empleado.obtenerTotalEmpleados()}`);

   // Serializar a JSON
   console.log("\n📄 SERIALIZACIÓN A JSON:");
   console.log("─".repeat(60));
   console.log(JSON.stringify(dev1.toJSON(), null, 2));
   ```

7. Ejecuta la demostración de POO:

   ```bash
   npm run poo
   ```

**Resultado Esperado:**

```
╔══════════════════════════════════════════════════╗
║         DEMOSTRACIÓN DE POO EN JAVASCRIPT        ║
╚══════════════════════════════════════════════════╝

📋 POLIMORFISMO - obtenerDescripcion():
────────────────────────────────────────────────────────────
  [Senior] Ana Torres | Tecnologías: JavaScript, React, Node.js | Salario: $45,000
  [Junior] Carlos Vega | Tecnologías: Python, Django | Salario: $32,000
  [GERENTE] María Directora | Tecnología | Equipo: 2 personas | Salario: $85,000

💰 CÁLCULO DE BONOS (polimorfismo en calcularBono):
────────────────────────────────────────────────────────────
  Ana Torres           | Tipo: EmpleadoTecnico    | Bono: $5,400.00
  Carlos Vega          | Tipo: EmpleadoTecnico    | Bono: $1,600.00
  María Directora      | Tipo: Gerente            | Bono: $18,000.00

🔒 ENCAPSULAMIENTO - Validaciones:
────────────────────────────────────────────────────────────
  ✅ Error esperado al reducir salario: No se puede reducir el salario de un empleado
  ✅ Error esperado con email inválido: Email inválido: email-invalido
  ✅ Email actualizado correctamente: ana.torres@empresa.com

🔗 HERENCIA - instanceof:
────────────────────────────────────────────────────────────
  dev1 instanceof EmpleadoTecnico: true
  dev1 instanceof Empleado:        true
  gerente instanceof Gerente:      true
  gerente instanceof Empleado:     true
```

**Verificación:**

- Confirma que `instanceof` devuelve `true` para la clase directa y también para las clases padre (herencia).
- Verifica que los errores de encapsulamiento son lanzados correctamente (reducir salario, email inválido).
- Comprueba que el polimorfismo funciona: cada clase implementa `calcularBono()` y `obtenerDescripcion()` de forma diferente.

---

### Paso 7: Comparación JavaScript vs Java

**Objetivo:** Implementar el mismo sistema de gestión de empleados en Java 17, identificando similitudes y diferencias con JavaScript en cuanto a sintaxis, tipos, colecciones, streams y manejo de excepciones.

**Instrucciones:**

1. Crea el directorio y archivo Java:

   ```bash
   # En macOS / Linux
   mkdir -p src/java-comparacion
   touch src/java-comparacion/SistemaEmpleados.java

   # En Windows (PowerShell)
   New-Item src\java-comparacion\SistemaEmpleados.java -ItemType File
   ```

2. Escribe el código Java en el archivo. Ábrelo en VS Code y escribe:

   ```java
   // src/java-comparacion/SistemaEmpleados.java
   // Comparación directa con el código JavaScript del laboratorio

   import java.util.*;
   import java.util.stream.*;
   import java.time.*;
   import java.time.format.*;

   // ============================================================
   // Clase Empleado (equivalente a la clase JS)
   // ============================================================
   class Empleado {
       // Campos privados (encapsulamiento - equivalente a # en JS)
       private final int id;
       private final String nombre;
       private String email;
       private double salario;
       private final String departamento;
       private static int contadorEmpleados = 0;

       // Constructor (equivalente a constructor() en JS)
       public Empleado(String nombre, String email, double salario, String departamento) {
           this.id = ++contadorEmpleados;
           this.nombre = validarNombre(nombre);
           this.email = validarEmail(email);
           this.salario = validarSalario(salario);
           this.departamento = departamento != null ? departamento : "Sin asignar";
       }

       // Validaciones privadas
       private String validarNombre(String nombre) {
           if (nombre == null || nombre.trim().length() < 2) {
               throw new IllegalArgumentException("El nombre debe tener al menos 2 caracteres");
           }
           return nombre.trim();
       }

       private String validarEmail(String email) {
           if (email == null || !email.matches("^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$")) {
               throw new IllegalArgumentException("Email inválido: " + email);
           }
           return email.toLowerCase();
       }

       private double validarSalario(double salario) {
           if (salario <= 0) {
               throw new IllegalArgumentException("Salario inválido: " + salario);
           }
           return salario;
       }

       // Getters (equivalente a get nombre() en JS)
       public int getId() { return id; }
       public String getNombre() { return nombre; }
       public String getEmail() { return email; }
       public double getSalario() { return salario; }
       public String getDepartamento() { return departamento; }
       public static int getTotalEmpleados() { return contadorEmpleados; }

       // Setter con validación
       public void setSalario(double nuevoSalario) {
           if (nuevoSalario < this.salario) {
               throw new IllegalStateException("No se puede reducir el salario");
           }
           this.salario = validarSalario(nuevoSalario);
       }

       // Método polimórfico (equivalente a obtenerDescripcion() en JS)
       public String obtenerDescripcion() {
           return String.format("Empleado: %s | Depto: %s | Salario: $%,.2f",
               nombre, departamento, salario);
       }

       public double calcularBono() {
           return salario * 0.05;
       }

       @Override
       public String toString() {
           return String.format("Empleado[id=%d, nombre=%s, email=%s, salario=%.2f]",
               id, nombre, email, salario);
       }
   }

   // ============================================================
   // Subclase EmpleadoTecnico (herencia)
   // ============================================================
   class EmpleadoTecnico extends Empleado {
       private List<String> tecnologias;
       private String nivelSeniority;

       private static final List<String> NIVELES = List.of(
           "Junior", "Semi-Senior", "Senior", "Lead", "Principal"
       );

       public EmpleadoTecnico(String nombre, String email, double salario,
                               List<String> tecnologias, String nivelSeniority) {
           super(nombre, email, salario, "Tecnología");
           this.tecnologias = new ArrayList<>(tecnologias);
           if (!NIVELES.contains(nivelSeniority)) {
               throw new IllegalArgumentException("Nivel inválido: " + nivelSeniority);
           }
           this.nivelSeniority = nivelSeniority;
       }

       public List<String> getTecnologias() {
           return Collections.unmodifiableList(tecnologias);
       }

       public String getNivelSeniority() { return nivelSeniority; }

       @Override
       public String obtenerDescripcion() {
           return String.format("[%s] %s | Tecnologías: %s | Salario: $%,.2f",
               nivelSeniority, getNombre(),
               String.join(", ", tecnologias), getSalario());
       }

       @Override
       public double calcularBono() {
           Map<String, Double> multiplicadores = Map.of(
               "Junior", 0.05, "Semi-Senior", 0.08,
               "Senior", 0.12, "Lead", 0.18, "Principal", 0.25
           );
           return getSalario() * multiplicadores.getOrDefault(nivelSeniority, 0.05);
       }
   }

   // ============================================================
   // Clase principal con main()
   // ============================================================
   public class SistemaEmpleados {

       public static void main(String[] args) {
           System.out.println("╔══════════════════════════════════════════════════╗");
           System.out.println("║    COMPARACIÓN JAVA vs JAVASCRIPT - EMPLEADOS    ║");
           System.out.println("╚══════════════════════════════════════════════════╝\n");

           // --------------------------------------------------------
           // SECCIÓN 1: Crear empleados (equivalente a new en JS)
           // --------------------------------------------------------
           System.out.println("=== SECCIÓN 1: Creación de objetos ===\n");

           List<Empleado> empleados = new ArrayList<>();
           empleados.add(new EmpleadoTecnico("Ana Torres", "ana@empresa.com", 45000,
               List.of("JavaScript", "React", "Node.js"), "Senior"));
           empleados.add(new EmpleadoTecnico("Carlos Vega", "carlos@empresa.com", 32000,
               List.of("Python", "Django"), "Junior"));
           empleados.add(new Empleado("Luis Ramírez", "luis@empresa.com", 38000, "RRHH"));
           empleados.add(new Empleado("Roberto Silva", "roberto@empresa.com", 61000, "Finanzas"));

           empleados.forEach(e -> System.out.println("  " + e.obtenerDescripcion()));

           // --------------------------------------------------------
           // SECCIÓN 2: Streams (equivalente a filter/map/reduce en JS)
           // --------------------------------------------------------
           System.out.println("\n=== SECCIÓN 2: Streams de Java vs Array methods JS ===\n");
           System.out.println("  JavaScript: empleados.filter(e => e.salario > 40000)");
           System.out.println("  Java:       empleados.stream().filter(e -> e.getSalario() > 40000)\n");

           // Equivalente a .filter() en JS
           List<Empleado> altoPago = empleados.stream()
               .filter(e -> e.getSalario() > 40000)
               .collect(Collectors.toList());

           System.out.println("  Empleados con salario > $40,000:");
           altoPago.forEach(e -> System.out.printf("    - %s: $%,.2f%n", e.getNombre(), e.getSalario()));

           // Equivalente a .map() en JS
           List<String> nombres = empleados.stream()
               .map(Empleado::getNombre) // Method reference (equivalente a e => e.getNombre())
               .collect(Collectors.toList());
           System.out.println("\n  Nombres (equivalente a .map()): " + nombres);

           // Equivalente a .reduce() en JS
           double totalSalarios = empleados.stream()
               .mapToDouble(Empleado::getSalario)
               .sum(); // equivalente a .reduce((sum, e) => sum + e.salario, 0)
           System.out.printf("%n  Total masa laboral (.reduce()): $%,.2f%n", totalSalarios);

           // --------------------------------------------------------
           // SECCIÓN 3: Manejo de excepciones (checked vs unchecked)
           // --------------------------------------------------------
           System.out.println("\n=== SECCIÓN 3: Manejo de Excepciones ===\n");
           System.out.println("  JavaScript: try { ... } catch(error) { ... }");
           System.out.println("  Java:       try { ... } catch(Exception e) { ... }\n");

           // Excepción no verificada (unchecked) - equivalente a throw en JS
           try {
               Empleado invalido = new Empleado("X", "no-es-email", -1000, "Test");
           } catch (IllegalArgumentException e) {
               System.out.println("  ✅ Excepción capturada (unchecked): " + e.getMessage());
           }

           // Try con múltiples catch
           try {
               empleados.get(0).setSalario(30000); // Intento de reducir salario
           } catch (IllegalStateException e) {
               System.out.println("  ✅ Excepción de estado: " + e.getMessage());
           } catch (Exception e) {
               System.out.println("  ❌ Error inesperado: " + e.getMessage());
           } finally {
               System.out.println("  ✅ Bloque finally siempre se ejecuta (igual que en JS)");
           }

           // --------------------------------------------------------
           // SECCIÓN 4: Colecciones - Map (equivalente a objetos JS)
           // --------------------------------------------------------
           System.out.println("\n=== SECCIÓN 4: Colecciones Map ===\n");
           System.out.println("  JavaScript: const mapa = {}; mapa[clave] = valor;");
           System.out.println("  Java:       Map<String, Object> mapa = new HashMap<>();\n");

           Map<String, Double> bonos = new HashMap<>();
           empleados.forEach(e -> bonos.put(e.getNombre(), e.calcularBono()));

           System.out.println("  Bonos calculados:");
           bonos.entrySet().stream()
               .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
               .forEach(entry ->
                   System.out.printf("    %-20s -> $%,.2f%n", entry.getKey(), entry.getValue())
               );

           // --------------------------------------------------------
           // SECCIÓN 5: Estadísticas con Streams
           // --------------------------------------------------------
           System.out.println("\n=== SECCIÓN 5: Estadísticas con Streams ===\n");

           DoubleSummaryStatistics estadisticas = empleados.stream()
               .mapToDouble(Empleado::getSalario)
               .summaryStatistics();

           System.out.printf("  Total empleados:  %d%n", estadisticas.getCount());
           System.out.printf("  Salario promedio: $%,.2f%n", estadisticas.getAverage());
           System.out.printf("  Salario máximo:   $%,.2f%n", estadisticas.getMax());
           System.out.printf("  Salario mínimo:   $%,.2f%n", estadisticas.getMin());
           System.out.printf("  Masa laboral:     $%,.2f%n", estadisticas.getSum());

           System.out.println("\n✅ Comparación completada. Total empleados creados: " +
               Empleado.getTotalEmpleados());
       }
   }
   ```

3. Compila el archivo Java:

   ```bash
   # En macOS / Linux
   cd src/java-comparacion
   javac SistemaEmpleados.java
   cd ../..

   # En Windows (PowerShell)
   cd src\java-comparacion
   javac SistemaEmpleados.java
   cd ..\..
   ```

4. Ejecuta el programa Java:

   ```bash
   # En macOS / Linux
   java -cp src/java-comparacion SistemaEmpleados

   # En Windows (PowerShell)
   java -cp src\java-comparacion SistemaEmpleados
   ```

**Resultado Esperado:**

```
╔══════════════════════════════════════════════════╗
║    COMPARACIÓN JAVA vs JAVASCRIPT - EMPLEADOS    ║
╚══════════════════════════════════════════════════╝

=== SECCIÓN 1: Creación de objetos ===

  [Senior] Ana Torres | Tecnologías: JavaScript, React, Node.js | Salario: $45,000.00
  [Junior] Carlos Vega | Tecnologías: Python, Django | Salario: $32,000.00
  Empleado: Luis Ramírez | Depto: RRHH | Salario: $38,000.00
  Empleado: Roberto Silva | Depto: Finanzas | Salario: $61,000.00

=== SECCIÓN 2: Streams de Java vs Array methods JS ===

  Empleados con salario > $40,000:
    - Ana Torres: $45,000.00
    - Roberto Silva: $61,000.00

  Nombres: [Ana Torres, Carlos Vega, Luis Ramírez, Roberto Silva]

  Total masa laboral: $176,000.00

=== SECCIÓN 3: Manejo de Excepciones ===

  ✅ Excepción capturada (unchecked): Email inválido: no-es-email
  ✅ Excepción de estado: No se puede reducir el salario
  ✅ Bloque finally siempre se ejecuta (igual que en JS)

=== SECCIÓN 4: Colecciones Map ===

  Bonos calculados:
    Roberto Silva        -> $3,050.00
    Ana Torres           -> $5,400.00
    Luis Ramírez         -> $1,900.00
    Carlos Vega          -> $1,600.00

=== SECCIÓN 5: Estadísticas con Streams ===

  Total empleados:  4
  Salario promedio: $44,000.00
  Salario máximo:   $61,000.00
  Salario mínimo:   $32,000.00
  Masa laboral:     $176,000.00

✅ Comparación completada. Total empleados creados: 5
```

**Verificación:**

- Confirma que el código Java compila sin errores (`javac` no muestra mensajes de error).
- Verifica que la salida de estadísticas en Java es equivalente a la producida por JavaScript.
- Compara mentalmente las diferencias de sintaxis: tipos explícitos en Java vs tipado dinámico en JS, `stream()` vs métodos de array, `throws` vs `throw`.

---

### Paso 8: Configurar Git y Realizar el Primer Push a GitHub

**Objetivo:** Inicializar un repositorio Git local, crear commits descriptivos con la convención Conventional Commits, configurar el repositorio remoto en GitHub y realizar el primer push estableciendo el flujo de trabajo que se usará durante todo el curso.

**Instrucciones:**

1. Primero, configura tu identidad en Git si aún no lo has hecho:

   ```bash
   git config --global user.name "Tu Nombre Completo"
   git config --global user.email "tu.email@ejemplo.com"
   git config --global init.defaultBranch main
   ```

2. Verifica la configuración:

   ```bash
   git config --global --list
   ```

3. Inicializa el repositorio Git en la raíz del proyecto (asegúrate de estar en el directorio `lab-01`):

   ```bash
   # Verifica que estás en el directorio correcto
   # En macOS / Linux
   pwd
   # Debe mostrar: .../curso-fullstack/lab-01

   # En Windows (PowerShell)
   Get-Location
   # Debe mostrar: C:\curso-fullstack\lab-01

   # Inicializar repositorio
   git init
   ```

4. Crea el archivo `.gitignore` para excluir archivos innecesarios:

   ```bash
   # En macOS / Linux
   touch .gitignore

   # En Windows (PowerShell)
   New-Item .gitignore -ItemType File
   ```

   Escribe el siguiente contenido en `.gitignore`:

   ```
   # Node.js
   node_modules/
   npm-debug.log*
   .npm

   # Java compilados
   *.class
   *.jar
   out/
   build/

   # IDE
   .idea/
   .vscode/settings.json
   *.iml

   # Sistema operativo
   .DS_Store
   Thumbs.db
   desktop.ini

   # Variables de entorno
   .env
   .env.local
   ```

5. Verifica el estado del repositorio:

   ```bash
   git status
   ```

6. Agrega todos los archivos al área de staging:

   ```bash
   git add .
   ```

7. Verifica qué archivos están en staging:

   ```bash
   git status
   ```

8. Realiza el primer commit con mensaje descriptivo siguiendo la convención **Conventional Commits**:

   ```bash
   git commit -m "feat: inicializar proyecto Lab 01 con estructura base

   - Configurar package.json con type module para ES Modules
   - Crear estructura de directorios src/js-moderno, src/poo, src/java-comparacion
   - Agregar .gitignore para Node.js, Java e IDEs"
   ```

9. Crea una rama para el trabajo del laboratorio:

   ```bash
   git checkout -b feature/javascript-moderno
   ```

10. Verifica las ramas existentes:

    ```bash
    git branch -a
    ```

11. Ahora crea el repositorio en GitHub. Abre tu navegador y ve a [https://github.com/new](https://github.com/new). Configura:
    - **Repository name:** `curso-fullstack-lab01`
    - **Description:** `Laboratorio 1: JavaScript Moderno, POO, Java y Git`
    - **Visibility:** Public (o Private según tu preferencia)
    - **NO** marques "Add a README file" (ya tenemos archivos locales)
    - Haz clic en **Create repository**

12. Conecta tu repositorio local con el remoto (reemplaza `TU_USUARIO` con tu nombre de usuario de GitHub):

    ```bash
    git remote add origin https://github.com/TU_USUARIO/curso-fullstack-lab01.git
    ```

13. Verifica la conexión remota:

    ```bash
    git remote -v
    ```

14. Sube la rama `main` al repositorio remoto:

    ```bash
    # Primero asegúrate de estar en main y que tenga el commit inicial
    git checkout main
    git push -u origin main
    ```

15. Sube también la rama de trabajo:

    ```bash
    git checkout feature/javascript-moderno
    git push -u origin feature/javascript-moderno
    ```

16. Crea un commit adicional con el código desarrollado en esta rama:

    ```bash
    # Agregar cualquier cambio pendiente
    git add .
    git status
    ```

    Si hay cambios pendientes (archivos `.class` de Java deberían estar ignorados):

    ```bash
    git commit -m "feat(js): implementar módulos ES6+ con sistema de empleados

    - Agregar variables-y-funciones.js con let/const y arrow functions
    - Agregar destructuring-spread-rest.js con operadores modernos
    - Agregar async-await.js con Promises y Promise.all/allSettled
    - Agregar ServicioEmpleados.js con módulos ES y campos privados
    - Agregar utilidades.js con funciones de formato y validación"
    ```

    ```bash
    git push origin feature/javascript-moderno
    ```

17. Verifica el historial de commits:

    ```bash
    git log --oneline --graph --all
    ```

**Resultado Esperado:**

```bash
# git remote -v
origin  https://github.com/TU_USUARIO/curso-fullstack-lab01.git (fetch)
origin  https://github.com/TU_USUARIO/curso-fullstack-lab01.git (push)

# git log --oneline --graph --all
* a1b2c3d (HEAD -> feature/javascript-moderno, origin/feature/javascript-moderno) feat(js): implementar módulos ES6+
* d4e5f6g (origin/main, main) feat: inicializar proyecto Lab 01 con estructura base

# git branch -a
* feature/javascript-moderno
  main
  remotes/origin/feature/javascript-moderno
  remotes/origin/main
```

**Verificación:**

- Abre tu navegador y navega a `https://github.com/TU_USUARIO/curso-fullstack-lab01` para confirmar que el repositorio existe y tiene los archivos.
- Verifica que aparecen ambas ramas (`main` y `feature/javascript-moderno`) en la interfaz de GitHub.
- Confirma que `.class` (archivos compilados de Java) NO aparecen en el repositorio de GitHub gracias al `.gitignore`.

---

### Paso 9: Crear el README del Proyecto

**Objetivo:** Documentar el proyecto con un archivo README profesional en formato Markdown que sirva como referencia para el repositorio y demuestre buenas prácticas de documentación.

**Instrucciones:**

1. Crea el archivo README en la raíz del proyecto:

   ```bash
   # En macOS / Linux
   touch README.md

   # En Windows (PowerShell)
   New-Item README.md -ItemType File
   ```

2. Escribe el siguiente contenido en `README.md`:

   ```markdown
   # Lab 01: JavaScript Moderno, POO, Java y Git

   Laboratorio 1 del Curso Full Stack Empresarial. Este proyecto demuestra las
   características fundamentales de JavaScript ES6+, Programación Orientada a
   Objetos, comparación con Java 17 y flujo de trabajo con Git/GitHub.

   ## Estructura del Proyecto

   ```
   lab-01/
   ├── src/
   │   ├── js-moderno/          # Módulos ES6+ de JavaScript
   │   │   ├── variables-y-funciones.js
   │   │   ├── destructuring-spread-rest.js
   │   │   ├── async-await.js
   │   │   ├── ServicioEmpleados.js
   │   │   ├── utilidades.js
   │   │   └── index.js
   │   ├── poo/                 # POO con clases JavaScript
   │   │   ├── Persona.js
   │   │   ├── Empleado.js
   │   │   ├── EmpleadoTecnico.js
   │   │   ├── Gerente.js
   │   │   └── index.js
   │   └── java-comparacion/    # Código Java equivalente
   │       └── SistemaEmpleados.java
   ├── package.json
   ├── .gitignore
   └── README.md
   ```

   ## Requisitos

   - Node.js 20.x LTS
   - JDK 17
   - Git 2.44+

   ## Ejecución

   ```bash
   # JavaScript Moderno (módulos ES)
   npm run js-moderno

   # POO con clases JavaScript
   npm run poo

   # Java (compilar y ejecutar)
   javac src/java-comparacion/SistemaEmpleados.java
   java -cp src/java-comparacion SistemaEmpleados
   ```

   ## Conceptos Demostrados

   ### JavaScript ES6+
   - `let`/`const` con alcance de bloque
   - Arrow functions y contexto `this`
   - Desestructuración de objetos y arrays
   - Operadores spread y rest
   - Promesas y `async/await`
   - `Promise.all` y `Promise.allSettled`
   - Módulos ES (`import`/`export`)

   ### POO en JavaScript
   - Clases con campos privados (`#campo`)
   - Herencia con `extends` y `super`
   - Encapsulamiento con getters/setters
   - Polimorfismo con sobrescritura de métodos
   - Miembros estáticos
   - Prevención de instanciación de clases abstractas

   ### Comparación Java vs JavaScript
   | Concepto | JavaScript | Java |
   |----------|-----------|------|
   | Variables | `let`/`const` | Tipos explícitos |
   | Privado | `#campo` | `private` |
   | Herencia | `extends` | `extends` |
   | Iteración | `.filter()/.map()/.reduce()` | `.stream().filter()...` |
   | Errores | `throw new Error()` | `throw new Exception()` |
   | Colecciones | Arrays/Objects | `List`/`Map` |

   ## Autor

   Participante del Curso Full Stack Empresarial
   ```

3. Agrega y confirma el README:

   ```bash
   git add README.md
   git commit -m "docs: agregar README con documentación del proyecto"
   git push origin feature/javascript-moderno
   ```

**Resultado Esperado:**

El archivo `README.md` aparece en GitHub con formato Markdown renderizado correctamente, mostrando la tabla de comparación y la estructura del proyecto.

**Verificación:**

- Navega a `https://github.com/TU_USUARIO/curso-fullstack-lab01/tree/feature/javascript-moderno` y confirma que el README se renderiza correctamente.
- Verifica que la tabla de comparación JavaScript vs Java se muestra con el formato correcto.

---

## Validación y Pruebas

### Criterios de Éxito

- [ ] `node src/js-moderno/variables-y-funciones.js` ejecuta sin errores y muestra el reporte de producto
- [ ] `node src/js-moderno/destructuring-spread-rest.js` ejecuta sin errores y muestra el intercambio de variables y cálculo de bonificación
- [ ] `node src/js-moderno/async-await.js` ejecuta sin errores y demuestra ejecución paralela con `Promise.all`
- [ ] `npm run js-moderno` ejecuta el sistema integrado de empleados con estadísticas correctas
- [ ] `npm run poo` demuestra polimorfismo, encapsulamiento y herencia sin errores
- [ ] `javac src/java-comparacion/SistemaEmpleados.java` compila sin errores
- [ ] `java -cp src/java-comparacion SistemaEmpleados` ejecuta y muestra estadísticas correctas
- [ ] `git log --oneline` muestra al menos 2 commits con mensajes descriptivos
- [ ] El repositorio remoto en GitHub existe con ambas ramas (`main` y `feature/javascript-moderno`)
- [ ] El archivo `.gitignore` excluye correctamente los archivos `.class`

### Procedimiento de Pruebas

1. Prueba completa del módulo JavaScript moderno:

   ```bash
   node src/js-moderno/variables-y-funciones.js
   ```
   **Resultado Esperado:** Muestra el reporte de producto con precio calculado correctamente (precio con 10% descuento + 16% IVA).

2. Prueba del módulo de desestructuración:

   ```bash
   node src/js-moderno/destructuring-spread-rest.js
   ```
   **Resultado Esperado:** El intercambio de variables muestra `A=200, B=100` después del intercambio. La bonificación promedio es `12.50%`.

3. Prueba de operaciones asíncronas:

   ```bash
   node src/js-moderno/async-await.js
   ```
   **Resultado Esperado:** El dashboard carga en menos de 400ms (paralelo). `Promise.allSettled` muestra éxito para IDs 1 y 4, y error para ID 99.

4. Prueba del sistema integrado:

   ```bash
   npm run js-moderno
   ```
   **Resultado Esperado:** La masa laboral total es $244,000 (incluyendo a Patricia Mendoza con $48,000).

5. Prueba de POO:

   ```bash
   npm run poo
   ```
   **Resultado Esperado:** `instanceof EmpleadoTecnico` devuelve `true` para `dev1`. El error de reducción de salario es capturado correctamente.

6. Prueba de Java:

   ```bash
   # En macOS / Linux
   javac src/java-comparacion/SistemaEmpleados.java && java -cp src/java-comparacion SistemaEmpleados

   # En Windows (PowerShell)
   javac src\java-comparacion\SistemaEmpleados.java; java -cp src\java-comparacion SistemaEmpleados
   ```
   **Resultado Esperado:** Masa laboral total de $176,000 con 4 empleados.

7. Prueba de Git:

   ```bash
   git log --oneline --graph --all
   ```
   **Resultado Esperado:** Al menos 3 commits visibles con mensajes que siguen la convención `feat:`, `docs:`, etc.

8. Verificar que `.class` está ignorado:

   ```bash
   git status
   ```
   **Resultado Esperado:** Los archivos `.class` generados por `javac` NO aparecen en la lista de archivos sin seguimiento.

---

## Solución de Problemas

### Problema 1: Error `Cannot use import statement in a module`

**Síntomas:**
- Al ejecutar `node src/js-moderno/index.js` aparece el error: `SyntaxError: Cannot use import statement outside a module`

**Causa:**
Node.js está tratando el archivo como CommonJS en lugar de ES Module. Esto ocurre cuando falta `"type": "module"` en `package.json` o cuando el archivo no tiene extensión `.mjs`.

**Solución:**

```bash
# Verificar que package.json tiene "type": "module"
cat package.json | grep type
# Debe mostrar: "type": "module",

# Si no aparece, editar package.json y agregar la línea:
# Abrir en VS Code
code package.json
# Agregar "type": "module" dentro del objeto JSON principal
```

---

### Problema 2: Error `javac: command not found` o `'javac' no se reconoce`

**Síntomas:**
- Al ejecutar `javac SistemaEmpleados.java` aparece: `command not found` (macOS/Linux) o `'javac' no se reconoce como un comando` (Windows)

**Causa:**
`JAVA_HOME` no está configurado o el directorio `bin` del JDK no está en el `PATH`.

**Solución:**

```bash
# Verificar instalación de Java
java --version

# En macOS - encontrar la ubicación del JDK
/usr/libexec/java_home -V

# En macOS - agregar al PATH en ~/.zshrc o ~/.bash_profile
echo 'export JAVA_HOME=$(/usr/libexec/java_home -v 17)' >> ~/.zshrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.zshrc
source ~/.zshrc

# En Windows (PowerShell) - verificar JAVA_HOME
echo $env:JAVA_HOME
# Si está vacío, configurarlo:
[System.Environment]::SetEnvironmentVariable("JAVA_HOME", "C:\Program Files\Java\jdk-17", "User")
$env:PATH += ";$env:JAVA_HOME\bin"

# En Linux (Ubuntu/Debian)
sudo apt update && sudo apt install openjdk-17-jdk -y
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
```

---

### Problema 3: Error de autenticación al hacer `git push`

**Síntomas:**
- Al ejecutar `git push -u origin main` aparece: `remote: Support for password authentication was removed` o `Authentication failed`

**Causa:**
GitHub eliminó la autenticación por contraseña en agosto de 2021. Ahora se requiere un **Personal Access Token (PAT)** o autenticación SSH.

**Solución (Personal Access Token):**

```bash
# 1. Generar un PAT en GitHub:
#    GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
#    → Generate new token → Seleccionar scope: repo → Generate token
#    IMPORTANTE: Copiar el token, no se mostrará de nuevo

# 2. Usar el token como contraseña al hacer push:
git push -u origin main
# Username: TU_USUARIO_GITHUB
# Password: PEGAR_EL_TOKEN_AQUI (no tu contraseña de GitHub)

# 3. Para no ingresar credenciales cada vez, configurar el credential helper:
# En macOS
git config --global credential.helper osxkeychain

# En Windows
git config --global credential.helper manager

# En Linux
git config --global credential.helper store
# NOTA: En Linux, 'store' guarda las credenciales en texto plano.
# Para mayor seguridad, usar 'cache' que las guarda temporalmente:
git config --global credential.helper "cache --timeout=3600"
```

---

### Problema 4: Los archivos `.class` aparecen en `git status`

**Síntomas:**
- Después de compilar con `javac`, los archivos `.class` aparecen como `Untracked files` en `git status`

**Causa:**
El archivo `.gitignore` no fue creado correctamente o no contiene la entrada `*.class`.

**Solución:**

```bash
# Verificar el contenido actual del .gitignore
cat .gitignore

# Si no contiene *.class, agregar la línea:
echo "*.class" >> .gitignore

# Si los archivos .class ya fueron rastreados accidentalmente, removerlos del índice:
git rm --cached src/java-comparacion/*.class

# Verificar que ahora están ignorados
git status
# Los archivos .class no deben aparecer
```

---

### Problema 5: Error `Cannot find module './utilidades.js'`

**Síntomas:**
- Al ejecutar el `index.js` aparece: `Error [ERR_MODULE_NOT_FOUND]: Cannot find module '/ruta/src/js-moderno/utilidades.js'`

**Causa:**
En módulos ES de Node.js, las importaciones deben incluir la extensión del archivo (`.js`). A diferencia de CommonJS, los módulos ES no resuelven extensiones automáticamente.

**Solución:**

```bash
# Verificar que todas las importaciones incluyen la extensión .js
grep -r "from './" src/js-moderno/
# Todas las líneas de import deben terminar en .js

# Ejemplo correcto:
# import { formatearMoneda } from "./utilidades.js";   ✅

# Ejemplo incorrecto:
# import { formatearMoneda } from "./utilidades";      ❌
```

---

## Limpieza

Una vez completado el laboratorio, puedes limpiar los archivos generados durante la compilación de Java:

```bash
# Eliminar archivos .class compilados de Java
# En macOS / Linux
find src/java-comparacion -name "*.class" -delete

# En Windows (PowerShell)
Get-ChildItem -Path "src\java-comparacion" -Filter "*.class" | Remove-Item

# Verificar que fueron eliminados
ls src/java-comparacion/
# Solo debe mostrar SistemaEmpleados.java

# Verificar que git no rastrea los archivos eliminados
git status
# No debe mostrar cambios pendientes
```

> ⚠️ **Advertencia:** NO elimines los archivos del repositorio Git ni el directorio `node_modules` si fue creado. El directorio `node_modules` debería estar en `.gitignore` y no debería existir ya que no instalamos dependencias externas en este laboratorio. Si aparece, puedes eliminarlo con `rm -rf node_modules` (macOS/Linux) o `Remove-Item -Recurse -Force node_modules` (Windows).

> ⚠️ **Nota sobre el repositorio GitHub:** El repositorio `curso-fullstack-lab01` en GitHub debe **mantenerse** ya que se usará como referencia en laboratorios posteriores. No lo elimines al finalizar.

---

## Resumen

### Lo que Lograste

- **Configuraste** un proyecto Node.js moderno con soporte para módulos ES (`"type": "module"`) y scripts npm personalizados
- **Aplicaste** características ES6+ en código funcional real: `let`/`const`, arrow functions, template literals, desestructuración, spread/rest, Promises y `async/await`
- **Implementaste** operaciones asíncronas robustas con manejo de errores usando `try/catch`, `Promise.all` para ejecución paralela y `Promise.allSettled` para tolerancia a fallos
- **Organizaste** código en módulos ES con exportaciones nombradas y por defecto, estableciendo un patrón de arquitectura modular
- **Diseñaste** una jerarquía de clases JavaScript con herencia (`extends`), encapsulamiento (campos `#privados`), polimorfismo (sobrescritura de métodos) y miembros estáticos
- **Comparaste** JavaScript y Java 17 identificando equivalencias en POO, colecciones (`Array` vs `List`), iteración (`map/filter/reduce` vs `Stream`), y manejo de excepciones
- **Configuraste** Git con identidad, `.gitignore` y flujo de trabajo con ramas, realizando el primer push a GitHub con commits descriptivos siguiendo Conventional Commits

### Conceptos Clave Aprendidos

- **Alcance de bloque**: `let` y `const` tienen alcance de bloque `{}`, evitando los problemas de `var` con hoisting y scope de función
- **Arrow functions**: Heredan el contexto `this` léxicamente, ideales para callbacks y métodos de array; no deben usarse como métodos de objetos
- **Desestructuración**: Permite extraer valores de objetos y arrays en variables con nombres específicos, con soporte para valores por defecto y alias
- **Asincronía**: `async/await` hace el código asíncrono legible; `Promise.all` optimiza operaciones independientes ejecutándolas en paralelo
- **Módulos ES**: Cada archivo es un módulo con su propio scope; `export` expone elementos, `import` los consume con resolución estática en tiempo de carga
- **Encapsulamiento JS**: Los campos `#privados` son verdaderamente privados (no accesibles fuera de la clase), a diferencia de las convenciones `_privado` anteriores
- **Conventional Commits**: Formato `tipo(scope): descripción` para commits legibles y compatibles con herramientas de changelog automático

### Próximos Pasos

- En la **Lección 1.2** profundizarás en POO avanzada con JavaScript: mixins, composición vs herencia, y patrones de diseño aplicados a clases
- En el **Lab 02** aplicarás pruebas unitarias con JUnit y Mockito en el backend Java, usando los conceptos de POO aprendidos hoy
- Practica los conceptos de este laboratorio modificando el `ServicioEmpleados` para agregar un método `buscarPorSalario(min, max)` usando `filter` y desestructuración
- Explora la documentación de MDN sobre [Clases de JavaScript](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Classes) para profundizar en características avanzadas como campos estáticos privados y métodos de clase

---

## Recursos Adicionales

- **MDN Web Docs - JavaScript**: Referencia completa de todas las características ES6+ con ejemplos interactivos - [https://developer.mozilla.org/es/docs/Web/JavaScript](https://developer.mozilla.org/es/docs/Web/JavaScript)
- **JavaScript.info**: Tutorial moderno y profundo de JavaScript en español, desde fundamentos hasta temas avanzados - [https://javascript.info](https://javascript.info)
- **Conventional Commits**: Especificación completa del estándar para mensajes de commit estructurados - [https://www.conventionalcommits.org/es/v1.0.0/](https://www.conventionalcommits.org/es/v1.0.0/)
- **Node.js - ES Modules**: Documentación oficial sobre el uso de módulos ES en Node.js - [https://nodejs.org/api/esm.html](https://nodejs.org/api/esm.html)
- **Git - Pro Git Book**: Libro completo y gratuito sobre Git en español - [https://git-scm.com/book/es/v2](https://git-scm.com/book/es/v2)
- **Java Streams API**: Documentación oficial de la API de Streams de Java 17 - [https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html)
- **TC39 Proposals**: Repositorio oficial de propuestas de nuevas características para JavaScript - [https://github.com/tc39/proposals](https://github.com/tc39/proposals)
