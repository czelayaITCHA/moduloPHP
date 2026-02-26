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
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```
Si se tiene problemas al ejecutar **npx tailwindcss init -p**, es por incompatibilidad con la versión de node, es este caso crear los archivos **tailwind.config.js** y **postcss.config.js**, en la raíz del proyecto

* Configuración del archivo **tailwind.config.js**
  
```JS
export default {
  content: [
     "./index.html",
    "./src/**/*.{js,jsx}"
  ],
  theme: {
    extend: {},
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

* hacer importaciones de directivas en **src/assets/main.css**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## 3. Instalar PrimeVue
**¿Qué es PrimeVue?** es una completa biblioteca de componentes de interfaz de usuario (UI) de código abierto diseñada específicamente para Vue.js. Ofrece más de 80 componentes listos para usar (tablas, formularios, menús), enfocados en alto rendimiento, personalización y temas integrados, facilitando la creación de aplicaciones web modernas y responsivas.

* Instalación de paquetes
```bash
npm install primevue primeicons
```
* Hacer importaciones en archivo **main.js**
```js
import PrimeVue from 'primevue/config'
import 'primevue/resources/themes/lara-dark-blue/theme.css'
import 'primevue/resources/primevue.min.css'
import 'primeicons/primeicons.css'

app.use(PrimeVue)
```
## 4. Instalar el cliente http Axios 
```bash
npm install axios
```
## 5. instalar vue router y pinia
por alguna razón no se instaló vue-router en la creación del proyecto, se hará manualmente
* Instalar paquete
```bash
npm install vue-router@4 pinia
```
* Crear archivo de rutas del frontend en *src/router/index.js*
  
## 6. crear un servicio en src/services/api.js, para interceptar peticios http, debe crear la carpeta **service** dentro de src

## 7. Definir la siguiente estructura para el proyecto
```Plain Text
src/
├── router/
│     └── index.js
├── services/
│     └── api.js
├── stores/
│     └── auth.js
├── layouts/
│     ├── PublicLayout.vue
│     └── AdminLayout.vue
├── components/
│     ├── public/
│     │     ├── Home.vue
│     │     ├── Products.vue
│     │     └── Login.vue
│     └── admin/
│           ├── Dashboard.vue
│           └── Products.vue
```
