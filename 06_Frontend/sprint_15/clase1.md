# Uso de Redux en React

Redux es una librería para gestionar el **estado global** de una aplicación de forma centralizada y predecible. Junto con Redux Toolkit, permite manejar datos compartidos entre múltiples componentes sin necesidad de pasar props manualmente entre ellos.

## ¿Qué es Redux?

Redux centraliza el estado de la aplicación en un único lugar llamado **store**. Los componentes no modifican el estado directamente: en su lugar, **despachan acciones** (`dispatch`) que son procesadas por funciones llamadas **reducers**, las cuales determinan cómo debe cambiar el estado. Esto es especialmente útil cuando varios componentes necesitan acceder o modificar los mismos datos, o cuando se quiere evitar el llamado *prop drilling*.

## Instalación

Para empezar a usar Redux con Redux Toolkit, instalamos las siguientes dependencias:

```bash
npm install @reduxjs/toolkit react-redux
```

## Creación de un Slice

Un **slice** agrupa el estado inicial, los reducers y las acciones relacionadas con una parte concreta del estado (por ejemplo, un contador, un carrito de compra, un usuario, etc.).

```jsx
import { createSlice } from "@reduxjs/toolkit";

const initialState = {
  value: 0
};

const counterSlice = createSlice({
  name: "counter",
  initialState,
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    }
  }
});

export const { increment, decrement } = counterSlice.actions;
export default counterSlice.reducer;
```

En este ejemplo, `counterSlice` define un estado inicial con un valor numérico y dos reducers (`increment` y `decrement`) que modifican ese valor. Redux Toolkit permite escribir estas mutaciones de forma directa gracias a Immer, aunque internamente el estado sigue siendo inmutable.

## Creación del Store

El **store** es el almacén central donde se combinan todos los reducers de la aplicación.

```jsx
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "./features/counter/counterSlice";

export const store = configureStore({
  reducer: {
    counter: counterReducer
  }
});
```

## Conexión con React

Para que los componentes puedan acceder al store, envolvemos la aplicación con el componente `<Provider>`.

```jsx
import { Provider } from "react-redux";
import { store } from "./store/store";
import App from "./App";

function Root() {
  return (
    <Provider store={store}>
      <App />
    </Provider>
  );
}

export default Root;
```

## Lectura y modificación del estado

Dentro de los componentes, utilizamos `useSelector` para **leer** el estado global y `useDispatch` para **modificarlo** mediante acciones.

```jsx
import { useSelector, useDispatch } from "react-redux";
import { increment, decrement } from "./features/counter/counterSlice";

function Counter() {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return (
    <div>
      <p>Valor actual: {count}</p>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
    </div>
  );
}

export default Counter;
```

En este ejemplo, `useSelector` obtiene el valor actual del contador desde el store, mientras que `useDispatch` permite despachar las acciones `increment` y `decrement` para modificar ese estado, provocando el re-render automático del componente.

## Casos de Uso Comunes

- Compartir información (usuario autenticado, carrito de compra, tema visual) entre múltiples componentes sin pasar props manualmente.
- Centralizar lógica de negocio compleja fuera de los componentes de UI.
- Evitar el *prop drilling* en árboles de componentes profundos.
- Mantener un flujo de datos predecible y fácil de depurar en aplicaciones de gran tamaño.