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
* 
