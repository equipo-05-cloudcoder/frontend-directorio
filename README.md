# Sesión 3: Frontend: HTML/JS y React consumiendo servicios web

## Objetivos de la sesión

Al finalizar, el participante podrá:

1. Configurar un `devcontainer.json` con Node.js para desarrollo frontend.
2. Consumir una API REST desde HTML/JavaScript puro usando `fetch`.
3. Crear un proyecto React con Vite dentro de un Codespace.
4. Organizar un frontend React con componentes, estado y servicios separados.
5. Conectar el frontend React al backend Flask del repositorio de la sesión 2.

---

## `devcontainer.json` para frontend (React + Vite)

Guarda este archivo en `.devcontainer/devcontainer.json`:

```json
{
  "name": "Frontend React + Vite",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:20",
  "features": {
    "ghcr.io/devcontainers/features/node:1": {
      "version": "20"
    },
    "ghcr.io/devcontainers/features/python:1": {
      "version": "3.12"
    }
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "bradlc.vscode-tailwindcss",
        "formulahendry.auto-rename-tag"
      ]
    }
  }
}
```

**Extensiones instaladas:**

| Extensión | Propósito |
|---|---|
| `dbaeumer.vscode-eslint` | Verificación de errores y estilo en JavaScript/JSX |
| `esbenp.prettier-vscode` | Formateo automático de código |
| `bradlc.vscode-tailwindcss` | Soporte de Tailwind CSS (opcional) |
| `formulahendry.auto-rename-tag` | Renombra las etiquetas HTML/JSX en pares automáticamente |

### Inicia el codespace

Inicia el codespace para que tome la configuración del archivo que acabas de crear

---

## Parte 1: HTML y JavaScript puro con `fetch`

Antes de usar un framework, es importante entender cómo el navegador hace peticiones HTTP.

### Crea `index.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Estudiantes</title>
  <style>
    body { font-family: Arial, sans-serif; max-width: 800px; margin: 40px auto; padding: 0 20px; }
    table { width: 100%; border-collapse: collapse; }
    th, td { border: 1px solid #ccc; padding: 8px; text-align: left; }
    th { background: #f4f4f4; }
    .form-group { margin: 10px 0; }
    input { padding: 6px; width: 300px; }
    button { padding: 8px 16px; cursor: pointer; margin: 4px; }
  </style>
</head>
<body>

  <h1>Lista de Estudiantes</h1>
  <button onclick="cargarEstudiantes()">Recargar</button>
  <table id="tabla">
    <thead><tr><th>ID</th><th>Nombre</th><th>Email</th><th>Carrera</th></tr></thead>
    <tbody id="cuerpo"></tbody>
  </table>

  <h2>Agregar estudiante</h2>
  <div class="form-group"><input id="nombre" placeholder="Nombre"></div>
  <div class="form-group"><input id="email" placeholder="Email"></div>
  <div class="form-group"><input id="carrera" placeholder="Carrera"></div>
  <button onclick="agregarEstudiante()">Guardar</button>

  <p id="mensaje"></p>

  <script>
    // IMPORTANTE: cambia esta URL si el backend está en un puerto público de Codespaces
    const URL_BASE = "http://localhost:5000";

    async function cargarEstudiantes() {
      const resp = await fetch(`${URL_BASE}/estudiantes`);
      const datos = await resp.json();
      const cuerpo = document.getElementById("cuerpo");
      cuerpo.innerHTML = "";
      datos.forEach(e => {
        cuerpo.innerHTML += `
          <tr>
            <td>${e.id}</td>
            <td>${e.nombre}</td>
            <td>${e.email}</td>
            <td>${e.carrera}</td>
          </tr>`;
      });
    }

    async function agregarEstudiante() {
      const body = {
        nombre:  document.getElementById("nombre").value,
        email:   document.getElementById("email").value,
        carrera: document.getElementById("carrera").value
      };
      const resp = await fetch(`${URL_BASE}/estudiantes`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body)
      });
      const resultado = await resp.json();
      document.getElementById("mensaje").textContent = resultado.mensaje || resultado.error;
      cargarEstudiantes();
    }

    // Cargar al abrir la página
    cargarEstudiantes();
  </script>

</body>
</html>
```

Para abrirlo en el Codespace, instala la extensión **Live Server** o usa Python:

```bash
python -m http.server 8080
```

Luego abre el puerto 8080 desde la pestaña **Ports** de VS Code.

---

## Parte 2: Introducción a React

### ¿Qué es React?

React es una librería JavaScript para construir interfaces de usuario mediante **componentes** reutilizables. Un componente es una función que recibe datos (**props**) y retorna HTML (JSX).

### Conceptos mínimos

| Concepto | Descripción |
|---|---|
| **Componente** | Función que retorna JSX (HTML + JS combinados) |
| **Props** | Datos que el padre le pasa al hijo (solo lectura) |
| **Estado (`useState`)** | Datos internos del componente; al cambiar, React re-renderiza la UI |
| **`useEffect`** | Ejecuta código cuando el componente se monta o cuando cambia una dependencia |

### Ejemplo mínimo

```jsx
import { useState } from "react";

function Contador() {
  const [cuenta, setCuenta] = useState(0);   // [valor, función para cambiar]

  return (
    <div>
      <p>Clicks: {cuenta}</p>
      <button onClick={() => setCuenta(cuenta + 1)}>+1</button>
    </div>
  );
}
```

---

## Parte 3: Crear proyecto React con Vite

### `.gitignore` para React

Crea el archivo en la raíz del proyecto:

```bash
touch .gitignore
```

Luego pega este contenido:

```gitignore
# Dependencies
node_modules/
/.pnp
.pnp.js
.pnp.cjs

# Build output
/dist
/dist-ssr
/build
/.next/
/out/

# Environment variables
.env
.env.local
.env.development.local
.env.test.local
.env.production.local
.env.*.local

# Logs
logs/
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*

# Testing & coverage
/coverage
/.nyc_output
/cypress/videos/
/cypress/screenshots/

# Cache
.cache/
.eslintcache
.vite/
*.tsbuildinfo

# Editor
.vscode/*
!.vscode/extensions.json
!.vscode/settings.json
.idea/
.DS_Store
*.sw?
```

### Crear el proyecto

```bash
# Crear el proyecto
npm create vite@latest mi-app -- --template react

# Entrar a la carpeta
cd mi-app

# Instalar dependencias
npm install

# Correr en modo desarrollo
npm run dev
```

Vite levantará el servidor en el puerto **5173**. Abre ese puerto en la pestaña **Ports** de VS Code.

---

## Estructura del proyecto recomendada

```
mi-app/
├── public/
├── src/
│   ├── services/
│   │   └── estudianteService.js   ← Todas las llamadas HTTP al backend
│   ├── components/
│   │   ├── EstudianteForm.jsx      ← Formulario crear / editar
│   │   └── EstudianteList.jsx      ← Tabla con la lista
│   └── App.jsx                    ← Componente principal (orquestador)
├── index.html
├── vite.config.js
└── package.json
```

---

## `estudianteService.js`

```javascript
const URL_BASE = "http://localhost:5000";

export async function getAll() {
  const r = await fetch(`${URL_BASE}/estudiantes`);
  return r.json();
}

export async function getOne(id) {
  const r = await fetch(`${URL_BASE}/estudiantes/${id}`);
  return r.json();
}

export async function create(data) {
  const r = await fetch(`${URL_BASE}/estudiantes`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(data)
  });
  return r.json();
}

export async function update(id, data) {
  const r = await fetch(`${URL_BASE}/estudiantes/${id}`, {
    method: "PUT",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(data)
  });
  return r.json();
}

export async function remove(id) {
  const r = await fetch(`${URL_BASE}/estudiantes/${id}`, { method: "DELETE" });
  return r.json();
}
```

---

## `EstudianteList.jsx`

```jsx
export default function EstudianteList({ estudiantes, onEditar, onEliminar }) {
  return (
    <table border="1" style={{ width: "100%", borderCollapse: "collapse" }}>
      <thead>
        <tr>
          <th>ID</th><th>Nombre</th><th>Email</th><th>Carrera</th><th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        {estudiantes.map(e => (
          <tr key={e.id}>
            <td>{e.id}</td>
            <td>{e.nombre}</td>
            <td>{e.email}</td>
            <td>{e.carrera}</td>
            <td>
              <button onClick={() => onEditar(e)}>Editar</button>
              <button onClick={() => onEliminar(e.id)}>Eliminar</button>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

---

## `EstudianteForm.jsx`

```jsx
import { useState, useEffect } from "react";

export default function EstudianteForm({ estudianteEditar, onGuardado, onCancelar }) {
  const [nombre,  setNombre]  = useState("");
  const [email,   setEmail]   = useState("");
  const [carrera, setCarrera] = useState("");

  // Cargar datos cuando se va a editar
  useEffect(() => {
    if (estudianteEditar) {
      setNombre(estudianteEditar.nombre);
      setEmail(estudianteEditar.email);
      setCarrera(estudianteEditar.carrera || "");
    } else {
      setNombre(""); setEmail(""); setCarrera("");
    }
  }, [estudianteEditar]);

  const handleSubmit = (e) => {
    e.preventDefault();
    onGuardado({ nombre, email, carrera });
  };

  return (
    <form onSubmit={handleSubmit}>
      <h3>{estudianteEditar ? "Editar estudiante" : "Nuevo estudiante"}</h3>
      <input value={nombre}  onChange={e => setNombre(e.target.value)}  placeholder="Nombre"  required />
      <input value={email}   onChange={e => setEmail(e.target.value)}   placeholder="Email"   required />
      <input value={carrera} onChange={e => setCarrera(e.target.value)} placeholder="Carrera" />
      <button type="submit">Guardar</button>
      {estudianteEditar && <button type="button" onClick={onCancelar}>Cancelar</button>}
    </form>
  );
}
```

---

## `App.jsx`

```jsx
import { useState, useEffect } from "react";
import EstudianteList from "./components/EstudianteList";
import EstudianteForm from "./components/EstudianteForm";
import * as svc from "./services/estudianteService";

export default function App() {
  const [estudiantes,      setEstudiantes]      = useState([]);
  const [estudianteEditar, setEstudianteEditar] = useState(null);
  const [mensaje,          setMensaje]          = useState("");

  const cargar = async () => {
    const datos = await svc.getAll();
    setEstudiantes(datos);
  };

  useEffect(() => { cargar(); }, []);

  const handleGuardado = async (datos) => {
    if (estudianteEditar) {
      await svc.update(estudianteEditar.id, datos);
      setMensaje("Estudiante actualizado.");
    } else {
      await svc.create(datos);
      setMensaje("Estudiante creado.");
    }
    setEstudianteEditar(null);
    cargar();
  };

  const handleEliminar = async (id) => {
    await svc.remove(id);
    setMensaje(`Estudiante ${id} eliminado.`);
    cargar();
  };

  return (
    <div style={{ maxWidth: 900, margin: "40px auto", padding: "0 20px" }}>
      <h1>Gestión de Estudiantes</h1>
      {mensaje && <p style={{ color: "green" }}>{mensaje}</p>}
      <EstudianteForm
        estudianteEditar={estudianteEditar}
        onGuardado={handleGuardado}
        onCancelar={() => setEstudianteEditar(null)}
      />
      <hr />
      <EstudianteList
        estudiantes={estudiantes}
        onEditar={setEstudianteEditar}
        onEliminar={handleEliminar}
      />
    </div>
  );
}
```

---

## Flujo de datos en React (resumen)

```
App.jsx  (estado: estudiantes, estudianteEditar)
   │                              │
   │   props + callbacks          │   props + callbacks
   ▼                              ▼
EstudianteForm               EstudianteList
(formulario)                 (tabla)
```

- **Los datos bajan** del padre al hijo vía `props`.
- **Los eventos suben** del hijo al padre vía funciones (`onGuardado`, `onEditar`, `onEliminar`).
- El estado **siempre vive en el componente más alto** que lo necesite.

---

## Conectar con backend en otro Codespace

Cuando el backend y el frontend están en **repositorios separados** (escenario real de producción), necesitas:

1. En el Codespace del backend, hacer el puerto 5000 **público**.
2. Copiar la URL pública (p.ej. `https://abc123-5000.app.github.dev`).
3. En `estudianteService.js`, cambiar:

```javascript
const URL_BASE = "https://abc123-5000.app.github.dev";
```

---

## Actividad: agregar tu entidad al frontend

Usando el backend que creaste en la Sesión 2 (con tu entidad personalizada):

1. Crea un nuevo proyecto React con Vite.
2. Escribe el archivo `<entidad>Service.js` con los 5 métodos CRUD.
3. Crea los componentes `<Entidad>List.jsx` y `<Entidad>Form.jsx`.
4. Ensambla todo en `App.jsx`.
5. Prueba que el flujo completo funcione: crear, leer, editar y eliminar.

---

## Actividad de cierre: crea tu plantilla de repositorio frontend

La plantilla de frontend es diferente a las anteriores: el proyecto React se crea con `npm create vite`, por lo que la plantilla puede incluir ese código ya generado o solo el `devcontainer.json` y un README que indica al alumno cómo crearlo.

Existen dos estrategias; elige la que mejor se adapte a tu curso:

### Estrategia A: plantilla vacía (el alumno crea el proyecto React)

El alumno aprende el proceso completo de inicialización. La plantilla solo aporta el entorno.

```
template-frontend/
├── .devcontainer/
│   └── devcontainer.json     ← Node.js 20 listo para usar
└── README.md                 ← Instrucciones paso a paso
```

### Estrategia B: plantilla con proyecto React preconfigurado

El alumno recibe la estructura lista y solo completa los componentes o el servicio. Útil cuando el tiempo de la actividad es corto.

```
template-frontend/
├── .devcontainer/
│   └── devcontainer.json
├── src/
│   ├── services/
│   │   └── apiService.js     ← Solo falta llenar URL_BASE
│   ├── components/           ← Carpeta vacía; el alumno crea sus componentes
│   └── App.jsx               ← Esqueleto mínimo
├── index.html
├── package.json
├── vite.config.js
└── README.md
```

### `devcontainer.json` de la plantilla

```json
{
  "name": "Frontend React + Vite",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:20",
  "features": {
    "ghcr.io/devcontainers/features/node:1": {
      "version": "20"
    }
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "formulahendry.auto-rename-tag"
      ]
    }
  }
}
```

> Esta imagen ya incluye Node.js 20 y npm. El alumno puede ejecutar `npm create vite` o `npm install` + `npm run dev` sin instalar nada.

### `README.md` para el alumno (Estrategia A)

```markdown
# [Nombre de la actividad]

## Descripción
[Qué debe construir el alumno]

## Paso 1: abrir el Codespace
1. Haz clic en **Code** → **Codespaces** → **Create codespace on main**.
2. Espera ~30 segundos. Node.js 20 ya estará disponible.
3. Verifica con: `node --version` y `npm --version`

## Paso 2: crear el proyecto React
\```bash
npm create vite@latest mi-app -- --template react
cd mi-app
npm install
\```

## Paso 3: configurar la URL del backend
Edita `src/services/apiService.js` y reemplaza `URL_BASE` con la URL pública
del backend (la encontrarás en la pestaña **Ports** del Codespace del backend).

## Paso 4: iniciar la aplicación
\```bash
npm run dev
\```
Abre el puerto 5173 desde la pestaña **Ports** de VS Code.

## Lo que debes implementar
[Lista de componentes o funcionalidades que debe crear el alumno]

## Entregable
[Qué debe entregar y cómo]
```

Usa este prompt con IA para generar el README:

```
Genera un README.md en español para una actividad de frontend llamada "[NOMBRE]".
El alumno usa GitHub Codespaces con Node.js 20 disponible.
El proyecto usa React + Vite. El backend ya existe en otro Codespace y el alumno
debe configurar su URL en un archivo apiService.js.
Incluye: Descripción, pasos para abrir el Codespace, crear el proyecto con Vite,
configurar la URL del backend, correr la aplicación, y descripción del entregable.
```

### Activa Template repository

1. Ve a **Settings → General** de tu repositorio.
2. Activa **Template repository** y guarda.

### Prueba la plantilla

1. Usa **Use this template → Create a new repository**.
2. Lanza un Codespace.
3. Ejecuta `node --version` — debe mostrar v20.x.x.
4. Ejecuta `npm create vite@latest prueba -- --template react && cd prueba && npm install && npm run dev` y verifica que Vite levante en el puerto 5173.

---

## Actividad extra (si queda tiempo)

### Agregar navegación entre vistas con estado

Sin instalar React Router, puedes simular múltiples "páginas" usando un estado en `App.jsx` que controla qué vista se muestra:

```jsx
import { useState } from "react";
import VistaEstudiantes from "./components/VistaEstudiantes";
import VistaCarreras from "./components/VistaCarreras";

export default function App() {
  const [vista, setVista] = useState("estudiantes");

  return (
    <div style={{ maxWidth: 900, margin: "40px auto", padding: "0 20px" }}>
      <nav style={{ marginBottom: 20 }}>
        <button onClick={() => setVista("estudiantes")}
          style={{ fontWeight: vista === "estudiantes" ? "bold" : "normal", marginRight: 12 }}>
          Estudiantes
        </button>
        <button onClick={() => setVista("carreras")}
          style={{ fontWeight: vista === "carreras" ? "bold" : "normal" }}>
          Carreras
        </button>
      </nav>

      {vista === "estudiantes" && <VistaEstudiantes />}
      {vista === "carreras"   && <VistaCarreras />}
    </div>
  );
}
```

Cada componente de vista (`VistaEstudiantes`, `VistaCarreras`) encapsula su propio estado y llamadas al servicio, exactamente igual al `App.jsx` de la actividad principal.

Este patrón es una introducción natural a React Router: el concepto de "qué componente se muestra según una condición" es el mismo que usa el router, solo que aquí la condición es un estado en lugar de la URL.

### Agregar indicador de carga

Agrega un estado `cargando` para mostrar retroalimentación visual mientras se espera la respuesta del backend:

```jsx
const [cargando, setCargando] = useState(false);

const cargar = async () => {
  setCargando(true);
  const datos = await svc.getAll();
  setEstudiantes(datos);
  setCargando(false);
};

// En el JSX:
{cargando ? <p>Cargando...</p> : <EstudianteList estudiantes={estudiantes} ... />}
```

Prueba la diferencia de experiencia con y sin el indicador, especialmente cuando el backend tarda en responder (puedes agregar `time.sleep(1)` temporalmente en el endpoint Flask para simularlo).

---

## Recursos adicionales

- [Repositorio de referencia: ejemplo_frontend](https://github.com/TMP-DESARROLLO-WEB/ejemplo_frontend)
- [Vite: Guía de inicio rápido](https://vitejs.dev/guide/)
- [React: Documentación oficial](https://react.dev)
- [MDN: Usando Fetch](https://developer.mozilla.org/es/docs/Web/API/Fetch_API/Using_Fetch)
