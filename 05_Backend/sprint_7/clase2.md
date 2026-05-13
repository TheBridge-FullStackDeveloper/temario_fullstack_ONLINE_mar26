# Express.js

## ¿Qué es Express.js?

Express.js es un marco de aplicación web para Node.js, diseñado para simplificar el desarrollo de aplicaciones web y APIs. Es minimalista y flexible, proporcionando una serie de funciones y herramientas para construir aplicaciones web de manera rápida y sencilla.

## Características Principales

### 1. **Manejo de Rutas:**

Express facilita la definición de rutas para manejar diversas solicitudes HTTP. Puedes responder a solicitudes GET, POST, PUT, DELETE y más.

```javascript
const express = require("express");
const app = express();

// Ruta básica
app.get("/", (req, res) => {
  res.send("Hola, mundo!");
});

// Ruta con parámetros
app.get("/usuario/:id", (req, res) => {
  const userId = req.params.id;
  res.send(`Detalles del usuario ${userId}`);
});
```

### 2. Middlewares:

Express utiliza middlewares para procesar solicitudes antes de llegar a las rutas definidas. Pueden realizar tareas como el análisis de cuerpos de solicitud, autenticación, manejo de errores, etc.

```javascript
const express = require("express");
const app = express();

// Middleware de registro de tiempo
app.use((req, res, next) => {
  console.log(`Time: ${Date.now()}`);
  next();
});

// Middleware de análisis de cuerpo
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Middleware de autenticación
app.use((req, res, next) => {
  // Lógica de autenticación
  next();
});
```

### 3. Plantillas de Vistas:

Express es compatible con varios motores de plantillas (pug, ejs, handlebars, etc.), permitiendo la generación dinámica de HTML.

```javascript
app.set("view engine", "pug");

app.get("/vista", (req, res) => {
  res.render("mi_vista", { titulo: "Express.js" });
});
```

### 4. Enrutamiento Modular:

Express permite organizar el código de las rutas en módulos independientes, facilitando la gestión de rutas en aplicaciones grandes.

```javascript
// routes/usuarios.js
const express = require("express");
const router = express.Router();

router.get("/", (req, res) => {
  res.send("Lista de usuarios");
});

module.exports = router;

// index.js
const express = require("express");
const usuariosRouter = require("./routes/usuarios");

const app = express();
app.use("/usuarios", usuariosRouter);
```

### 5. Manejo de Archivos Estáticos:

Express puede servir archivos estáticos, como CSS, imágenes y JavaScript, de manera eficiente.

```javascript
app.use(express.static("public"));
```

## Instalación y Configuración

Para empezar con Express.js, primero debes instalarlo utilizando npm:

```bash
npm install express
```

Luego, puedes crear una aplicación básica de Express:

```javascript
const express = require("express");
const app = express();

const puerto = 3000;
app.listen(puerto, () => {
  console.log(`Servidor escuchando en el puerto ${puerto}`);
});
```

# Middlewares en Express.js

En Express.js, los middlewares son funciones que tienen acceso al objeto de solicitud (`req`), al objeto de respuesta (`res`), y a la siguiente función de middleware en el ciclo de solicitud-respuesta de la aplicación. Estos middlewares pueden realizar diversas tareas, como modificar la solicitud o respuesta, realizar validaciones, autenticar usuarios, y más.

## Uso Básico de Middlewares

En Express, puedes usar middlewares con la función `use`. Aquí hay un ejemplo básico:

```javascript
const express = require("express");
const app = express();

// Middleware de registro de tiempo
app.use((req, res, next) => {
  console.log(`Time: ${Date.now()}`);
  next(); // Llama a la siguiente función de middleware
});

// Ruta principal
app.get("/", (req, res) => {
  res.send("Hola, mundo!");
});

const puerto = 3000;
app.listen(puerto, () => {
  console.log(`Servidor escuchando en el puerto ${puerto}`);
});
```

En este ejemplo, el middleware de registro de tiempo se ejecutará antes de cada solicitud, mostrando la hora actual en la consola.

## Tipos Comunes de Middlewares

### Middleware de Análisis de Cuerpo (Body Parser)

El middleware de análisis de cuerpo permite procesar el cuerpo de la solicitud, especialmente útil para formularios y solicitudes POST. Puedes instalar el middleware `body-parser` para esto.

```javascript
const express = require("express");
const bodyParser = require("body-parser");
const app = express();

// Middleware de análisis de cuerpo
app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());

// Ruta para manejar datos POST
app.post("/ejemplo", (req, res) => {
  const datos = req.body;
  res.json({ mensaje: "Datos recibidos correctamente", datos });
});

const puerto = 3000;
app.listen(puerto, () => {
  console.log(`Servidor escuchando en el puerto ${puerto}`);
});
```

### Middleware de Autenticación

Los middlewares de autenticación verifican si un usuario tiene permisos para acceder a ciertas rutas. Aquí hay un ejemplo simple:

```javascript
const express = require("express");
const app = express();

// Middleware de autenticación
const middlewareAutenticacion = (req, res, next) => {
  const usuarioAutenticado = false; // Lógica de autenticación

  if (usuarioAutenticado) {
    next(); // Usuario autenticado, permite el acceso
  } else {
    res.status(401).send("Acceso no autorizado");
  }
};

// Ruta protegida con el middleware de autenticación
app.get("/ruta-protegida", middlewareAutenticacion, (req, res) => {
  res.send("Bienvenido a la ruta protegida");
});

const puerto = 3000;
app.listen(puerto, () => {
  console.log(`Servidor escuchando en el puerto ${puerto}`);
});
```