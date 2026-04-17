---LAB_START---
LAB_ID: 02-00-01
---MARKDOWN---
# Laboratorio 2. Proyecto Integrado usando tecnologías Frontend

## Metadatos

| Campo            | Valor                                                                 |
|------------------|-----------------------------------------------------------------------|
| **Duración**     | 260 minutos (recomendado: 3 sesiones de ~90 min con pausas)          |
| **Complejidad**  | Alta                                                                  |
| **Nivel Bloom**  | Crear                                                                 |
| **Módulo**       | Módulo 2 — Tecnologías Frontend Modernas                             |
| **Laboratorio**  | Lab 02-00-01                                                          |

---

## Descripción General

En este laboratorio construirás una **aplicación web de página única (SPA)** que funciona como catálogo interactivo de productos empresariales. Comenzarás desde cero con HTML5 semántico y CSS3 avanzado, añadirás interactividad con JavaScript ES6+, refactorizarás la interfaz hacia Web Components nativos y finalmente migrarás los componentes a **Lit 3.x** con propiedades reactivas y Shadow DOM. Al finalizar, tendrás un proyecto frontend modular, probado con Vitest y listo para integrarse con el backend del Laboratorio 7.

> **Punto de pausa sugerido para el instructor:**
> - **Sesión 1 (0–90 min):** Pasos 1–3 (Entorno, HTML5 y CSS3)
> - **Sesión 2 (90–180 min):** Pasos 4–6 (JavaScript, Web Components nativos)
> - **Sesión 3 (180–260 min):** Pasos 7–9 (Lit, comunicación entre componentes, pruebas)

---

## Objetivos de Aprendizaje

- [ ] Construir la estructura HTML5 semántica completa de una SPA usando elementos como `<header>`, `<main>`, `<section>`, `<article>`, `<aside>` y `<footer>`, aplicando atributos `data-*` para enlazar datos con comportamiento JavaScript.
- [ ] Aplicar CSS3 avanzado con Flexbox, CSS Grid, Custom Properties (variables CSS) y animaciones para crear una interfaz responsiva y visualmente coherente.
- [ ] Implementar Web Components nativos con el ciclo de vida completo (`connectedCallback`, `disconnectedCallback`, `attributeChangedCallback`) y Shadow DOM para encapsular estilos y comportamiento.
- [ ] Desarrollar componentes Lit con `@property`, `@state`, directivas `repeat` e `ifDefined`, y establecer comunicación padre-hijo mediante `CustomEvent`.
- [ ] Configurar un entorno profesional con Vite como bundler y escribir pruebas unitarias de componentes con Vitest.

---

## Prerrequisitos

### Conocimientos

| Área                     | Nivel requerido                                              |
|--------------------------|--------------------------------------------------------------|
| HTML y CSS básico        | Estructura de documento, selectores, box model              |
| JavaScript               | Variables, funciones, arrays, objetos, eventos básicos      |
| POO                      | Clases, herencia, encapsulamiento                           |
| Terminal/Línea de comandos | Navegación básica, ejecución de comandos npm               |

### Software y Acceso

| Herramienta          | Versión mínima | Verificación                     |
|----------------------|----------------|----------------------------------|
| Node.js              | 20.x LTS       | `node -v`                        |
| npm                  | 10.x           | `npm -v`                         |
| Visual Studio Code   | 1.88+          | Extensiones: Lit-plugin, ESLint  |
| Google Chrome        | Última estable | DevTools disponibles             |
| Git                  | 2.44+          | `git --version`                  |

> **Checkpoint de recuperación:** Si no completaste un laboratorio anterior, clona la rama de inicio:
> ```bash
> git clone https://github.com/curso-fullstack/labs.git
> cd labs
> git checkout lab02-start
> ```

---

## Entorno de Laboratorio

### Verificación del Entorno

Antes de comenzar, ejecuta los siguientes comandos en tu terminal para confirmar que el entorno está listo:

```bash
# Verificar Node.js y npm
node -v    # Debe mostrar v20.x.x
npm -v     # Debe mostrar 10.x.x

# Verificar Git
git --version   # Debe mostrar 2.44.x o superior
```

### Extensiones de VS Code Recomendadas

Abre VS Code y verifica que estas extensiones estén instaladas:

```
lit-plugin           (runem.lit-plugin)
ESLint               (dbaeumer.vscode-eslint)
Prettier             (esbenp.prettier-vscode)
```

Para instalarlas desde la terminal:

```bash
code --install-extension runem.lit-plugin
code --install-extension dbaeumer.vscode-eslint
code --install-extension esbenp.prettier-vscode
```

---

## Instrucciones Paso a Paso

---

### Paso 1 — Inicialización del Proyecto con Vite

**Objetivo:** Crear la estructura base del proyecto usando Vite como bundler y configurar las herramientas de calidad de código.

**Instrucciones:**

1. Abre una terminal en el directorio donde deseas crear el proyecto y ejecuta:

```bash
npm create vite@latest catalogo-productos -- --template vanilla
cd catalogo-productos
npm install
```

2. Instala las dependencias principales del proyecto:

```bash
# Dependencias de producción
npm install lit

# Dependencias de desarrollo
npm install -D vitest @web/test-runner eslint prettier eslint-config-prettier
```

3. Reemplaza el contenido de `package.json` con la siguiente configuración completa:

```json
{
  "name": "catalogo-productos",
  "version": "1.0.0",
  "description": "Catálogo interactivo de productos - Lab 02",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "test": "vitest run",
    "test:watch": "vitest",
    "lint": "eslint src --ext .js",
    "format": "prettier --write src/**/*.{js,html,css}"
  },
  "dependencies": {
    "lit": "^3.1.0"
  },
  "devDependencies": {
    "vite": "^5.2.0",
    "vitest": "^1.5.0",
    "eslint": "^8.57.0",
    "prettier": "^3.2.5",
    "eslint-config-prettier": "^9.1.0"
  }
}
```

4. Crea el archivo de configuración de Vite `vite.config.js` en la raíz del proyecto:

```javascript
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  server: {
    port: 3000,
    open: true
  },
  build: {
    outDir: 'dist',
    sourcemap: true
  },
  test: {
    environment: 'happy-dom',
    globals: true
  }
});
```

5. Crea el archivo `.eslintrc.json` en la raíz:

```json
{
  "env": {
    "browser": true,
    "es2022": true
  },
  "extends": ["eslint:recommended", "prettier"],
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "rules": {
    "no-unused-vars": "warn",
    "no-console": "off"
  }
}
```

6. Crea el archivo `.prettierrc` en la raíz:

```json
{
  "singleQuote": true,
  "semi": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100
}
```

7. Limpia los archivos de ejemplo que genera Vite:

```bash
# Elimina archivos de ejemplo
rm src/main.js src/style.css src/counter.js
rm public/vite.svg
```

8. Crea la estructura de directorios del proyecto:

```bash
mkdir -p src/components src/styles src/data src/utils src/tests
```

La estructura final debe verse así:

```
catalogo-productos/
├── src/
│   ├── components/
│   ├── styles/
│   ├── data/
│   ├── utils/
│   └── tests/
├── public/
├── index.html
├── vite.config.js
├── package.json
├── .eslintrc.json
└── .prettierrc
```

**Resultado Esperado:** El directorio `catalogo-productos` contiene la estructura de proyecto completa sin errores de instalación.

**Verificación:**

```bash
npm run dev
# El servidor debe iniciar en http://localhost:3000 sin errores
# Presiona Ctrl+C para detenerlo
```

---

### Paso 2 — Estructura HTML5 Semántica

**Objetivo:** Construir el documento HTML5 base de la SPA con estructura semántica completa, usando los elementos aprendidos en la Lección 2.1.

**Instrucciones:**

1. Reemplaza el contenido de `index.html` en la raíz del proyecto con la siguiente estructura semántica completa:

```html
<!DOCTYPE html>
<html lang="es">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta
      name="description"
      content="Catálogo interactivo de productos empresariales"
    />
    <meta name="author" content="Equipo de Desarrollo" />
    <title>CatálogoTech — Gestión de Productos</title>
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <link rel="stylesheet" href="/src/styles/main.css" />
  </head>
  <body>
    <!-- Cabecera principal de la aplicación -->
    <header class="app-header" role="banner">
      <div class="header-brand">
        <span class="brand-icon" aria-hidden="true">🛒</span>
        <h1 class="brand-title">CatálogoTech</h1>
        <span class="brand-subtitle">Gestión de Productos</span>
      </div>

      <nav class="header-nav" role="navigation" aria-label="Navegación principal">
        <ul class="nav-list">
          <li class="nav-item">
            <a href="#catalogo" class="nav-link active" data-section="catalogo">
              Catálogo
            </a>
          </li>
          <li class="nav-item">
            <a href="#agregar" class="nav-link" data-section="agregar">
              Agregar Producto
            </a>
          </li>
          <li class="nav-item">
            <a href="#estadisticas" class="nav-link" data-section="estadisticas">
              Estadísticas
            </a>
          </li>
        </ul>
      </nav>

      <div class="header-actions">
        <button
          class="btn-icon"
          id="btn-toggle-theme"
          aria-label="Cambiar tema"
          title="Alternar tema claro/oscuro"
        >
          🌙
        </button>
        <div class="cart-indicator" aria-live="polite">
          <span class="cart-icon">🛍️</span>
          <span class="cart-count" id="cart-count" data-count="0">0</span>
        </div>
      </div>
    </header>

    <!-- Contenido principal de la aplicación -->
    <main class="app-main" id="app-main" role="main">

      <!-- Sección: Barra de búsqueda y filtros -->
      <section
        class="search-section"
        aria-labelledby="search-heading"
        id="seccion-busqueda"
      >
        <h2 id="search-heading" class="visually-hidden">
          Búsqueda y filtros
        </h2>

        <div class="search-bar">
          <label for="input-busqueda" class="visually-hidden">
            Buscar productos
          </label>
          <input
            type="search"
            id="input-busqueda"
            class="search-input"
            placeholder="Buscar productos por nombre o categoría..."
            aria-label="Buscar productos"
            autocomplete="off"
          />
          <button class="btn-search" aria-label="Ejecutar búsqueda">
            🔍
          </button>
        </div>

        <div class="filter-bar" role="group" aria-label="Filtros de categoría">
          <button
            class="filter-chip active"
            data-categoria="todas"
            aria-pressed="true"
          >
            Todas
          </button>
          <button
            class="filter-chip"
            data-categoria="electronica"
            aria-pressed="false"
          >
            Electrónica
          </button>
          <button
            class="filter-chip"
            data-categoria="software"
            aria-pressed="false"
          >
            Software
          </button>
          <button
            class="filter-chip"
            data-categoria="perifericos"
            aria-pressed="false"
          >
            Periféricos
          </button>
          <button
            class="filter-chip"
            data-categoria="servicios"
            aria-pressed="false"
          >
            Servicios
          </button>
        </div>
      </section>

      <!-- Sección: Catálogo de productos -->
      <section
        class="catalog-section"
        aria-labelledby="catalog-heading"
        id="catalogo"
      >
        <div class="section-header">
          <h2 id="catalog-heading">Catálogo de Productos</h2>
          <p class="product-count" id="product-count" aria-live="polite">
            Cargando productos...
          </p>
        </div>

        <!-- Contenedor donde se inyectarán los componentes de productos -->
        <div
          class="products-grid"
          id="products-grid"
          role="list"
          aria-label="Lista de productos"
        >
          <!-- Los productos se renderizan dinámicamente aquí -->
        </div>
      </section>

      <!-- Sección: Formulario para agregar producto -->
      <section
        class="form-section"
        aria-labelledby="form-heading"
        id="agregar"
        hidden
      >
        <h2 id="form-heading">Agregar Nuevo Producto</h2>

        <form
          class="product-form"
          id="form-producto"
          novalidate
          aria-label="Formulario de nuevo producto"
        >
          <fieldset class="form-fieldset">
            <legend>Información del Producto</legend>

            <div class="form-group">
              <label for="producto-nombre">
                Nombre del producto <span aria-hidden="true">*</span>
              </label>
              <input
                type="text"
                id="producto-nombre"
                name="nombre"
                required
                minlength="3"
                maxlength="100"
                placeholder="Ej: Laptop Pro 15"
                data-field="nombre"
              />
              <span class="field-error" id="error-nombre" role="alert"></span>
            </div>

            <div class="form-group">
              <label for="producto-categoria">Categoría</label>
              <select id="producto-categoria" name="categoria" required data-field="categoria">
                <option value="">Selecciona una categoría</option>
                <option value="electronica">Electrónica</option>
                <option value="software">Software</option>
                <option value="perifericos">Periféricos</option>
                <option value="servicios">Servicios</option>
              </select>
              <span class="field-error" id="error-categoria" role="alert"></span>
            </div>

            <div class="form-group">
              <label for="producto-precio">
                Precio (USD) <span aria-hidden="true">*</span>
              </label>
              <input
                type="number"
                id="producto-precio"
                name="precio"
                required
                min="0.01"
                max="99999.99"
                step="0.01"
                placeholder="0.00"
                data-field="precio"
              />
              <span class="field-error" id="error-precio" role="alert"></span>
            </div>

            <div class="form-group">
              <label for="producto-descripcion">Descripción</label>
              <textarea
                id="producto-descripcion"
                name="descripcion"
                rows="3"
                maxlength="500"
                placeholder="Descripción breve del producto..."
                data-field="descripcion"
              ></textarea>
            </div>

            <div class="form-group">
              <label for="producto-stock">Stock disponible</label>
              <input
                type="number"
                id="producto-stock"
                name="stock"
                min="0"
                value="0"
                data-field="stock"
              />
            </div>
          </fieldset>

          <div class="form-actions">
            <button type="submit" class="btn btn-primary">
              Guardar Producto
            </button>
            <button type="reset" class="btn btn-secondary">
              Limpiar
            </button>
          </div>
        </form>
      </section>

      <!-- Sección: Estadísticas -->
      <section
        class="stats-section"
        aria-labelledby="stats-heading"
        id="estadisticas"
        hidden
      >
        <h2 id="stats-heading">Estadísticas del Catálogo</h2>
        <div class="stats-grid" id="stats-grid">
          <!-- Tarjetas de estadísticas se renderizan dinámicamente -->
        </div>

        <figure class="stats-chart">
          <canvas
            id="chart-categorias"
            width="600"
            height="300"
            aria-label="Gráfico de distribución por categorías"
            role="img"
          ></canvas>
          <figcaption>Distribución de productos por categoría</figcaption>
        </figure>
      </section>
    </main>

    <!-- Barra lateral: detalles del producto seleccionado -->
    <aside
      class="product-detail-panel"
      id="detail-panel"
      role="complementary"
      aria-label="Detalle del producto"
      hidden
    >
      <div class="panel-header">
        <h3>Detalle del Producto</h3>
        <button
          class="btn-icon btn-close-panel"
          id="btn-close-panel"
          aria-label="Cerrar panel de detalle"
        >
          ✕
        </button>
      </div>
      <div class="panel-content" id="panel-content">
        <!-- Contenido dinámico -->
      </div>
    </aside>

    <!-- Pie de página -->
    <footer class="app-footer" role="contentinfo">
      <div class="footer-content">
        <p class="footer-copy">
          &copy; 2024 CatálogoTech — Laboratorio de Tecnologías Frontend
        </p>
        <nav aria-label="Navegación de pie de página">
          <a href="#" class="footer-link">Documentación</a>
          <a href="#" class="footer-link">API Reference</a>
          <a href="#" class="footer-link">Soporte</a>
        </nav>
      </div>
    </footer>

    <!-- Notificaciones tipo toast -->
    <div
      class="toast-container"
      id="toast-container"
      aria-live="assertive"
      aria-atomic="true"
      role="status"
    ></div>

    <!-- Punto de entrada JavaScript -->
    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```

**Resultado Esperado:** El archivo `index.html` contiene una estructura HTML5 semántica completa con `<header>`, `<nav>`, `<main>`, múltiples `<section>`, `<aside>`, `<footer>` y atributos ARIA correctos.

**Verificación:** Abre Chrome DevTools (F12) → pestaña **Elements** y verifica que el árbol DOM muestra la jerarquía semántica correcta. En la pestaña **Accessibility** verifica que el árbol de accesibilidad reconoce los landmarks.

---

### Paso 3 — Estilos CSS3 con Variables, Flexbox y Grid

**Objetivo:** Aplicar CSS3 avanzado para crear la interfaz visual responsiva usando Custom Properties, Flexbox y CSS Grid.

**Instrucciones:**

1. Crea el archivo `src/styles/main.css`:

```css
/* ============================================================
   VARIABLES CSS (Custom Properties)
   ============================================================ */
:root {
  /* Paleta de colores */
  --color-primary: #2563eb;
  --color-primary-dark: #1d4ed8;
  --color-primary-light: #dbeafe;
  --color-secondary: #7c3aed;
  --color-success: #16a34a;
  --color-warning: #d97706;
  --color-danger: #dc2626;

  /* Escala de grises */
  --color-bg: #f8fafc;
  --color-surface: #ffffff;
  --color-surface-alt: #f1f5f9;
  --color-border: #e2e8f0;
  --color-text: #1e293b;
  --color-text-muted: #64748b;
  --color-text-inverse: #ffffff;

  /* Tipografía */
  --font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  --font-size-2xl: 1.5rem;
  --font-size-3xl: 1.875rem;

  /* Espaciado */
  --spacing-1: 0.25rem;
  --spacing-2: 0.5rem;
  --spacing-3: 0.75rem;
  --spacing-4: 1rem;
  --spacing-6: 1.5rem;
  --spacing-8: 2rem;
  --spacing-12: 3rem;

  /* Bordes y sombras */
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-xl: 1rem;
  --radius-full: 9999px;
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.07);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
  --shadow-hover: 0 20px 25px rgba(0, 0, 0, 0.12);

  /* Transiciones */
  --transition-fast: 150ms ease;
  --transition-base: 250ms ease;
  --transition-slow: 400ms ease;

  /* Layout */
  --header-height: 64px;
  --sidebar-width: 360px;
  --content-max-width: 1400px;
}

/* Tema oscuro */
[data-theme='dark'] {
  --color-bg: #0f172a;
  --color-surface: #1e293b;
  --color-surface-alt: #334155;
  --color-border: #475569;
  --color-text: #f1f5f9;
  --color-text-muted: #94a3b8;
  --color-primary-light: #1e3a5f;
}

/* ============================================================
   RESET Y BASE
   ============================================================ */
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html {
  scroll-behavior: smooth;
  font-size: 16px;
}

body {
  font-family: var(--font-family);
  font-size: var(--font-size-base);
  color: var(--color-text);
  background-color: var(--color-bg);
  line-height: 1.6;
  min-height: 100vh;
  display: grid;
  grid-template-rows: var(--header-height) 1fr auto;
  grid-template-columns: 1fr;
  grid-template-areas:
    'header'
    'main'
    'footer';
}

.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

/* ============================================================
   HEADER
   ============================================================ */
.app-header {
  grid-area: header;
  position: sticky;
  top: 0;
  z-index: 100;
  background-color: var(--color-surface);
  border-bottom: 1px solid var(--color-border);
  box-shadow: var(--shadow-sm);
  display: flex;
  align-items: center;
  gap: var(--spacing-6);
  padding: 0 var(--spacing-6);
  height: var(--header-height);
}

.header-brand {
  display: flex;
  align-items: center;
  gap: var(--spacing-2);
  flex-shrink: 0;
}

.brand-icon {
  font-size: var(--font-size-xl);
}

.brand-title {
  font-size: var(--font-size-xl);
  font-weight: 700;
  color: var(--color-primary);
}

.brand-subtitle {
  font-size: var(--font-size-xs);
  color: var(--color-text-muted);
  display: none;
}

@media (min-width: 768px) {
  .brand-subtitle {
    display: inline;
  }
}

.header-nav {
  flex: 1;
}

.nav-list {
  display: flex;
  list-style: none;
  gap: var(--spacing-1);
}

.nav-link {
  display: inline-block;
  padding: var(--spacing-2) var(--spacing-4);
  border-radius: var(--radius-md);
  text-decoration: none;
  color: var(--color-text-muted);
  font-size: var(--font-size-sm);
  font-weight: 500;
  transition: all var(--transition-fast);
}

.nav-link:hover,
.nav-link.active {
  color: var(--color-primary);
  background-color: var(--color-primary-light);
}

.header-actions {
  display: flex;
  align-items: center;
  gap: var(--spacing-3);
  flex-shrink: 0;
}

/* ============================================================
   BOTONES
   ============================================================ */
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--spacing-2);
  padding: var(--spacing-2) var(--spacing-4);
  border: none;
  border-radius: var(--radius-md);
  font-size: var(--font-size-sm);
  font-weight: 500;
  cursor: pointer;
  transition: all var(--transition-fast);
  text-decoration: none;
}

.btn-primary {
  background-color: var(--color-primary);
  color: var(--color-text-inverse);
}

.btn-primary:hover {
  background-color: var(--color-primary-dark);
  box-shadow: var(--shadow-md);
  transform: translateY(-1px);
}

.btn-secondary {
  background-color: var(--color-surface-alt);
  color: var(--color-text);
  border: 1px solid var(--color-border);
}

.btn-secondary:hover {
  background-color: var(--color-border);
}

.btn-icon {
  width: 36px;
  height: 36px;
  padding: 0;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  background-color: var(--color-surface);
  cursor: pointer;
  font-size: var(--font-size-base);
  transition: all var(--transition-fast);
  display: flex;
  align-items: center;
  justify-content: center;
}

.btn-icon:hover {
  background-color: var(--color-surface-alt);
}

/* ============================================================
   MAIN LAYOUT
   ============================================================ */
.app-main {
  grid-area: main;
  max-width: var(--content-max-width);
  width: 100%;
  margin: 0 auto;
  padding: var(--spacing-6);
  display: flex;
  flex-direction: column;
  gap: var(--spacing-8);
}

/* ============================================================
   SECCIÓN DE BÚSQUEDA Y FILTROS
   ============================================================ */
.search-section {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-4);
}

.search-bar {
  display: flex;
  gap: var(--spacing-2);
}

.search-input {
  flex: 1;
  padding: var(--spacing-3) var(--spacing-4);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-lg);
  font-size: var(--font-size-base);
  background-color: var(--color-surface);
  color: var(--color-text);
  transition: border-color var(--transition-fast);
}

.search-input:focus {
  outline: none;
  border-color: var(--color-primary);
  box-shadow: 0 0 0 3px var(--color-primary-light);
}

.btn-search {
  padding: var(--spacing-3) var(--spacing-4);
  background-color: var(--color-primary);
  color: white;
  border: none;
  border-radius: var(--radius-lg);
  cursor: pointer;
  font-size: var(--font-size-base);
  transition: background-color var(--transition-fast);
}

.btn-search:hover {
  background-color: var(--color-primary-dark);
}

.filter-bar {
  display: flex;
  flex-wrap: wrap;
  gap: var(--spacing-2);
}

.filter-chip {
  padding: var(--spacing-1) var(--spacing-3);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-full);
  background-color: var(--color-surface);
  color: var(--color-text-muted);
  font-size: var(--font-size-sm);
  cursor: pointer;
  transition: all var(--transition-fast);
}

.filter-chip:hover,
.filter-chip.active {
  background-color: var(--color-primary);
  color: white;
  border-color: var(--color-primary);
}

/* ============================================================
   CATÁLOGO — CSS GRID
   ============================================================ */
.catalog-section .section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: var(--spacing-4);
}

.catalog-section h2 {
  font-size: var(--font-size-2xl);
  font-weight: 700;
}

.product-count {
  font-size: var(--font-size-sm);
  color: var(--color-text-muted);
}

.products-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: var(--spacing-6);
}

/* ============================================================
   TARJETA DE PRODUCTO
   ============================================================ */
.product-card {
  background-color: var(--color-surface);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-xl);
  padding: var(--spacing-6);
  display: flex;
  flex-direction: column;
  gap: var(--spacing-3);
  box-shadow: var(--shadow-sm);
  transition: all var(--transition-base);
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.product-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: linear-gradient(90deg, var(--color-primary), var(--color-secondary));
  opacity: 0;
  transition: opacity var(--transition-base);
}

.product-card:hover {
  box-shadow: var(--shadow-hover);
  transform: translateY(-4px);
  border-color: var(--color-primary);
}

.product-card:hover::before {
  opacity: 1;
}

.product-card-badge {
  display: inline-flex;
  align-items: center;
  padding: var(--spacing-1) var(--spacing-2);
  border-radius: var(--radius-full);
  font-size: var(--font-size-xs);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  background-color: var(--color-primary-light);
  color: var(--color-primary);
  width: fit-content;
}

.product-card-name {
  font-size: var(--font-size-lg);
  font-weight: 600;
  color: var(--color-text);
}

.product-card-description {
  font-size: var(--font-size-sm);
  color: var(--color-text-muted);
  line-height: 1.5;
  flex: 1;
}

.product-card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: var(--spacing-3);
  border-top: 1px solid var(--color-border);
}

.product-card-price {
  font-size: var(--font-size-xl);
  font-weight: 700;
  color: var(--color-primary);
}

.product-card-stock {
  font-size: var(--font-size-xs);
  color: var(--color-text-muted);
}

.product-card-stock.low-stock {
  color: var(--color-warning);
  font-weight: 600;
}

/* ============================================================
   FORMULARIO
   ============================================================ */
.form-section {
  max-width: 640px;
}

.form-section h2 {
  font-size: var(--font-size-2xl);
  font-weight: 700;
  margin-bottom: var(--spacing-6);
}

.product-form {
  background-color: var(--color-surface);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-xl);
  padding: var(--spacing-8);
  box-shadow: var(--shadow-md);
}

.form-fieldset {
  border: none;
  padding: 0;
  margin: 0 0 var(--spacing-6) 0;
}

.form-fieldset legend {
  font-size: var(--font-size-lg);
  font-weight: 600;
  margin-bottom: var(--spacing-6);
  color: var(--color-text);
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-1);
  margin-bottom: var(--spacing-4);
}

.form-group label {
  font-size: var(--font-size-sm);
  font-weight: 500;
  color: var(--color-text);
}

.form-group input,
.form-group select,
.form-group textarea {
  padding: var(--spacing-3);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  font-size: var(--font-size-base);
  background-color: var(--color-surface);
  color: var(--color-text);
  transition: border-color var(--transition-fast);
  font-family: var(--font-family);
}

.form-group input:focus,
.form-group select:focus,
.form-group textarea:focus {
  outline: none;
  border-color: var(--color-primary);
  box-shadow: 0 0 0 3px var(--color-primary-light);
}

.form-group input.invalid,
.form-group select.invalid {
  border-color: var(--color-danger);
  box-shadow: 0 0 0 3px rgba(220, 38, 38, 0.1);
}

.field-error {
  font-size: var(--font-size-xs);
  color: var(--color-danger);
  min-height: 1rem;
}

.form-actions {
  display: flex;
  gap: var(--spacing-3);
  justify-content: flex-end;
}

/* ============================================================
   ESTADÍSTICAS
   ============================================================ */
.stats-section h2 {
  font-size: var(--font-size-2xl);
  font-weight: 700;
  margin-bottom: var(--spacing-6);
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: var(--spacing-4);
  margin-bottom: var(--spacing-8);
}

.stat-card {
  background-color: var(--color-surface);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-xl);
  padding: var(--spacing-6);
  text-align: center;
  box-shadow: var(--shadow-sm);
}

.stat-card-value {
  font-size: var(--font-size-3xl);
  font-weight: 700;
  color: var(--color-primary);
}

.stat-card-label {
  font-size: var(--font-size-sm);
  color: var(--color-text-muted);
  margin-top: var(--spacing-1);
}

.stats-chart {
  background-color: var(--color-surface);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-xl);
  padding: var(--spacing-6);
  box-shadow: var(--shadow-sm);
}

.stats-chart figcaption {
  text-align: center;
  font-size: var(--font-size-sm);
  color: var(--color-text-muted);
  margin-top: var(--spacing-3);
}

/* ============================================================
   PANEL LATERAL (ASIDE)
   ============================================================ */
.product-detail-panel {
  position: fixed;
  top: var(--header-height);
  right: 0;
  width: var(--sidebar-width);
  height: calc(100vh - var(--header-height));
  background-color: var(--color-surface);
  border-left: 1px solid var(--color-border);
  box-shadow: var(--shadow-lg);
  z-index: 50;
  display: flex;
  flex-direction: column;
  transform: translateX(100%);
  transition: transform var(--transition-slow);
}

.product-detail-panel:not([hidden]) {
  transform: translateX(0);
  display: flex;
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: var(--spacing-4) var(--spacing-6);
  border-bottom: 1px solid var(--color-border);
}

.panel-header h3 {
  font-size: var(--font-size-lg);
  font-weight: 600;
}

.panel-content {
  flex: 1;
  overflow-y: auto;
  padding: var(--spacing-6);
}

/* ============================================================
   FOOTER
   ============================================================ */
.app-footer {
  grid-area: footer;
  background-color: var(--color-surface);
  border-top: 1px solid var(--color-border);
  padding: var(--spacing-4) var(--spacing-6);
}

.footer-content {
  max-width: var(--content-max-width);
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: var(--spacing-3);
}

.footer-copy {
  font-size: var(--font-size-sm);
  color: var(--color-text-muted);
}

.footer-link {
  font-size: var(--font-size-sm);
  color: var(--color-text-muted);
  text-decoration: none;
  margin-left: var(--spacing-4);
  transition: color var(--transition-fast);
}

.footer-link:hover {
  color: var(--color-primary);
}

/* ============================================================
   TOAST NOTIFICATIONS
   ============================================================ */
.toast-container {
  position: fixed;
  bottom: var(--spacing-6);
  right: var(--spacing-6);
  z-index: 200;
  display: flex;
  flex-direction: column;
  gap: var(--spacing-3);
}

.toast {
  padding: var(--spacing-3) var(--spacing-4);
  border-radius: var(--radius-lg);
  color: white;
  font-size: var(--font-size-sm);
  font-weight: 500;
  box-shadow: var(--shadow-lg);
  min-width: 280px;
  animation: slideInRight var(--transition-base) ease forwards;
}

.toast.success { background-color: var(--color-success); }
.toast.error   { background-color: var(--color-danger); }
.toast.warning { background-color: var(--color-warning); }

@keyframes slideInRight {
  from {
    opacity: 0;
    transform: translateX(100%);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes fadeOut {
  from { opacity: 1; }
  to   { opacity: 0; transform: translateY(-8px); }
}

/* ============================================================
   INDICADOR DE CARRITO
   ============================================================ */
.cart-indicator {
  display: flex;
  align-items: center;
  gap: var(--spacing-1);
  padding: var(--spacing-1) var(--spacing-3);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-full);
  font-size: var(--font-size-sm);
  cursor: pointer;
  transition: all var(--transition-fast);
}

.cart-indicator:hover {
  border-color: var(--color-primary);
  color: var(--color-primary);
}

.cart-count {
  font-weight: 700;
  min-width: 1.25rem;
  text-align: center;
}

/* ============================================================
   RESPONSIVE
   ============================================================ */
@media (max-width: 768px) {
  .app-header {
    padding: 0 var(--spacing-4);
    gap: var(--spacing-3);
  }

  .header-nav {
    display: none;
  }

  .products-grid {
    grid-template-columns: 1fr;
  }

  .product-detail-panel {
    width: 100%;
  }

  .form-actions {
    flex-direction: column;
  }
}
```

**Resultado Esperado:** La aplicación muestra la interfaz visual completa con header, sección de búsqueda, catálogo y footer con estilos aplicados.

**Verificación:** Ejecuta `npm run dev` y abre `http://localhost:3000`. La página debe mostrar el header con navegación, la barra de búsqueda y filtros. Redimensiona la ventana para verificar el comportamiento responsivo.

---

### Paso 4 — Datos y Lógica JavaScript ES6+

**Objetivo:** Crear el módulo de datos de productos y el controlador principal de la aplicación usando JavaScript ES6+ moderno.

**Instrucciones:**

1. Crea `src/data/productos.js` con los datos iniciales del catálogo:

```javascript
// src/data/productos.js

/**
 * Dataset inicial de productos del catálogo.
 * Cada producto usa atributos data-* en el HTML para enlazar con el DOM.
 */
export const productosIniciales = [
  {
    id: 1,
    nombre: 'Laptop Pro 15',
    categoria: 'electronica',
    precio: 1299.99,
    descripcion: 'Laptop empresarial con procesador Intel Core i7, 16GB RAM y SSD 512GB.',
    stock: 15,
    destacado: true,
  },
  {
    id: 2,
    nombre: 'Suite Office 365',
    categoria: 'software',
    precio: 149.99,
    descripcion: 'Licencia anual de Microsoft Office 365 para empresas con 5 dispositivos.',
    stock: 100,
    destacado: false,
  },
  {
    id: 3,
    nombre: 'Monitor 4K UltraWide',
    categoria: 'perifericos',
    precio: 599.99,
    descripcion: 'Monitor 34 pulgadas UltraWide 4K con panel IPS y 144Hz de refresco.',
    stock: 8,
    destacado: true,
  },
  {
    id: 4,
    nombre: 'Teclado Mecánico RGB',
    categoria: 'perifericos',
    precio: 129.99,
    descripcion: 'Teclado mecánico con switches Cherry MX Red e iluminación RGB personalizable.',
    stock: 25,
    destacado: false,
  },
  {
    id: 5,
    nombre: 'Servicio Cloud Backup',
    categoria: 'servicios',
    precio: 29.99,
    descripcion: 'Respaldo automático en la nube con 2TB de almacenamiento y encriptación AES-256.',
    stock: 999,
    destacado: false,
  },
  {
    id: 6,
    nombre: 'Servidor NAS 8TB',
    categoria: 'electronica',
    precio: 849.99,
    descripcion: 'Almacenamiento en red con 8TB RAID 5, acceso remoto y gestión centralizada.',
    stock: 3,
    destacado: true,
  },
];

/**
 * Genera un ID único para nuevos productos.
 * @param {Array} productos - Lista actual de productos
 * @returns {number} - Nuevo ID único
 */
export function generarId(productos) {
  return productos.length > 0 ? Math.max(...productos.map((p) => p.id)) + 1 : 1;
}

/**
 * Formatea un precio como moneda USD.
 * @param {number} precio
 * @returns {string}
 */
export function formatearPrecio(precio) {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(precio);
}

/**
 * Obtiene la etiqueta de categoría para mostrar.
 * @param {string} categoria
 * @returns {string}
 */
export function etiquetaCategoria(categoria) {
  const etiquetas = {
    electronica: 'Electrónica',
    software: 'Software',
    perifericos: 'Periféricos',
    servicios: 'Servicios',
  };
  return etiquetas[categoria] ?? categoria;
}
```

2. Crea `src/utils/dom.js` con utilidades de manipulación del DOM:

```javascript
// src/utils/dom.js

/**
 * Muestra una notificación toast en pantalla.
 * @param {string} mensaje - Texto del mensaje
 * @param {'success'|'error'|'warning'} tipo - Tipo de notificación
 * @param {number} duracion - Duración en milisegundos
 */
export function mostrarToast(mensaje, tipo = 'success', duracion = 3000) {
  const contenedor = document.getElementById('toast-container');
  if (!contenedor) return;

  const toast = document.createElement('div');
  toast.className = `toast ${tipo}`;
  toast.textContent = mensaje;
  toast.setAttribute('role', 'alert');

  contenedor.appendChild(toast);

  setTimeout(() => {
    toast.style.animation = `fadeOut ${250}ms ease forwards`;
    toast.addEventListener('animationend', () => toast.remove());
  }, duracion);
}

/**
 * Actualiza el contador del carrito en el header.
 * @param {number} cantidad
 */
export function actualizarContadorCarrito(cantidad) {
  const contador = document.getElementById('cart-count');
  if (contador) {
    contador.textContent = cantidad;
    contador.dataset.count = cantidad;
  }
}

/**
 * Navega entre secciones de la SPA ocultando/mostrando secciones.
 * @param {string} seccionId - ID de la sección a mostrar
 */
export function navegarA(seccionId) {
  const secciones = ['catalogo', 'agregar', 'estadisticas'];

  secciones.forEach((id) => {
    const seccion = document.getElementById(id);
    if (seccion) {
      if (id === seccionId) {
        seccion.removeAttribute('hidden');
      } else {
        seccion.setAttribute('hidden', '');
      }
    }
  });

  // Actualizar estado activo en nav
  document.querySelectorAll('.nav-link').forEach((link) => {
    const esActivo = link.dataset.section === seccionId;
    link.classList.toggle('active', esActivo);
  });
}
```

3. Crea `src/main.js` — el punto de entrada y controlador principal:

```javascript
// src/main.js
import { productosIniciales, generarId, formatearPrecio, etiquetaCategoria } from './data/productos.js';
import { mostrarToast, actualizarContadorCarrito, navegarA } from './utils/dom.js';

// ============================================================
// ESTADO DE LA APLICACIÓN
// ============================================================
const estado = {
  productos: [...productosIniciales],
  carrito: [],
  filtroActual: 'todas',
  busquedaActual: '',
  productoSeleccionado: null,
};

// ============================================================
// RENDERIZADO DE PRODUCTOS
// ============================================================

/**
 * Filtra los productos según búsqueda y categoría activa.
 * @returns {Array} Productos filtrados
 */
function obtenerProductosFiltrados() {
  return estado.productos.filter((producto) => {
    const coincideBusqueda =
      estado.busquedaActual === '' ||
      producto.nombre.toLowerCase().includes(estado.busquedaActual.toLowerCase()) ||
      producto.categoria.toLowerCase().includes(estado.busquedaActual.toLowerCase());

    const coincideCategoria =
      estado.filtroActual === 'todas' || producto.categoria === estado.filtroActual;

    return coincideBusqueda && coincideCategoria;
  });
}

/**
 * Renderiza las tarjetas de productos en el grid.
 * Usa atributos data-* para enlazar datos con el DOM.
 */
function renderizarProductos() {
  const grid = document.getElementById('products-grid');
  const contador = document.getElementById('product-count');
  if (!grid) return;

  const productosFiltrados = obtenerProductosFiltrados();

  // Actualizar contador
  if (contador) {
    contador.textContent = `${productosFiltrados.length} producto${productosFiltrados.length !== 1 ? 's' : ''} encontrado${productosFiltrados.length !== 1 ? 's' : ''}`;
  }

  if (productosFiltrados.length === 0) {
    grid.innerHTML = `
      <div style="grid-column: 1 / -1; text-align: center; padding: 3rem; color: var(--color-text-muted);">
        <p style="font-size: 2rem; margin-bottom: 1rem;">🔍</p>
        <p>No se encontraron productos con los filtros actuales.</p>
      </div>
    `;
    return;
  }

  // Renderizar tarjetas usando template literals
  grid.innerHTML = productosFiltrados
    .map(
      (producto) => `
      <article
        class="product-card"
        role="listitem"
        data-producto-id="${producto.id}"
        data-producto-nombre="${producto.nombre}"
        data-producto-precio="${producto.precio}"
        tabindex="0"
        aria-label="Producto: ${producto.nombre}, precio ${formatearPrecio(producto.precio)}"
      >
        <span class="product-card-badge">${etiquetaCategoria(producto.categoria)}</span>
        <h3 class="product-card-name">${producto.nombre}</h3>
        <p class="product-card-description">${producto.descripcion}</p>
        <div class="product-card-footer">
          <span class="product-card-price">${formatearPrecio(producto.precio)}</span>
          <span class="product-card-stock ${producto.stock <= 5 ? 'low-stock' : ''}">
            ${producto.stock <= 5 ? '⚠️ ' : ''}Stock: ${producto.stock}
          </span>
        </div>
        <div style="margin-top: 0.75rem; display: flex; gap: 0.5rem;">
          <button
            class="btn btn-primary"
            style="flex: 1; font-size: 0.75rem; padding: 0.375rem 0.5rem;"
            data-accion="agregar-carrito"
            data-producto-id="${producto.id}"
          >
            🛒 Agregar
          </button>
          <button
            class="btn btn-secondary"
            style="font-size: 0.75rem; padding: 0.375rem 0.75rem;"
            data-accion="ver-detalle"
            data-producto-id="${producto.id}"
          >
            Ver
          </button>
        </div>
      </article>
    `
    )
    .join('');
}

// ============================================================
// PANEL DE DETALLE
// ============================================================

function mostrarDetalle(productoId) {
  const producto = estado.productos.find((p) => p.id === Number(productoId));
  if (!producto) return;

  estado.productoSeleccionado = producto;

  const panel = document.getElementById('detail-panel');
  const contenido = document.getElementById('panel-content');

  if (!panel || !contenido) return;

  contenido.innerHTML = `
    <div style="display: flex; flex-direction: column; gap: 1rem;">
      <span class="product-card-badge">${etiquetaCategoria(producto.categoria)}</span>
      <h4 style="font-size: 1.25rem; font-weight: 700;">${producto.nombre}</h4>
      <p style="color: var(--color-text-muted); line-height: 1.6;">${producto.descripcion}</p>
      <div style="display: flex; justify-content: space-between; padding: 1rem; background: var(--color-surface-alt); border-radius: 0.5rem;">
        <div>
          <div style="font-size: 0.75rem; color: var(--color-text-muted);">PRECIO</div>
          <div style="font-size: 1.5rem; font-weight: 700; color: var(--color-primary);">${formatearPrecio(producto.precio)}</div>
        </div>
        <div>
          <div style="font-size: 0.75rem; color: var(--color-text-muted);">STOCK</div>
          <div style="font-size: 1.5rem; font-weight: 700;">${producto.stock}</div>
        </div>
      </div>
      <button
        class="btn btn-primary"
        data-accion="agregar-carrito"
        data-producto-id="${producto.id}"
      >
        🛒 Agregar al carrito
      </button>
    </div>
  `;

  panel.removeAttribute('hidden');
}

// ============================================================
// ESTADÍSTICAS
// ============================================================

function renderizarEstadisticas() {
  const statsGrid = document.getElementById('stats-grid');
  if (!statsGrid) return;

  const totalProductos = estado.productos.length;
  const precioPromedio =
    estado.productos.reduce((acc, p) => acc + p.precio, 0) / totalProductos || 0;
  const stockTotal = estado.productos.reduce((acc, p) => acc + p.stock, 0);
  const categorias = [...new Set(estado.productos.map((p) => p.categoria))].length;

  const stats = [
    { valor: totalProductos, etiqueta: 'Total Productos', icono: '📦' },
    { valor: formatearPrecio(precioPromedio), etiqueta: 'Precio Promedio', icono: '💰' },
    { valor: stockTotal, etiqueta: 'Unidades en Stock', icono: '🏭' },
    { valor: categorias, etiqueta: 'Categorías', icono: '🏷️' },
  ];

  statsGrid.innerHTML = stats
    .map(
      (s) => `
      <div class="stat-card">
        <div style="font-size: 2rem; margin-bottom: 0.5rem;">${s.icono}</div>
        <div class="stat-card-value">${s.valor}</div>
        <div class="stat-card-label">${s.etiqueta}</div>
      </div>
    `
    )
    .join('');

  // Dibujar gráfico de barras en Canvas
  dibujarGraficoCanvas();
}

function dibujarGraficoCanvas() {
  const canvas = document.getElementById('chart-categorias');
  if (!canvas) return;

  const ctx = canvas.getContext('2d');
  const categorias = {};

  estado.productos.forEach((p) => {
    categorias[p.categoria] = (categorias[p.categoria] || 0) + 1;
  });

  const etiquetas = Object.keys(categorias);
  const valores = Object.values(categorias);
  const colores = ['#2563eb', '#7c3aed', '#16a34a', '#d97706'];

  const ancho = canvas.width;
  const alto = canvas.height;
  const margen = 60;
  const anchoBarras = (ancho - margen * 2) / etiquetas.length;
  const maxValor = Math.max(...valores);

  ctx.clearRect(0, 0, ancho, alto);
  ctx.fillStyle = getComputedStyle(document.documentElement)
    .getPropertyValue('--color-surface')
    .trim() || '#fff';
  ctx.fillRect(0, 0, ancho, alto);

  etiquetas.forEach((etiqueta, i) => {
    const altoBarra = ((valores[i] / maxValor) * (alto - margen * 2));
    const x = margen + i * anchoBarras + anchoBarras * 0.1;
    const y = alto - margen - altoBarra;
    const anchoReal = anchoBarras * 0.8;

    ctx.fillStyle = colores[i % colores.length];
    ctx.beginPath();
    ctx.roundRect(x, y, anchoReal, altoBarra, 4);
    ctx.fill();

    ctx.fillStyle = '#64748b';
    ctx.font = '12px system-ui';
    ctx.textAlign = 'center';
    ctx.fillText(etiquetaCategoria(etiqueta), x + anchoReal / 2, alto - margen + 20);

    ctx.fillStyle = '#1e293b';
    ctx.font = 'bold 14px system-ui';
    ctx.fillText(valores[i], x + anchoReal / 2, y - 8);
  });
}

// ============================================================
// MANEJO DE EVENTOS
// ============================================================

function inicializarEventos() {
  // Navegación SPA
  document.querySelectorAll('.nav-link').forEach((link) => {
    link.addEventListener('click', (e) => {
      e.preventDefault();
      const seccion = link.dataset.section;
      navegarA(seccion);
      if (seccion === 'estadisticas') renderizarEstadisticas();
    });
  });

  // Búsqueda en tiempo real
  const inputBusqueda = document.getElementById('input-busqueda');
  if (inputBusqueda) {
    inputBusqueda.addEventListener('input', (e) => {
      estado.busquedaActual = e.target.value;
      renderizarProductos();
    });
  }

  // Filtros por categoría
  document.querySelectorAll('.filter-chip').forEach((chip) => {
    chip.addEventListener('click', () => {
      document.querySelectorAll('.filter-chip').forEach((c) => {
        c.classList.remove('active');
        c.setAttribute('aria-pressed', 'false');
      });
      chip.classList.add('active');
      chip.setAttribute('aria-pressed', 'true');
      estado.filtroActual = chip.dataset.categoria;
      renderizarProductos();
    });
  });

  // Delegación de eventos en el grid de productos
  const grid = document.getElementById('products-grid');
  if (grid) {
    grid.addEventListener('click', (e) => {
      const boton = e.target.closest('[data-accion]');
      if (!boton) {
        // Click en tarjeta
        const tarjeta = e.target.closest('.product-card');
        if (tarjeta) mostrarDetalle(tarjeta.dataset.productoId);
        return;
      }

      const { accion, productoId } = boton.dataset;

      if (accion === 'agregar-carrito') {
        agregarAlCarrito(Number(productoId));
      } else if (accion === 'ver-detalle') {
        mostrarDetalle(productoId);
      }
    });
  }

  // Cerrar panel de detalle
  const btnCerrar = document.getElementById('btn-close-panel');
  if (btnCerrar) {
    btnCerrar.addEventListener('click', () => {
      const panel = document.getElementById('detail-panel');
      if (panel) panel.setAttribute('hidden', '');
    });
  }

  // Toggle de tema
  const btnTema = document.getElementById('btn-toggle-theme');
  if (btnTema) {
    btnTema.addEventListener('click', () => {
      const temaActual = document.documentElement.dataset.theme;
      if (temaActual === 'dark') {
        delete document.documentElement.dataset.theme;
        btnTema.textContent = '🌙';
      } else {
        document.documentElement.dataset.theme = 'dark';
        btnTema.textContent = '☀️';
      }
    });
  }

  // Formulario de nuevo producto
  const formulario = document.getElementById('form-producto');
  if (formulario) {
    formulario.addEventListener('submit', (e) => {
      e.preventDefault();
      manejarEnvioFormulario(formulario);
    });
  }
}

// ============================================================
// CARRITO
// ============================================================

function agregarAlCarrito(productoId) {
  const producto = estado.productos.find((p) => p.id === productoId);
  if (!producto) return;

  const itemExistente = estado.carrito.find((item) => item.id === productoId);
  if (itemExistente) {
    itemExistente.cantidad += 1;
  } else {
    estado.carrito.push({ ...producto, cantidad: 1 });
  }

  const totalItems = estado.carrito.reduce((acc, item) => acc + item.cantidad, 0);
  actualizarContadorCarrito(totalItems);
  mostrarToast(`✅ "${producto.nombre}" agregado al carrito`, 'success');
}

// ============================================================
// FORMULARIO
// ============================================================

function manejarEnvioFormulario(formulario) {
  const datos = new FormData(formulario);
  const errores = validarFormulario(datos);

  limpiarErrores();

  if (Object.keys(errores).length > 0) {
    mostrarErrores(errores);
    return;
  }

  const nuevoProducto = {
    id: generarId(estado.productos),
    nombre: datos.get('nombre').trim(),
    categoria: datos.get('categoria'),
    precio: parseFloat(datos.get('precio')),
    descripcion: datos.get('descripcion').trim(),
    stock: parseInt(datos.get('stock')) || 0,
    destacado: false,
  };

  estado.productos.push(nuevoProducto);
  formulario.reset();
  mostrarToast(`✅ Producto "${nuevoProducto.nombre}" agregado exitosamente`, 'success');
  navegarA('catalogo');
  renderizarProductos();
}

function validarFormulario(datos) {
  const errores = {};
  const nombre = datos.get('nombre')?.trim();
  const categoria = datos.get('categoria');
  const precio = datos.get('precio');

  if (!nombre || nombre.length < 3) {
    errores.nombre = 'El nombre debe tener al menos 3 caracteres.';
  }
  if (!categoria) {
    errores.categoria = 'Selecciona una categoría.';
  }
  if (!precio || parseFloat(precio) <= 0) {
    errores.precio = 'El precio debe ser mayor a 0.';
  }

  return errores;
}

function mostrarErrores(errores) {
  Object.entries(errores).forEach(([campo, mensaje]) => {
    const errorEl = document.getElementById(`error-${campo}`);
    const inputEl = document.querySelector(`[data-field="${campo}"]`);
    if (errorEl) errorEl.textContent = mensaje;
    if (inputEl) inputEl.classList.add('invalid');
  });
}

function limpiarErrores() {
  document.querySelectorAll('.field-error').forEach((el) => (el.textContent = ''));
  document.querySelectorAll('.invalid').forEach((el) => el.classList.remove('invalid'));
}

// ============================================================
// INICIALIZACIÓN
// ============================================================

function inicializar() {
  renderizarProductos();
  inicializarEventos();
  console.log('✅ CatálogoTech inicializado correctamente');
  console.log(`📦 ${estado.productos.length} productos cargados`);
}

// Ejecutar cuando el DOM esté listo
document.addEventListener('DOMContentLoaded', inicializar);
```

**Resultado Esperado:** La aplicación muestra el catálogo de 6 productos con funcionalidad de búsqueda, filtros, carrito y formulario operativos.

**Verificación:** En el navegador, verifica que: (1) los 6 productos se muestran en el grid, (2) el filtro "Electrónica" muestra solo 2 productos, (3) agregar un producto al carrito actualiza el contador del header, (4) el formulario valida campos vacíos.

---

### Paso 5 — Web Component Nativo: `<product-card>`

**Objetivo:** Refactorizar la tarjeta de producto hacia un Web Component nativo con Shadow DOM y ciclo de vida completo.

**Instrucciones:**

1. Crea `src/components/product-card.js`:

```javascript
// src/components/product-card.js

/**
 * Web Component nativo para tarjeta de producto.
 * Demuestra el ciclo de vida completo de los Custom Elements.
 */
class ProductCard extends HTMLElement {
  // Define qué atributos observar para attributeChangedCallback
  static get observedAttributes() {
    return ['product-id', 'nombre', 'categoria', 'precio', 'descripcion', 'stock'];
  }

  constructor() {
    super();
    // Crear Shadow DOM en modo 'open' para permitir acceso externo
    this._shadowRoot = this.attachShadow({ mode: 'open' });
    this._carrito = 0;
  }

  // ── CICLO DE VIDA ──────────────────────────────────────────

  /**
   * Se ejecuta cuando el componente se inserta en el DOM.
   * Equivalente a componentDidMount en React.
   */
  connectedCallback() {
    console.log(`[ProductCard] Conectado al DOM: ${this.getAttribute('nombre')}`);
    this._render();
    this._attachEventListeners();
  }

  /**
   * Se ejecuta cuando el componente se elimina del DOM.
   * Ideal para limpiar event listeners y timers.
   */
  disconnectedCallback() {
    console.log(`[ProductCard] Desconectado del DOM: ${this.getAttribute('nombre')}`);
    // Limpiar event listeners internos (los del Shadow DOM se limpian automáticamente)
  }

  /**
   * Se ejecuta cuando un atributo observado cambia.
   * @param {string} nombre - Nombre del atributo
   * @param {string|null} valorAnterior
   * @param {string|null} valorNuevo
   */
  attributeChangedCallback(nombre, valorAnterior, valorNuevo) {
    if (valorAnterior !== valorNuevo && this._shadowRoot.innerHTML !== '') {
      console.log(`[ProductCard] Atributo "${nombre}" cambió: ${valorAnterior} → ${valorNuevo}`);
      this._render();
      this._attachEventListeners();
    }
  }

  // ── PROPIEDADES COMPUTADAS ─────────────────────────────────

  get productoId() { return this.getAttribute('product-id'); }
  get nombre()     { return this.getAttribute('nombre') ?? 'Sin nombre'; }
  get categoria()  { return this.getAttribute('categoria') ?? ''; }
  get precio()     { return parseFloat(this.getAttribute('precio')) || 0; }
  get descripcion(){ return this.getAttribute('descripcion') ?? ''; }
  get stock()      { return parseInt(this.getAttribute('stock')) || 0; }

  get precioFormateado() {
    return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(this.precio);
  }

  get etiquetaCategoria() {
    const etiquetas = {
      electronica: 'Electrónica',
      software: 'Software',
      perifericos: 'Periféricos',
      servicios: 'Servicios',
    };
    return etiquetas[this.categoria] ?? this.categoria;
  }

  get esStockBajo() { return this.stock > 0 && this.stock <= 5; }

  // ── RENDERIZADO ────────────────────────────────────────────

  _render() {
    this._shadowRoot.innerHTML = `
      <style>
        :host {
          display: block;
        }

        :host(:hover) .card {
          box-shadow: 0 20px 25px rgba(0, 0, 0, 0.12);
          transform: translateY(-4px);
          border-color: #2563eb;
        }

        .card {
          background-color: #ffffff;
          border: 1px solid #e2e8f0;
          border-radius: 1rem;
          padding: 1.5rem;
          display: flex;
          flex-direction: column;
          gap: 0.75rem;
          box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
          transition: all 250ms ease;
          cursor: pointer;
          position: relative;
          overflow: hidden;
        }

        .card::before {
          content: '';
          position: absolute;
          top: 0; left: 0; right: 0;
          height: 4px;
          background: linear-gradient(90deg, #2563eb, #7c3aed);
          opacity: 0;
          transition: opacity 250ms ease;
        }

        :host(:hover) .card::before { opacity: 1; }

        .badge {
          display: inline-flex;
          align-items: center;
          padding: 0.25rem 0.5rem;
          border-radius: 9999px;
          font-size: 0.75rem;
          font-weight: 600;
          text-transform: uppercase;
          letter-spacing: 0.05em;
          background-color: #dbeafe;
          color: #2563eb;
          width: fit-content;
        }

        .nombre {
          font-size: 1.125rem;
          font-weight: 600;
          color: #1e293b;
          margin: 0;
        }

        .descripcion {
          font-size: 0.875rem;
          color: #64748b;
          line-height: 1.5;
          flex: 1;
        }

        .footer {
          display: flex;
          justify-content: space-between;
          align-items: center;
          padding-top: 0.75rem;
          border-top: 1px solid #e2e8f0;
        }

        .precio {
          font-size: 1.25rem;
          font-weight: 700;
          color: #2563eb;
        }

        .stock {
          font-size: 0.75rem;
          color: #64748b;
        }

        .stock.bajo { color: #d97706; font-weight: 600; }

        .acciones {
          display: flex;
          gap: 0.5rem;
          margin-top: 0.75rem;
        }

        .btn {
          flex: 1;
          padding: 0.375rem 0.5rem;
          border: none;
          border-radius: 0.375rem;
          font-size: 0.75rem;
          font-weight: 500;
          cursor: pointer;
          transition: all 150ms ease;
        }

        .btn-primary {
          background-color: #2563eb;
          color: white;
        }

        .btn-primary:hover { background-color: #1d4ed8; }

        .btn-secondary {
          background-color: #f1f5f9;
          color: #1e293b;
          border: 1px solid #e2e8f0;
          flex: 0 0 auto;
          padding: 0.375rem 0.75rem;
        }

        .btn-secondary:hover { background-color: #e2e8f0; }
      </style>
