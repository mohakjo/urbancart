<template>
  <div class="cart">
    <h2 class="page-title">Mon panier</h2>

    <div v-if="cartItems.length === 0" class="empty">
      Ton panier est vide.
    </div>

    <div v-else class="cart-layout">
      <div class="items">
        <div
          v-for="(item, index) in cartItems"
          :key="item.productId ? item.productId._id : index"
          class="item"
        >
          <div class="item-info">
            <span class="item-name">{{ item.productId ? item.productId.name : 'Article indisponible' }}</span>
            <span class="item-qty">x{{ item.quantity }}</span>
          </div>
          <div class="item-right">
            <span class="item-price">{{ item.productId ? item.productId.price : 'N/A' }} €</span>
            <button
              v-if="item.productId"
              @click="handleRemoveItem(item.productId._id)"
              class="btn-remove"
              title="Retirer"
            >×</button>
          </div>
        </div>
      </div>

      <div class="summary">
        <div class="summary-row total">
          <span>Total</span>
          <span>{{ total }} €</span>
        </div>

        <div v-if="showAddressForm" class="address-form">
          <p class="address-label">Adresse de livraison</p>
          <input v-model="shippingAddress.street" placeholder="Rue et numéro" class="field" />
          <input v-model="shippingAddress.city" placeholder="Ville" class="field" />
          <input v-model="shippingAddress.postalCode" placeholder="Code postal" class="field" />
        </div>

        <div v-if="error" class="error-msg">{{ error }}</div>

        <button
          v-if="!showAddressForm"
          @click="showAddressForm = true"
          class="btn-checkout"
        >
          Passer la commande
        </button>
        <button
          v-else
          @click="checkout"
          class="btn-checkout"
          :disabled="processing || !isAddressValid"
        >
          {{ processing ? 'Traitement en cours...' : 'Confirmer la commande' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'

export default {
  name: 'ShoppingCart',
  props: {
    cartItems: {
      type: Array,
      required: true,
      default: () => []
    }
  },
  emits: ['remove-from-cart', 'cart-cleared'],
  setup(props, { emit }) {
    const router = useRouter()
    const processing = ref(false)
    const error = ref('')
    const showAddressForm = ref(false)
    const shippingAddress = ref({ street: '', city: '', postalCode: '' })

    const total = computed(() =>
      props.cartItems.reduce((sum, item) =>
        sum + (item.productId ? item.productId.price * item.quantity : 0), 0
      ).toFixed(2)
    )

    const isAddressValid = computed(() =>
      shippingAddress.value.street.length > 0 &&
      shippingAddress.value.city.length > 0 &&
      shippingAddress.value.postalCode.length > 0
    )

    const handleRemoveItem = (productId) => emit('remove-from-cart', productId)

    const checkout = async () => {
      if (!isAddressValid.value) {
        error.value = 'Merci de renseigner tous les champs de livraison.'
        return
      }
      try {
        processing.value = true
        error.value = ''
        const token = localStorage.getItem('token')
        const orderData = {
          products: props.cartItems
            .filter(item => item.productId)
            .map(item => ({ productId: item.productId._id, quantity: item.quantity })),
          shippingAddress: shippingAddress.value
        }
        await axios.post('/api/orders', orderData, {
          headers: { Authorization: `Bearer ${token}` }
        })
        emit('cart-cleared')
        router.push('/orders')
      } catch (err) {
        console.error(err)
        error.value = 'Une erreur est survenue. Veuillez réessayer.'
      } finally {
        processing.value = false
      }
    }

    return {
      total, checkout, processing, error,
      showAddressForm, shippingAddress, isAddressValid, handleRemoveItem
    }
  }
}
</script>

<style scoped>
.cart {
  width: 100%;
  max-width: 720px;
}

.page-title {
  font-size: 1.3rem;
  font-weight: 600;
  letter-spacing: -0.02em;
  margin-bottom: 1.5rem;
}

.empty {
  text-align: center;
  padding: 3rem;
  color: #aaa;
  font-size: 0.875rem;
  background: #fff;
  border: 1px solid #e5e5e3;
  border-radius: 8px;
}

.cart-layout {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.items {
  background: #fff;
  border: 1px solid #e5e5e3;
  border-radius: 8px;
  overflow: hidden;
}

.item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 1rem;
  border-bottom: 1px solid #f0f0ee;
}

.item:last-child {
  border-bottom: none;
}

.item-info {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.item-name {
  font-size: 0.875rem;
  font-weight: 500;
}

.item-qty {
  font-size: 0.75rem;
  color: #aaa;
}

.item-right {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.item-price {
  font-size: 0.875rem;
  font-weight: 500;
}

.btn-remove {
  background: none;
  border: 1px solid #e5e5e3;
  color: #aaa;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
  line-height: 1;
  transition: background 0.15s, color 0.15s;
}

.btn-remove:hover {
  background: #fef2f2;
  color: #e53e3e;
  border-color: #fecaca;
}

.summary {
  background: #fff;
  border: 1px solid #e5e5e3;
  border-radius: 8px;
  padding: 1.25rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.summary-row {
  display: flex;
  justify-content: space-between;
  font-size: 0.875rem;
}

.summary-row.total {
  font-weight: 600;
  font-size: 1rem;
  padding-top: 0.5rem;
  border-top: 1px solid #f0f0ee;
}

.address-form {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.address-label {
  font-size: 0.8rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: #888;
}

.field {
  width: 100%;
  padding: 0.6rem 0.75rem;
  border: 1px solid #e5e5e3;
  border-radius: 6px;
  font-family: inherit;
  font-size: 0.875rem;
  background: #fafafa;
  color: #1a1a1a;
  outline: none;
  transition: border-color 0.15s;
}

.field:focus {
  border-color: #2563eb;
  background: #fff;
}

.field::placeholder {
  color: #bbb;
}

.error-msg {
  font-size: 0.8rem;
  color: #e53e3e;
  background: #fef2f2;
  border: 1px solid #fecaca;
  border-radius: 6px;
  padding: 0.6rem 0.75rem;
}

.btn-checkout {
  width: 100%;
  padding: 0.75rem;
  background: #2563eb;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-family: inherit;
  font-size: 0.875rem;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.15s;
}

.btn-checkout:hover:not(:disabled) {
  background: #1d4ed8;
}

.btn-checkout:disabled {
  background: #d1d5db;
  cursor: not-allowed;
}
</style>