# 10. Diseñar y programar el Panel Administrativo de la Aplicación

## 10.1 Crear y Diseñar AdminLayout.vue

````vue
  <template>
  <div class="min-h-screen flex flex-col bg-gray-100">
    <!-- Inyectamos componente AdminNavbar -->
    <AdminNavbar @toggleSidebar="toggleSidebar" />

    <div class="flex flex-1 relative">
      <!-- Esconder el AdminSidebar automaticamente cuando es movil -->
      <div
        v-if="sidebarOpen && isMobile"
        class="fixed inset-0 bg-black/40 z-30"
        @click="closeSidebar"
      ></div>

      <!-- Inyectamos componente AdminSidebar -->
      <AdminSidebar
        :open="sidebarOpen"
        :isMobile="isMobile"
        @navigate="closeSidebar"
      />

      <!-- En main se renderizan los componentes con las opciones y rutas -->
      <main class="flex-1 p-6 overflow-y-auto">
        <router-view />
      </main>
    </div>

    <AdminFooter />
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

import AdminNavbar from "@/components/admin/AdminNavbar.vue";
import AdminSidebar from "@/components/admin/AdminSidebar.vue";
import AdminFooter from "@/components/admin/AdminFooter.vue";

const sidebarOpen = ref(false);
const isMobile = ref(false);

const toggleSidebar = () => {
  sidebarOpen.value = !sidebarOpen.value;
}

const closeSidebar = () =>{
  if (isMobile.value) {
    sidebarOpen.value = false;
  }
}

const checkScreen = () => {
  isMobile.value = window.innerWidth < 768;

  if (!isMobile.value) {
    sidebarOpen.value = true;
  } else {
    sidebarOpen.value = false;
  }
}

onMounted(() => {
  checkScreen();

  window.addEventListener("resize", checkScreen);
});
</script>
```` 
## 10.2 Crear los componentes que conformarán el Panel Administrativo
* Crear el componente **components/admin/AdminNavbar.vue**
  
````vue
<template>
  <header class="w-full bg-blue-600 text-white shadow">
    <div class="flex items-center justify-between px-6 py-3">
      <!-- Botón tipo hamburguesa para mostrar/ocultar el Sidebar -->
      <div class="flex items-center gap-4">
        <button
          @click="$emit('toggleSidebar')"
          class="hover:bg-blue-500 p-2 rounded transition"
        >
          <i class="pi pi-bars text-xl"></i>
        </button>

        <h1 class="font-bold text-lg hidden sm:block">
          Sistema de Gestión ShopApp
        </h1>
      </div>

      <!-- Div en la parte derecha para mostrar datos del usuario logueado -->

      <div class="flex items-center gap-4">
        <div
          class="flex items-center gap-2 bg-blue-500 px-3 py-1 rounded-full text-sm"
        >
          <i class="pi pi-user"></i>
          <span>{{ authStore.user?.email }}</span>
        </div>

        <button
          @click="logout"
          class="bg-red-500 hover:bg-red-600 px-3 py-1 rounded flex items-center gap-2"
        >
          <i class="pi pi-sign-out"></i>
          Salir
        </button>
      </div>
    </div>
  </header>
</template>

<script setup>
import { useAuthStore } from "@/stores/authStore";
import { useRouter } from "vue-router";

const authStore = useAuthStore();
const router = useRouter();

const logout = () => {
  authStore.logout();
  router.push("/login");
}
</script>  
````
* Crear el componente **components/admin/AdminSidebar.vue**
````vue
<template>
  <aside
    :class="[
      'bg-slate-900 text-gray-200 w-64 min-h-screen fixed md:relative z-40 transition-transform duration-300',

      open ? 'translate-x-0' : '-translate-x-full',

      !isMobile && open ? 'md:translate-x-0' : '',
    ]"
  >
    <div class="p-6 border-b border-gray-700">
      <div class="flex items-center gap-3">
        <div class="bg-blue-600 p-2 rounded">
          <i class="pi pi-shield text-white"></i>
        </div>

        <h2 class="font-bold text-lg">Shop-App</h2>
      </div>
    </div>

    <nav class="mt-4 flex flex-col text-sm">
      <router-link
        to="/admin/dashboard"
        class="menu-item"
        @click="$emit('navigate')"
      >
        <i class="pi pi-home"></i>
        Inicio
      </router-link>

      <router-link
        to="/admin/catalogos"
        class="menu-item"
        @click="$emit('navigate')"
      >
        <i class="pi pi-book"></i>
        Catálogos
      </router-link>

      <router-link
        to="/admin/productos"
        class="menu-item"
        @click="$emit('navigate')"
      >
        <i class="pi pi-box"></i>
        Productos
      </router-link>

      <router-link
        to="/admin/ordenes"
        class="menu-item"
        @click="$emit('navigate')"
      >
        <i class="pi pi-shopping-cart"></i>
        Órdenes
      </router-link>

      <router-link
        to="/admin/usuarios"
        class="menu-item"
        @click="$emit('navigate')"
      >
        <i class="pi pi-users"></i>
        Usuarios
      </router-link>

      <router-link
        to="/admin/reportes"
        class="menu-item"
        @click="$emit('navigate')"
      >
        <i class="pi pi-chart-bar"></i>
        Reportes
      </router-link>
    </nav>
  </aside>
</template>

<script setup>
defineProps({
  open: Boolean,
  isMobile: Boolean,
});
</script>

<style scoped>
.menu-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 20px;
  transition: all 0.2s;
}

.menu-item:hover {
  background: #1e293b;
}

.router-link-active {
  background: #2563eb;
  color: white;
}
</style>
````
* Crear el componente **components/admin/AdminFooter.vue**
````vue
<template>
  <footer class="bg-white border-t text-center py-3 text-sm text-gray-500">
    Sistema de Gestión ShopApp © {{ new Date().getFullYear() }}
  </footer>
</template>
````
## 10.3 Crear componente views/admin/Dashboard.vue para mostrar información de primera mano en la ruta home del panel administrativo
````vue
<template>
  <div>
    <h2 class="text-2xl font-bold mb-6">Resumen Operativo</h2>

    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-5 gap-6">
      <!-- Ventas -->
      <div class="card">
        <div class="icon bg-blue-100 text-blue-600">
          <i class="pi pi-chart-line"></i>
        </div>

        <div>
          <p class="label">Ventas Totales</p>
          <h3 class="value">$1,250.75</h3>
        </div>
      </div>

      <!-- Ordenes Pendientes -->
      <div class="card">
        <div class="icon bg-yellow-100 text-yellow-600">
          <i class="pi pi-clock"></i>
        </div>

        <div>
          <p class="label">Órdenes Pendientes</p>
          <h3 class="value">8</h3>
        </div>
      </div>

      <!-- Ordenes Pagadas -->
      <div class="card">
        <div class="icon bg-green-100 text-green-600">
          <i class="pi pi-check-circle"></i>
        </div>

        <div>
          <p class="label">Órdenes Pagadas</p>
          <h3 class="value">25</h3>
        </div>
      </div>

      <!-- Ordenes Canceladas -->
      <div class="card">
        <div class="icon bg-red-100 text-red-600">
          <i class="pi pi-times-circle"></i>
        </div>

        <div>
          <p class="label">Órdenes Canceladas</p>
          <h3 class="value">3</h3>
        </div>
      </div>

      <!-- Ordenes Reembolsadas -->
      <div class="card">
        <div class="icon bg-purple-100 text-purple-600">
          <i class="pi pi-refresh"></i>
        </div>

        <div>
          <p class="label">Órdenes Reembolsadas</p>
          <h3 class="value">2</h3>
        </div>
      </div>
    </div>
  </div>
</template>


<style scoped>
.card {
  @apply bg-white p-5 rounded-xl shadow-sm flex items-center gap-4 transition hover:shadow-md;
}

.icon {
  @apply flex items-center justify-center w-12 h-12 rounded-lg text-xl;
}

.label {
  @apply text-gray-500 text-sm;
}

.value {
  @apply font-bold text-xl text-gray-800;
}
</style>
````
## 10.4 Actualizar el archivo de rutas
```js
import { createRouter, createWebHistory } from 'vue-router'
import { useAuthStore } from '@/stores/authStore'

import HomeView from '../views/HomeView.vue'
import Login from '@/views/Login.vue'
import Register from '@/views/Register.vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),

  routes: [

    {
      path: '/login',
      name: 'login',
      component: Login,
      meta: { guest: true }
    },

    {
      path: '/register',
      name: 'register',
      component: Register,
      meta: { guest: true }
    },

    {
      path: '/',
      name: 'home',
      component: HomeView
    },

    {
      path: '/order',
      name: 'order',
      component: () => import('@/views/OrderView.vue'),
      meta: { requiresAuth: true, role: 'CLIENTE' }
    },

    {
      path: '/mis-ordenes',
      name: 'mis-ordenes',
      component: () => import('@/views/MisOrdenes.vue'),
      meta: { requiresAuth: true, role: 'CLIENTE' }
    },

    // ADMIN PANEL
    {
      path:'/admin',
      component: () => import('@/components/layouts/AdminLayout.vue'),
      meta:{ requiresAuth:true, role:['ADMIN','VENDEDOR'] },

      children:[

        {
          path:'dashboard',
          name:'admin-dashboard',
          component: () => import('@/views/admin/Dashboard.vue')
        },

        {
          path:'catalogos',
          name:'admin-catalogos',
          component: () => import('@/views/admin/Catalogos.vue')
        },

        {
          path:'productos',
          name:'admin-productos',
          component: () => import('@/views/admin/Productos.vue')
        },

        {
          path:'ordenes',
          name:'admin-ordenes',
          component: () => import('@/views/admin/Ordenes.vue')
        },

        {
          path:'usuarios',
          name:'admin-usuarios',
          component: () => import('@/views/admin/Usuarios.vue')
        },

        {
          path:'reportes',
          name:'admin-reportes',
          component: () => import('@/views/admin/Reportes.vue')
        }

      ]
    }

  ]
})


//para proteger las rutas
router.beforeEach((to) => {

  const authStore = useAuthStore()

  // requiere login
  if (to.meta.requiresAuth && !authStore.isAuthenticated) {
    return '/login'
  }

  // validación de roles
  if (to.meta.role) {

    const userRoles = authStore.user?.roles?.map(r => r.name) || []

    // si la ruta acepta varios roles
    if (Array.isArray(to.meta.role)) {

      const hasRole = to.meta.role.some(role => userRoles.includes(role))

      if (!hasRole) return '/'

    } 
    // si la ruta acepta un solo rol
    else {

      if (!userRoles.includes(to.meta.role)) {
        return '/'
      }

    }

  }

  // rutas solo invitados
  if (to.meta.guest && authStore.isAuthenticated) {
    return '/'
  }

})

export default router
```
## 10.5 La estructura de carpetas y archivos para el Panel Administrativo debe verse como el sisguiente:
```bash
src
 ├ layouts
 │   └ AdminLayout.vue
 │
 ├ components/admin
 │   ├ AdminNavbar.vue
 │   ├ AdminSidebar.vue
 │   └ AdminFooter.vue
 │
 └ views/admin
     ├ Dashboard.vue
     ├ Productos.vue
     ├ Ordenes.vue
     ├ Usuarios.vue
     └ Reportes.vue
```
## 10.11 Programar CRUD de tabla independiente (marcas)
* Como se usarán compomentes de primevue, se debe importar, para ello vamos a crear la carpeta **utils** dentro de **src** y luego el archivo **primevue.js** que contendrá lo siguiente:
```js
import PrimeVue from 'primevue/config'

//servicios
import ToastService from 'primevue/toastservice';
import ConfirmationService from 'primevue/confirmationservice'

//componentes
import Button from 'primevue/button';
import Dialog from 'primevue/dialog';
import InputText from 'primevue/inputtext';
import DataTable from 'primevue/datatable';
import Column from 'primevue/column';
import Toolbar from 'primevue/toolbar';
import Toast from 'primevue/toast';
import ConfirmDialog from 'primevue/confirmdialog';
import Card from 'primevue/card';


export default function setupPrimeVue(app){
    //registramos en app
    app.use(PrimeVue)
    //registramos los servicios
    app.use(ToastService)
    app.use(ConfirmationService)
    //componentes
    app.component('Button', Button)
    app.component('Dialog', Dialog)
    app.component('InputText', InputText)
    app.component('DataTable', DataTable)
    app.component('Column', Column)
    app.component('Toolbar', Toolbar)
    app.component('Toast', Toast)
    app.component('ConfirmDialog', ConfirmDialog)
    app.component('Card', Card)
}
``` 
* importamos el archivo en **main.js** quedando de la siguiente manera
```js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import { createPinia } from 'pinia'
//importamos el plugin de pinia que mantiene la persistencia del estado
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

import setupPrimeVue from './utils/primevue'

// 1. IMPORTAR ESTILOS DE Tailwind
import './assets/main.css' 

// 2. ESTILOS DE PRIMEVUE
import 'primevue/resources/themes/saga-blue/theme.css'
import 'primevue/resources/primevue.css'
import 'primeicons/primeicons.css'

const app = createApp(App)

setupPrimeVue(app)
//creamos instancia de pinia
const pinia = createPinia()
//hacemos que pinia use el plugin de persistencia del estado
pinia.use(piniaPluginPersistedstate)
//hacemos que el objeto vue(app) use pinia
app.use(pinia)
app.use(router)
app.mount('#app')

```
*  Componente Marcas.vue ajustado para cargar las marcas, actualizado con todas las funcionalidades de un CRUD

````vue

<template>
    <div>
        <Toast />
        <div class="card">
            <Toolbar class="mb-4">
                <template #start>
                    <Button label="Nueva Marca" icon="pi pi-plus" severity="success" class="mr-2" @click="openNew" />
                </template>
            </Toolbar>

            <DataTable ref="dt" :value="marcas" dataKey="id"
                :paginator="true" :rows="10" :filters="filters"
                paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown" 
                :rowsPerPageOptions="[5,10,25]"
                currentPageReportTemplate="Mostrando del {first} al {last} de {totalRecords} marcas">
                <template #header>
                    <div class="flex flex-wrap gap-2 items-center justify-between">
                        <h4 class="m-0">Gestión de Marcas</h4>

                        <div class="p-input-icon-left">
                            <i class="pi pi-search"></i>
                            <InputText v-model="filters['global'].value" placeholder="Buscar..." />
                        </div>
                    </div>
                </template>
                <Column field="nombre" header="Marca" sortable style="min-width:16rem"></Column>
                <Column :exportable="false" style="min-width:8rem">
                    <template #body="slotProps">
                        <div class="flex gap-2">
                            <Button icon="pi pi-pencil" outlined rounded @click="editMarca(slotProps.data)" />
                            <Button icon="pi pi-trash" outlined rounded severity="danger" @click="confirmDeleteMarca(slotProps.data)" />
                        </div>
                    </template>
                </Column>
            </DataTable>
        </div>
        <!--Formulario modal de primevue-->
        <Dialog v-model:visible="dialog" :style="{width: '450px'}" :header="titleDialog" :modal="true" class="p-fluid">
            <div class="field">
                <label for="name">Nombre</label>
                <InputText id="nombre" v-model.trim="marca.nombre" required="true" autofocus :invalid="submitted && !marca.nombre" />
                <small class="p-error" v-if="submitted && !marca.nombre">Nombre es requerido.</small>
            </div>
            
            <template #footer>
                <Button label="Cancelar" icon="pi pi-times" text @click="hideDialog"/>
                <Button :label="labelButton" security="primary" icon="pi pi-check" text @click="saveOrUpdate" />
            </template>
        </Dialog>

        <Dialog v-model:visible="deleteMarcaDialog" :style="{width: '450px'}" header="Confirmación" :modal="true">
            <div class="confirmation-content">
                <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                <span v-if="marca">Seguro/a de eliminar la marca <b>{{marca.nombre}}</b>?</span>
            </div>
            <template #footer>
                <Button label="No" icon="pi pi-times" text @click="deleteMarcaDialog = false"/>
                <Button label="Si" icon="pi pi-check" text @click="deleteMarca" />
            </template>
        </Dialog>
    </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import { FilterMatchMode } from 'primevue/api';
import { useToast } from 'primevue/usetoast';
import api from '@/services/api';

onMounted(() => {
    loadMarcas()
});

const toast = useToast();
const dt = ref();
const marcas = ref();
const marca = ref({
    id:null,
    nombre:''
});
const dialog = ref(false);
const deleteMarcaDialog = ref(false);

const filters = ref({
    'global': {value: null, matchMode: FilterMatchMode.CONTAINS},
});
const submitted = ref(false);

const openNew = () => {
    marca.value = {};
    submitted.value = false;
    dialog.value = true;
};
const hideDialog = () => {
    dialog.value = false;
    submitted.value = false;
};

//función para obtener las marcas
const loadMarcas = async () =>{
    try{
        const {data} = await api.get('/marcas');
        marcas.value = data;
    }catch(err){
        console.log("Error al obtener las marcas", err)
    }
}

const saveOrUpdate = async () => {

    submitted.value = true;

    if (!marca?.value.nombre?.trim()) {
        return;
    }

    try {
        let response;
        if (marca.value.id) {
            // Petición para actualizar la marca
            response = await api.put(`/marcas/${marca.value.id}`, marca.value);
        } else {
            // Registro de nueva marca
            response = await api.post('/marcas', marca.value);
        }
        //desestructurando status y data de la respuesta
        const { status, data } = response;
        if (status === 201) {
            marcas.value.unshift(data.marca);
            toast.add({
                severity: 'success',
                summary: 'Registro exitoso',
                detail: data.message,
                life: 3000
            });
        }
        if (status === 202) {
            const index = marcas.value.findIndex(m => m.id === data.marca.id);
            if (index !== -1) {
                marcas.value[index] = data.marca;
            }
            toast.add({
                severity: 'success',
                summary: 'Actualización exitosa',
                detail: data.message,
                life: 3000
            });
        }
        dialog.value = false;
        marca.value = {};
        submitted.value = false;
    } catch (error) {
        let message = "Error inesperado";

        if (error.response?.data?.errores) {
            const errores = error.response.data.errores;
            message = Object.values(errores)[0][0];
        } else if (error.response?.data?.message) {
            message = error.response.data.message;
        }
        toast.add({
            severity: 'error',
            summary: 'Error',
            detail: message,
            life: 4000
        });
        console.error(error);
    }
};

const editMarca = (mark) => {
    marca.value = {...mark};
    dialog.value = true;
};

const confirmDeleteMarca = (mark) => {
    marca.value = mark;
    deleteMarcaDialog.value = true;
};

const deleteMarca = async () => {

    try {
        const response = await api.delete(`/marcas/${marca.value.id}`);
        const { status, data } = response;

        if (status === 200) {
            marcas.value = marcas.value.filter(val => val.id !== marca.value.id);
            toast.add({
                severity: 'success',
                summary: 'Eliminado',
                detail: data.message,
                life: 3000
            });

        }
        deleteMarcaDialog.value = false;
        marca.value = {};

    } catch (error) {
        let message = "Error inesperado";
        if (error.response?.status === 409) {
            message = error.response.data.message;
        }
        else if (error.response?.status === 404) {
            message = error.response.data.message;
        }
        else if (error.response?.data?.message) {
            message = error.response.data.message;
        }
        toast.add({
            severity: 'error',
            summary: 'Error',
            detail: message,
            life: 4000
        });
        deleteMarcaDialog.value = false;
        console.error(error);
    }
};

//funciones computables para determinar si esta agregando o etidando un registro
const titleDialog = computed(() => {
    return marca.value.id ? "Edición de Marcas" : "Registro de Marcas"
})

const labelButton = computed(() => {
    return marca.value.id ? "Actualizar" : "Guardar"
})

</script>
````





