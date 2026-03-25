<template>
  <div class="orders">
    <h2 class="page-title">Historique des commandes</h2>

    <div v-if="loading" class="state-block">Chargement...</div>
    <div v-else-if="error" class="state-block error-block">{{ error }}</div>
    <div v-else-if="orders.length === 0" class="state-block">
      Tu n'as pas encore passé de commande.
    </div>

    <div v-else class="list">
      <div v-for="order in orders" :key="order._id" class="order-card">
        <div class="order-head">
          <span class="order-id">Commande #{{ order._id.slice(-6).toUpperCase() }}</span>
          <span :class="['badge', order.status]">{{ translateStatus(order.status) }}</span>
        </div>

        <div class="order-products">
          <div v-for="product in order.products" :key="product._id" class="order-product">
            <span class="product-name">{{ product.name }}</span>
            <span class="product-detail">{{ product.price }} € × {{ product.quantity }}</span>
          </div>
        </div>

        <div class="order-address">
          <span class="address-label">Livraison</span>
          <span class="address-text">
            {{ order.shippingAddress.street }}, {{ order.shippingAddress.postalCode }} {{ order.shippingAddress.city }}
          </span>
        </div>

        <div class="order-foot">
          <span class="order-date">{{ formatDate(order.createdAt) }}</span>
          <span class="order-total">{{ order.totalAmount }} €</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import { orderService } from '../services/orderService'

export default {
  name: 'OrderHistory',
  setup() {
    const orders = ref([])
    const loading = ref(true)
    const error = ref(null)

    const translateStatus = (status) => {
      const map = {
        pending: 'En attente',
        confirmed: 'Confirmée',
        shipped: 'Expédiée',
        delivered: 'Livrée',
        cancelled: 'Annulée'
      }
      return map[status] || status
    }

    const formatDate = (dateString) =>
      new Date(dateString).toLocaleDateString('fr-FR', {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      })

    const fetchOrders = async () => {
      try {
        loading.value = true
        orders.value = await orderService.getOrders()
      } catch (err) {
        error.value = 'Impossible de charger les commandes.'
        console.error(err)
      } finally {
        loading.value = false
      }
    }

    onMounted(fetchOrders)

    return { orders, loading, error, translateStatus, formatDate }
  }
}
</script>

<style scoped>
.orders {
  width: 100%;
  max-width: 680px;
}

.page-title {
  font-size: 1.3rem;
  font-weight: 600;
  letter-spacing: -0.02em;
  margin-bottom: 1.5rem;
}

.state-block {
  text-align: center;
  padding: 2.5rem;
  background: #fff;
  border: 1px solid #e5e5e3;
  border-radius: 8px;
  font-size: 0.875rem;
  color: #aaa;
}

.error-block {
  color: #e53e3e;
  background: #fef2f2;
  border-color: #fecaca;
}

.list {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.order-card {
  background: #fff;
  border: 1px solid #e5e5e3;
  border-radius: 8px;
  overflow: hidden;
}

.order-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.875rem 1rem;
  border-bottom: 1px solid #f0f0ee;
}

.order-id {
  font-size: 0.8rem;
  font-weight: 600;
  letter-spacing: 0.05em;
}

.badge {
  font-size: 0.72rem;
  font-weight: 500;
  padding: 0.25rem 0.6rem;
  border-radius: 999px;
}

.badge.pending    { background: #fef9c3; color: #854d0e; }
.badge.confirmed  { background: #dcfce7; color: #166534; }
.badge.shipped    { background: #dbeafe; color: #1e40af; }
.badge.delivered  { background: #bbf7d0; color: #14532d; }
.badge.cancelled  { background: #fee2e2; color: #991b1b; }

.order-products {
  padding: 0.5rem 1rem;
  border-bottom: 1px solid #f0f0ee;
}

.order-product {
  display: flex;
  justify-content: space-between;
  padding: 0.4rem 0;
  font-size: 0.8rem;
  border-bottom: 1px solid #f7f7f5;
}

.order-product:last-child {
  border-bottom: none;
}

.product-name {
  color: #1a1a1a;
}

.product-detail {
  color: #888;
}

.order-address {
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
  padding: 0.75rem 1rem;
  border-bottom: 1px solid #f0f0ee;
}

.address-label {
  font-size: 0.7rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: #aaa;
}

.address-text {
  font-size: 0.8rem;
  color: #555;
}

.order-foot {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 1rem;
}

.order-date {
  font-size: 0.75rem;
  color: #aaa;
}

.order-total {
  font-size: 0.9rem;
  font-weight: 600;
}
</style>