<template>
  <div class="app">
    <header class="header">
      <div class="logo">
        <span class="logo-icon">◈</span>
        <span class="logo-name">UrbanCart</span>
      </div>

      <nav v-if="isAuthenticated" class="nav">
        <router-link to="/" class="nav-link">Boutique</router-link>
        <router-link to="/cart" class="nav-link">
          Panier
          <span v-if="cartCount > 0" class="count">{{ cartCount }}</span>
        </router-link>
        <router-link to="/orders" class="nav-link">Historique</router-link>
        <button @click="logout" class="btn-logout">Quitter</button>
      </nav>
    </header>

    <main class="main">
      <div v-if="!isAuthenticated" class="login-wrap">
        <h2 class="login-title">Accès à la boutique</h2>
        <p class="login-sub">Crée un compte ou connecte-toi pour commencer.</p>
        <AuthTest @login-success="handleLoginSuccess" />
      </div>

      <div v-else class="view-wrap">
        <router-view
          :key="$route.fullPath"
          :products="products"
          :cart-items="cartItems"
          @add-to-cart="addToCart"
          @remove-from-cart="removeFromCart"
          @cart-cleared="handleCartCleared"
        />
      </div>
    </main>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'
import AuthTest from './components/AuthTest.vue'

export default {
  name: 'App',
  components: { AuthTest },
  setup() {
    const router = useRouter()
    const products = ref([])
    const cartItems = ref([])
    const cartCount = ref(0)
    const isAuthenticated = ref(false)

    const getAuthData = () => ({
      token: localStorage.getItem('token'),
      userId: localStorage.getItem('userId')
    })

    const loadProducts = async () => {
      try {
        const { token } = getAuthData()
        if (!token) return
        const res = await axios.get('/api/products', {
          headers: { Authorization: `Bearer ${token}` }
        })
        products.value = res.data
      } catch (e) {
        console.error(e)
      }
    }

    const loadCart = async () => {
      try {
        const { token, userId } = getAuthData()
        if (!token || !userId) return
        const res = await axios.get('/api/cart', {
          headers: { Authorization: `Bearer ${token}`, userId }
        })
        cartItems.value = res.data.items || []
        cartCount.value = cartItems.value.length
      } catch (e) {
        console.error(e)
      }
    }

    const addToCart = async (product) => {
      try {
        const { token, userId } = getAuthData()
        if (!token || !userId) return
        await axios.post('/api/cart/add', { userId, productId: product._id }, {
          headers: { Authorization: `Bearer ${token}`, userId, 'Content-Type': 'application/json' }
        })
        await loadCart()
      } catch (e) {
        console.error(e)
      }
    }

    const removeFromCart = async (productId) => {
      try {
        const { token, userId } = getAuthData()
        if (!token || !userId) return
        await axios.delete(`/api/cart/remove/${productId}`, {
          headers: { Authorization: `Bearer ${token}`, userId }
        })
        await loadCart()
      } catch (e) {
        console.error(e)
      }
    }

    const handleCartCleared = async () => {
      cartItems.value = []
      cartCount.value = 0
      await loadCart()
    }

    const handleLoginSuccess = async () => {
      isAuthenticated.value = true
      await loadProducts()
      await loadCart()
      router.push('/')
    }

    const logout = () => {
      localStorage.removeItem('token')
      localStorage.removeItem('userId')
      isAuthenticated.value = false
      products.value = []
      cartItems.value = []
      cartCount.value = 0
      router.push('/')
    }

    const checkAuth = async () => {
      const { token, userId } = getAuthData()
      if (token && userId) {
        isAuthenticated.value = true
        await loadProducts()
        await loadCart()
      }
    }

    router.beforeEach((to, from, next) => {
      if (to.path === '/' && isAuthenticated.value) loadProducts()
      next()
    })

    onMounted(checkAuth)

    return {
      products, cartItems, cartCount, isAuthenticated,
      handleLoginSuccess, logout, addToCart, removeFromCart, handleCartCleared
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Geist:wght@400;500;600&display=swap');

*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Geist', sans-serif;
  background: #f7f7f5;
  color: #1a1a1a;
  font-size: 14px;
}

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 2rem;
  height: 56px;
  border-bottom: 1px solid #e5e5e3;
  background: #fff;
  position: sticky;
  top: 0;
  z-index: 50;
}

.logo {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.logo-icon {
  font-size: 1.1rem;
  color: #2563eb;
}

.logo-name {
  font-size: 0.95rem;
  font-weight: 600;
  letter-spacing: -0.01em;
}

.nav {
  display: flex;
  align-items: center;
  gap: 0.25rem;
}

.nav-link {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  color: #555;
  text-decoration: none;
  font-size: 0.875rem;
  padding: 0.4rem 0.75rem;
  border-radius: 6px;
  transition: background 0.15s, color 0.15s;
}

.nav-link:hover {
  background: #f3f3f1;
  color: #1a1a1a;
}

.nav-link.router-link-active {
  background: #eff6ff;
  color: #2563eb;
  font-weight: 500;
}

.count {
  background: #2563eb;
  color: #fff;
  font-size: 0.7rem;
  font-weight: 600;
  padding: 0 5px;
  height: 16px;
  border-radius: 999px;
  display: inline-flex;
  align-items: center;
}

.btn-logout {
  margin-left: 0.5rem;
  background: none;
  border: 1px solid #e5e5e3;
  color: #888;
  font-family: inherit;
  font-size: 0.875rem;
  padding: 0.4rem 0.75rem;
  border-radius: 6px;
  cursor: pointer;
  transition: background 0.15s, color 0.15s, border-color 0.15s;
}

.btn-logout:hover {
  background: #fef2f2;
  color: #e53e3e;
  border-color: #fecaca;
}

.main {
  flex: 1;
  max-width: 1100px;
  margin: 0 auto;
  padding: 3rem 2rem;
  width: 100%;
}

.login-wrap {
  max-width: 400px;
  margin: 4rem auto 0;
}

.login-title {
  font-size: 1.4rem;
  font-weight: 600;
  letter-spacing: -0.02em;
  margin-bottom: 0.4rem;
}

.login-sub {
  color: #888;
  font-size: 0.875rem;
  margin-bottom: 2rem;
}

.view-wrap {
  animation: in 0.2s ease;
}

@keyframes in {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}

@media (max-width: 640px) {
  .header { padding: 0 1rem; }
  .main { padding: 2rem 1rem; }
}
</style>