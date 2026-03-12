# 12. Programar Componente Gestión de Ordenes - Panel Administrativo

## 12.1 Componente Ordenes.vue
````vue
<template>
    <div class="card">
        <OrderTable :orders="orders" @ver="verOrden" @estado="cambiarEstado" />
        <OrderDetailDialog v-model:visible="showDetail" :order="selectedOrder" />
        <OrderEstadoDialog v-model:visible="showEstado" :order="selectedOrder" @actualizado="loadOrders" />
    </div>

</template>

<script setup>

import { ref, onMounted } from 'vue'
import api from '@/services/api'

import OrderTable from '@/components/admin/ordenes/OrderTable.vue'
import OrderDetailDialog from '@/components/admin/ordenes/OrderDetailDialog.vue'
import OrderEstadoDialog from '@/components/admin/ordenes/OrderEstadoDialog.vue'

const orders = ref([])

const selectedOrder = ref(null)

const showDetail = ref(false)
const showEstado = ref(false)

const loadOrders = async () => {
    const res = await api.get('/ordenes')
    orders.value = res.data
}

const verOrden = (order) => {
    selectedOrder.value = order
    showDetail.value = true
}

const cambiarEstado = (order) => {
    selectedOrder.value = order
    showEstado.value = true
}
onMounted(loadOrders)
</script>
````

## 12.2 Crear componente OrderTable.vue
````vue
<template>
    <div class="card">
        <div class="table-header">
            <h3 class="text-2xl font-bold">Gestión de Órdenes</h3>
            <div class="filters">
                <span class="p-input-icon-left">
                    <i class="pi pi-search" />
                    <InputText v-model="filtroTexto" placeholder="Buscar por orden o cliente" />
                </span>

                <Dropdown v-model="filtroEstado" :options="estados" placeholder="Filtrar estado" showClear
                    class="estado-filter" />

            </div>

        </div>

        <p class="total-registros">
            Total órdenes: {{ filteredOrders.length }}
        </p>

        <DataTable :value="filteredOrders" paginator :rows="10" :rowsPerPageOptions="[5, 10, 20, 50]"
            responsiveLayout="scroll" class="p-datatable-sm">

            <Column field="correlativo" header="Orden" />

            <Column header="Cliente">
                <template #body="slotProps">
                    {{ slotProps.data.user?.name }}
                </template>
            </Column>

            <Column header="Fecha">
                <template #body="slotProps">
                    {{ formatFecha(slotProps.data.fecha) }}
                </template>
            </Column>

            <Column header="Total">
                <template #body="slotProps">
                    {{ formatMoney(slotProps.data.total) }}
                </template>
            </Column>

            <Column header="Estado">

                <template #body="slotProps">

                    <Tag :value="slotProps.data.estado" :severity="getEstadoColor(slotProps.data.estado)" />

                </template>

            </Column>

            <Column header="Acciones">

                <template #body="slotProps">

                    <div class="acciones">

                        <Button icon="pi pi-eye" class="p-button-info p-button-sm"
                            @click="$emit('ver', slotProps.data)" />

                        <Button icon="pi pi-sync" class="p-button-warning p-button-sm"
                            @click="$emit('estado', slotProps.data)" />

                    </div>

                </template>

            </Column>

        </DataTable>

    </div>

</template>

<script setup>

import { ref, computed } from "vue"
import Tag from "primevue/tag"

const props = defineProps({
    orders: Array
})

const filtroTexto = ref("")
const filtroEstado = ref(null)

const estados = [
    "PENDIENTE",
    "PAGADA",
    "ENTREGADA",
    "CANCELADA",
    "REEMBOLSADA"
]

const filteredOrders = computed(() => {
    return props.orders.filter(order => {
        const texto = filtroTexto.value.toLowerCase()
        const matchTexto =
            order.correlativo?.toLowerCase().includes(texto) ||
            order.user?.name?.toLowerCase().includes(texto)

        const matchEstado =
            !filtroEstado.value || order.estado === filtroEstado.value

        return matchTexto && matchEstado
    })
})

const formatFecha = (fecha) => {
    if (!fecha) return ""
    const d = new Date(fecha)
    const dia = String(d.getDate()).padStart(2, '0')
    const mes = String(d.getMonth() + 1).padStart(2, '0')
    const anio = d.getFullYear()
    return `${dia}-${mes}-${anio}`
}

const formatMoney = (valor) => {
    return new Intl.NumberFormat('es-SV', {
        style: 'currency',
        currency: 'USD'
    }).format(valor)

}

const getEstadoColor = (estado) => {
    switch (estado) {
        case 'PENDIENTE': return 'warning'
        case 'PAGADA': return 'info'
        case 'ENTREGADA': return 'success'
        case 'CANCELADA': return 'danger'
        case 'REEMBOLSADA': return 'secondary'

    }

}
</script>

<style scoped>
.table-header {

    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 10px;
    margin-bottom: 10px;

}

.filters {

    display: flex;
    gap: 10px;
    flex-wrap: wrap;

}

.estado-filter {
    min-width: 180px;
}

.acciones {
    display: flex;
    gap: 8px;
}

.total-registros {
    margin-bottom: 10px;
    font-size: 0.9rem;
    color: #666;
}
</style>
````
## 12.3 Crear componente OrderEstadoDialog.vue

````vue
<template>
    <Dialog v-model:visible="dialogVisible" header="Cambiar Estado" modal :style="{ width: '400px' }">
        <div v-if="order">
            <p><b>Orden: </b> {{ order.correlativo }}</p>
            <Dropdown v-model="estado" :options="estados" placeholder="Seleccione estado" class="w-full mt-2 mb-3" />
            <Button label="Actualizar" icon="pi pi-check" class="mt-3" @click="actualizarEstado" />
        </div>
    </Dialog>
</template>

<script setup>

import { ref, watch, computed } from "vue"
import api from "@/services/api"

const props = defineProps({
    visible: Boolean,
    order: Object
})

const emit = defineEmits(['update:visible', 'actualizado'])

const dialogVisible = computed({
    get: () => props.visible,
    set: (value) => emit('update:visible', value)
})

const estado = ref(null)

const estados = [
    'PAGADA',
    'ENTREGADA',
    'CANCELADA',
    'REEMBOLSADA'
]

watch(() => props.order, (o) => {
    estado.value = o?.estado
})

const actualizarEstado = async () => {
    await api.put(`/ordenes/estado/${props.order.id}`, {
        estado: estado.value
    })
    emit('actualizado')
    emit('update:visible', false)
}

</script>
````
## 12.4 Crear el componente OrderDetailDialog.vue

````vue
<template>
    <Dialog v-model:visible="dialogVisible" header="Detalle de Orden" :style="{ width: '700px' }" modal :breakpoints="{ '960px': '75vw', '641px': '95vw' }">
        <div v-if="order" class="p-1">
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
                <div class="flex flex-col gap-1">
                    <p class="m-0"><span class="font-bold text-gray-700">Orden:</span> {{ order.correlativo }}</p>
                    <p class="m-0"><span class="font-bold text-gray-700">Cliente:</span> {{ order.user?.name }}</p>
                </div>
                <div class="flex flex-col gap-1 md:items-end">
                    <p class="m-0"><span class="font-bold text-gray-700">Fecha:</span> {{ formatFecha(order.fecha) }}</p>
                    <p class="m-0">
                        <span class="font-bold text-gray-700">Estado:</span> 
                        <Tag :value="order.estado" :severity="getSeverity(order.estado)" />
                    </p>
                </div>
            </div>

            <DataTable :value="order.items" responsiveLayout="scroll" class="p-datatable-sm">
                <Column header="Producto">
                    <template #body="slotProps">
                        {{ slotProps.data.producto?.nombre }}
                    </template>
                </Column>
                <Column field="cantidad" header="Cant." class="text-center" />
                <Column header="Precio" class="text-right">
                    <template #body="slotProps">
                        {{ formatMoney(slotProps.data.precio_unitario) }}
                    </template>
                </Column>
                <Column header="Subtotal" class="text-right">
                    <template #body="slotProps">
                        {{ formatMoney(slotProps.data.subtotal) }}
                    </template>
                </Column>
            </DataTable>

            <div class="flex justify-end mt-4">
                <div class="w-full md:w-5/12 border-t pt-3">
                    <div class="flex justify-between mb-1">
                        <span class="text-gray-600">Subtotal:</span>
                        <span>{{ formatMoney(order.subtotal) }}</span>
                    </div>
                    <div class="flex justify-between mb-1">
                        <span class="text-gray-600">Impuesto:</span>
                        <span>{{ formatMoney(order.impuesto) }}</span>
                    </div>
                    <div class="flex justify-between text-xl font-bold border-t pt-2 mt-2">
                        <span>Total:</span>
                        <span class="text-primary">{{ formatMoney(order.total) }}</span>
                    </div>
                </div>
            </div>
        </div>
    </Dialog>
</template>

<script setup>
import { computed } from "vue"
import Tag from 'primevue/tag' // para mejorar visualmente el estado

const props = defineProps({
    visible: Boolean,
    order: Object
})

const emit = defineEmits(['update:visible'])

const dialogVisible = computed({
    get: () => props.visible,
    set: (value) => emit('update:visible', value)
})

const formatFecha = (fecha) => {
    if (!fecha) return ""
    const d = new Date(fecha)
    return d.toLocaleDateString('es-SV', {
        day: '2-digit',
        month: '2-digit',
        year: 'numeric'
    })
}

const formatMoney = (valor) => {
    return new Intl.NumberFormat('es-SV', {
        style: 'currency',
        currency: 'USD'
    }).format(valor || 0)
}

// Colores para el Tag de estado
const getSeverity = (estado) => {
    switch (estado?.toUpperCase()) {
        case 'COMPLETADO': return 'success';
        case 'PENDIENTE': return 'warning';
        case 'CANCELADO': return 'danger';
        default: return 'info';
    }
}
</script>
````
