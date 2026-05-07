# Crear y configurar proyecto full stack en laravel 12
## Introducción
## 1. Crear proyecto en Laravel 12
```bash
composer create-project laravel/laravel:^12.0 demo-app
```
## 2. Instalar kit de autenticación con laravel/breeze
```bash
composer require laravel/breeze –dev
```
intregar inertia.js y autenticación con Vue 3
```bash
php artisan breeze:install vue
```
## 3. Instalar dependencias de primevue
```bash
npm install primevue @primevue/themes
```
## 4. Personalizar el archivo de estilos app.css 
```css
@layer tailwind-base, primevue, tailwind-utilities;

@import "tailwindcss/base";
@import "tailwindcss/components";
@import "tailwindcss/utilities";

@layer primevue {
    /* Aquí se inyectarán los estilos de Aura */
}

/* Para fondo mas suave */
body {
    @apply bg-slate-50;
}
```
