# Gestión de Carrito y Wishlist en un E-commerce

En esta sesión aprenderás a gestionar el flujo completo de un e-commerce integrando el **carrito** y la **wishlist** con el backend, calculando totales dinámicos y diferenciando entre **estado global** y **estado local** para lograr una arquitectura escalable.

## ¿Qué es el carrito y la wishlist en una arquitectura de estado?

En un e-commerce, tanto el carrito como la wishlist son datos que varios componentes de la aplicación necesitan consultar y modificar (header, listado de productos, página de checkout, etc.). Por eso se gestionan como **estado global**, normalmente con Redux Toolkit, mientras que aspectos puntuales de la UI (por ejemplo, si un modal está abierto) se mantienen como **estado local** con `useState`.

| Tipo de estado | Ejemplo | Herramienta |
|---|---|---|
| Global | carrito, wishlist, usuario | Redux Toolkit |
| Local | modal abierto, input de búsqueda | useState |

## Slice del carrito

El slice del carrito almacena los productos añadidos, junto con las acciones necesarias para modificarlos.

```javascript
// src/features/cart/cartSlice.js
import { createSlice } from "@reduxjs/toolkit";

const initialState = {
  items: [],
  loading: false,
  error: null
};

const cartSlice = createSlice({
  name: "cart",
  initialState,
  reducers: {
    addToCart: (state, action) => {
      const existing = state.items.find(item => item.id === action.payload.id);
      if (existing) {
        existing.quantity += 1;
      } else {
        state.items.push({ ...action.payload, quantity: 1 });
      }
    },
    removeFromCart: (state, action) => {
      state.items = state.items.filter(item => item.id !== action.payload);
    },
    updateQuantity: (state, action) => {
      const item = state.items.find(item => item.id === action.payload.id);
      if (item) item.quantity = action.payload.quantity;
    },
    clearCart: (state) => {
      state.items = [];
    }
  }
});

export const { addToCart, removeFromCart, updateQuantity, clearCart } = cartSlice.actions;
export default cartSlice.reducer;
```

## Slice de la wishlist

La wishlist sigue una estructura similar, pero sin cantidades, ya que solo guarda referencias a productos favoritos.

```javascript
// src/features/wishlist/wishlistSlice.js
import { createSlice } from "@reduxjs/toolkit";

const initialState = {
  items: []
};

const wishlistSlice = createSlice({
  name: "wishlist",
  initialState,
  reducers: {
    toggleWishlist: (state, action) => {
      const exists = state.items.some(item => item.id === action.payload.id);
      if (exists) {
        state.items = state.items.filter(item => item.id !== action.payload.id);
      } else {
        state.items.push(action.payload);
      }
    }
  }
});

export const { toggleWishlist } = wishlistSlice.actions;
export default wishlistSlice.reducer;
```

## Conexión de ambos slices al store

```javascript
// src/store/store.js
import { configureStore } from "@reduxjs/toolkit";
import cartReducer from "../features/cart/cartSlice";
import wishlistReducer from "../features/wishlist/wishlistSlice";

export const store = configureStore({
  reducer: {
    cart: cartReducer,
    wishlist: wishlistReducer
  }
});
```

## Sincronización con el backend

Aunque el carrito y la wishlist viven en el estado global, es habitual persistirlos en el backend (por ejemplo, con Supabase) para que el usuario no los pierda al recargar la página o cambiar de dispositivo.

```jsx
import { useEffect } from "react";
import { useDispatch } from "react-redux";
import axios from "axios";
import { addToCart } from "../features/cart/cartSlice";

function CartSync() {
  const dispatch = useDispatch();

  useEffect(() => {
    const fetchCart = async () => {
      try {
        const res = await axios.get("/api/cart");
        res.data.forEach(item => dispatch(addToCart(item)));
      } catch (err) {
        console.error("Error al sincronizar el carrito", err);
      }
    };

    fetchCart();
  }, [dispatch]);

  return null;
}

export default CartSync;
```

✔ El patrón `useEffect + axios + dispatch` permite hidratar el estado global con los datos guardados en el backend al montar la aplicación.

## Cálculo de totales dinámicos

El cálculo del total del carrito es un dato que **depende** de los ítems, por lo que conviene memorizarlo con `useMemo` para evitar recalcularlo en cada render si los ítems no han cambiado.

```jsx
import { useSelector } from "react-redux";
import { useMemo } from "react";

function CartSummary() {
  const items = useSelector((state) => state.cart.items);

  const total = useMemo(() => {
    return items.reduce((acc, item) => acc + item.price * item.quantity, 0);
  }, [items]);

  return (
    <div>
      <p>Total: {total.toFixed(2)} €</p>
    </div>
  );
}

export default CartSummary;
```

## Uso combinado: carrito y wishlist en un componente de producto

```jsx
import { useSelector, useDispatch } from "react-redux";
import { addToCart } from "../features/cart/cartSlice";
import { toggleWishlist } from "../features/wishlist/wishlistSlice";

function ProductCard({ product }) {
  const dispatch = useDispatch();
  const wishlist = useSelector((state) => state.wishlist.items);
  const isFavorite = wishlist.some((item) => item.id === product.id);

  return (
    <div>
      <h3>{product.name}</h3>
      <p>{product.price} €</p>
      <button onClick={() => dispatch(addToCart(product))}>Añadir al carrito</button>
      <button onClick={() => dispatch(toggleWishlist(product))}>
        {isFavorite ? "★ Quitar de favoritos" : "☆ Añadir a favoritos"}
      </button>
    </div>
  );
}

export default ProductCard;
```

## Flujo general