<template>
  <div class="auth">
    <div v-if="!userProfile">
      <div class="form-block">
        <h3 class="form-title">Créer un compte</h3>
        <input v-model="registerForm.email" type="email" placeholder="Adresse e-mail" class="field" />
        <input v-model="registerForm.password" type="password" placeholder="Mot de passe" class="field" />
        <button @click="register" class="btn-primary">S'inscrire</button>
      </div>

      <div class="divider"><span>ou</span></div>

      <div class="form-block">
        <h3 class="form-title">Se connecter</h3>
        <input v-model="loginForm.email" type="email" placeholder="Adresse e-mail" class="field" />
        <input v-model="loginForm.password" type="password" placeholder="Mot de passe" class="field" />
        <button @click="login" class="btn-primary">Connexion</button>
      </div>
    </div>

    <div v-if="userProfile" class="profile-block">
      <p class="profile-email">{{ userProfile.email }}</p>
      <button @click="logout" class="btn-secondary">Se déconnecter</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'AuthTest',
  emits: ['login-success', 'logout'],
  data() {
    return {
      registerForm: { email: '', password: '' },
      loginForm: { email: '', password: '' },
      userProfile: null
    }
  },
  methods: {
    async register() {
      try {
        const res = await axios.post('/api/auth/register', this.registerForm)
        localStorage.setItem('token', res.data.token)
        localStorage.setItem('userId', res.data.userId)
        await this.getProfile()
        this.$emit('login-success', res.data.userId)
      } catch (err) {
        alert(err.response?.data?.message || 'Inscription impossible, réessaie.')
      }
    },
    async login() {
      try {
        const res = await axios.post('/api/auth/login', this.loginForm)
        localStorage.setItem('token', res.data.token)
        localStorage.setItem('userId', res.data.userId)
        await this.getProfile()
        this.$emit('login-success', res.data.userId)
      } catch (err) {
        alert(err.response?.data?.message || 'Identifiants incorrects.')
      }
    },
    async getProfile() {
      try {
        const token = localStorage.getItem('token')
        if (!token) return
        const res = await axios.get('/api/auth/profile', {
          headers: { Authorization: `Bearer ${token}` }
        })
        this.userProfile = res.data
        if (this.userProfile) {
          localStorage.setItem('userId', this.userProfile._id)
          this.$emit('login-success', this.userProfile._id)
        }
      } catch (err) {
        this.handleLogout()
      }
    },
    handleLogout() {
      localStorage.removeItem('token')
      localStorage.removeItem('userId')
      this.userProfile = null
      this.loginForm = { email: '', password: '' }
      this.registerForm = { email: '', password: '' }
      this.$emit('logout')
    },
    logout() {
      this.handleLogout()
      window.location.reload()
    }
  },
  mounted() {
    this.getProfile()
  }
}
</script>

<style scoped>
.auth {
  width: 100%;
}

.form-block {
  display: flex;
  flex-direction: column;
  gap: 0.6rem;
}

.form-title {
  font-size: 0.8rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: #888;
  margin-bottom: 0.25rem;
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
  transition: border-color 0.15s;
  outline: none;
}

.field:focus {
  border-color: #2563eb;
  background: #fff;
}

.field::placeholder {
  color: #bbb;
}

.btn-primary {
  width: 100%;
  padding: 0.65rem;
  background: #2563eb;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-family: inherit;
  font-size: 0.875rem;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.15s;
  margin-top: 0.25rem;
}

.btn-primary:hover {
  background: #1d4ed8;
}

.divider {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  margin: 1.25rem 0;
  color: #ccc;
  font-size: 0.8rem;
}

.divider::before,
.divider::after {
  content: '';
  flex: 1;
  height: 1px;
  background: #e5e5e3;
}

.profile-block {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.profile-email {
  font-size: 0.875rem;
  color: #555;
}

.btn-secondary {
  width: 100%;
  padding: 0.65rem;
  background: none;
  color: #555;
  border: 1px solid #e5e5e3;
  border-radius: 6px;
  font-family: inherit;
  font-size: 0.875rem;
  cursor: pointer;
  transition: background 0.15s, color 0.15s;
}

.btn-secondary:hover {
  background: #fef2f2;
  color: #e53e3e;
  border-color: #fecaca;
}
</style>