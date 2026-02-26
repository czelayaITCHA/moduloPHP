# 7 - Creación de proyecto Vue 3 tipo SPA
## I. Introducción
## 1. Crear proyecto Vue 3
**¿Que es Vue 3?** es la última versión del framework JavaScript progresivo para construir interfaces de usuario, lanzado en 2020. Destaca por ser más rápido, pequeño y eficiente que Vue 2, 
introduciendo la Composition API para organizar código complejo, un mejor soporte para TypeScript y un sistema de reactividad basado en Proxies ES6. para crear un proyeco se hace de la siguiente manera:

```bash
  npm create vue@latest shop-ft
```
<img width="1023" height="637" alt="image" src="https://github.com/user-attachments/assets/d41ba88f-373f-4fb8-8577-27df247fa0e1" />

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
Correr la aplicación con la url local en el navegador de su prefenencia, debe mostrar la pantalla inicial de un proyecto Vue 3, como se muestra a continuación

<img width="1359" height="655" alt="image" src="https://github.com/user-attachments/assets/7b8ee978-a18b-4b80-98d4-9e46164bf763" />

Si abrimos el proyecto con VSCode, la estructura inicial de un proyecto vue 3, es como el de la siguiente imágen
<img width="1360" height="700" alt="image" src="https://github.com/user-attachments/assets/ff903526-dd42-4e15-a19d-edfd86731903" />

## 2. Instalar tailwind css
Tailwind es un framework de CSS que, a diferencia de los tradicionales como Bootstrap, no crea componentes con una sola clase, sino que tiene algo llamado Utility Classes, que son clases específicas para casa cosa. Por ejemplo, una clase para los textos, otra clase para las sobras, una para el color, entre otros. El objetivo es que puedas personalizar a fondo.

* Instalación de los paquetes
```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```
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
* hacer importaciones de directivas en **src/assets/main.css**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```
## 3. Instalar PrimeVue

## 4. Instalar el cliente http Axios 



