# 9. Programación del proceso de generación de ordenes en la interfaz pública

## 9.1. Instalar Sweetalert2 
```bash
npm install sweetalert2
```
## 9.2. Hacer que el backend devuelva los roles en el objeto $user, modificar el método privado responseWithToken, de AuthController, quedando de la siguiente manera:
```php
protected function responseWithToken($token){
        return response()->json([
            'access_token' => $token,
            'token_type' => 'bearer',
            'user' => auth()->user()->load('roles'),
             'expires_in' => auth()->factory()->getTTL() * 60
        ]);
    }
```
## 9.3. Instalar plugin oficial para mantener en memoria los valores de los estados
```bash
npm install pinia-plugin-persistedstate
```
## 9.4. Actualizar el archivo main.js
```js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import { createPinia } from 'pinia'
import PrimeVue from 'primevue/config'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

// 1. IMPORTAR ESTILOS DE Tailwind
import './assets/main.css' 

// 2. ESTILOS DE PRIMEVUE
import 'primevue/resources/themes/saga-blue/theme.css'
import 'primevue/resources/primevue.css'
import 'primeicons/primeicons.css'

const app = createApp(App)
const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)


app.use(pinia)
app.use(router)
app.use(PrimeVue)
app.mount('#app')

```
## 9.5. Actualizar authStore
```js
import { defineStore } from 'pinia'
import api from '@/services/api'
import router from '@/router'

export const useAuthStore = defineStore('auth', {
  state: () => ({
    token: null,
    user: null
  }),

  // Persistencia automática
  persist: true,

  getters: {
    isAuthenticated: (state) => !!state.token,

    isAdmin: (state) => {
      return state.user?.roles?.some(role => role.name === 'ADMIN')
    },

    isVendedor: (state) => {
      return state.user?.roles?.some(role => role.name === 'VENDEDOR')
    },

    isCliente: (state) => {
      return state.user?.roles?.some(role => role.name === 'CLIENTE')
    }
  },

  actions: {

    async login(credentials) {
      try {
        const { data } = await api.post('/auth/login', credentials)

        this.token = data.access_token
        this.user = data.user

        // Redirección según rol
        if (this.isAdmin || this.isVendedor) {
          router.push('/dashboard')
        } else {
          router.push('/')
        }

      } catch (error) {
        console.error('Error en login:', error)
        //throw error
      }
    },

    async register(payload) {
      try {
        const { data } = await api.post('/auth/register', payload)

        this.token = data.access_token
        this.user = data.user

        router.push('/')
      } catch (error) {
        console.error('Error en registro:', error)
        throw error
      }
    },

    async logout() {
      try {
        if (this.token) {
          await api.post('/auth/logout')
        }
      } catch (error) {
        console.warn('Error al cerrar sesión:', error)
      } finally {
        this.$reset()
        router.push('/')
      }
    }
  }
})
```
## 9.6. Crear stores/orderStore.js (pinia), para gestionar estos de la orden 

```js
import { defineStore } from "pinia";
import api from "@/services/api";

export const useOrderStore = defineStore("order",{
    state: () => ({
        items: [],
        loading: false
    }),

    getters: {
        total: (state) => state.items.reduce((acc, item) => acc + item.precio * item.cantidad,0),

        totalItems: (state) => state.items.reduce((acc, item) => acc + item.cantidad,0),
    },

    actions: {
        //definimos las acciones o funcionalidades que se utilizaran en el componente de OrderView
        //método para agregar productos a la orden
        addItem(product){
            const existing = this.items.find((p) => p.id === product.id);
            if(existing){
                //si un producto ya esta en items, se incrementa la cantidad
                existing.cantidad++
            }else{
                this.items.push({
                    id: product.id,
                    nombre: product.nombre,
                    descripcion: product.descripcion,
                    modelo: product.modelo,
                    marca: product.marca?.nombre,
                    precio: Number(product.precio),
                    cantidad: 1,
                });
            }
        },

        removeItem(id){
            this.items = this.items.filter((p) => p.id !== id);
        },

        increment(id){
            const item = this.items.find((p) => p.id === id);
            if(item) item.cantidad++; 
        },
        decrement(id){
            const item = this.items.find((p) => p.id === id);
            if(item && item.cantidad > 1) item.cantidad--; 
        },

        clearOrder(){
            this.items = [];
        },
        //método para enviar la orden al backend
        async confirmOrder(order){
            this.loading = true; 
            try{
               const response = await api.post(`/ordenes`, order);
               return response; 
            }finally{
                this.loading = false;
            }
        },
    },
})
```
## 9.7 Agregar botón para acceder al componente OrderView en el componente Navbar.vue
* Agregar la importación de **useOrderStore**
  ```js
  import { useOrderStore } from '@/stores/orderStore';
  ```
* Crear el objeto orderStore
  ```js
  const orderStore = useOrderStore();
  ``` 
* En el template agregar link, hacerlo después del link de ofertas
```vue
 <router-link v-if="authStore.isAuthenticated && authStore.isCliente" 
   to="/order"
   class="relative ml-6 hover:text-blue-200 transition"
  >
     <i class="pi pi-shopping-cart text-2xl"></i>
     <!--Contador de productos -->
     <span v-if="orderStore.totalItems > 0"
        class="absolute -top-2 -right-3
        bg-red-500 text-white text-xs font-bold rounded-full px-2 py-0.5"
     >
      {{ orderStore.totalItems }}
    </span>
 </router-link>
```
  
## 9.8. Programar boton "Agregar" de ProductCard para hacer que se agreguen productos a la orden

Después de **import { ref, computed } from "vue";**, agregar el siguiente código:

```vue
import { useAuthStore } from "@/stores/authStore";
import { useOrderStore } from '@/stores/orderStore'

const orderStore = useOrderStore()
const authStore = useAuthStore()

const addToOrder = (product) => {
  orderStore.addItem(product)
}
```
Y el template, busca el boton Agregar y llama en el evento click la función **addToOrder()** y le pasas como parámetro **product**
```vue
<button v-if="authStore.isAuthenticated && authStore.isCliente"
   :disabled="product.stock <= 0"
   @click="addToOrder(product)"
   class="px-4 py-2 bg-primary text-white rounded-xl text-sm font-medium hover:bg-primary/90 transition disabled:bg-gray-300 disabled:cursor-not-allowed"
 >
 Agregar
</button>
```
## 9.9. Crear componente views/OrderView.vue
````vue
<template>
  <div class="container mx-auto px-6 py-10">
    <!-- Encabezado -->
    <div class="flex justify-between items-center mb-8">
      <div>
        <h1 class="text-3xl font-bold">Detalle de Orden</h1>
        <p class="text-gray-600 mt-2">
          Cliente: <span class="font-semibold">{{ authStore.user?.name }}</span>
        </p>
        <p class="text-gray-600">
          Fecha: {{ currentDate }}
        </p>
        <p class="text-gray-500 text-sm mt-1">
          Total de artículos: {{ orderStore.totalItems }}
        </p>
      </div>

      <router-link
        to="/"
        class="bg-blue-600 text-white px-6 py-2 rounded-lg shadow hover:bg-blue-700 transition"
      >
        Seguir agregando
      </router-link>
    </div>

    <!-- Tabla -->
    <div v-if="orderStore.items.length > 0" class="bg-white shadow-xl rounded-xl overflow-hidden">

      <table class="min-w-full text-sm text-left">

        <thead class="bg-gray-100 text-gray-700 uppercase text-xs">
          <tr>
            <th class="px-6 py-4">Item</th>
            <th class="px-6 py-4">Producto</th>
            <th class="px-6 py-4">Marca</th>
            <th class="px-6 py-4 text-center">Cantidad</th>
            <th class="px-6 py-4 text-right">Precio</th>
            <th class="px-6 py-4 text-right">Subtotal</th>
            <th class="px-6 py-4 text-center">Eliminar</th>
          </tr>
        </thead>

        <tbody>
          <tr
            v-for="(item, index) in orderStore.items"
            :key="item.id"
            class="border-b hover:bg-gray-50 transition"
          >
            <td class="px-6 py-4 font-medium">
              {{ index + 1 }}
            </td>

            <td class="px-6 py-4">
              <div class="font-semibold">
                {{ item.nombre }}
              </div>
              <div class="text-gray-500 text-xs">
                {{ item.descripcion }} - {{ item.modelo }}
              </div>
            </td>

            <td class="px-6 py-4">
              {{ item.marca }}
            </td>

            <!-- CANTIDAD -->
            <td class="px-6 py-4 text-center">
              <div class="flex justify-center items-center gap-2">
                <button
                    @click="orderStore.decrement(item.id)"
                    class="w-8 h-8 flex items-center justify-center bg-gray-200 rounded hover:bg-gray-300"
                    >-</button>

                    <span class="font-semibold w-8 text-center">
                    {{ item.cantidad }}
                    </span>

                    <button
                    @click="orderStore.increment(item.id)"
                    class="w-8 h-8 flex items-center justify-center bg-gray-200 rounded hover:bg-gray-300"
                    >+</button>
              </div>
            </td>

            <td class="px-6 py-4 text-right">
              {{ formatCurrency(item.precio) }}
            </td>

            <td class="px-6 py-4 text-right font-semibold">
              {{ formatCurrency(item.precio * item.cantidad) }}
            </td>

            <td class="px-6 py-4 text-center">
              <button
                @click="orderStore.removeItem(item.id)"
                class="text-red-500 hover:text-red-700 transition"
              >
                <i class="pi pi-trash"></i>
              </button>
            </td>
          </tr>
        </tbody>
      </table>

      <!-- Totales -->
      <div class="p-8 bg-gray-50">

        <div class="flex justify-end">
          <div class="w-full md:w-1/3 space-y-3 text-sm">

            <div class="flex justify-between">
              <span>Subtotal (sin IVA)</span>
              <span>{{ formatCurrency(subtotalSinIVA) }}</span>
            </div>

            <div class="flex justify-between">
              <span>IVA (13%)</span>
              <span>{{ formatCurrency(iva) }}</span>
            </div>

            <div class="flex justify-between text-xl font-bold border-t pt-3 mt-3">
              <span>Total</span>
              <span>{{ formatCurrency(orderStore.total) }}</span>
            </div>

          </div>
        </div>

        <!-- BOTONES -->
        <div class="flex justify-between mt-8">

          <button
            @click="handleClearOrder"
            class="bg-red-500 text-white px-6 py-3 rounded-xl hover:bg-red-600 transition"
          >
            Vaciar Orden
          </button>

          <button
            @click="confirm"
            :disabled="orderStore.loading"
            class="bg-green-600 text-white px-8 py-3 rounded-xl font-semibold hover:bg-green-700 transition shadow disabled:opacity-50"
          >
            <span v-if="orderStore.loading">Procesando...</span>
            <span v-else>Confirmar Orden</span>
          </button>

        </div>

      </div>

    </div>

    <div v-else class="text-gray-500 text-center py-20">
      No hay productos en la orden.
    </div>

  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useOrderStore } from '@/stores/orderStore'
import { useAuthStore } from '@/stores/authStore'
import { useRouter } from 'vue-router'
import Swal from 'sweetalert2'

const router = useRouter()
const orderStore = useOrderStore()
const authStore = useAuthStore()

const currentDate = new Date().toLocaleDateString('es-SV')

const subtotalSinIVA = computed(() => {
  return orderStore.total / 1.13
})

const iva = computed(() => {
  return orderStore.total - subtotalSinIVA.value
})

const formatCurrency = (value) => {
  return new Intl.NumberFormat('es-SV', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2
  }).format(value)
}

const handleClearOrder = async () => {
  const result = await Swal.fire({
    title: '¿Vaciar orden?',
    text: 'Se eliminarán todos los productos de la orden.',
    icon: 'question',
    showCancelButton: true,
    confirmButtonColor: '#dc2626',
    cancelButtonColor: '#6b7280',
    confirmButtonText: 'Sí, vaciar',
    cancelButtonText: 'Cancelar'
  })

  if (result.isConfirmed) {
    orderStore.clearOrder()

    Swal.fire({
      icon: 'success',
      title: 'Orden vaciada',
      timer: 1500,
      showConfirmButton: false
    })
  }
}

const confirm = async () => {

   console.log("Entró a confirmOrder");  
  if (!authStore.user) {
    router.push('/login')
    return
  }

  if (orderStore.items.length === 0) {
    Swal.fire({
      icon: 'info',
      title: 'No hay productos en la orden'
    })
    return
  }

  const result = await Swal.fire({
    title: '¿Confirmar orden?',
    html: `
      <p>Total a pagar:</p>
      <strong>${formatCurrency(orderStore.total)}</strong>
    `,
    icon: 'question',
    showCancelButton: true,
    confirmButtonColor: '#16a34a',
    cancelButtonColor: '#6b7280',
    confirmButtonText: 'Sí, confirmar',
    cancelButtonText: 'Cancelar'
  })

  if (!result.isConfirmed) return

  try {

    const payload = {
      user_id: authStore.user.id,
      fecha: new Date().toISOString().split('T')[0],
      subtotal: subtotalSinIVA.value,
      impuesto: iva.value,
      total: orderStore.total,
      items: orderStore.items.map(item => ({
        producto_id: item.id,
        cantidad: item.cantidad
      }))
    }

    Swal.fire({
      title: 'Procesando orden...',
      allowOutsideClick: false,
      didOpen: () => {
        Swal.showLoading()
      }
    })

    const response = await orderStore.confirmOrder(payload)

    Swal.close()

    // VALIDACIÓN CORRECTA
    if (response.status === 201) {

      const { message, order } = response.data

      await Swal.fire({
        icon: 'success',
        title: message,
        confirmButtonColor: '#16a34a'
      })

      orderStore.clearOrder()

      router.push({
        path: '/',
        query: { id: order.id }
      })

    }

  } catch (error) {

    Swal.close()

    Swal.fire({
      icon: 'error',
      title: 'Error al crear la orden',
      text: error.response?.data?.message || 'Error inesperado'
    })
  }
}

</script>
````
## 9.10 Crear ruta para acceder al componente
```js
// ruta para componente order - interfaz publica
    {
      path: '/order',
      name: 'Order',
      component: () => import('@/views/OrderView.vue'),
      meta: { requiresAuth: true }
    },
```
## 9.11 Programar componente MisOrdenes.vue
El propósito de este componente es mostrar las ordenes que ha realizado el cliente que se loguea en la aplicación, además de mostrar el detalle en un formulario modal, pueda CANCELAR("ANULAR") una orden que este en estado PENDIENTE.

### 9.11.1 Crear los métodos en orderStore.js
importar useAuthStore, en la sección de importaciones
```js
import { useAuthStore } from "./authStore";
```
En la sección **action** crear los métodos para **fetchMyOrders** y **cancelOrder**

```js
async fetchMyOrders() {
      const authStore = useAuthStore();

      this.loading = true;

      try {
        const { data } = await api.get("/ordenes", {
          params: {
            user_id: authStore.user.id,
          },
        });

        this.orders = data;
      } catch (error) {
        console.error("Error cargando órdenes", error);
      } finally {
        this.loading = false;
      }
    },

    async cancelOrder(id) {
      try {
        const { data } = await api.put(`/ordenes/estado/${id}`, {
          estado: "CANCELADA",
        });

        // actualizar en el store
        const order = this.orders.find((o) => o.id === id);

        if (order) {
          order.estado = "CANCELADA";
        }

        return data;
      } catch (error) {
        console.error("Error al anular la orden", error);
        throw error;
      }
    },
```
### 9.11.2 Programar el componente views/MisOrdenes.vue

````vue
<template>
  <div class="container mx-auto px-6 py-10">
    <div
      class="flex flex-col md:flex-row justify-between md:items-center mb-8 gap-4"
    >
      <div>
        <h1 class="text-3xl font-bold">Mis Órdenes</h1>
        <p class="text-gray-600">
          Cliente: <span class="font-semibold">{{ authStore.user?.name }}</span>
        </p>
      </div>

      <router-link
        to="/"
        class="bg-blue-600 text-white px-6 py-2 rounded-lg shadow hover:bg-blue-700 transition flex items-center gap-2 w-max"
      >
        <i class="pi pi-arrow-left"></i>
        Seguir comprando
      </router-link>
    </div>

    <div v-if="orderStore.loading" class="text-center py-20">
      <i class="pi pi-spin pi-spinner text-4xl text-gray-500"></i>
    </div>

    <div
      v-else-if="orderStore.orders.length > 0"
      class="bg-white shadow-xl rounded-xl overflow-x-auto"
    >
      <table class="min-w-full text-sm">
        <thead class="bg-gray-100 text-gray-700 uppercase text-xs">
          <tr>
            <th class="px-6 py-4 text-left">Orden</th>
            <th class="px-6 py-4 text-left">Fecha</th>
            <th class="px-6 py-4 text-right">Total</th>
            <th class="px-6 py-4 text-center">Estado</th>
            <th class="px-6 py-4 text-center">Acciones</th>
          </tr>
        </thead>

        <tbody>
          <tr
            v-for="order in orderStore.orders"
            :key="order.id"
            class="border-b hover:bg-gray-50 transition"
          >
            <td class="px-6 py-4 font-semibold">
              {{ order.correlativo }}
            </td>

            <td class="px-6 py-4">
              {{ formatDate(order.fecha) }}
            </td>

            <td class="px-6 py-4 text-right font-semibold">
              {{ formatCurrency(order.total) }}
            </td>

            <td class="px-6 py-4 text-center">
              <span
                class="px-3 py-1 rounded-full text-xs font-semibold"
                :class="statusClass(order.estado)"
              >
                {{ order.estado }}
              </span>
            </td>

            <td class="px-6 py-4 text-center space-x-2">
              <button
                @click="openDetail(order)"
                class="bg-gray-200 hover:bg-gray-300 px-3 py-2 rounded"
              >
                <i class="pi pi-eye"></i>
              </button>

              <button
                v-if="order.estado === 'PENDIENTE'"
                @click="cancel(order)"
                class="bg-red-500 hover:bg-red-600 text-white px-3 py-2 rounded"
              >
                <i class="pi pi-times"></i>
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div v-else class="text-center text-gray-500 py-20">
      <i class="pi pi-shopping-bag text-4xl mb-4"></i>

      <p>No tienes órdenes registradas.</p>

      <router-link
        to="/"
        class="mt-4 inline-block bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700 transition"
      >
        Explorar productos
      </router-link>
    </div>
  </div>

  <div
    v-if="selectedOrder"
    class="fixed inset-0 bg-black bg-opacity-40 flex items-center justify-center z-50 p-4"
  >
    <div
      class="bg-white rounded-xl shadow-xl max-w-4xl w-full max-h-[90vh] overflow-y-auto"
    >
      
      <div class="flex justify-between items-center border-b p-6">
        <div>
          <h2 class="text-xl font-bold">
            Detalle de Orden {{ selectedOrder.correlativo }}
          </h2>

          <p class="text-gray-500 text-sm">
            {{ formatDate(selectedOrder.fecha) }}
          </p>
        </div>

        <button
          @click="selectedOrder = null"
          class="text-gray-500 hover:text-gray-800"
        >
          <i class="pi pi-times text-xl"></i>
        </button>
      </div>

      <div class="p-6">
        <table class="min-w-full text-sm">
          <thead class="bg-gray-100 text-xs uppercase text-gray-600">
            <tr>
              <th class="px-4 py-3 text-left">Producto</th>
              <th class="px-4 py-3 text-center">Cantidad</th>
              <th class="px-4 py-3 text-right">Precio</th>
              <th class="px-4 py-3 text-right">Subtotal</th>
            </tr>
          </thead>

          <tbody>
            <tr
              v-for="item in selectedOrder.items"
              :key="item.id"
              class="border-b"
            >
              <td class="px-4 py-3">
                <div class="font-semibold">
                  {{ item.producto.nombre }}
                </div>

                <div class="text-gray-500 text-xs">
                  {{ item.producto.descripcion }} - {{ item.producto.modelo }}
                </div>
              </td>

              <td class="px-4 py-3 text-center">
                {{ item.cantidad }}
              </td>

              <td class="px-4 py-3 text-right">
                {{ formatCurrency(item.precio_unitario) }}
              </td>

              <td class="px-4 py-3 text-right font-semibold">
                {{ formatCurrency(item.subtotal) }}
              </td>
            </tr>
          </tbody>
        </table>

        <div class="flex justify-end mt-6">
          <div class="w-full md:w-1/3 space-y-2 text-sm">
            <div class="flex justify-between">
              <span>Subtotal</span>
              <span>{{ formatCurrency(selectedOrder.subtotal) }}</span>
            </div>

            <div class="flex justify-between">
              <span>IVA</span>
              <span>{{ formatCurrency(selectedOrder.impuesto) }}</span>
            </div>

            <div class="flex justify-between text-lg font-bold border-t pt-2">
              <span>Total</span>
              <span>{{ formatCurrency(selectedOrder.total) }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { useOrderStore } from "@/stores/orderStore";
import { useAuthStore } from "@/stores/authStore";
import Swal from "sweetalert2";

const orderStore = useOrderStore();
const authStore = useAuthStore();

const selectedOrder = ref(null);

onMounted(async () => {
  await orderStore.fetchMyOrders(authStore.user.id);
});

function openDetail(order) {
  selectedOrder.value = order;
}

function formatCurrency(value) {
  return new Intl.NumberFormat("es-SV", {
    style: "currency",
    currency: "USD",
    minimumFractionDigits: 2,
  }).format(value);
}

function formatDate(date) {
  return new Date(date).toLocaleDateString("es-SV");
}

function statusClass(status) {
  switch (status) {
    case "PENDIENTE":
      return "bg-yellow-100 text-yellow-700";

    case "PAGADA":
      return "bg-green-100 text-green-700";

    case "CANCELADA":
      return "bg-red-100 text-red-700";

    case "REEMBOLSADA":
      return "bg-purple-100 text-purple-700";

    default:
      return "bg-gray-100 text-gray-700";
  }
}

async function cancel(order) {
  const result = await Swal.fire({
    title: "¿Cancelar orden?",
    text: `Orden ${order.correlativo}`,
    icon: "question",
    showCancelButton: true,
    confirmButtonText: "Sí cancelar",
    cancelButtonText: "No",
  });

  if (!result.isConfirmed) return;

  try {
    const resp = await orderStore.cancelOrder(order.id);

    Swal.fire("Orden cancelada", resp.message, "success");

    await orderStore.fetchMyOrders(authStore.user.id);
  } catch (error) {
    Swal.fire(
      "Error",
      error.response?.data?.message || "No se pudo cancelar",
      "error"
    );
  }
}
</script>
````
### 9.11.3 Crear la ruta para cargar el compomnente MisOrdenes.vue en el archivo index.js

```js
{
   path: '/mis-ordenes',
   name: 'mis-ordenes',
   component: () => import('@/views/MisOrdenes.vue'),
   meta: { requiresAuth: true }
},
```
### 9.11.4 Crear el link en Navbar.vue para cargar la ruta del componente

Colocar después del enlace de ofertas
```vue
<router-link
   v-if="authStore.isAuthenticated && authStore.isCliente"
   to="/mis-ordenes"
   class="flex items-center gap-2 text-white hover:text-blue-600"
>
   <i class="pi pi-shopping-bag"></i>
   Mis Órdenes
</router-link>
```

El resultado debe ser algo similar a la siguiente imágen:

<img width="1365" height="534" alt="image" src="https://github.com/user-attachments/assets/af98abe2-983f-4d48-a92b-7d766954370a" />

Y al acceder al componente:

<img width="1339" height="350" alt="image" src="https://github.com/user-attachments/assets/1fae2a3b-a321-4302-a502-2798b1541ee5" />

Formulario modal mostrando el detalle de la orden

<img width="1241" height="480" alt="image" src="https://github.com/user-attachments/assets/952de34a-eb05-4753-aff9-c3f106f17e41" />

## 9.12 implementar guards para protección de rutas en el archivo index.js

importar useAuthStore 

```js
import { useAuthStore } from '@/stores/authStore'
```
agregar el siguiente código antes de export default 

```js
router.beforeEach((to, from, next) => {

  const authStore = useAuthStore()

  if (to.meta.requiresAuth && !authStore.isAuthenticated) {
    next('/login')
    return
  }

  if (to.meta.role) {

    const hasRole = authStore.user?.roles?.some(
      r => r.name === to.meta.role
    )

    if (!hasRole) {
      next('/')
      return
    }

  }

  if (to.meta.guest && authStore.isAuthenticated) {
    next('/')
    return
  }

  next()

})
```
