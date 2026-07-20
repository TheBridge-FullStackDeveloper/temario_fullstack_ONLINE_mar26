# Checkout, Despliegue y Optimización Final

En esta sesión aprenderás a finalizar el flujo del cliente conectando el proceso de **checkout** con el backend y desplegando tu aplicación en **Netlify**, optimizando el rendimiento con `useMemo` y revisando el proyecto completo antes de tu etapa autónoma.

## ¿Qué es el checkout?

El checkout es el proceso final de compra: el usuario confirma los productos de su carrito, introduce sus datos (envío, pago) y la aplicación envía esa información al backend para crear el pedido. Es el punto donde convergen el estado global del carrito, las peticiones HTTP y la gestión de errores.

## Envío del pedido al backend

```jsx
import { useState } from "react";
import { useSelector, useDispatch } from "react-redux";
import axios from "axios";
import { clearCart } from "../features/cart/cartSlice";

function Checkout() {
  const items = useSelector((state) => state.cart.items);
  const dispatch = useDispatch();
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [success, setSuccess] = useState(false);

  const handleCheckout = async (shippingData) => {
    setLoading(true);
    setError(null);

    try {
      await axios.post("/api/orders", {
        items,
        shipping: shippingData
      });

      dispatch(clearCart());
      setSuccess(true);
    } catch (err) {
      setError("No se pudo procesar el pedido");
    } finally {
      setLoading(false);
    }
  };

  if (loading) return <p>Procesando pedido...</p>;
  if (error) return <p>{error}</p>;
  if (success) return <p>¡Pedido confirmado!</p>;

  return (
    <button onClick={() => handleCheckout({ address: "Calle Falsa 123" })}>
      Confirmar compra
    </button>
  );
}

export default Checkout;
```

✔ El patrón `loading / error / success` controla todos los estados posibles del proceso, igual que en cualquier flujo de `useEffect + axios`.

## Optimización del resumen del pedido con useMemo

En la pantalla de checkout es habitual mostrar un desglose de precios (subtotal, impuestos, envío). Estos cálculos no deben repetirse en cada render si los datos del carrito no han cambiado.

```jsx
import { useSelector } from "react-redux";
import { useMemo } from "react";

function OrderSummary() {
  const items = useSelector((state) => state.cart.items);

  const { subtotal, taxes, total } = useMemo(() => {
    const subtotal = items.reduce((acc, item) => acc + item.price * item.quantity, 0);
    const taxes = subtotal * 0.21;
    const total = subtotal + taxes;
    return { subtotal, taxes, total };
  }, [items]);

  return (
    <div>
      <p>Subtotal: {subtotal.toFixed(2)} €</p>
      <p>Impuestos: {taxes.toFixed(2)} €</p>
      <p>Total: {total.toFixed(2)} €</p>
    </div>
  );
}

export default OrderSummary;
```

✔ `useMemo` evita recalcular subtotal, impuestos y total en cada render, solo lo hace cuando cambian los `items`.

## Preparar el proyecto para producción

Antes de desplegar, generamos la build optimizada de la aplicación:

```bash
npm run build
```

Este comando genera una carpeta `dist/` con los archivos estáticos listos para producción (HTML, CSS y JS minificados).

## Despliegue en Netlify

Netlify permite desplegar aplicaciones frontend de forma sencilla conectando directamente el repositorio.

**Pasos principales:**

1. Subir el proyecto a un repositorio de GitHub.
2. Crear un nuevo sitio en Netlify y seleccionar el repositorio.
3. Configurar el comando de build y el directorio de salida:

| Configuración | Valor |
|---|---|
| Build command | `npm run build` |
| Publish directory | `dist` |

4. Añadir las variables de entorno necesarias (por ejemplo, las credenciales de Supabase) en la sección **Site settings → Environment variables**.
5. Desplegar el sitio.

✔ Netlify vuelve a desplegar automáticamente cada vez que se hace push a la rama configurada.

## Revisión final del proyecto

Antes de dar por finalizado el proyecto, conviene repasar los siguientes puntos:

✔ El estado global (Redux) contiene solo los datos que realmente se comparten entre componentes
✔ Las peticiones a la API controlan `loading` y `error` en todos los flujos
✔ Los cálculos costosos (totales, filtros) están memorizados con `useMemo`
✔ Las variables sensibles (claves de API) están en variables de entorno, no en el código
✔ La aplicación funciona correctamente en la versión de producción (`npm run build` + `npm run preview`)
✔ El sitio está desplegado y accesible públicamente en Netlify

## Casos de Uso Comunes

- Enviar el contenido del carrito al backend para generar un pedido.
- Vaciar el carrito tras una compra confirmada.
- Mostrar un resumen de precios optimizado con `useMemo`.
- Desplegar la aplicación en Netlify de forma continua a partir del repositorio.
- Revisar la configuración de producción antes de entregar el proyecto.