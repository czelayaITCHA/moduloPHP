# 8. Programación de Interfaz Pública 

## 8.1 Configurar CORS en el backend
Antes de hacer peticiones a las APIs del backend se debe configurar el archivo **config/cors.php**, para que permita peticiones del dominio del proyecto frontend, de la siguiente manera:

```php
'paths' => ['api/*'],
    'allowed_origins' => ['http://localhost:5173'],
```
En Laravel 12, el archivo config/cors.php puede no aparecer por defecto. Para solucionarlo, publica la configuración ejecutando
```php
php artisan config:publish cors
```
## 8.2 Crear la siguiente estructura del proyecto
```bash
src/
│
├── components/
│   ├── home/
│   │   ├── CategoryBar.vue
│   │   ├── ProductCard.vue
│   │   ├── ProductGrid.vue
│   │   ├── ProductSlider.vue
│   │   ├── SearchBar.vue
│   │
│   ├── layouts/
│   │   ├── Footer.vue
│   │   ├── HeroSection.vue
│   │   ├── Navbar.vue
│   │
│   └── icons/
│
├── router/
│   └── index.js
│
├── services/
│   └── api.js
│
├── stores/
│   ├── productStore.js
│   ├── categoryStore.js
│   ├── authStore.js
│   └── counter.js
│
├── views/
│   ├── HomeView.vue
│   ├── CategoryView.vue
│   └── AboutView.vue
│
├── assets/
│   └── main.css
│
├── main.js
└── App.vue
```

Explicación de la estructura anterior:


---

# Carpeta `/services`

## api.js

### Propósito
Centralizar la configuración de Axios.

### Función
- Define `baseURL`
- Permite agregar interceptores
- Evita repetir endpoints

### Principio aplicado
> Desacoplamiento entre frontend y backend.

---

# Carpeta `/stores` (Pinia)

Contiene el **estado global de la aplicación**.

---

## productStore.js

### Responsabilidad
Gestionar todo lo relacionado con productos.

### Estado que contiene
- `products` → Lista original desde API
- `activeCategory` → Categoría seleccionada
- `searchQuery` → Texto de búsqueda
- `filteredProducts` → Productos filtrados (computed)

### Acciones
- `fetchProducts()` → Llamada a API
- `setCategory()` → Cambia categoría activa

### Conceptos Aplicados

- Estado global
- Reactividad declarativa
- Estado derivado (`computed`)
- Separación lógica / presentación

---

## categoryStore.js

### Función
Manejar categorías de forma independiente.

Permite escalar a:
- CRUD de categorías
- Gestión independiente del producto

---

## authStore.js

### Preparado para
- Manejo de usuario autenticado
- Tokens
- Roles
- Control de acceso

---

# Carpeta `/components`

Contiene la **capa de presentación (UI)**.

Cada componente sigue el principio:

> **Single Responsibility Principle**

---

# components/home

Componentes específicos de la página principal.

---

## ProductCard.vue

### Función
Representar un producto individual.

### Muestra:
- Imágenes
- Nombre
- Marca
- Modelo
- Precio
- Botón de acción

### Concepto aplicado
Componente puramente visual.

No contiene lógica de negocio hasta el momento.
Solo recibe datos.

---

## ProductGrid.vue

### Función
Mostrar múltiples productos.

### Origen de datos
`store.filteredProducts`

### Concepto aplicado
Componente contenedor que conecta UI con estado global.

---

## ProductSlider.vue

### Función
Mostrar múltiples imágenes por producto.

### Conceptos aplicados
- Estado local
- Renderizado condicional
- Navegación manual

---

## CategoryBar.vue

### Función
Permitir filtrar por categoría.

### Conceptos aplicados
- Eventos
- Estado compartido
- Reactividad

---

## SearchBar.vue

### Función
Filtrar productos por nombre.

### Concepto
`v-model` conectado al estado global.

---

# components/layouts

Componentes estructurales.

---

## Navbar.vue

Barra de navegación principal.

Puede incluir:
- Logo
- Enlaces
- Carrito
- Usuario

---

## HeroSection.vue

Sección destacada principal.
Función visual y de marketing.

---

## Footer.vue

Pie de página informativo.

---

# Carpeta `/views`

En Vue Router:

> Las views representan páginas completas.

---

## HomeView.vue

Orquesta múltiples componentes:

- HeroSection
- CategoryBar
- SearchBar
- ProductGrid
- Footer

---

## CategoryView.vue

Página dedicada a una categoría específica.

---

## AboutView.vue

Página estática informativa.

---

#  `/router/index.js`

Define:

- Rutas
- Componentes asociados
- Posibles guards de autenticación

### Concepto
SPA (Single Page Application).

---

# `/assets/main.css`

Contiene:

- Configuración global
- Imports de PrimeVue
- Estilos base

---

# main.js

Punto de entrada de la aplicación.

Inicializa:

- Vue
- Pinia
- Router

---

# App.vue

Componente raíz.

Contiene:

```vue
<router-view />
```
## 8.3 Crear interceptor services/api.js
```php
import axios from 'axios'
import { useAuthStore } from '@/stores/authStore'

const api = axios.create({
  baseURL: 'http://localhost:8000/api'
})

api.interceptors.request.use((config) => {
  const authStore = useAuthStore()

  if (authStore.token) {
    config.headers.Authorization = `Bearer ${authStore.token}`
  }

  return config
})

export default api
```
## 8.4 Implementar Pinia para gestionar el estado global y crear en la carpeta store los siguientes archivos:
* productService.js
```js
import { defineStore } from "pinia";
import api from "@/services/api";

export const useProductStore = defineStore("products", {
  state: () => ({
    products: [],
    activeCategory: "all",
    searchQuery: "",
    loading: false,
  }),

  getters: {
    filteredProducts: (state) => {
      return state.products
        .filter((p) => p.activo) // solo activos
        .filter((p) => {
          const matchesCategory =
            state.activeCategory === "all" || p.categoria?.nombre === state.activeCategory;

          const matchesSearch = p.nombre.toLowerCase().includes(state.searchQuery.toLowerCase());

          return matchesCategory && matchesSearch;
        });
    },
  },

  actions: {
    async fetchProducts() {
      this.loading = true;
      try {
        const response = await api.get("/productos");
        this.products = response.data;
      } catch (error) {
        console.error(error);
      } finally {
        this.loading = false;
      }
    },

    setCategory(category) {
      this.activeCategory = category;
    },

    setSearch(query) {
      this.searchQuery = query;
    },
  },
});

```
  
* productStore.js
```js
import { defineStore } from "pinia";
import { ref, computed } from "vue";
import api from "@/services/api"; // axios instancia con el interceptor

export const useProductStore = defineStore("products", () => {
  const products = ref([]);
  const activeCategory = ref("all");
  const searchQuery = ref("");
  const loading = ref(false);

  
  const filteredProducts = computed(() => {
    return products.value
      .filter((p) => p.activo)
      .filter((p) => {
        const matchesCategory =
          activeCategory.value === "all" ||
          p.categoria?.nombre?.toLowerCase().trim() === activeCategory.value?.toLowerCase().trim();

        const matchesSearch = p.nombre?.toLowerCase().includes(searchQuery.value.toLowerCase());

        return matchesCategory && matchesSearch;
      });
  });

  const setCategory = (category) => {
    activeCategory.value = category;
  };

  const fetchProducts = async () => {
    loading.value = true;
    try {
      const { data } = await api.get("/productos");
      products.value = data;
    } catch (error) {
      console.error(error);
    } finally {
      loading.value = false;
    }
  };

  const setSearch = (query) => {
    searchQuery.value = query;
  };

  return {
    products,
    activeCategory,
    searchQuery,
    loading,
    filteredProducts,
    fetchProducts,
    setCategory,
    setSearch,
  };
});

```
* authStore.js
  
```js
import { defineStore } from 'pinia'
import api from '@/services/api'
import router from '@/router'

export const useAuthStore = defineStore('auth', {
  state: () => ({
    token: null,
    user: null
  }),

  getters: {
    isAuthenticated: (state) => !!state.token,
    isAdmin: (state) => state.user?.role === 'ADMIN'
  },

  actions: {

    async login(credentials) {
      const { data } = await api.post('/auth/login', credentials)

      this.token = data.access_token
      this.user = data.user

      if (this.user.role === 'ADMIN' || this.user.role === 'VENDEDOR') {
        router.push('/dashboard')
      } else {
        router.push('/')
      }
    },

    async register(payload) {
      const { data } = await api.post('/auth/register', payload)

      this.token = data.access_token
      this.user = data.user

      router.push('/')
    },

    async logout() {
      await api.post('/auth/logout')
      this.token = null
      this.user = null
      router.push('/')
    }
  }

})
```
## 8.5 En la carpeta components/home, programar los siguientes componentes:

* ProductCard.vue
  
````vue
<template>
  <div
    class="bg-white rounded-2xl shadow-sm hover:shadow-xl transition duration-300 overflow-hidden border border-gray-100"
  >
    <!-- IMAGEN / SLIDER -->
    <div
      class="relative h-56 bg-gray-50 flex items-center justify-center overflow-hidden"
    >
      <!-- Imagen actual -->
      <img
        v-if="currentImage"
        :src="imageUrl(currentImage)"
        class="h-full object-contain p-6 transition duration-300"
      />

      <div v-else class="text-gray-400 text-sm">Sin imagen</div>

      <!-- Flecha izquierda -->
      <button
        v-if="hasMultipleImages"
        @click="prevImage"
        class="absolute left-3 bg-black/40 text-white w-8 h-8 rounded-full flex items-center justify-center hover:bg-black/60 transition"
      >
        ‹
      </button>

      <!-- Flecha derecha -->
      <button
        v-if="hasMultipleImages"
        @click="nextImage"
        class="absolute right-3 bg-black/40 text-white w-8 h-8 rounded-full flex items-center justify-center hover:bg-black/60 transition"
      >
        ›
      </button>

      <!-- Indicador -->
      <div
        v-if="hasMultipleImages"
        class="absolute bottom-3 bg-black/50 text-white text-xs px-2 py-1 rounded"
      >
        {{ currentIndex + 1 }} / {{ product.imagenes.length }}
      </div>

      <!-- Badge Sin stock -->
      <span
        v-if="product.stock <= 0"
        class="absolute top-3 left-3 bg-red-500 text-white text-xs px-3 py-1 rounded-full shadow"
      >
        Sin stock
      </span>

      <!-- Badge Inactivo -->
      <span
        v-if="!product.activo"
        class="absolute top-3 right-3 bg-gray-700 text-white text-xs px-3 py-1 rounded-full shadow"
      >
        Inactivo
      </span>
    </div>

    <!-- INFORMACION DEL PRODUCTO -->
    <div class="p-5 flex flex-col gap-2">
      <!-- Nombre -->
      <h3 class="text-gray-900 font-semibold text-lg leading-snug line-clamp-2">
        {{ product.nombre }}
      </h3>

      <!-- Descripción -->
      <p class="text-sm text-gray-600 line-clamp-2">
        {{ product.descripcion }}
      </p>

      <!-- Detalles -->
      <div class="text-xs text-gray-500 space-y-1 mt-1">
        <div>
          Marca:
          <span class="font-medium text-gray-700">
            {{ product.marca?.nombre }}
          </span>
        </div>

        <div>
          Categoría:
          <span class="font-medium text-gray-700">
            {{ product.categoria?.nombre }}
          </span>
        </div>

        <div>
          Modelo:
          <span class="text-gray-700">
            {{ product.modelo }}
          </span>
        </div>
      </div>

      <!-- Precio y botón -->
      <div class="flex justify-between items-center mt-4">
        <span class="text-2xl font-bold text-primary">
          ${{ parseFloat(product.precio).toFixed(2) }}
        </span>

        <button
          :disabled="product.stock <= 0"
          class="px-4 py-2 bg-primary text-white rounded-xl text-sm font-medium hover:bg-primary/90 transition disabled:bg-gray-300 disabled:cursor-not-allowed"
        >
          Agregar
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

const props = defineProps({
  product: Object,
});

/* SLIDER */

const currentIndex = ref(0);

const hasMultipleImages = computed(() => props.product.imagenes?.length > 1);

const currentImage = computed(
  () => props.product.imagenes?.[currentIndex.value]?.nombre || null
);

const baseUrl = import.meta.env.VITE_API_URL || "http://localhost:8000";

const imageUrl = (imageName) => {
  return `${baseUrl}/images/productos/${imageName}`;
};

const nextImage = () => {
  if (!props.product.imagenes) return;
  currentIndex.value =
    (currentIndex.value + 1) % props.product.imagenes.length;
};

const prevImage = () => {
  if (!props.product.imagenes) return;
  currentIndex.value =
    (currentIndex.value - 1 + props.product.imagenes.length) %
    props.product.imagenes.length;
};
</script>
````
* ProductGrid.vue
  
````vue
<template>
  <section class="bg-gray-50 py-10">
    <div class="container mx-auto px-6">
      <div v-if="store.loading" class="text-center py-20">
        Cargando productos...
      </div>

      <div
        v-else
        class="grid gap-8 grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4"
      >
        <ProductCard
          v-for="product in store.filteredProducts"
          :key="product.id"
          :product="product"
        />
      </div>
    </div>
  </section>
</template>

<script setup>
import { onMounted } from "vue";
import { useProductStore } from "@/stores/productStore";
import ProductCard from "./ProductCard.vue";

const store = useProductStore();

onMounted(() => {
  store.fetchProducts();
});
</script>

````
* CategoryBar.vue

````vue
<template>
  <section class="bg-white border-b border-gray-100 sticky top-20 z-30">
    <div class="container mx-auto px-6 py-6">
      <div class="flex gap-6 overflow-x-auto md:justify-center scrollbar-hide">
        <button
          v-for="cat in categories"
          :key="cat"
          @click="productStore.setCategory(cat)"
          class="relative flex items-center gap-2 pb-2 font-medium whitespace-nowrap transition-all duration-300"
          :class="
            productStore.activeCategory === cat
              ? 'text-primary'
              : 'text-gray-500 hover:text-primary'
          "
        >
          <i :class="getIcon(cat)"></i>

          {{ cat === "all" ? "Todos" : cat }}

          <span
            v-if="productStore.activeCategory === cat"
            class="absolute bottom-0 left-0 w-full h-[3px] bg-primary rounded-full animate-slide"
          ></span>
        </button>
      </div>
    </div>
  </section>
</template>

<script setup>
import { computed } from "vue";
import { useProductStore } from "@/stores/productStore";

const productStore = useProductStore();

const categories = computed(() => {
  const unique = new Set(
    productStore.products
      .map((p) => p.categoria?.nombre?.trim())
      .filter(Boolean)
  );

  return ["all", ...unique];
});

/* Iconos tecnológicos dinámicos */
const getIcon = (name) => {
  if (name === "all") return "pi pi-th-large";

  const n = name.toLowerCase();

  if (n.includes("laptop")) return "pi pi-desktop";
  if (n.includes("monitor")) return "pi pi-window-maximize";
  if (n.includes("hardware")) return "pi pi-cog";
  if (n.includes("red")) return "pi pi-share-alt";
  if (n.includes("accesorio")) return "pi pi-headphones";
  if (n.includes("almacen")) return "pi pi-database";
  if (n.includes("componente")) return "pi pi-microchip";
  if (n.includes("impresora")) return "pi pi-print";
  if (n.includes("audio")) return "pi pi-volume-up";
  if (n.includes("gaming")) return "pi pi-star";

  return "pi pi-tag";
};
</script>

<style scoped>
.scrollbar-hide::-webkit-scrollbar {
  display: none;
}
.scrollbar-hide {
  -ms-overflow-style: none;
  scrollbar-width: none;
}

.animate-slide {
  animation: slideIn 0.3s ease forwards;
}

@keyframes slideIn {
  from {
    width: 0%;
  }
  to {
    width: 100%;
  }
}
</style>
````
* ProductSlider.vue
````vue
<template>
  <section class="container mx-auto px-6 py-16">
    <h3 class="text-3xl font-bold mb-10 text-primary">
      Nuevos Productos
    </h3>

    <Carousel
      :value="products"
      :numVisible="1"
      :numScroll="1"
      :circular="true"
      :autoplayInterval="5000"
      :responsiveOptions="responsiveOptions"
      showNavigators
      showIndicators
      class="premium-carousel"
    >
      <template #item="slotProps">
        <div class="p-4">
          <Card
            class="shadow-xl rounded-2xl overflow-hidden 
                   transition-all duration-500 ease-in-out
                   hover:scale-[1.03] hover:shadow-2xl"
          >
            <template #header>
              <img
                :src="slotProps.data.image"
                class="w-full h-56 object-cover"
              />
            </template>

            <template #title>
              <span class="text-xl font-bold">
                {{ slotProps.data.name }}
              </span>
            </template>

            <template #content>
              <p class="text-primary font-bold text-lg">
                ${{ slotProps.data.price }}
              </p>
            </template>
          </Card>
        </div>
      </template>
    </Carousel>
  </section>
</template>

<script setup>
import Carousel from 'primevue/carousel'
import Card from 'primevue/card'

const products = [
  {
    name: 'Laptop Gamer RTX',
    price: 1500,
    image: 'https://images.unsplash.com/photo-1517336714731-489689fd1ca8'
  },
  {
    name: 'Monitor 27" 144Hz',
    price: 320,
    image: 'https://images.unsplash.com/photo-1587829741301-dc798b83add3'
  },
  {
    name: 'Teclado Mecánico RGB',
    price: 120,
    image: 'https://images.unsplash.com/photo-1618384887929-16ec33fab9ef'
  },
  {
    name: 'Mouse Gamer Pro',
    price: 80,
    image: 'https://images.unsplash.com/photo-1587202372775-989c3e5b9b18'
  },
  {
    name: 'SSD NVMe 1TB',
    price: 150,
    image: 'https://images.unsplash.com/photo-1593642532973-d31b6557fa68'
  }   
]

const responsiveOptions = [
  {
    breakpoint: '1024px',
    numVisible: 2,
    numScroll: 1
  },
  {
    breakpoint: '768px',
    numVisible: 1,
    numScroll: 1
  }
]
</script>
````
* SearchBar.vue
````vue
<template>
  <section class="bg-gray-50">
    <div class="container mx-auto px-6 py-6">

      <div class="relative max-w-2xl mx-auto">

        <!-- Icono -->
        <i class="pi pi-search absolute left-4 top-1/2 -translate-y-1/2 text-gray-400"></i>

        <!-- Input -->
        <input
          v-model="search"
          type="text"
          placeholder="Buscar productos..."
          class="w-full pl-12 pr-12 py-3 rounded-xl border border-gray-200
                 focus:outline-none focus:ring-2 focus:ring-primary
                 transition-all duration-300 shadow-sm"
        />

        <!-- Botón limpiar -->
        <button
          v-if="search"
          @click="clearSearch"
          class="absolute right-4 top-1/2 -translate-y-1/2 text-gray-400 hover:text-primary transition"
        >
          <i class="pi pi-times"></i>
        </button>

      </div>

    </div>
  </section>
</template>

<script setup>
import { computed } from 'vue'
import { useProductStore } from '@/stores/productStore'

const productStore = useProductStore()

const search = computed({
  get: () => productStore.searchQuery,
  set: (value) => productStore.setSearch(value)
})

const clearSearch = () => {
  productStore.setSearch('')
}
</script>
````

## 8.6 En la carpeta components/layouts, programar los siguientes componentes:

* Navbar.vue

````vue
<template>
  <nav class="bg-primary text-white shadow-lg fixed w-full z-50">
    <div class="container mx-auto px-6 py-4 flex justify-between items-center">
      
      <router-link to="/" class="text-2xl font-bold tracking-wide flex items-center gap-2">
        💻 <span>TechStore</span>
      </router-link>

      <div class="hidden md:flex gap-8 font-medium items-center">
        <router-link to="/" class="hover:text-blue-200 transition">Inicio</router-link>
        <a href="#" class="hover:text-blue-200 transition">Categorías</a>
        <a href="#" class="hover:text-blue-200 transition">Ofertas</a>

        <div class="flex gap-4 ml-6 items-center">
          <template v-if="!authStore.isAuthenticated">
            <router-link to="/login" class="px-4 py-2 border border-white rounded-lg hover:bg-white hover:text-primary transition">
              Iniciar sesión
            </router-link>
            <router-link to="/register" class="px-4 py-2 bg-white text-primary rounded-lg font-semibold hover:bg-gray-200 transition">
              Registrarse
            </router-link>
          </template>

          <template v-else>
            <span class="text-white font-semibold border-r border-blue-400 pr-4">
              Hola, {{ authStore.user?.name }}
            </span>
            <button @click="authStore.logout()" class="text-amber-400 hover:text-amber-300 font-bold transition">
              Cerrar sesión
            </button>
          </template>
        </div>
      </div>

      <button @click="toggleMenu" class="md:hidden text-2xl focus:outline-none" aria-label="Menu">
        <i :class="isOpen ? 'pi pi-times' : 'pi pi-bars'"></i>
      </button>
    </div>

    <transition name="slide">
      <div v-if="isOpen" class="md:hidden bg-primary-dark border-t border-blue-800 px-6 py-6 space-y-6 shadow-xl">
        <router-link to="/" @click="isOpen = false" class="block hover:text-blue-200 transition">Inicio</router-link>
        <a href="#" class="block hover:text-blue-200 transition">Categorías</a>
        <a href="#" class="block hover:text-blue-200 transition">Ofertas</a>

        <div class="flex flex-col gap-3 pt-4 border-t border-blue-400">
          <template v-if="!authStore.isAuthenticated">
            <router-link 
              to="/login" 
              @click="isOpen = false"
              class="px-4 py-2 border border-white rounded-lg text-center hover:bg-white hover:text-primary transition"
            >
              Iniciar Sesión
            </router-link>
            <router-link 
              to="/register" 
              @click="isOpen = false"
              class="px-4 py-2 bg-white text-primary rounded-lg text-center font-semibold"
            >
              Registrarse
            </router-link>
          </template>
          
          <template v-else>
            <div class="flex flex-col gap-4">
              <span class="text-white font-bold text-lg text-center">
                {{ authStore.user?.name }}
              </span>
              <button 
                @click="handleLogout" 
                class="px-4 py-2 bg-red-500 text-white rounded-lg font-semibold hover:bg-red-600 transition"
              >
                Cerrar sesión
              </button>
            </div>
          </template>
        </div>
      </div>
    </transition>
  </nav>

  <div class="h-20"></div>
</template>

<script setup>
import { ref } from 'vue'
import { useAuthStore } from '@/stores/authStore'

const authStore = useAuthStore();

const isOpen = ref(false)

const toggleMenu = () => {
  isOpen.value = !isOpen.value
}
</script>

<style>
.slide-enter-active,
.slide-leave-active {
  transition: all 0.3s ease;
}

.slide-enter-from,
.slide-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}
</style>
````

* Footer.vue
````vue
<template>
  <footer class="bg-primary text-white py-8 mt-auto">
    <div class="container mx-auto px-6 text-center">
      <p class="font-semibold">© 2026 TechStore</p>
      <p class="text-blue-200 text-sm">
        Tecnología • Innovación • Confianza
      </p>
    </div>
  </footer>
</template>
````
* HeroSection.vue
````vue
  <template>
    <section class="bg-gradient-to-r from-primary to-blue-500 text-white py-20">
      <div class="container mx-auto px-6 grid md:grid-cols-2 items-center gap-10">
        
        <div>
          <h2 class="text-4xl md:text-5xl font-bold mb-6 leading-tight">
            Tecnología de última generación
          </h2>
          <p class="text-lg text-blue-100 mb-6">
            Encuentra laptops, componentes y accesorios al mejor precio.
          </p>
          <button class="bg-white text-primary px-6 py-3 rounded-xl font-semibold shadow-lg hover:scale-105 transition">
            Ver productos
          </button>
        </div>

        <div class="hidden md:block">
          <img src="https://images.unsplash.com/photo-1517336714731-489689fd1ca8"
              class="rounded-2xl shadow-2xl" />
        </div>

      </div>
    </section>
  </template>
````
## 8.9 Configurar archivo de rutas **router/index.js**
```js
import { createRouter, createWebHistory } from 'vue-router'
import HomeView from '../views/HomeView.vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: HomeView,
    },

    {
      path: '/categoria/:slug',
      name: 'categoria',
      component: () => import('@/views/CategoryView.vue')
    },
    {
      path: '/about',
      name: 'about',
      // route level code-splitting
      // this generates a separate chunk (About.[hash].js) for this route
      // which is lazy-loaded when the route is visited.
      component: () => import('../views/AboutView.vue'),
    },
  ],
})

export default router
```
## 8.10 Programar componente HomeView.vue 

````vue
<template>
  <div class="bg-gray-50 min-h-screen flex flex-col">
    <Navbar />
    <HeroSection />
    <CategoryBar />
    <SearchBar />
    <ProductSlider />
    <ProductGrid />
    <Footer />
  </div>
</template>

<script setup>
  import Navbar from "@/components/layouts/Navbar.vue";
  import Footer from "@/components/layouts/Footer.vue";
  import HeroSection from "@/components/layouts/HeroSection.vue";
  import CategoryBar from "@/components/home/CategoryBar.vue";
  import ProductSlider from "@/components/home/ProductSlider.vue";
  import ProductGrid from "@/components/home/ProductGrid.vue";
  import SearchBar from "@/components/home/SearchBar.vue";
</script>
````
## 8.11 Configurar archivo principal main.js

```js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import { createPinia } from 'pinia'
import PrimeVue from 'primevue/config'

// 1. IMPORTAR ESTILOS DE Tailwind
import './assets/main.css' 

// 2. ESTILOS DE PRIMEVUE

import 'primevue/resources/themes/saga-blue/theme.css'
import 'primevue/resources/primevue.css'
import 'primeicons/primeicons.css'

const app = createApp(App)
app.use(createPinia())
app.use(router)
app.use(PrimeVue)
app.mount('#app')
```
## 8.12 El archivo de configuración main.css debe quedar así

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```
**Recomendación:** 

Desintalar primevue 3.53.1
```bash
npm uninstall primevue@3.53.1
```
Instalar la versión 3.29.2
```bash
npm install primevue@3.29.2
```
## 8.13 Crear componente views/Login.vue

````vue
<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-100 px-4">
    <div class="w-full max-w-md bg-white shadow-xl rounded-2xl p-8">
      <h2 class="text-2xl font-bold text-center text-gray-800 mb-6">Iniciar Sesión</h2>
      <div v-if="errorMessage" class="mb-4 p-3 text-sm text-red-600 bg-red-100 rounded-lg">
        {{ errorMessage }}
      </div>

      <form @submit.prevent="sendLogin" class="space-y-5">
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1"> Email </label>
          <input
            v-model="form.email"
            type="email"
            required
            class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none"
          />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1"> Password </label>
          <input
            v-model="form.password"
            type="password"
            required
            class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none"
          />
        </div>

        <button
          type="submit"
          :disabled="loading"
          class="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 rounded-lg transition duration-300 disabled:opacity-50"
        >
          {{ loading ? "Ingresando..." : "Ingresar" }}
        </button>
      </form>

      <p class="mt-6 text-center text-sm text-gray-600">
        ¿No tienes cuenta?
        <router-link to="/register" class="text-blue-600 hover:underline font-medium">
          Regístrate
        </router-link>
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from "vue";
import { useAuthStore } from "@/stores/authStore";

const authStore = useAuthStore();
//definiendo variables de estado
const loading = ref(false);
const errorMessage = ref(null);

const form = reactive({
  email: "",
  password: "",
});

//función para enviar peticion
const sendLogin = async () => {
  loading.value = true;
  errorMessage.value = null;
  try {
    await authStore.login(form);
  } catch (err) {
    if(err.response.status===401){
        const {message} = err.response.data;
        errorMessage.value = message;
    }
    
  } finally {
    loading.value = false;
  }
};
</script>

``
