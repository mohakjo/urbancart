<template>
  <div class="product-list">
    <h2 class="page-title">Boutique</h2>
    <div v-if="products.length === 0" class="empty">Aucun article disponible pour le moment.</div>
    <div v-else class="grid">
      <div v-for="product in products" :key="product._id" class="card">
        <div class="card-body">
          <h3 class="card-name">{{ product.name }}</h3>
          <p class="card-desc">{{ product.description }}</p>
        </div>
        <div class="card-footer">
          <span class="card-price">{{ product.price }} €</span>
          <button
            @click="handleAddToCart(product)"
            class="btn-add"
            :disabled="isAddingToCart"
          >
            {{ isAddingToCart ? 'Ajout...' : '+ Panier' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  name: 'ProductList',
  props: {
    products: {
      type: Array,
      required: true
    }
  },
  emits: ['add-to-cart'],
  setup(props, { emit }) {
    const isAddingToCart = ref(false)

    const handleAddToCart = async (product) => {
      try {
        isAddingToCart.value = true
        emit('add-to-cart', product)
      } catch (e) {
        console.error(e)
      } finally {
        isAddingToCart.value = false
      }
    }

    return { handleAddToCart, isAddingToCart }
  }
}
</script>

<style scoped>
.product-list {
  width: 100%;
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

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  gap: 1rem;
}

.card {
  background: #fff;
  border: 1px solid #e5e5e3;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  transition: box-shadow 0.15s;
}

.card:hover {
  box-shadow: 0 4px 16px rgba(0,0,0,0.07);
}

.card-body {
  padding: 1rem 1rem 0.75rem;
}

.card-name {
  font-size: 0.9rem;
  font-weight: 600;
  margin-bottom: 0.35rem;
}

.card-desc {
  font-size: 0.8rem;
  color: #888;
  line-height: 1.5;
}

.card-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.75rem 1rem;
  border-top: 1px solid #f0f0ee;
}

.card-price {
  font-size: 0.95rem;
  font-weight: 600;
  color: #1a1a1a;
}

.btn-add {
  background: #2563eb;
  color: #fff;
  border: none;
  padding: 0.4rem 0.8rem;
  border-radius: 6px;
  font-family: inherit;
  font-size: 0.8rem;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.15s;
}

.btn-add:hover:not(:disabled) {
  background: #1d4ed8;
}

.btn-add:disabled {
  background: #d1d5db;
  cursor: not-allowed;
}
</style>