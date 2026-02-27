# 7 - Creación de proyecto Vue 3 tipo SPA
## I. Introducción
## 1. Crear proyecto Vue 3
**¿Que es Vue 3?** es la última versión del framework JavaScript progresivo para construir interfaces de usuario, lanzado en 2020. Destaca por ser más rápido, pequeño y eficiente que Vue 2, 
introduciendo la Composition API para organizar código complejo, un mejor soporte para TypeScript y un sistema de reactividad basado en Proxies ES6. para crear un proyeco se hace de la siguiente manera:

```bash
  npm create vue@latest shop-ft
```
<img width="986" height="401" alt="image" src="https://github.com/user-attachments/assets/aa6c9ce9-8de5-44cd-adae-fd8130deaad0" />
Marcar las tres opciones anteriores con la **barra espaciadora** y luego dar **Enter**

Luego hacer lo siguiente:
```bash
cd shop-ft
```
Instalar dependencias
```bash
npm install
```
Compilar y ejecutar el proyecto 
```bash
npm run dev
```
Aparecerá una ventana similar a la siguiente:

<img width="819" height="207" alt="image" src="https://github.com/user-attachments/assets/825268ae-a1d4-4a79-b2a3-2cc8bcbd6653" />

Correr la aplicación con la url local en el navegador de su prefenencia, debe mostrar la pantalla inicial de un proyecto Vue 3, puede verificar que ya una aplicación de tipo SPA, como se muestra a continuación

<img width="1355" height="662" alt="image" src="https://github.com/user-attachments/assets/0d57f399-f5fe-4030-b2e4-03c9f31bb4de" />


Si abrimos el proyecto con VSCode, la estructura inicial de un proyecto vue 3, es como el de la siguiente imágen
<img width="1278" height="702" alt="image" src="https://github.com/user-attachments/assets/1be3fd27-9bef-4663-80be-30db9e97e01c" />


## 2. Instalar tailwind css
Tailwind es un framework de CSS que, a diferencia de los tradicionales como Bootstrap, no crea componentes con una sola clase, sino que tiene algo llamado Utility Classes, que son clases específicas para casa cosa. Por ejemplo, una clase para los textos, otra clase para las sobras, una para el color, entre otros. El objetivo es que puedas personalizar a fondo.

* Instalación de los paquetes
```bash
npm install -D tailwindcss@3.4.4 postcss autoprefixer
npx tailwindcss init -p
```
Si tiene problemas al ejecutar **npx tailwindcss init -p**, es por incompatibilidad con la versión de node, en este caso crear los archivos **tailwind.config.js** y **postcss.config.js**, en la raíz del proyecto manualmente

* Configuración del archivo **tailwind.config.js**
  
```JS
export default {
  content: [
    "./index.html",
    "./src/**/*.{vue,js,ts,jsx,tsx}",
  ],
  // AÑADE ESTO: Es lo que evita que los botones de primevue pierdan el fondo
  corePlugins: {
    preflight: false,
  },
  theme: {
    extend: {
      colors:{
        primary: "#0F172A",
        secondary: "#1E293B",
        accent: "#3B82F6",
      }
    },
  },
  plugins: [],
}
```
* Configuración del archivo **postcss.config.js**

```JS
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

* hacer importaciones de directivas en **src/assets/main.css**, se crean capas para evitar conflictos entre estilos de tailwind y primevue
```css
/* 1. Definimos el orden */
@layer tailwind-base, primevue, tailwind-utilities;

/* 2. Base de Tailwind */
@layer tailwind-base {
  @tailwind base;
  
  /* Reset mínimo manual para que la parte pública no se rompa */
  *, ::before, ::after { box-sizing: border-box; border-width: 0; border-style: solid; }
  img, svg { display: block; vertical-align: middle; }
}

/* 3. Forzamos a que PrimeVue use su capa (IMPORTANTE) */
@layer primevue {
  /* No pongas nada aquí, lo inyectaremos vía CSS imports o déjalo listo para overrides */
}

/* 4. Utilidades de Tailwind al final para que siempre ganen */
@layer tailwind-utilities {
  @tailwind components;
  @tailwind utilities;
}```


* Probar que tailwind este funcionando, elimina el contenido del componente App.vue y cambialo por este:
```JS
  <template>
  <div class="bg-red-500 text-white p-10 text-3xl">
    Tailwind funcionando 🚀
  </div>
</template>
```
Si el resultado es como la imágen de abajo, significa que tailwind esta funcionando correctamente

<img width="1355" height="168" alt="image" src="https://github.com/user-attachments/assets/b64e64d6-2c3d-40a7-9d96-229c33c0cfcb" />

## 3. Instalar PrimeVue
**¿Qué es PrimeVue?** es una completa biblioteca de componentes de interfaz de usuario (UI) de código abierto diseñada específicamente para Vue.js. Ofrece más de 80 componentes listos para usar (tablas, formularios, menús), enfocados en alto rendimiento, personalización y temas integrados, facilitando la creación de aplicaciones web modernas y responsivas.

* Instalación de paquetes de la version 3
```bash
npm install primevue@3.53.1 primeicons
```
* Hacer importaciones en archivo **main.js**
```js
import PrimeVue from 'primevue/config'
import 'primevue/resources/themes/lara-dark-blue/theme.css'
import 'primevue/resources/primevue.min.css'
import 'primeicons/primeicons.css'

app.use(PrimeVue)
```
El archivo **main.js**, debe quedar así como se muestra a continuación
```JS
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import { createPinia } from 'pinia'
import PrimeVue from 'primevue/config'

// 1. IMPORTAR ESTILOS DE Tailwind
import './assets/main.css' 

// 2. ESTILOS DE PRIMEVUE
import 'primevue/resources/themes/lara-dark-blue/theme.css'
import 'primevue/resources/primevue.min.css'
import 'primeicons/primeicons.css'

const app = createApp(App)
app.use(createPinia())
app.use(router)
app.use(PrimeVue)
app.mount('#app')
```
* Probar que tailwind y primevue funcionen correctamente, cambiar el contenido del componente principal App.vue, por el siguiente código

```JS
<script setup>
import Button from 'primevue/button'
import Card from 'primevue/card'
</script>

<template>
  <div class="min-h-screen bg-slate-900 flex items-center justify-center p-10">

    <Card class="w-full max-w-md shadow-2xl">
      <template #title>
        <div class="text-center text-2xl font-bold">
          🚀 TechStore UI Test
        </div>
      </template>

      <template #content>
        <p class="mb-6 text-center">
          Si ves esto con estilo moderno,
          PrimeVue y Tailwind están funcionando correctamente.
        </p>

        <div class="flex justify-center gap-4">
          <Button label="Primario" icon="pi pi-check" />
          <Button label="Secundario" severity="secondary" />
        </div>
      </template>
    </Card>

  </div>
</template>
```
Si el resultado es como el que se muestra en la siguiente imágen, indica que todo esta bien por el momento

<img width="411" height="249" alt="image" src="https://github.com/user-attachments/assets/64cda6de-e30e-4635-baef-10eba6c22e3e" />

## 4. Instalar el cliente http Axios 
```bash
npm install axios
```
