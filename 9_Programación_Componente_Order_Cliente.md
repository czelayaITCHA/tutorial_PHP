# 9. Programación del proceso de generación de ordenes en la interfaz pública

## 1. Instalar Sweetalert2 
```bash
npm install sweetalert2
```
## 2. Hacer que el backend devuelva los roles en el objeto $user, modificar el método privado responseWithToken, de AuthController, quedando de la siguiente manera:
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
## 3. Instalar plugin oficial para mantener en memoria los valores de los estados
```bash
npm install pinia-plugin-persistedstate
```
## 4. Actualizar el archivo main.js
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
## 5. Actualizar authStore
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
## 6. Crear stores/orderStore.js (pinia), para gestionar estos de la orden, 

```js
// stores/orderStore.js

import { defineStore } from "pinia";
import api from "@/services/api";

export const useOrderStore = defineStore("order", {
  state: () => ({
    items: [],
    loading: false,
  }),

  getters: {
    total: (state) => state.items.reduce((acc, item) => acc + item.precio * item.cantidad, 0),

    totalItems: (state) => state.items.reduce((acc, item) => acc + item.cantidad, 0),
  },

  actions: {
    addItem(product) {
      const existing = this.items.find((p) => p.id === product.id);

      if (existing) {
        existing.cantidad++;
      } else {
        this.items.push({
          id: product.id,
          nombre: product.nombre,
          descripcion: product.descripcion,
          modelo: product.modelo,
          marca: product.marca?.nombre ?? "",
          precio: Number(product.precio),
          cantidad: 1,
        });
      }
    },

    removeItem(id) {
      this.items = this.items.filter((p) => p.id !== id);
    },

    increment(id) {
      const item = this.items.find((p) => p.id === id);
      if (item) item.cantidad++;
    },

    decrement(id) {
      const item = this.items.find((p) => p.id === id);
      if (item && item.cantidad > 1) item.cantidad--;
    },

    clearOrder() {
      this.items = [];
    },

    async confirmOrder(payload) {
      this.loading = true;

      try {
        const response = await api.post("/ordenes", payload);
        return response; // se retorna todo el response
      } finally {
        this.loading = false;
      }
    },
    
  },
});

```
## 7. Programar boton "Agregar" de ProductCard para hacer que se agreguen productos a la orden

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
## 8. Crear componente views/OrderView.vue
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
## 9 Crear ruta
```js
// ruta para componente order - interfaz publica
    {
      path: '/order',
      name: 'Order',
      component: () => import('@/views/OrderView.vue'),
      meta: { requiresAuth: true }
    },
```

## 9. Crear ruta 
