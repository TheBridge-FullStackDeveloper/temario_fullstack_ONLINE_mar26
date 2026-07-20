# Autenticación con Supabase y Redux Toolkit

En esta sesión aprenderás a implementar un sistema de autenticación real conectando tu frontend con **Supabase** y **Redux Toolkit** para gestionar el flujo de login y registro mediante el uso de **tokens JWT**.

## ¿Qué es Supabase?

Supabase es una plataforma backend que ofrece, entre otros servicios, un sistema de autenticación listo para usar. Permite registrar usuarios, iniciar sesión y gestionar sesiones mediante **tokens JWT**, sin necesidad de construir un backend de autenticación propio.

## Instalación

Para conectar nuestra aplicación React con Supabase, instalamos su cliente oficial:

```bash
npm install @supabase/supabase-js
```

## Configuración del cliente de Supabase

Creamos un archivo de configuración donde inicializamos el cliente con la URL y la clave pública (`anon key`) del proyecto.

```javascript
// src/supabase/client.js
import { createClient } from "@supabase/supabase-js";

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseKey = import.meta.env.VITE_SUPABASE_ANON_KEY;

export const supabase = createClient(supabaseUrl, supabaseKey);
```

✅ Las credenciales se guardan en variables de entorno (`.env`) y nunca directamente en el código.

## Creación del Slice de autenticación

Al igual que con cualquier otra parte del estado global, la autenticación se gestiona mediante un **slice** de Redux Toolkit, que almacena el usuario actual, el token y los estados de carga y error.

```javascript
// src/features/auth/authSlice.js
import { createSlice } from "@reduxjs/toolkit";

const initialState = {
  user: null,
  token: null,
  loading: false,
  error: null
};

const authSlice = createSlice({
  name: "auth",
  initialState,
  reducers: {
    loginStart: (state) => {
      state.loading = true;
      state.error = null;
    },
    loginSuccess: (state, action) => {
      state.loading = false;
      state.user = action.payload.user;
      state.token = action.payload.token;
    },
    loginFailure: (state, action) => {
      state.loading = false;
      state.error = action.payload;
    },
    logout: (state) => {
      state.user = null;
      state.token = null;
    }
  }
});

export const { loginStart, loginSuccess, loginFailure, logout } = authSlice.actions;
export default authSlice.reducer;
```

## Registro de usuario (Sign Up)

Utilizamos el método `signUp` de Supabase para crear un nuevo usuario.

```javascript
import { supabase } from "../../supabase/client";

export const registerUser = async (email, password) => {
  const { data, error } = await supabase.auth.signUp({
    email,
    password
  });

  if (error) throw error;
  return data;
};
```

## Inicio de sesión (Login) con Redux

Combinamos la llamada a Supabase con el `dispatch` de las acciones del slice para reflejar el estado de autenticación en toda la aplicación.

```jsx
import { useDispatch } from "react-redux";
import { supabase } from "../../supabase/client";
import { loginStart, loginSuccess, loginFailure } from "../../features/auth/authSlice";

function LoginForm() {
  const dispatch = useDispatch();

  const handleLogin = async (email, password) => {
    dispatch(loginStart());

    const { data, error } = await supabase.auth.signInWithPassword({
      email,
      password
    });

    if (error) {
      dispatch(loginFailure(error.message));
      return;
    }

    dispatch(
      loginSuccess({
        user: data.user,
        token: data.session.access_token
      })
    );
  };

  return (
    <button onClick={() => handleLogin("test@mail.com", "123456")}>
      Iniciar sesión
    </button>
  );
}

export default LoginForm;
```

## Persistencia de la sesión (JWT)

Supabase gestiona internamente el token JWT en la sesión, pero también podemos comprobar si existe una sesión activa al montar la aplicación, para mantener al usuario logueado tras recargar la página.

```jsx
import { useEffect } from "react";
import { useDispatch } from "react-redux";
import { supabase } from "../../supabase/client";
import { loginSuccess, logout } from "../../features/auth/authSlice";

function AuthListener() {
  const dispatch = useDispatch();

  useEffect(() => {
    const { data: listener } = supabase.auth.onAuthStateChange((event, session) => {
      if (session) {
        dispatch(
          loginSuccess({
            user: session.user,
            token: session.access_token
          })
        );
      } else {
        dispatch(logout());
      }
    });

    return () => listener.subscription.unsubscribe();
  }, [dispatch]);

  return null;
}

export default AuthListener;
```

✔ `onAuthStateChange` mantiene sincronizado el estado global con la sesión real de Supabase.

## Cierre de sesión (Logout)

```javascript
import { supabase } from "../../supabase/client";

export const logoutUser = async () => {
  const { error } = await supabase.auth.signOut();
  if (error) throw error;
};
```

## Flujo de autenticación