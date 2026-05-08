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
@import "tailwindcss/base";
@import "tailwindcss/components";
@import "tailwindcss/utilities";

/* 1. Definimos el orden de capas para que PrimeVue tenga su espacio */
@layer tailwind-base, primevue, tailwind-utilities;

@layer base {
    /* Evitar que Tailwind resetee componentes de PrimeVue */
    [class^='p-'], [class*=' p-'] {
        border-style: solid;
        border-width: 0;
        box-sizing: border-box;
    }

    /* 3. Forzamos que el Dialog tenga su fondo blanco y sombra originales */
    .p-dialog {
        @apply bg-white shadow-2xl border border-slate-200 !important;
    }
    
    .p-dialog-mask {
        @apply bg-black/50 backdrop-blur-sm !important;
    }
}

@layer primevue {
    /* PrimeVue inyectará Aura aquí. No borrar. */
}

body {
    @apply bg-slate-50 antialiased text-slate-800;
}
```
## 5. Configurar el archivo app.js 
```js
import '../css/app.css';
import './bootstrap';
import 'primeicons/primeicons.css'; 

import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';
import { ZiggyVue } from '../../vendor/tightenco/ziggy';

// PrimeVue
import PrimeVue from 'primevue/config';
import Aura from '@primevue/themes/aura';

const appName = import.meta.env.VITE_APP_NAME || 'Laravel';

createInertiaApp({
    title: (title) => `${title} - ${appName}`,
    resolve: (name) =>
        resolvePageComponent(
            `./Pages/${name}.vue`,
            import.meta.glob('./Pages/**/*.vue'),
        ),
    setup({ el, App, props, plugin }) {
        const app = createApp({ render: () => h(App, props) });
        
        app.use(plugin)
           .use(ZiggyVue)
           .use(PrimeVue, {
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
        color: '#4f46e5', // Color indigo-600 para la barra de carga
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
<script setup>
import { Link } from '@inertiajs/vue3';

const props = defineProps({ 
    isVisible: Boolean 
});

const emit = defineEmits(['item-click']);

const menuGroups = [
    {
        title: 'Menú Principal',
        items: [
            { label: 'Inicio', icon: 'pi pi-home', route: 'dashboard' },
            { label: 'Catálogos', icon: 'pi pi-database', route: '#' },
            { label: 'Configuración', icon: 'pi pi-cog', route: '#' },
            { label: 'Registrar Usuario', icon: 'pi pi-user-plus', route: 'register' },
        ]
    }
];

// Función para gestionar el click
const handleLinkClick = (routeStr) => {
    // Emitimos el cierre para móviles
    emit('item-click');
    
    // Si la ruta es '#', podemos prevenir o manejar algo aquí
    if (routeStr === '#') {
        console.log('Ruta no definida');
    }
};
</script>

<template>
    <aside 
        :class="[
            isVisible ? 'w-64 translate-x-0' : 'w-0 -translate-x-full lg:w-0',
            'bg-slate-100 text-slate-700 transition-all duration-300 fixed lg:relative z-40 h-full border-r border-slate-200 flex flex-col shadow-2xl lg:shadow-none overflow-hidden'
        ]"
    >
        <div class="h-16 flex items-center px-6 bg-slate-200/50 border-b border-slate-200 shrink-0">
            <i class="pi pi-desktop text-indigo-600 mr-3 text-xl font-bold"></i>
            <span class="font-black text-slate-800 tracking-tighter text-xl">DemoApp</span>
        </div>

        <nav class="flex-1 p-4 space-y-6 overflow-y-auto">
            <div v-for="group in menuGroups" :key="group.title">
                <p v-if="isVisible" class="text-[10px] font-bold uppercase tracking-[2px] text-slate-400 mb-4 px-2">
                    {{ group.title }}
                </p>
                <div class="space-y-1">
                    <Link 
                        v-for="item in group.items" 
                        :key="item.label"
                        :href="item.route !== '#' ? route(item.route) : '#'"
                        @click="handleLinkClick(item.route)"
                        class="flex items-center gap-3 p-3 rounded-xl hover:bg-white hover:text-indigo-600 hover:shadow-sm border border-transparent hover:border-slate-200 transition-all group"
                    >
                        <i :class="[item.icon, 'text-lg text-slate-400 group-hover:text-indigo-600']"></i>
                        <span v-if="isVisible" class="text-sm font-semibold">{{ item.label }}</span>
                    </Link>
                </div>
            </div>
        </nav>
    </aside>
</template>
````
* Footer.vue
  
````vue
<template>
    <footer class="h-10 bg-white border-t border-slate-200 flex items-center justify-between px-6 text-[10px] text-slate-400 font-bold uppercase tracking-widest shrink-0">
        <div>&copy; 2026 <span class="text-indigo-600 italic">Demo-App</span></div>
        <div class="flex items-center gap-2">
            <span class="w-2 h-2 rounded-full bg-green-500 animate-pulse"></span>
            Sistema en Línea
        </div>
    </footer>
</template>
````
## 8. Crear el componente AdminLayout.vue
````vue
<script setup>
import { ref, onMounted } from 'vue';
import Sidebar from '@/Components/nav/Sidebar.vue';
import Navbar from '@/Components/nav/Navbar.vue';
import Footer from '@/Components/nav/Footer.vue';

const isSidebarVisible = ref(true);

// Detectar tamaño de pantalla al montar el componente
onMounted(() => {
    checkScreenSize();
    window.addEventListener('resize', checkScreenSize);
});

const checkScreenSize = () => {
    // Si la pantalla es menor a 1024px (Lg en Tailwind), ocultar sidebar
    if (window.innerWidth < 1024) {
        isSidebarVisible.value = false;
    } else {
        isSidebarVisible.value = true;
    }
};

const toggleSidebar = () => {
    isSidebarVisible.value = !isSidebarVisible.value;
};

// Función para cerrar el sidebar automáticamente en móviles al hacer click en una opción
const handleSidebarItemClick = () => {
    if (window.innerWidth < 1024) {
        isSidebarVisible.value = false;
    }
};
</script>

<template>
    <div class="flex flex-col h-screen w-full bg-slate-50 overflow-hidden font-sans">
        <Navbar @toggle-sidebar="toggleSidebar" />

        <div class="flex flex-1 overflow-hidden relative">
            <Sidebar 
                :isVisible="isSidebarVisible" 
                @item-click="handleSidebarItemClick"
            />

            <transition name="fade">
                <div 
                    v-if="isSidebarVisible" 
                    @click="toggleSidebar" 
                    class="lg:hidden fixed inset-0 bg-slate-900/50 z-30 mt-16"
                ></div>
            </transition>

            <div class="flex-1 flex flex-col min-w-0">
                <main class="flex-1 overflow-y-auto p-4 md:p-6">
                    <slot />
                </main>
                <Footer />
            </div>
        </div>
    </div>
</template>

<style scoped>
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
</style>
````

## 9. Actualizar archivos de rutas auth.web, web.php
* web.php
  
```php
  <?php

use App\Http\Controllers\ProfileController;
use App\Http\Controllers\Auth\RegisteredUserController;
use Illuminate\Support\Facades\Route;
use Inertia\Inertia;

// 1. Redirección de Raíz
Route::get('/', function () {
    return redirect()->route('dashboard');
});

// 2. Rutas Protegidas (Solo usuarios autenticados)
Route::middleware(['auth', 'verified'])->group(function () {
    
    // Panel Principal
    Route::get('/dashboard', function () {
        return Inertia::render('Dashboard');
    })->name('dashboard');

    // Gestión de Usuarios (Registro Interno)
    // Ahora accesible solo si estás logueado
    Route::get('register', [RegisteredUserController::class, 'create'])
                ->name('register');
    
    Route::post('register', [RegisteredUserController::class, 'store']);
    

    // Perfil de Usuario
    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');
    
});

require __DIR__.'/auth.php';
```
* auth.php

```php
<?php

use App\Http\Controllers\Auth\AuthenticatedSessionController;
use App\Http\Controllers\Auth\ConfirmablePasswordController;
use App\Http\Controllers\Auth\EmailVerificationNotificationController;
use App\Http\Controllers\Auth\EmailVerificationPromptController;
use App\Http\Controllers\Auth\NewPasswordController;
use App\Http\Controllers\Auth\PasswordController;
use App\Http\Controllers\Auth\PasswordResetLinkController;
use App\Http\Controllers\Auth\RegisteredUserController;
use App\Http\Controllers\Auth\VerifyEmailController;
use Illuminate\Support\Facades\Route;

Route::middleware('guest')->group(function () {
        
    Route::get('login', [AuthenticatedSessionController::class, 'create'])
        ->name('login');

    Route::post('login', [AuthenticatedSessionController::class, 'store']);

    Route::get('forgot-password', [PasswordResetLinkController::class, 'create'])
        ->name('password.request');

    Route::post('forgot-password', [PasswordResetLinkController::class, 'store'])
        ->name('password.email');

    Route::get('reset-password/{token}', [NewPasswordController::class, 'create'])
        ->name('password.reset');

    Route::post('reset-password', [NewPasswordController::class, 'store'])
        ->name('password.store');
});

Route::middleware('auth')->group(function () {
    Route::get('verify-email', EmailVerificationPromptController::class)
        ->name('verification.notice');

    Route::get('verify-email/{id}/{hash}', VerifyEmailController::class)
        ->middleware(['signed', 'throttle:6,1'])
        ->name('verification.verify');

    Route::post('email/verification-notification', [EmailVerificationNotificationController::class, 'store'])
        ->middleware('throttle:6,1')
        ->name('verification.send');

    Route::get('confirm-password', [ConfirmablePasswordController::class, 'show'])
        ->name('password.confirm');

    Route::post('confirm-password', [ConfirmablePasswordController::class, 'store']);

    Route::put('password', [PasswordController::class, 'update'])->name('password.update');

    Route::post('logout', [AuthenticatedSessionController::class, 'destroy'])
        ->name('logout');
});
```
##  10. Actualizar controlador RegisterUserController.php
```php
public function create(): Response
    {
        //return Inertia::render('Auth/Register');
        return Inertia::render('Auth/Register', [
            'users' => User::all() // Enviamos todos los usuarios a la vista
        ]);
    }
```
## 11. limpiar rutas 
```bash
php artisan route:clear

php artisan ziggy:generate
```
## 12. Registrar usuario con tinker

```php
php artisan tinker
```

```php
$user = new App\Models\User();
$user->name = 'Admon del Sistema';
$user->email = 'admin@example.com';
$user->password = Hash::make('12345'); 
$user->email_verified_at = now(); // Para saltar la verificación de correo
$user->save();
```
