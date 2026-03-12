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
