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

## 🔹 AboutView.vue

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
import { ref } from 'vue'
import api from '@/services/api'

export const useAuthStore = defineStore('auth', () => {

  const user = ref(null)
  const token = ref(null)

  const login = async (credentials) => {
    const { data } = await api.post('/login', credentials)
    user.value = data.user
    token.value = data.token
  }

  const logout = () => {
    user.value = null
    token.value = null
  }

  return {
    user,
    token,
    login,
    logout
  }
})
```
## 8.5 En la carpeta components/home, programar los siguientes componentes:
* ProductCard.vue
  ## 🧩 ProductCard.vue

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
* 
