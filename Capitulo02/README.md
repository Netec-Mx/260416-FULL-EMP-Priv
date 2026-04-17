# Laboratorio 2. Proyecto Integrado usando tecnologías Frontend

## 1. Metadatos

| Campo            | Valor                                       |
|------------------|---------------------------------------------|
| **Duración**     | 260 minutos (7 fases; ver tabla de tiempos) |
| **Complejidad**  | Alta                                        |
| **Nivel Bloom**  | Crear                                       |
| **Laboratorio**  | 02-00-01                                    |

### Distribución de Tiempos por Fase

| Fase | Título                                        | Duración |
|------|-----------------------------------------------|----------|
| 1    | Configuración del Entorno y Estructura Base   | 40 min   |
| 2    | Web Components Nativos                        | 40 min   |
| 3    | Introducción a Lit                            | 35 min   |
| 4    | Propiedades Reactivas y Templates             | 40 min   |
| 5    | Comunicación entre Componentes                | 35 min   |
| 6    | Pruebas Unitarias con Vitest                  | 30 min   |
| 7    | Integración Final y Refinamiento              | 40 min   |

> **⏸ Puntos de pausa recomendados:** Entre Fase 2 y 3 (80 min), y entre Fase 4 y 5 (155 min). Cada pausa debe durar al menos 10 minutos.

---

## 2. Descripción General

En este laboratorio integrador construirás un **Dashboard de Gestión de Tareas y Proyectos** completo, aplicando directamente los fundamentos de HTML5 semántico aprendidos en la Lección 2.1. Partirás desde cero con un entorno profesional basado en Vite, progresarás por la API nativa de Web Components y finalizarás con componentes Lit reactivos, comunicación bidireccional entre componentes y una suite de pruebas unitarias con Vitest. El resultado será una aplicación frontend empresarial lista para integrarse con un backend en laboratorios posteriores.

---

## 3. Objetivos de Aprendizaje

- [ ] Configurar un entorno de desarrollo profesional con Node.js, NPM, Vite y Vitest, y estructurar el proyecto con HTML5 semántico correcto.
- [ ] Crear Web Components nativos implementando el ciclo de vida completo (`connectedCallback`, `disconnectedCallback`, `attributeChangedCallback`) y Shadow DOM.
- [ ] Migrar componentes nativos a Lit, utilizando `LitElement`, `` html`\`` ``, `` css`\`` ``, `@property` y `@state` con directivas avanzadas (`repeat`, `when`, `classMap`).
- [ ] Implementar comunicación bidireccional entre componentes mediante propiedades, `CustomEvent` y un servicio de estado simple.
- [ ] Escribir y ejecutar pruebas unitarias con Vitest y `happy-dom` validando renderizado, reactividad y emisión de eventos.

---

## 4. Prerrequisitos

### Conocimiento Previo

| Área                          | Nivel requerido                                          |
|-------------------------------|----------------------------------------------------------|
| HTML y CSS básicos            | Nociones básicas (la Lección 2.1 cubre lo necesario)     |
| JavaScript (ES6+)             | Variables, funciones, clases, módulos, promesas          |
| Línea de comandos             | Navegación de directorios, ejecución de comandos         |
| Conceptos de componentes      | Deseable; se explica desde cero en este laboratorio      |

### Acceso y Software Requerido

| Software           | Versión mínima | Verificación                    |
|--------------------|----------------|---------------------------------|
| Node.js            | 20.x LTS       | `node --version`                |
| NPM                | 10.x           | `npm --version`                 |
| Visual Studio Code | 1.88+          | `code --version`                |
| Google Chrome      | Última estable | Abrir `chrome://version`        |
| Git                | 2.44+          | `git --version`                 |

### Extensiones de VS Code Recomendadas

Instalar antes de comenzar:

```bash
code --install-extension lit-plugin.vscode-lit-plugin
code --install-extension esbenp.prettier-vscode
code --install-extension dbaeumer.vscode-eslint
code --install-extension ZixuanChen.vitest-explorer
```

---

## 5. Entorno del Laboratorio

### Configuración Previa (ejecutar antes del laboratorio)

Verificar que Node.js y NPM estén disponibles:

```bash
# Windows (PowerShell), macOS y Linux
node --version    # debe mostrar v20.x.x
npm --version     # debe mostrar 10.x.x
```

Si Node.js no está instalado, descargarlo desde [https://nodejs.org](https://nodejs.org) (versión LTS).

### Estructura Final del Proyecto

Al terminar el laboratorio, el proyecto tendrá esta estructura:

```
task-dashboard/
├── index.html
├── package.json
├── vite.config.js
├── vitest.config.js
├── .eslintrc.cjs
├── .prettierrc
├── src/
│   ├── main.js
│   ├── styles/
│   │   ├── global.css
│   │   └── variables.css
│   ├── services/
│   │   └── state-service.js
│   ├── components/
│   │   ├── native/
│   │   │   └── status-badge.js
│   │   └── lit/
│   │       ├── app-dashboard.js
│   │       ├── project-card.js
│   │       ├── task-list.js
│   │       ├── notification-panel.js
│   │       └── theme-toggle.js
│   └── data/
│       └── mock-data.js
└── src/tests/
    ├── project-card.test.js
    ├── task-list.test.js
    └── notification-panel.test.js
```

---

## 6. Instrucciones Paso a Paso

---

### Fase 1: Configuración del Entorno y Estructura Base (40 min)

**Objetivo:** Inicializar el proyecto con Vite, definir la estructura de directorios y crear el esqueleto HTML5 semántico con el sistema de estilos base.

#### Paso 1.1 – Crear el Proyecto con Vite

**Instrucciones:**

1. Abrir una terminal en el directorio donde almacenarás el proyecto.
2. Ejecutar el siguiente comando para crear el proyecto:

```bash
npm create vite@latest task-dashboard -- --template vanilla
```

3. Ingresar al directorio del proyecto e instalar dependencias base:

```bash
cd task-dashboard
npm install
```

4. Instalar Lit y las dependencias de desarrollo:

```bash
npm install lit
npm install --save-dev vitest @vitest/ui happy-dom eslint prettier
```

5. Instalar el plugin de Vite para soporte de decoradores:

```bash
npm install --save-dev @babel/core @babel/plugin-proposal-decorators
```

**Resultado esperado:**

```
added 150 packages in 15s
```

El directorio `node_modules/` debe existir y `package.json` debe contener `"lit"` en `dependencies`.

**Verificación:**

```bash
npm run dev
# Debe mostrar: VITE vX.X.X ready in XXXms
# ➜  Local: http://localhost:5173/
```

Abrir `http://localhost:5173` en Chrome. Debe verse la página de bienvenida de Vite. Detener el servidor con `Ctrl+C`.

---

#### Paso 1.2 – Configurar Vite y Vitest

**Instrucciones:**

1. Reemplazar el contenido de `vite.config.js` con:

```javascript
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  esbuild: {
    target: 'es2022',
  },
  build: {
    target: 'es2022',
  },
});
```

2. Crear el archivo `vitest.config.js` en la raíz del proyecto:

```javascript
// vitest.config.js
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'happy-dom',
    globals: true,
    setupFiles: [],
    include: ['src/tests/**/*.test.js'],
    coverage: {
      reporter: ['text', 'html'],
    },
  },
});
```

3. Actualizar la sección `"scripts"` en `package.json`:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:ui": "vitest --ui"
  }
}
```

**Verificación:**

```bash
npm test
# Debe mostrar: "No test files found"
# Esto es correcto – aún no hemos escrito pruebas
```

---

#### Paso 1.3 – Crear la Estructura HTML5 Semántica

**Instrucciones:**

1. Reemplazar completamente el contenido de `index.html` con la siguiente estructura semántica:

```html
<!DOCTYPE html>
<html lang="es">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="description" content="Dashboard de gestión de tareas y proyectos empresariales" />
    <title>TaskDash – Gestión de Proyectos</title>
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <link rel="stylesheet" href="/src/styles/variables.css" />
    <link rel="stylesheet" href="/src/styles/global.css" />
  </head>
  <body>
    <!-- Encabezado principal de la aplicación -->
    <header class="app-header" role="banner">
      <div class="header-brand">
        <h1 class="app-title">
          <span aria-hidden="true">📋</span> TaskDash
        </h1>
        <p class="app-subtitle">Gestión de Proyectos Empresariales</p>
      </div>
      <nav class="header-nav" aria-label="Navegación principal">
        <ul role="list">
          <li><a href="#dashboard" aria-current="page">Dashboard</a></li>
          <li><a href="#proyectos">Proyectos</a></li>
          <li><a href="#tareas">Tareas</a></li>
        </ul>
      </nav>
      <!-- theme-toggle se montará aquí en Fase 4 -->
      <div id="theme-toggle-container"></div>
    </header>

    <!-- Contenido principal -->
    <main id="main-content" class="app-main" role="main">
      <!-- El componente principal del dashboard se montará aquí -->
      <app-dashboard id="dashboard"></app-dashboard>
    </main>

    <!-- Panel de notificaciones (fuera del flujo principal) -->
    <aside id="notification-area" aria-label="Notificaciones" aria-live="polite">
      <notification-panel></notification-panel>
    </aside>

    <!-- Pie de página -->
    <footer class="app-footer" role="contentinfo">
      <p>&copy; 2024 TaskDash – Plataforma de Gestión Empresarial</p>
    </footer>

    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```

> **Nota pedagógica:** Observa el uso de `role="banner"`, `role="main"`, `role="contentinfo"` y `aria-label`. Estos atributos ARIA complementan los elementos semánticos HTML5 para mejorar la accesibilidad con lectores de pantalla, tal como se explicó en la Lección 2.1.

---

#### Paso 1.4 – Crear el Sistema de Estilos CSS3

**Instrucciones:**

1. Crear el directorio de estilos:

```bash
# Windows (PowerShell)
New-Item -ItemType Directory -Path src/styles

# macOS / Linux
mkdir -p src/styles
```

2. Crear `src/styles/variables.css`:

```css
/* src/styles/variables.css – Variables CSS globales (Design Tokens) */
:root {
  /* Paleta de colores – Tema Claro */
  --color-primary: #2563eb;
  --color-primary-hover: #1d4ed8;
  --color-secondary: #7c3aed;
  --color-success: #16a34a;
  --color-warning: #d97706;
  --color-danger: #dc2626;
  --color-info: #0891b2;

  /* Superficies */
  --color-bg: #f8fafc;
  --color-surface: #ffffff;
  --color-surface-raised: #f1f5f9;
  --color-border: #e2e8f0;

  /* Texto */
  --color-text-primary: #0f172a;
  --color-text-secondary: #475569;
  --color-text-muted: #94a3b8;

  /* Espaciado */
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 1.5rem;
  --space-xl: 2rem;
  --space-2xl: 3rem;

  /* Tipografía */
  --font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  --font-size-2xl: 1.5rem;
  --font-size-3xl: 1.875rem;

  /* Bordes y sombras */
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-full: 9999px;
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1);

  /* Transiciones */
  --transition-fast: 150ms ease;
  --transition-base: 250ms ease;
  --transition-slow: 400ms ease;
}

/* Tema Oscuro */
[data-theme="dark"] {
  --color-bg: #0f172a;
  --color-surface: #1e293b;
  --color-surface-raised: #334155;
  --color-border: #475569;
  --color-text-primary: #f1f5f9;
  --color-text-secondary: #cbd5e1;
  --color-text-muted: #64748b;
}
```

3. Crear `src/styles/global.css`:

```css
/* src/styles/global.css – Estilos globales de la aplicación */
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html {
  font-size: 16px;
  scroll-behavior: smooth;
}

body {
  font-family: var(--font-family);
  font-size: var(--font-size-base);
  color: var(--color-text-primary);
  background-color: var(--color-bg);
  line-height: 1.6;
  min-height: 100vh;
  display: grid;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header"
    "main"
    "footer";
  transition: background-color var(--transition-base), color var(--transition-base);
}

/* Header */
.app-header {
  grid-area: header;
  background-color: var(--color-surface);
  border-bottom: 1px solid var(--color-border);
  box-shadow: var(--shadow-sm);
  padding: var(--space-md) var(--space-xl);
  display: flex;
  align-items: center;
  gap: var(--space-xl);
  position: sticky;
  top: 0;
  z-index: 100;
}

.app-title {
  font-size: var(--font-size-2xl);
  font-weight: 700;
  color: var(--color-primary);
}

.app-subtitle {
  font-size: var(--font-size-sm);
  color: var(--color-text-muted);
}

.header-nav ul {
  list-style: none;
  display: flex;
  gap: var(--space-lg);
}

.header-nav a {
  text-decoration: none;
  color: var(--color-text-secondary);
  font-weight: 500;
  padding: var(--space-xs) var(--space-sm);
  border-radius: var(--radius-md);
  transition: color var(--transition-fast), background-color var(--transition-fast);
}

.header-nav a:hover,
.header-nav a[aria-current="page"] {
  color: var(--color-primary);
  background-color: var(--color-surface-raised);
}

/* Main */
.app-main {
  grid-area: main;
  padding: var(--space-xl);
  max-width: 1440px;
  margin: 0 auto;
  width: 100%;
}

/* Footer */
.app-footer {
  grid-area: footer;
  background-color: var(--color-surface);
  border-top: 1px solid var(--color-border);
  padding: var(--space-md) var(--space-xl);
  text-align: center;
  color: var(--color-text-muted);
  font-size: var(--font-size-sm);
}

/* Aside de notificaciones */
#notification-area {
  position: fixed;
  bottom: var(--space-xl);
  right: var(--space-xl);
  z-index: 200;
  display: flex;
  flex-direction: column;
  gap: var(--space-sm);
  max-width: 380px;
}

/* Responsive */
@media (max-width: 768px) {
  .app-header {
    flex-wrap: wrap;
    padding: var(--space-md);
  }
  .app-main {
    padding: var(--space-md);
  }
  .header-nav ul {
    gap: var(--space-md);
  }
}
```

4. Crear el archivo de entrada `src/main.js` (vacío por ahora):

```javascript
// src/main.js – Punto de entrada de la aplicación
// Las importaciones se añadirán en fases posteriores
console.log('TaskDash iniciando...');
```

**Resultado esperado:** Al ejecutar `npm run dev` y abrir el navegador, se debe ver el header con "TaskDash", la navegación y el footer con estilos aplicados.

---

### Fase 2: Web Components Nativos (40 min)

**Objetivo:** Crear un componente web nativo usando la API del navegador, implementando Shadow DOM, HTML Templates y el ciclo de vida completo.

#### Paso 2.1 – Crear los Datos de Prueba

**Instrucciones:**

1. Crear el directorio y archivo de datos mock:

```bash
# Windows (PowerShell)
New-Item -ItemType Directory -Path src/data
New-Item -ItemType Directory -Path src/components/native
New-Item -ItemType Directory -Path src/components/lit
New-Item -ItemType Directory -Path src/tests

# macOS / Linux
mkdir -p src/data src/components/native src/components/lit src/tests
```

2. Crear `src/data/mock-data.js`:

```javascript
// src/data/mock-data.js – Datos de prueba para el dashboard

export const PROJECTS = [
  {
    id: 'proj-001',
    name: 'Portal de Clientes',
    description: 'Rediseño completo del portal web para clientes corporativos',
    status: 'active',
    progress: 65,
    priority: 'high',
    dueDate: '2024-03-31',
    team: ['Ana García', 'Carlos López', 'María Rodríguez'],
    taskCount: { total: 24, completed: 15, pending: 6, blocked: 3 },
  },
  {
    id: 'proj-002',
    name: 'API de Pagos',
    description: 'Integración con pasarela de pagos internacionales',
    status: 'planning',
    progress: 20,
    priority: 'critical',
    dueDate: '2024-04-15',
    team: ['Pedro Martínez', 'Laura Sánchez'],
    taskCount: { total: 18, completed: 4, pending: 14, blocked: 0 },
  },
  {
    id: 'proj-003',
    name: 'App Móvil v2',
    description: 'Nueva versión de la aplicación móvil con funciones offline',
    status: 'completed',
    progress: 100,
    priority: 'medium',
    dueDate: '2024-01-15',
    team: ['Ana García', 'Roberto Kim'],
    taskCount: { total: 32, completed: 32, pending: 0, blocked: 0 },
  },
  {
    id: 'proj-004',
    name: 'Dashboard Analytics',
    description: 'Panel de métricas y reportes en tiempo real',
    status: 'active',
    progress: 45,
    priority: 'medium',
    dueDate: '2024-05-01',
    team: ['Carlos López', 'Sofía Torres', 'Juan Pérez'],
    taskCount: { total: 20, completed: 9, pending: 10, blocked: 1 },
  },
];

export const TASKS = [
  { id: 'task-001', projectId: 'proj-001', title: 'Diseño de wireframes', status: 'completed', priority: 'high', assignee: 'Ana García', dueDate: '2024-02-10' },
  { id: 'task-002', projectId: 'proj-001', title: 'Implementar autenticación SSO', status: 'in-progress', priority: 'critical', assignee: 'Carlos López', dueDate: '2024-03-15' },
  { id: 'task-003', projectId: 'proj-001', title: 'Pruebas de integración', status: 'pending', priority: 'high', assignee: 'María Rodríguez', dueDate: '2024-03-25' },
  { id: 'task-004', projectId: 'proj-001', title: 'Documentación API', status: 'blocked', priority: 'medium', assignee: 'Carlos López', dueDate: '2024-03-28' },
  { id: 'task-005', projectId: 'proj-002', title: 'Análisis de requisitos de seguridad', status: 'completed', priority: 'critical', assignee: 'Pedro Martínez', dueDate: '2024-02-20' },
  { id: 'task-006', projectId: 'proj-002', title: 'Configurar sandbox de pagos', status: 'in-progress', priority: 'high', assignee: 'Laura Sánchez', dueDate: '2024-03-30' },
  { id: 'task-007', projectId: 'proj-004', title: 'Diseño de esquema de base de datos', status: 'completed', priority: 'high', assignee: 'Sofía Torres', dueDate: '2024-02-28' },
  { id: 'task-008', projectId: 'proj-004', title: 'Componente de gráficos', status: 'in-progress', priority: 'medium', assignee: 'Juan Pérez', dueDate: '2024-04-10' },
];

export const NOTIFICATIONS = [
  { id: 'notif-001', type: 'warning', message: 'La tarea "Documentación API" está bloqueada', timestamp: new Date(Date.now() - 300000), read: false },
  { id: 'notif-002', type: 'success', message: 'Proyecto "App Móvil v2" completado exitosamente', timestamp: new Date(Date.now() - 3600000), read: false },
  { id: 'notif-003', type: 'info', message: 'Reunión de sprint programada para mañana 9:00 AM', timestamp: new Date(Date.now() - 7200000), read: true },
];

export const METRICS = {
  totalProjects: 4,
  activeProjects: 2,
  totalTasks: 94,
  completedTasks: 61,
  overdueTasks: 3,
  teamMembers: 7,
};
```

---

#### Paso 2.2 – Crear el Web Component Nativo `status-badge`

**Instrucciones:**

1. Crear `src/components/native/status-badge.js`:

```javascript
// src/components/native/status-badge.js
// Web Component nativo usando la API del navegador
// Demuestra: customElements, Shadow DOM, HTML Template, ciclo de vida completo

const template = document.createElement('template');
template.innerHTML = `
  <style>
    :host {
      display: inline-flex;
      align-items: center;
      gap: 0.25rem;
    }

    .badge {
      display: inline-flex;
      align-items: center;
      gap: 0.3rem;
      padding: 0.2rem 0.6rem;
      border-radius: 9999px;
      font-size: 0.75rem;
      font-weight: 600;
      text-transform: uppercase;
      letter-spacing: 0.05em;
      transition: all 150ms ease;
    }

    .badge--active      { background: #dcfce7; color: #16a34a; }
    .badge--planning    { background: #dbeafe; color: #1d4ed8; }
    .badge--completed   { background: #f3f4f6; color: #6b7280; }
    .badge--blocked     { background: #fee2e2; color: #dc2626; }
    .badge--in-progress { background: #fef3c7; color: #d97706; }
    .badge--pending     { background: #ede9fe; color: #7c3aed; }
    .badge--critical    { background: #fee2e2; color: #dc2626; }
    .badge--high        { background: #fef3c7; color: #d97706; }
    .badge--medium      { background: #dbeafe; color: #1d4ed8; }
    .badge--low         { background: #f3f4f6; color: #6b7280; }

    .dot {
      width: 6px;
      height: 6px;
      border-radius: 50%;
      background: currentColor;
    }
  </style>
  <span class="badge" part="badge" role="status">
    <span class="dot" aria-hidden="true"></span>
    <span class="label"></span>
  </span>
`;

const STATUS_LABELS = {
  active: 'Activo',
  planning: 'Planificación',
  completed: 'Completado',
  blocked: 'Bloqueado',
  'in-progress': 'En Progreso',
  pending: 'Pendiente',
  critical: 'Crítico',
  high: 'Alto',
  medium: 'Medio',
  low: 'Bajo',
};

class StatusBadge extends HTMLElement {
  // Atributos observados – cuando cambian, se llama attributeChangedCallback
  static get observedAttributes() {
    return ['status', 'label'];
  }

  constructor() {
    super();
    // Adjuntar Shadow DOM en modo abierto para encapsulación
    this._shadow = this.attachShadow({ mode: 'open' });
    this._shadow.appendChild(template.content.cloneNode(true));
  }

  // Ciclo de vida: el componente se añade al DOM
  connectedCallback() {
    console.log('[StatusBadge] Conectado al DOM');
    this._render();
  }

  // Ciclo de vida: el componente se elimina del DOM
  disconnectedCallback() {
    console.log('[StatusBadge] Desconectado del DOM');
  }

  // Ciclo de vida: un atributo observado cambió
  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue !== newValue) {
      console.log(`[StatusBadge] Atributo '${name}' cambió: ${oldValue} → ${newValue}`);
      this._render();
    }
  }

  // Ciclo de vida: el componente se mueve a otro documento
  adoptedCallback() {
    console.log('[StatusBadge] Adoptado por nuevo documento');
  }

  // Getters/Setters para acceso programático
  get status() { return this.getAttribute('status') || 'pending'; }
  set status(val) { this.setAttribute('status', val); }

  get label() { return this.getAttribute('label'); }
  set label(val) { this.setAttribute('label', val); }

  // Método privado de renderizado
  _render() {
    const badge = this._shadow.querySelector('.badge');
    const labelEl = this._shadow.querySelector('.label');
    const status = this.status;

    // Limpiar clases anteriores
    badge.className = 'badge';
    badge.classList.add(`badge--${status}`);

    // Actualizar texto
    labelEl.textContent = this.label || STATUS_LABELS[status] || status;

    // Accesibilidad: actualizar aria-label
    badge.setAttribute('aria-label', `Estado: ${labelEl.textContent}`);
  }
}

// Registrar el custom element
customElements.define('status-badge', StatusBadge);

export { StatusBadge };
```

2. Actualizar `src/main.js` para importar el componente nativo:

```javascript
// src/main.js
import './components/native/status-badge.js';
console.log('TaskDash: componentes nativos cargados');
```

3. Añadir una prueba visual temporal en `index.html` (dentro del `<main>`, antes de `<app-dashboard>`):

```html
<!-- Prueba temporal de Web Component nativo – remover en Fase 7 -->
<section id="native-test" style="padding: 1rem; display: flex; gap: 1rem; flex-wrap: wrap;">
  <status-badge status="active"></status-badge>
  <status-badge status="planning"></status-badge>
  <status-badge status="completed"></status-badge>
  <status-badge status="blocked"></status-badge>
  <status-badge status="in-progress"></status-badge>
  <status-badge status="critical" label="Urgente"></status-badge>
</section>
```

**Resultado esperado:** Al recargar el navegador, deben verse 6 badges de colores diferentes con los estados correspondientes. Abrir la consola del navegador (F12) para ver los mensajes del ciclo de vida.

**Verificación:** En la consola del navegador debe aparecer `[StatusBadge] Conectado al DOM` 6 veces.

---

### Fase 3: Introducción a Lit (35 min)

**Objetivo:** Configurar LitElement, crear el primer componente Lit y comparar con la implementación nativa.

#### Paso 3.1 – Crear el Componente `theme-toggle` con Lit

**Instrucciones:**

1. Crear `src/components/lit/theme-toggle.js`:

```javascript
// src/components/lit/theme-toggle.js
// Primer componente Lit – comparar con StatusBadge nativo
import { LitElement, html, css } from 'lit';

class ThemeToggle extends LitElement {
  // Estilos encapsulados con css`` tagged template literal
  static styles = css`
    :host {
      display: inline-flex;
      align-items: center;
    }

    button {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      padding: 0.4rem 0.8rem;
      border: 1px solid var(--color-border, #e2e8f0);
      border-radius: 9999px;
      background: var(--color-surface, #fff);
      color: var(--color-text-primary, #0f172a);
      font-size: 0.875rem;
      font-weight: 500;
      cursor: pointer;
      transition: all 150ms ease;
    }

    button:hover {
      background: var(--color-surface-raised, #f1f5f9);
      transform: scale(1.02);
    }

    button:focus-visible {
      outline: 2px solid var(--color-primary, #2563eb);
      outline-offset: 2px;
    }

    .icon { font-size: 1rem; }
  `;

  // Propiedades reactivas con @property
  static properties = {
    theme: { type: String, reflect: true },
  };

  constructor() {
    super();
    // Inicializar con el tema guardado o el del sistema
    const saved = localStorage.getItem('taskdash-theme');
    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    this.theme = saved || (prefersDark ? 'dark' : 'light');
  }

  // connectedCallback equivalente – se llama cuando Lit monta el componente
  connectedCallback() {
    super.connectedCallback(); // ¡Siempre llamar a super!
    this._applyTheme();
  }

  _applyTheme() {
    document.documentElement.setAttribute('data-theme', this.theme);
    localStorage.setItem('taskdash-theme', this.theme);
  }

  _toggleTheme() {
    this.theme = this.theme === 'light' ? 'dark' : 'light';
    this._applyTheme();
    // Emitir evento para comunicar el cambio
    this.dispatchEvent(new CustomEvent('theme-changed', {
      detail: { theme: this.theme },
      bubbles: true,
      composed: true, // Atraviesa el Shadow DOM
    }));
  }

  // Template con html`` tagged template literal
  render() {
    const isDark = this.theme === 'dark';
    return html`
      <button
        @click=${this._toggleTheme}
        aria-label="${isDark ? 'Cambiar a tema claro' : 'Cambiar a tema oscuro'}"
        aria-pressed="${isDark}"
        title="${isDark ? 'Activar tema claro' : 'Activar tema oscuro'}"
      >
        <span class="icon" aria-hidden="true">${isDark ? '☀️' : '🌙'}</span>
        <span>${isDark ? 'Claro' : 'Oscuro'}</span>
      </button>
    `;
  }
}

customElements.define('theme-toggle', ThemeToggle);
export { ThemeToggle };
```

2. Actualizar `src/main.js`:

```javascript
// src/main.js
import './components/native/status-badge.js';
import './components/lit/theme-toggle.js';

// Montar theme-toggle en el header
const container = document.getElementById('theme-toggle-container');
if (container) {
  const toggle = document.createElement('theme-toggle');
  container.appendChild(toggle);
}

console.log('TaskDash: todos los componentes cargados');
```

**Resultado esperado:** En el header debe aparecer el botón de cambio de tema. Al hacer clic, el fondo de la página cambia entre claro y oscuro gracias a las variables CSS definidas en Fase 1.

---

### Fase 4: Propiedades Reactivas y Templates (40 min)

**Objetivo:** Implementar los componentes principales del dashboard con propiedades reactivas y directivas Lit avanzadas.

#### Paso 4.1 – Crear el Componente `project-card`

**Instrucciones:**

1. Crear `src/components/lit/project-card.js`:

```javascript
// src/components/lit/project-card.js
import { LitElement, html, css } from 'lit';
import { classMap } from 'lit/directives/class-map.js';
import { when } from 'lit/directives/when.js';
import '../native/status-badge.js';

class ProjectCard extends LitElement {
  static styles = css`
    :host { display: block; }

    .card {
      background: var(--color-surface);
      border: 1px solid var(--color-border);
      border-radius: var(--radius-lg, 0.75rem);
      padding: var(--space-lg, 1.5rem);
      box-shadow: var(--shadow-sm);
      transition: box-shadow var(--transition-base, 250ms ease),
                  transform var(--transition-base, 250ms ease);
      cursor: pointer;
    }

    .card:hover {
      box-shadow: var(--shadow-md);
      transform: translateY(-2px);
    }

    .card--critical { border-left: 4px solid #dc2626; }
    .card--high     { border-left: 4px solid #d97706; }
    .card--medium   { border-left: 4px solid #2563eb; }
    .card--low      { border-left: 4px solid #6b7280; }

    .card-header {
      display: flex;
      justify-content: space-between;
      align-items: flex-start;
      margin-bottom: var(--space-md, 1rem);
      gap: var(--space-sm, 0.5rem);
    }

    .card-title {
      font-size: var(--font-size-lg, 1.125rem);
      font-weight: 600;
      color: var(--color-text-primary);
      margin: 0;
    }

    .card-description {
      font-size: var(--font-size-sm, 0.875rem);
      color: var(--color-text-secondary);
      margin-bottom: var(--space-md, 1rem);
      line-height: 1.5;
    }

    .progress-container { margin-bottom: var(--space-md, 1rem); }

    .progress-label {
      display: flex;
      justify-content: space-between;
      font-size: var(--font-size-sm, 0.875rem);
      color: var(--color-text-secondary);
      margin-bottom: var(--space-xs, 0.25rem);
    }

    .progress-bar {
      height: 6px;
      background: var(--color-surface-raised, #f1f5f9);
      border-radius: 9999px;
      overflow: hidden;
    }

    .progress-fill {
      height: 100%;
      border-radius: 9999px;
      background: var(--color-primary, #2563eb);
      transition: width var(--transition-slow, 400ms ease);
    }

    .progress-fill--completed { background: #16a34a; }
    .progress-fill--blocked   { background: #dc2626; }

    .card-footer {
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: var(--font-size-sm, 0.875rem);
      color: var(--color-text-muted);
      border-top: 1px solid var(--color-border);
      padding-top: var(--space-sm, 0.5rem);
      margin-top: var(--space-sm, 0.5rem);
    }

    .team-avatars { display: flex; }

    .avatar {
      width: 24px;
      height: 24px;
      border-radius: 50%;
      background: var(--color-primary, #2563eb);
      color: white;
      font-size: 0.625rem;
      font-weight: 700;
      display: flex;
      align-items: center;
      justify-content: center;
      border: 2px solid var(--color-surface);
      margin-left: -4px;
    }

    .avatar:first-child { margin-left: 0; }

    .due-date {
      display: flex;
      align-items: center;
      gap: 0.25rem;
    }

    .overdue { color: #dc2626; font-weight: 600; }
  `;

  static properties = {
    project: { type: Object },
    selected: { type: Boolean, reflect: true },
  };

  constructor() {
    super();
    this.project = null;
    this.selected = false;
  }

  _handleClick() {
    this.dispatchEvent(new CustomEvent('project-selected', {
      detail: { project: this.project },
      bubbles: true,
      composed: true,
    }));
  }

  _isOverdue(dateStr) {
    return new Date(dateStr) < new Date() && this.project?.status !== 'completed';
  }

  _getInitials(name) {
    return name.split(' ').map(n => n[0]).join('').substring(0, 2).toUpperCase();
  }

  render() {
    if (!this.project) return html`<div class="card">Cargando...</div>`;

    const { name, description, status, progress, priority, dueDate, team, taskCount } = this.project;
    const overdue = this._isOverdue(dueDate);

    const cardClasses = classMap({
      card: true,
      [`card--${priority}`]: true,
    });

    const fillClasses = classMap({
      'progress-fill': true,
      'progress-fill--completed': status === 'completed',
      'progress-fill--blocked': taskCount.blocked > 0,
    });

    return html`
      <article
        class=${cardClasses}
        @click=${this._handleClick}
        role="button"
        tabindex="0"
        aria-label="Proyecto: ${name}, ${progress}% completado"
        @keydown=${(e) => e.key === 'Enter' && this._handleClick()}
      >
        <header class="card-header">
          <h3 class="card-title">${name}</h3>
          <status-badge status=${status}></status-badge>
        </header>

        <p class="card-description">${description}</p>

        <div class="progress-container">
          <div class="progress-label">
            <span>Progreso</span>
            <span>${progress}%</span>
          </div>
          <div class="progress-bar" role="progressbar"
               aria-valuenow=${progress} aria-valuemin="0" aria-valuemax="100">
            <div class=${fillClasses} style="width: ${progress}%"></div>
          </div>
        </div>

        <footer class="card-footer">
          <div class="team-avatars" aria-label="Equipo: ${team.join(', ')}">
            ${team.slice(0, 4).map(member => html`
              <div class="avatar" title="${member}">${this._getInitials(member)}</div>
            `)}
            ${when(team.length > 4, () => html`
              <div class="avatar">+${team.length - 4}</div>
            `)}
          </div>
          <div class="due-date ${overdue ? 'overdue' : ''}"
               aria-label="${overdue ? 'Vencido' : 'Vence'}: ${dueDate}">
            <span aria-hidden="true">${overdue ? '⚠️' : '📅'}</span>
            <time datetime=${dueDate}>${dueDate}</time>
          </div>
        </footer>
      </article>
    `;
  }
}

customElements.define('project-card', ProjectCard);
export { ProjectCard };
```

---

#### Paso 4.2 – Crear el Componente `task-list`

**Instrucciones:**

1. Crear `src/components/lit/task-list.js`:

```javascript
// src/components/lit/task-list.js
import { LitElement, html, css } from 'lit';
import { repeat } from 'lit/directives/repeat.js';
import { classMap } from 'lit/directives/class-map.js';
import { when } from 'lit/directives/when.js';
import '../native/status-badge.js';

class TaskList extends LitElement {
  static styles = css`
    :host { display: block; }

    .task-list-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: var(--space-md, 1rem);
    }

    h2 {
      font-size: var(--font-size-xl, 1.25rem);
      font-weight: 600;
      color: var(--color-text-primary);
    }

    .filter-group { display: flex; gap: var(--space-xs, 0.25rem); }

    .filter-btn {
      padding: 0.3rem 0.7rem;
      border: 1px solid var(--color-border);
      border-radius: 9999px;
      background: transparent;
      font-size: 0.75rem;
      cursor: pointer;
      transition: all 150ms ease;
      color: var(--color-text-secondary);
    }

    .filter-btn--active,
    .filter-btn:hover {
      background: var(--color-primary, #2563eb);
      color: white;
      border-color: var(--color-primary, #2563eb);
    }

    .task-item {
      display: flex;
      align-items: center;
      gap: var(--space-md, 1rem);
      padding: var(--space-md, 1rem);
      border: 1px solid var(--color-border);
      border-radius: var(--radius-md, 0.5rem);
      margin-bottom: var(--space-sm, 0.5rem);
      background: var(--color-surface);
      transition: all 150ms ease;
      animation: slideIn 200ms ease;
    }

    @keyframes slideIn {
      from { opacity: 0; transform: translateX(-10px); }
      to   { opacity: 1; transform: translateX(0); }
    }

    .task-item:hover { box-shadow: var(--shadow-sm); }
    .task-item--completed { opacity: 0.6; }

    .task-title {
      flex: 1;
      font-weight: 500;
      color: var(--color-text-primary);
    }

    .task-title--completed { text-decoration: line-through; }

    .task-assignee {
      font-size: var(--font-size-sm, 0.875rem);
      color: var(--color-text-muted);
    }

    .task-due {
      font-size: var(--font-size-sm, 0.875rem);
      color: var(--color-text-muted);
    }

    .empty-state {
      text-align: center;
      padding: var(--space-2xl, 3rem);
      color: var(--color-text-muted);
    }

    .complete-btn {
      padding: 0.2rem 0.5rem;
      border: 1px solid var(--color-border);
      border-radius: var(--radius-sm, 0.25rem);
      background: transparent;
      cursor: pointer;
      font-size: 0.75rem;
      transition: all 150ms ease;
    }

    .complete-btn:hover {
      background: #dcfce7;
      border-color: #16a34a;
      color: #16a34a;
    }
  `;

  static properties = {
    tasks: { type: Array },
    filter: { type: String, state: true },
    projectId: { type: String },
  };

  constructor() {
    super();
    this.tasks = [];
    this.filter = 'all';
    this.projectId = null;
  }

  get filteredTasks() {
    const projectTasks = this.projectId
      ? this.tasks.filter(t => t.projectId === this.projectId)
      : this.tasks;

    if (this.filter === 'all') return projectTasks;
    return projectTasks.filter(t => t.status === this.filter);
  }

  _setFilter(filter) { this.filter = filter; }

  _completeTask(task) {
    this.dispatchEvent(new CustomEvent('task-completed', {
      detail: { taskId: task.id, task },
      bubbles: true,
      composed: true,
    }));
  }

  render() {
    const filters = ['all', 'pending', 'in-progress', 'completed', 'blocked'];
    const filtered = this.filteredTasks;

    return html`
      <section aria-labelledby="task-list-title">
        <div class="task-list-header">
          <h2 id="task-list-title">
            Tareas
            <span aria-live="polite">(${filtered.length})</span>
          </h2>
          <div class="filter-group" role="group" aria-label="Filtrar tareas por estado">
            ${filters.map(f => html`
              <button
                class="filter-btn ${this.filter === f ? 'filter-btn--active' : ''}"
                @click=${() => this._setFilter(f)}
                aria-pressed=${this.filter === f}
              >
                ${f === 'all' ? 'Todas' : f}
              </button>
            `)}
          </div>
        </div>

        ${when(
          filtered.length === 0,
          () => html`
            <div class="empty-state" role="status">
              <p>No hay tareas para mostrar</p>
            </div>
          `,
          () => html`
            <ul role="list" style="list-style: none; padding: 0;">
              ${repeat(
                filtered,
                (task) => task.id,
                (task) => html`
                  <li>
                    <article
                      class=${classMap({
                        'task-item': true,
                        'task-item--completed': task.status === 'completed',
                      })}
                    >
                      <status-badge status=${task.status}></status-badge>
                      <span class=${classMap({
                        'task-title': true,
                        'task-title--completed': task.status === 'completed',
                      })}>
                        ${task.title}
                      </span>
                      <span class="task-assignee">👤 ${task.assignee}</span>
                      <time class="task-due" datetime=${task.dueDate}>
                        📅 ${task.dueDate}
                      </time>
                      ${when(
                        task.status !== 'completed',
                        () => html`
                          <button
                            class="complete-btn"
                            @click=${() => this._completeTask(task)}
                            aria-label="Marcar '${task.title}' como completada"
                          >
                            ✔ Completar
                          </button>
                        `
                      )}
                    </article>
                  </li>
                `
              )}
            </ul>
          `
        )}
      </section>
    `;
  }
}

customElements.define('task-list', TaskList);
export { TaskList };
```

---

### Fase 5: Comunicación entre Componentes (35 min)

**Objetivo:** Implementar el servicio de estado, el panel de notificaciones y el componente orquestador `app-dashboard`.

#### Paso 5.1 – Crear el Servicio de Estado

**Instrucciones:**

1. Crear el directorio `src/services/` y el archivo `state-service.js`:

```javascript
// src/services/state-service.js
// Servicio de estado simple (patrón Observable) para comunicación entre componentes hermanos

class StateService extends EventTarget {
  constructor() {
    super();
    this._state = {
      selectedProjectId: null,
      tasks: [],
      projects: [],
      notifications: [],
      theme: 'light',
    };
  }

  // Getter del estado actual (inmutable)
  get state() {
    return Object.freeze({ ...this._state });
  }

  // Actualizar estado y notificar suscriptores
  setState(updates) {
    const prevState = { ...this._state };
    this._state = { ...this._state, ...updates };

    this.dispatchEvent(new CustomEvent('state-changed', {
      detail: {
        prev: prevState,
        next: this._state,
        changes: Object.keys(updates),
      },
    }));
  }

  // Suscribirse a cambios de estado
  subscribe(callback) {
    this.addEventListener('state-changed', callback);
    return () => this.removeEventListener('state-changed', callback);
  }

  // Acciones específicas del dominio
  selectProject(projectId) {
    this.setState({ selectedProjectId: projectId });
  }

  completeTask(taskId) {
    const tasks = this._state.tasks.map(t =>
      t.id === taskId ? { ...t, status: 'completed' } : t
    );
    this.setState({ tasks });
    this.addNotification('success', `Tarea completada exitosamente`);
  }

  addNotification(type, message) {
    const notification = {
      id: `notif-${Date.now()}`,
      type,
      message,
      timestamp: new Date(),
      read: false,
    };
    this.setState({
      notifications: [notification, ...this._state.notifications].slice(0, 10),
    });
  }

  markNotificationRead(notifId) {
    const notifications = this._state.notifications.map(n =>
      n.id === notifId ? { ...n, read: true } : n
    );
    this.setState({ notifications });
  }
}

// Singleton – una sola instancia compartida por toda la app
export const stateService = new StateService();
```

---

#### Paso 5.2 – Crear el Componente `notification-panel`

**Instrucciones:**

1. Crear `src/components/lit/notification-panel.js`:

```javascript
// src/components/lit/notification-panel.js
import { LitElement, html, css } from 'lit';
import { repeat } from 'lit/directives/repeat.js';
import { classMap } from 'lit/directives/class-map.js';
import { stateService } from '../../services/state-service.js';

class NotificationPanel extends LitElement {
  static styles = css`
    :host { display: block; }

    .notification {
      display: flex;
      align-items: flex-start;
      gap: 0.75rem;
      padding: 0.75rem 1rem;
      background: var(--color-surface);
      border: 1px solid var(--color-border);
      border-radius: var(--radius-md, 0.5rem);
      box-shadow: var(--shadow-md);
      min-width: 300px;
      max-width: 380px;
      animation: slideInRight 250ms ease;
      transition: opacity var(--transition-base, 250ms ease);
    }

    @keyframes slideInRight {
      from { opacity: 0; transform: translateX(20px); }
      to   { opacity: 1; transform: translateX(0); }
    }

    .notification--unread  { border-left: 3px solid var(--color-primary, #2563eb); }
    .notification--success { border-left-color: #16a34a; }
    .notification--warning { border-left-color: #d97706; }
    .notification--error   { border-left-color: #dc2626; }
    .notification--info    { border-left-color: #0891b2; }

    .notif-icon    { font-size: 1.2rem; flex-shrink: 0; }
    .notif-content { flex: 1; }

    .notif-message {
      font-size: 0.875rem;
      color: var(--color-text-primary);
      line-height: 1.4;
    }

    .notif-time {
      font-size: 0.75rem;
      color: var(--color-text-muted);
      margin-top: 0.25rem;
    }

    .dismiss-btn {
      background: none;
      border: none;
      cursor: pointer;
      color: var(--color-text-muted);
      padding: 0.2rem;
      border-radius: var(--radius-sm, 0.25rem);
      line-height: 1;
      transition: color 150ms ease;
    }

    .dismiss-btn:hover { color: var(--color-text-primary); }
  `;

  static properties = {
    _notifications: { type: Array, state: true },
  };

  constructor() {
    super();
    this._notifications = [];
    this._unsubscribe = null;
  }

  connectedCallback() {
    super.connectedCallback();
    this._unsubscribe = stateService.subscribe((event) => {
      if (event.detail.changes.includes('notifications')) {
        this._notifications = stateService.state.notifications.filter(n => !n.read);
      }
    });
    this._notifications = stateService.state.notifications.filter(n => !n.read);
  }

  disconnectedCallback() {
    super.disconnectedCallback();
    if (this._unsubscribe) this._unsubscribe();
  }

  _dismiss(notifId) {
    stateService.markNotificationRead(notifId);
  }

  _getIcon(type) {
    const icons = { success: '✅', warning: '⚠️', error: '❌', info: 'ℹ️' };
    return icons[type] || 'ℹ️';
  }

  _formatTime(timestamp) {
    const diff = Date.now() - new Date(timestamp).getTime();
    if (diff < 60000) return 'Ahora mismo';
    if (diff < 3600000) return `Hace ${Math.floor(diff / 60000)} min`;
    return `Hace ${Math.floor(diff / 3600000)} h`;
  }

  render() {
    return html`
      <div role="log" aria-label="Notificaciones">
        ${repeat(
          this._notifications,
          (n) => n.id,
          (n) => html`
            <div
              class=${classMap({
                notification: true,
                'notification--unread': !n.read,
                [`notification--${n.type}`]: true,
              })}
              role="alert"
            >
              <span class="notif-icon" aria-hidden="true">${this._getIcon(n.type)}</span>
              <div class="notif-content">
                <p class="notif-message">${n.message}</p>
                <time class="notif-time" datetime=${n.timestamp.toISOString()}>
                  ${this._formatTime(n.timestamp)}
                </time>
              </div>
              <button
                class="dismiss-btn"
                @click=${() => this._dismiss(n.id)}
                aria-label="Cerrar notificación"
              >✕</button>
            </div>
          `
        )}
      </div>
    `;
  }
}

customElements.define('notification-panel', NotificationPanel);
export { NotificationPanel };
```

---

#### Paso 5.3 – Crear el Componente Orquestador `app-dashboard`

**Instrucciones:**

1. Crear `src/components/lit/app-dashboard.js`:

```javascript
// src/components/lit/app-dashboard.js
// Componente raíz que orquesta toda la aplicación
import { LitElement, html, css } from 'lit';
import { repeat } from 'lit/directives/repeat.js';
import { when } from 'lit/directives/when.js';
import { PROJECTS, TASKS, NOTIFICATIONS, METRICS } from '../../data/mock-data.js';
import { stateService } from '../../services/state-service.js';
import './project-card.js';
import './task-list.js';

class AppDashboard extends LitElement {
  static styles = css`
    :host { display: block; }

    .metrics-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
      gap: var(--space-md, 1rem);
      margin-bottom: var(--space-xl, 2rem);
    }

    .metric-card {
      background: var(--color-surface);
      border: 1px solid var(--color-border);
      border-radius: var(--radius-lg, 0.75rem);
      padding: var(--space-lg, 1.5rem);
      text-align: center;
      box-shadow: var(--shadow-sm);
    }

    .metric-value {
      font-size: var(--font-size-3xl, 1.875rem);
      font-weight: 700;
      color: var(--color-primary, #2563eb);
      display: block;
    }

    .metric-label {
      font-size: var(--font-size-sm, 0.875rem);
      color: var(--color-text-secondary);
      margin-top: var(--space-xs, 0.25rem);
    }

    .section-title {
      font-size: var(--font-size-xl, 1.25rem);
      font-weight: 600;
      color: var(--color-text-primary);
      margin-bottom: var(--space-md, 1rem);
      display: flex;
      align-items: center;
      gap: var(--space-sm, 0.5rem);
    }

    .projects-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
      gap: var(--space-md, 1rem);
      margin-bottom: var(--space-xl, 2rem);
    }

    .dashboard-layout {
      display: grid;
      grid-template-columns: 1fr;
      gap: var(--space-xl, 2rem);
    }

    @media (min-width: 1024px) {
      .dashboard-layout { grid-template-columns: 1fr 380px; }
    }

    .filter-input {
      width: 100%;
      max-width: 300px;
      padding: 0.5rem 1rem;
      border: 1px solid var(--color-border);
      border-radius: var(--radius-md, 0.5rem);
      background: var(--color-surface);
      color: var(--color-text-primary);
      font-size: var(--font-size-sm, 0.875rem);
      margin-bottom: var(--space-md, 1rem);
    }
  `;

  static properties = {
    _selectedProjectId: { type: String, state: true },
    _projectFilter: { type: String, state: true },
    _tasks: { type: Array, state: true },
  };

  constructor() {
    super();
    this._selectedProjectId = null;
    this._projectFilter = '';
    this._tasks = [...TASKS];
    this._unsubscribe = null;

    stateService.setState({
      projects: PROJECTS,
      tasks: TASKS,
      notifications: NOTIFICATIONS,
    });
  }

  connectedCallback() {
    super.connectedCallback();
    this._unsubscribe = stateService.subscribe((event) => {
      const { changes, next } = event.detail;
      if (changes.includes('selectedProjectId')) {
        this._selectedProjectId = next.selectedProjectId;
      }
      if (changes.includes('tasks')) {
        this._tasks = next.tasks;
      }
    });
  }

  disconnectedCallback() {
    super.disconnectedCallback();
    if (this._unsubscribe) this._unsubscribe();
  }

  _handleProjectSelected(event) {
    stateService.selectProject(event.detail.project.id);
  }

  _handleTaskCompleted(event) {
    stateService.completeTask(event.detail.taskId);
  }

  get filteredProjects() {
    if (!this._projectFilter) return PROJECTS;
    const q = this._projectFilter.toLowerCase();
    return PROJECTS.filter(p =>
      p.name.toLowerCase().includes(q) ||
      p.description.toLowerCase().includes(q)
    );
  }

  render() {
    const selectedProject = this._selectedProjectId
      ? PROJECTS.find(p => p.id === this._selectedProjectId)
      : null;

    return html`
      <!-- Sección de métricas -->
      <section aria-labelledby="metrics-title">
        <h2 id="metrics-title" class="section-title">
          <span aria-hidden="true">📊</span> Resumen Ejecutivo
        </h2>
        <div class="metrics-grid">
          ${Object.entries({
            'Proyectos Totales': METRICS.totalProjects,
            'Proyectos Activos': METRICS.activeProjects,
            'Tareas Totales': METRICS.totalTasks,
            'Tareas Completadas': METRICS.completedTasks,
            'Tareas Vencidas': METRICS.overdueTasks,
            'Miembros del Equipo': METRICS.teamMembers,
          }).map(([label, value]) => html`
            <div class="metric-card">
              <span class="metric-value">${value}</span>
              <p class="metric-label">${label}</p>
            </div>
          `)}
        </div>
      </section>

      <div class="dashboard-layout">
        <div>
          <!-- Sección de proyectos -->
          <section aria-labelledby="projects-title">
            <h2 id="projects-title" class="section-title">
              <span aria-hidden="true">📁</span> Proyectos
            </h2>
            <input
              type="search"
              class="filter-input"
              placeholder="Buscar proyectos..."
              .value=${this._projectFilter}
              @input=${(e) => { this._projectFilter = e.target.value; }}
              aria-label="Filtrar proyectos"
            />
            <div class="projects-grid">
              ${repeat(
                this.filteredProjects,
                (p) => p.id,
                (p) => html`
                  <project-card
                    .project=${p}
                    ?selected=${p.id === this._selectedProjectId}
                    @project-selected=${this._handleProjectSelected}
                  ></project-card>
                `
              )}
            </div>
          </section>

          <!-- Sección de tareas -->
          <section aria-labelledby="tasks-title">
            ${when(
              selectedProject,
              () => html`
                <h2 id="tasks-title" class="section-title">
                  <span aria-hidden="true">✅</span>
                  Tareas de: ${selectedProject.name}
                </h2>
              `,
              () => html`
                <h2 id="tasks-title" class="section-title">
                  <span aria-hidden="true">✅</span> Todas las Tareas
                </h2>
              `
            )}
            <task-list
              .tasks=${this._tasks}
              .projectId=${this._selectedProjectId}
              @task-completed=${this._handleTaskCompleted}
            ></task-list>
          </section>
        </div>
      </div>
    `;
  }
}

customElements.define('app-dashboard', AppDashboard);
export { AppDashboard };
```

2. Actualizar `src/main.js` con todas las importaciones:

```javascript
// src/main.js – Punto de entrada final
import './components/native/status-badge.js';
import './components/lit/theme-toggle.js';
import './components/lit/notification-panel.js';
import './components/lit/app-dashboard.js';

// Montar theme-toggle en el header
const container = document.getElementById('theme-toggle-container');
if (container) {
  const toggle = document.createElement('theme-toggle');
  container.appendChild(toggle);
}

console.log('TaskDash: aplicación iniciada correctamente');
```

3. Eliminar la sección de prueba nativa del `index.html` (el `<section id="native-test">`).

**Resultado esperado:** El dashboard completo debe mostrarse con métricas, tarjetas de proyectos filtrables y lista de tareas. Al hacer clic en una tarjeta de proyecto, la lista de tareas se filtra automáticamente.

---

### Fase 6: Pruebas Unitarias con Vitest (30 min)

**Objetivo:** Escribir y ejecutar pruebas unitarias para 3 componentes Lit validando renderizado, reactividad y emisión de eventos.

#### Paso 6.1 – Pruebas para `project-card`

**Instrucciones:**

1. Crear `src/tests/project-card.test.js`:

```javascript
// src/tests/project-card.test.js
import { describe, it, expect, beforeEach, afterEach } from 'vitest';

import '../components/native/status-badge.js';
import '../components/lit/project-card.js';

const MOCK_PROJECT = {
  id: 'test-proj-001',
  name: 'Proyecto de Prueba',
  description: 'Descripción del proyecto de prueba',
  status: 'active',
  progress: 75,
  priority: 'high',
  dueDate: '2025-12-31',
  team: ['Ana García', 'Carlos López', 'María Rodríguez'],
  taskCount: { total: 10, completed: 7, pending: 2, blocked: 1 },
};

describe('ProjectCard Component', () => {
  let element;

  beforeEach(async () => {
    element = document.createElement('project-card');
    document.body.appendChild(element);
    await element.updateComplete;
  });

  afterEach(() => {
    if (element && element.parentNode) {
      element.parentNode.removeChild(element);
    }
  });

  describe('Renderizado inicial', () => {
    it('debe renderizarse en el DOM sin errores', () => {
      expect(element).toBeDefined();
      expect(element.shadowRoot).toBeTruthy();
    });

    it('debe mostrar "Cargando..." cuando no hay proyecto', async () => {
      const shadowText = element.shadowRoot.textContent;
      expect(shadowText).toContain('Cargando');
    });
  });

  describe('Propiedades reactivas', () => {
    it('debe mostrar el nombre del proyecto cuando se asigna .project', async () => {
      element.project = MOCK_PROJECT;
      await element.updateComplete;

      const title = element.shadowRoot.querySelector('.card-title');
      expect(title).toBeTruthy();
      expect(title.textContent).toBe('Proyecto de Prueba');
    });

    it('debe mostrar el progreso correcto', async () => {
      element.project = MOCK_PROJECT;
      await element.updateComplete;

      const progressFill = element.shadowRoot.querySelector('.progress-fill');
      expect(progressFill).toBeTruthy();
      expect(progressFill.style.width).toBe('75%');
    });

    it('debe aplicar la clase de prioridad correcta', async () => {
      element.project = MOCK_PROJECT;
      await element.updateComplete;

      const card = element.shadowRoot.querySelector('.card');
      expect(card.classList.contains('card--high')).toBe(true);
    });

    it('debe actualizar el renderizado cuando cambia el proyecto', async () => {
      element.project = MOCK_PROJECT;
      await element.updateComplete;

      element.project = { ...MOCK_PROJECT, name: 'Proyecto Actualizado', progress: 90 };
      await element.updateComplete;

      const title = element.shadowRoot.querySelector('.card-title');
      expect(title.textContent).toBe('Proyecto Actualizado');

      const progressFill = element.shadowRoot.querySelector('.progress-fill');
      expect(progressFill.style.width).toBe('90%');
    });
  });

  describe('Emisión de eventos', () => {
    it('debe emitir "project-selected" al hacer clic', async () => {
      element.project = MOCK_PROJECT;
      await element.updateComplete;

      const eventPromise = new Promise(resolve => {
        element.addEventListener('project-selected', resolve);
      });

      const card = element.shadowRoot.querySelector('.card');
      card.click();

      const event = await eventPromise;
      expect(event.type).toBe('project-selected');
      expect(event.detail.project.id).toBe('test-proj-001');
      expect(event.detail.project.name).toBe('Proyecto de Prueba');
    });
  });
});
```

---

#### Paso 6.2 – Pruebas para `task-list`

**Instrucciones:**

1. Crear `src/tests/task-list.test.js`:

```javascript
// src/tests/task-list.test.js
import { describe, it, expect, beforeEach, afterEach } from 'vitest';
import '../components/native/status-badge.js';
import '../components/lit/task-list.js';

const MOCK_TASKS = [
  { id: 't1', projectId: 'p1', title: 'Tarea Pendiente',    status: 'pending',     priority: 'high',   assignee: 'Ana',    dueDate: '2024-12-31' },
  { id: 't2', projectId: 'p1', title: 'Tarea En Progreso',  status: 'in-progress', priority: 'medium', assignee: 'Carlos', dueDate: '2024-12-31' },
  { id: 't3', projectId: 'p1', title: 'Tarea Completada',   status: 'completed',   priority: 'low',    assignee: 'María',  dueDate: '2024-12-31' },
  { id: 't4', projectId: 'p2', title: 'Tarea Otro Proyecto',status: 'pending',     priority: 'high',   assignee: 'Pedro',  dueDate: '2024-12-31' },
];

describe('TaskList Component', () => {
  let element;

  beforeEach(async () => {
    element = document.createElement('task-list');
    document.body.appendChild(element);
    await element.updateComplete;
  });

  afterEach(() => {
    if (element?.parentNode) element.parentNode.removeChild(element);
  });

  describe('Renderizado inicial', () => {
    it('debe renderizarse correctamente', () => {
      expect(element.shadowRoot).toBeTruthy();
    });

    it('debe mostrar estado vacío cuando no hay tareas', async () => {
      const text = element.shadowRoot.textContent;
      expect(text).toContain('No hay tareas');
    });
  });

  describe('Filtrado de tareas', () => {
    it('debe mostrar todas las tareas con filtro "all"', async () => {
      element.tasks = MOCK_TASKS;
      await element.updateComplete;

      const items = element.shadowRoot.querySelectorAll('.task-item');
      expect(items.length).toBe(4);
    });

    it('debe filtrar tareas por projectId', async () => {
      element.tasks = MOCK_TASKS;
      element.projectId = 'p1';
      await element.updateComplete;

      const items = element.shadowRoot.querySelectorAll('.task-item');
      expect(items.length).toBe(3);
    });

    it('debe actualizar la lista al cambiar el filtro de estado', async () => {
      element.tasks = MOCK_TASKS;
      await element.updateComplete;

      const filterBtn = [...element.shadowRoot.querySelectorAll('.filter-btn')]
        .find(btn => btn.textContent.trim() === 'completed');
      filterBtn.click();
      await element.updateComplete;

      const items = element.shadowRoot.querySelectorAll('.task-item');
      expect(items.length).toBe(1);
    });
  });

  describe('Emisión de eventos', () => {
    it('debe emitir "task-completed" al hacer clic en Completar', async () => {
      element.tasks = MOCK_TASKS;
      await element.updateComplete;

      const eventPromise = new Promise(resolve => {
        element.addEventListener('task-completed', resolve);
      });

      const completeBtn = element.shadowRoot.querySelector('.complete-btn');
      completeBtn.click();

      const event = await eventPromise;
      expect(event.type).toBe('task-completed');
      expect(event.detail.taskId).toBeDefined();
    });
  });
});
```

---

#### Paso 6.3 – Pruebas para `notification-panel`

**Instrucciones:**

1. Crear `src/tests/notification-panel.test.js`:

```javascript
// src/tests/notification-panel.test.js
import { describe, it, expect, beforeEach, afterEach } from 'vitest';
import '../components/lit/notification-panel.js';
import { stateService } from '../services/state-service.js';

describe('NotificationPanel Component', () => {
  let element;

  beforeEach(async () => {
    // Limpiar estado antes de cada prueba
    stateService.setState({ notifications: [] });
    element = document.createElement('notification-panel');
    document.body.appendChild(element);
    await element.updateComplete;
  });

  afterEach(() => {
    if (element?.parentNode) element.parentNode.removeChild(element);
  });

  describe('Renderizado inicial', () => {
    it('debe renderizarse sin errores', () => {
      expect(element.shadowRoot).toBeTruthy();
    });

    it('debe mostrar panel vacío sin notificaciones', async () => {
      const notifications = element.shadowRoot.querySelectorAll('.notification');
      expect(notifications.length).toBe(0);
    });
  });

  describe('Reactividad con el servicio de estado', () => {
    it('debe mostrar nuevas notificaciones al agregarlas al servicio', async () => {
      stateService.addNotification('info', 'Mensaje de prueba');
      await element.updateComplete;

      const notifications = element.shadowRoot.querySelectorAll('.notification');
      expect(notifications.length).toBe(1);
    });

    it('debe mostrar el mensaje correcto en la notificación', async () => {
      stateService.addNotification('success', 'Operación exitosa');
      await element.updateComplete;

      const message = element.shadowRoot.querySelector('.notif-message');
      expect(message.textContent).toBe('Operación exitosa');
    });

    it('debe aplicar la clase de tipo correcta', async () => {
      stateService.addNotification('warning', 'Advertencia importante');
      await element.updateComplete;

      const notif = element.shadowRoot.querySelector('.notification');
      expect(notif.classList.contains('notification--warning')).toBe(true);
    });
  });

  describe('Interacción – descartar notificaciones', () => {
    it('debe ocultar la notificación al hacer clic en dismiss', async () => {
      stateService.addNotification('info', 'Notificación a descartar');
      await element.updateComplete;

      const dismissBtn = element.shadowRoot.querySelector('.dismiss-btn');
      dismissBtn.click();
      await element.updateComplete;

      const notifications = element.shadowRoot.querySelectorAll('.notification');
      expect(notifications.length).toBe(0);
    });
  });
});
```

2. Ejecutar todas las pruebas:

```bash
npm test
```

**Resultado esperado:**

```
✓ src/tests/project-card.test.js (5 tests)
✓ src/tests/task-list.test.js (5 tests)
✓ src/tests/notification-panel.test.js (5 tests)

Test Files  3 passed (3)
Tests       15 passed (15)
```

---

### Fase 7: Integración Final y Refinamiento (40 min)

**Objetivo:** Limpiar el código, verificar la integración completa de todos los componentes y confirmar el funcionamiento end-to-end del dashboard.

#### Paso 7.1 – Limpieza y Verificación Final

**Instrucciones:**

1. Confirmar que `index.html` **no** contiene la sección `#native-test` de prueba de la Fase 2.

2. Verificar que `src/main.js` importa todos los componentes en el orden correcto:

```javascript
// src/main.js – Versión final
import './components/native/status-badge.js';
import './components/lit/theme-toggle.js';
import './components/lit/notification-panel.js';
import './components/lit/app-dashboard.js';

const container = document.getElementById('theme-toggle-container');
if (container) {
  const toggle = document.createElement('theme-toggle');
  container.appendChild(toggle);
}

console.log('TaskDash: aplicación lista ✓');
```

3. Ejecutar el servidor de desarrollo y verificar manualmente cada funcionalidad:

```bash
npm run dev
```

**Lista de verificación funcional:**

- [ ] El header muestra "TaskDash" con navegación y botón de tema.
- [ ] El botón de tema alterna entre modo claro y oscuro y persiste al recargar.
- [ ] La sección "Resumen Ejecutivo" muestra las 6 métricas correctas.
- [ ] Las 4 tarjetas de proyecto se muestran con colores de prioridad correctos.
- [ ] El buscador de proyectos filtra en tiempo real.
- [ ] Al hacer clic en una tarjeta, la lista de tareas se filtra por ese proyecto.
- [ ] Los filtros de la lista de tareas (Todas, pending, in-progress, etc.) funcionan.
- [ ] El botón "Completar" actualiza el estado de la tarea y genera una notificación.
- [ ] La notificación aparece en la esquina inferior derecha y se puede descartar.
- [ ] El footer es visible al fondo de la página.

#### Paso 7.2 – Ejecutar Suite Completa de Pruebas

```bash
npm test
```

**Resultado esperado:** 15 pruebas pasando en 3 archivos.

#### Paso 7.3 – Build de Producción

```bash
npm run build
```

**Resultado esperado:**

```
vite v5.x.x building for production...
✓ built in Xs
dist/index.html          x.xx kB
dist/assets/index-XXX.js  xxx kB
```

---

## 7. Resumen

### Lo que Lograste

- **Configuraste** un entorno de desarrollo frontend moderno con Node.js, Vite y Vitest, incluyendo soporte para módulos ES, hot-reload y un pipeline de pruebas automatizadas con `happy-dom`.
- **Creaste** un Web Component nativo (`status-badge`) implementando el ciclo de vida completo del navegador: `connectedCallback`, `disconnectedCallback`, `attributeChangedCallback` y `adoptedCallback`, con Shadow DOM y HTML Templates para encapsulación real.
- **Migraste** la lógica de componentes a Lit, comparando la verbosidad del API nativa con la declaratividad de `LitElement`, `` html`\`` `` y `` css`\`` ``, y aplicando directivas avanzadas (`repeat`, `when`, `classMap`) para renderizado eficiente.
- **Implementaste** propiedades reactivas con `static properties` y estado interno con `state: true`, comprobando cómo Lit actualiza el DOM de forma eficiente sin reemplazar el árbol completo.
- **Estableciste** comunicación bidireccional entre componentes: de padre a hijo mediante propiedades (`.project`, `.tasks`), y de hijo a padre mediante `CustomEvent` con `bubbles: true` y `composed: true` para atravesar el Shadow DOM.
- **Diseñaste** un servicio de estado centralizado basado en el patrón Observable usando `EventTarget`, permitiendo comunicación entre componentes sin relación jerárquica directa (hermanos).
- **Escribiste** 15 pruebas unitarias con Vitest y `happy-dom`, validando renderizado inicial, reactividad ante cambios de propiedades, filtrado de datos y emisión correcta de eventos.
- **Aplicaste** principios de HTML5 semántico y accesibilidad (ARIA roles, `aria-label`, `aria-live`, `role="progressbar"`) en toda la arquitectura de componentes.

### Conceptos Clave Aprendidos

- **Web Components y Shadow DOM:** Los Custom Elements son la base nativa de los componentes en el navegador. El Shadow DOM crea un árbol DOM aislado que encapsula estilos y estructura, evitando colisiones con el CSS global. Los atributos observados con `observedAttributes` y `attributeChangedCallback` permiten reaccionar a cambios externos.

- **LitElement como capa de abstracción:** Lit no es un framework, sino una librería ligera (~5 KB) sobre la API nativa. Elimina el boilerplate del ciclo de vida manual y añade reactividad declarativa: cuando una `@property` o `@state` cambia, Lit programa una actualización asíncrona del DOM usando `requestAnimationFrame`, evitando renders innecesarios.

- **Tagged Template Literals:** `html\`\`` y `css\`\`` son funciones JavaScript ordinarias que reciben arrays de strings y valores interpolados. Lit los usa para construir un árbol de partes actualizables, lo que le permite hacer diff eficiente en lugar de reemplazar todo el innerHTML.

- **Directivas avanzadas de Lit:** `repeat()` usa claves para reutilizar nodos DOM existentes (como React keys), `when()` es una expresión condicional tipada, y `classMap()` sincroniza objetos de condiciones booleanas con clases CSS sin manipulación manual del `classList`.

- **CustomEvent y comunicación entre componentes:** El patrón estándar para comunicación hijo→padre es emitir un `CustomEvent` con `bubbles: true` (sube por el árbol DOM) y `composed: true` (atraviesa los límites del Shadow DOM). El padre escucha con `@event-name` en el template de Lit.

- **Patrón Observable con EventTarget:** `EventTarget` es la clase base de todos los elementos del DOM. Extenderla para crear un servicio de estado permite implementar el patrón pub/sub sin dependencias externas. El método `subscribe()` retorna una función de limpieza para evitar memory leaks en `disconnectedCallback`.

- **Pruebas con happy-dom y Vitest:** `happy-dom` simula el entorno del navegador en Node.js con mejor rendimiento que jsdom. `element.updateComplete` es la Promise de Lit que resuelve cuando el DOM está sincronizado con el estado, y es fundamental esperarla en pruebas antes de hacer assertions sobre el Shadow DOM.

- **CSS Variables como Design Tokens:** Las variables CSS definidas en `:root` actúan como tokens de diseño globales. Los componentes con Shadow DOM pueden consumirlas porque las Custom Properties heredan a través de los límites del Shadow DOM, lo que permite theming consistente sin romper la encapsulación.

### Próximos Pasos

- En el **Capítulo 3** conectarás este dashboard con un backend Spring Boot: implementarás un `ApiService` que reemplace el `mock-data.js` con llamadas reales a endpoints REST, y aprenderás a manejar estados de carga y error en los componentes Lit.
- En el **Laboratorio 3** integrarás el frontend con el backend mediante fetch y CORS, y desplegarás ambos servicios en contenedores Docker tal como se introduce en el Capítulo 6.
- Practica los conceptos de este laboratorio extendiendo el dashboard: agrega un componente `project-form` con Lit para crear nuevos proyectos, utilizando `@state` para manejar el estado del formulario y emitiendo un `project-created` `CustomEvent` al guardar.
- Explora el sistema de reactividad avanzado de Lit estudiando `willUpdate()`, `updated()` y los `PropertyDeclaration` con `hasChanged` para optimizar renders en listas grandes.

### Recursos Adicionales

- **Documentación oficial de Lit:** Guía completa de LitElement, directivas, decoradores y testing – [https://lit.dev/docs/](https://lit.dev/docs/)
- **MDN – Web Components:** Referencia completa de Custom Elements, Shadow DOM y HTML Templates – [https://developer.mozilla.org/es/docs/Web/API/Web_components](https://developer.mozilla.org/es/docs/Web/API/Web_components)
- **Vitest – Documentación oficial:** Configuración avanzada, mocking, cobertura y UI mode – [https://vitest.dev/guide/](https://vitest.dev/guide/)
- **Vite – Guía del usuario:** Features, plugins, optimización de build y variables de entorno – [https://vitejs.dev/guide/](https://vitejs.dev/guide/)
- **CSS Custom Properties (MDN):** Uso de variables CSS para theming y design tokens – [https://developer.mozilla.org/es/docs/Web/CSS/Using_CSS_custom_properties](https://developer.mozilla.org/es/docs/Web/CSS/Using_CSS_custom_properties)
- **Web.dev – Accesibilidad con ARIA:** Guía práctica de roles y propiedades ARIA para componentes interactivos – [https://web.dev/learn/accessibility](https://web.dev/learn/accessibility)
- **Open Web Components:** Guías de mejores prácticas para testing y desarrollo de Web Components – [https://open-wc.org/guides/](https://open-wc.org/guides/)

