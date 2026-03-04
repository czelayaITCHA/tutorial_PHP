# 9. Programación del proceso de generación de ordenes en la interfaz pública

## 1. Instalar Sweetalert2 
```bash
npm install sweetalert2
```
## 2. Crear stores/orderStore.js (pinia), para gestionar estos de la orden, 

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
