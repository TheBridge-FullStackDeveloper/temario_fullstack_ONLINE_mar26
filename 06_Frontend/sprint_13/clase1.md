# React

React es una biblioteca de JavaScript utilizada para construir interfaces de usuario interactivas y eficientes para aplicaciones web. Una de las características fundamentales de React es el uso de componentes reutilizables que encapsulan la lógica y la interfaz de usuario de una parte específica de la aplicación.

## ¿Qué es React?

React es una biblioteca de JavaScript de código abierto desarrollada por Facebook. Se centra en la creación de interfaces de usuario (UI) declarativas y basadas en componentes. En lugar de trabajar con el DOM directamente, React introduce un modelo de programación basado en componentes, donde cada componente es una pieza aislada y reutilizable de la interfaz de usuario.

## JSX en React

Una de las características más distintivas de React es su uso de JSX (JavaScript XML), una extensión de JavaScript que permite escribir HTML dentro de archivos de JavaScript. JSX facilita la creación de componentes React al combinar la lógica JavaScript con la estructura del marcado HTML.

Por ejemplo, un componente simple en React utilizando JSX se vería así:

```jsx
import React from "react";

function Saludo(props) {
  return <h1>Hola, {props.nombre}!</h1>;
}

export default Saludo;
```

En este ejemplo, `Saludo` es un componente de función que toma un prop `nombre` y muestra un saludo personalizado utilizando JSX.

## Componentes en React

En React, los componentes son bloques de construcción fundamentales que representan partes independientes y reutilizables de la interfaz de usuario. Los componentes pueden ser simples o complejos, y se componen entre sí para formar la estructura de la aplicación. Esta arquitectura basada en componentes facilita la creación, mantenimiento y prueba de aplicaciones web escalables.

## El Hook useState

`useState` es un hook de React que permite a los componentes de función mantener y actualizar su propio estado interno. Antes de la introducción de los hooks en React 16.8, el estado solo podía ser utilizado en componentes de clase. Con el hook `useState`, los componentes de función pueden tener estado, lo que simplifica la gestión del estado en aplicaciones React y fomenta el uso de componentes funcionales sobre los componentes de clase.

### Uso Básico de useState

El hook `useState` se utiliza comúnmente dentro de un componente de función de la siguiente manera:

```jsx
import { useState } from "react";

function Contador() {
  const [contador, setContador] = useState(0);

  return (
    <div>
      <p>El contador es: {contador}</p>
      <button onClick={() => setContador(contador + 1)}>Incrementar</button>
    </div>
  );
}
```

En este ejemplo, `useState` inicializa una variable de estado llamada `contador` con un valor inicial de 0. También devuelve una función `setContador` que nos permite actualizar el valor de `contador`.

El hook `useState` es una herramienta poderosa que simplifica la gestión del estado en aplicaciones React. Al utilizar `useState`, los componentes de función pueden mantener su propio estado interno de manera eficiente y elegante.

# Props en React

En React, las props (abreviatura de "propiedades") son utilizadas para pasar datos y configuraciones desde un componente padre a un componente hijo. Esto permite la reutilización de componentes y la creación de componentes dinámicos y flexibles.

## Pasar Props

Para pasar props de un componente padre a un componente hijo, simplemente se incluyen como atributos al renderizar el componente hijo. Las props son accesibles dentro del componente hijo a través del objeto `props`.

```jsx
// Componente Padre
import ComponenteHijo from "./ComponenteHijo";

function App() {
  return (
    <div>
      <ComponenteHijo nombre="Juan" edad={30} />
    </div>
  );
}

export default App;
```

En el ejemplo anterior, estamos pasando dos props (`nombre` y `edad`) al componente `ComponenteHijo`.

## Acceder a Props en Componentes Hijos

Dentro del componente hijo, podemos acceder a las props utilizando el objeto `props`. Las props son de solo lectura y no deben ser modificadas directamente dentro del componente hijo (a menos que se trate de componentes controlados).

```jsx
// Componente Hijo

function ComponenteHijo(props) {
  return (
    <div>
      <p>Nombre: {props.nombre}</p>
      <p>Edad: {props.edad}</p>
    </div>
  );
}

export default ComponenteHijo;
```

En este ejemplo, el componente `ComponenteHijo` recibe las props `nombre` y `edad` del componente padre y las muestra en su renderizado.

## Props por Defecto

Es posible definir valores por defecto para las props en caso de que no se pasen desde el componente padre. Esto se logra utilizando la propiedad `defaultProps` en el componente hijo.

```jsx
// Componente Hijo con Props por Defecto

function ComponenteHijo(props) {
  return (
    <div>
      <p>Nombre: {props.nombre}</p>
      <p>Edad: {props.edad}</p>
    </div>
  );
}

ComponenteHijo.defaultProps = {
  nombre: "Usuario",
  edad: 18,
};

export default ComponenteHijo;
```

En este ejemplo, si el componente padre no pasa las props `nombre` y `edad`, el componente `ComponenteHijo` utilizará los valores por defecto especificados en `defaultProps`.

## Prop Types

Para garantizar que las props recibidas sean del tipo esperado, es recomendable utilizar PropTypes. Esto ayuda a detectar errores durante el desarrollo y proporciona una documentación clara sobre las props aceptadas por un componente.

```jsx
// Componente Hijo con PropTypes
import PropTypes from "prop-types";

function ComponenteHijo(props) {
  return (
    <div>
      <p>Nombre: {props.nombre}</p>
      <p>Edad: {props.edad}</p>
    </div>
  );
}

ComponenteHijo.propTypes = {
  nombre: PropTypes.string.isRequired,
  edad: PropTypes.number.isRequired,
};

export default ComponenteHijo;
```

En este ejemplo, estamos utilizando PropTypes para especificar que `nombre` debe ser una cadena (string) y `edad` debe ser un número (number). La propiedad `isRequired` indica que es obligatorio pasar estas props al componente hijo.