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
## 5. Configurar el archivo app.js 
```js
import '../css/app.css';
import './bootstrap';

import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';
import { createApp, h } from 'vue';
import { ZiggyVue } from '../../vendor/tightenco/ziggy';
import PrimeVue from 'primevue/config';
import Aura from '@primevue/themes/aura';
import 'primeicons/primeicons.css'; 

const appName = import.meta.env.VITE_APP_NAME || 'Laravel';

createInertiaApp({
    title: (title) => `${title} - ${appName}`,
    resolve: (name) =>
        resolvePageComponent(
            `./Pages/${name}.vue`,
            import.meta.glob('./Pages/**/*.vue'),
        ),
    setup({ el, App, props, plugin }) {
        return createApp({ render: () => h(App, props) })
            .use(plugin)
            .use(ZiggyVue)
            .use(PrimeVue,{
                theme: {
                    preset: Aura,
                    options: {
                        prefix: 'p',
                        darkModeSelector: '.dark',
                        cssLayer: {
                            name: 'primevue',
                            order: 'tailwind-base, primevue, tailwind-utilities'
                        }
                    }
                }
            })
            .mount(el);
    },
    progress: {
        color: '#4B5563',
    },
});

```
## 6. Mejorar el aspecto del componente Login.vue
````vue
<script setup>
import GuestLayout from '@/Layouts/GuestLayout.vue';
import { Head, useForm } from '@inertiajs/vue3';
import InputText from 'primevue/inputtext';
import Password from 'primevue/password';
import Button from 'primevue/button';
import FloatLabel from 'primevue/floatlabel';

const form = useForm({
    email: '',
    password: '',
    remember: false,
});

const submit = () => {
    form.post(route('login'), {
        onFinish: () => form.reset('password'),
    });
};
</script>

<template>
    <GuestLayout>
        <Head title="Acceso Privado" />

        <div class="text-center mb-10">
            <h1 class="text-3xl font-black text-slate-900 uppercase tracking-widest">Login</h1>
            <p class="text-slate-400 font-medium mt-2">Datos de Acceso</p>
        </div>

        <form @submit.prevent="submit" class="flex flex-col gap-8">
            <div class="flex flex-col gap-1">
                <FloatLabel>
                    <InputText 
                        id="email" 
                        v-model="form.email" 
                        class="w-full !p-3 !border-slate-200" 
                        :class="{ 'p-invalid': form.errors.email }"
                    />
                    <label for="email">Email</label>
                </FloatLabel>
                <small v-if="form.errors.email" class="text-red-500 font-bold ml-1">{{ form.errors.email }}</small>
            </div>

            <div class="flex flex-col gap-1">
                <FloatLabel>
                    <Password 
                        id="password" 
                        v-model="form.password" 
                        :feedback="false" 
                        toggleMask 
                        fluid
                        inputClass="w-full !p-3 !border-slate-200" 
                        :class="{ 'p-invalid': form.errors.password }"
                    />
                    <label for="password">Contraseña</label>
                </FloatLabel>
                <small v-if="form.errors.password" class="text-red-500 font-bold ml-1">{{ form.errors.password }}</small>
            </div>

            <Button 
                type="submit" 
                label="Ingresar" 
                :loading="form.processing" 
                class="w-full !p-4 !bg-indigo-600 !border-none !text-white font-bold"
            />
        </form>
    </GuestLayout>
</template>
```` 
## 7. Crear carpeta Components/nav, para personalizar el panel administrativo con los siguientes Archivos

<img width="172" height="135" alt="image" src="https://github.com/user-attachments/assets/b032bc1b-6e3d-4676-b781-7144f017a047" />
* Navbar.vue
  ````vue
  <script setup>
import { Link } from '@inertiajs/vue3';
import Button from 'primevue/button';

defineEmits(['toggle-sidebar']);
</script>

<template>
    <header class="h-16 bg-indigo-700 text-white flex items-center justify-between px-4 z-50 shadow-md shrink-0">
        <div class="flex items-center gap-4">
            <Button 
                icon="pi pi-bars" 
                text 
                @click="$emit('toggle-sidebar')" 
                class="!text-white hover:!bg-indigo-600 !w-10 !h-10 border-none" 
            />
            <h1 class="text-sm md:text-lg font-bold tracking-wide uppercase opacity-90 truncate">
                Panel Administrativo del Sistema
            </h1>
        </div>

        <div class="flex items-center gap-2 md:gap-4">
            <div class="hidden sm:flex flex-col text-right border-r border-indigo-500 pr-4">
                <span class="text-xs font-bold leading-none">{{ $page.props.auth.user.name }}</span>
                <span class="text-[10px] text-indigo-200 mt-1 font-medium italic">Sesión Activa</span>
            </div>
            
            <Link :href="route('logout')" method="post" as="button" 
                  class="flex items-center gap-2 bg-indigo-800 hover:bg-red-600 px-3 py-2 rounded-lg transition-all text-xs font-bold shadow-md border-none text-white">
                <i class="pi pi-power-off"></i>
                <span class="hidden md:inline uppercase text-[10px]">Salir</span>
            </Link>
        </div>
    </header>
</template>
  ````
* Sidebar.vue
````vue

````
* Footer.vue
## Crear el componente AdminLayout.vue
Rvisar archivos de ruta auth.web, web.php
