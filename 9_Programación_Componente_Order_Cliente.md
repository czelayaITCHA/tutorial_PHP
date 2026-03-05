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
import { defineStore } from "pinia";
import api from "@/services/api";
import { useAuthStore } from "./authStore";
import router from "@/router";
import Swal from "sweetalert2";

export const useOrderStore = defineStore("order", {
  state: () => ({
    items: [],
  }),

  getters: {
    totalItems: (state) => state.items.reduce((sum, item) => sum + item.quantity, 0),

    totalPrice: (state) => state.items.reduce((sum, item) => sum + item.price * item.quantity, 0),
  },

  actions: {
    addProduct(product) {
      const existing = this.items.find((p) => p.id === product.id);

      if (existing) {
        existing.quantity++;
      } else {
        this.items.push({ ...product, quantity: 1 });
      }
    },

    updateQuantity(productId, quantity) {
      const item = this.items.find((p) => p.id === productId);
      if (item && quantity > 0) {
        item.quantity = quantity;
      }
    },

    removeProduct(productId) {
      this.items = this.items.filter((p) => p.id !== productId);
    },

    clearOrder() {
      this.items = [];
    },

    async checkout() {
      const authStore = useAuthStore();

      if (!authStore.isAuthenticated) {
        router.push("/login");
        return;
      }

      await api.post("/orders", {
        products: this.items.map((item) => ({
          product_id: item.id,
          quantity: item.quantity,
        })),
      });

      this.clearOrder();
      Swal.fire({
        title: "¡Éxito!",
        text: "Orden creada correctamente.",
        icon: "success",
      });
    },
  },
});

```
## 3. 
