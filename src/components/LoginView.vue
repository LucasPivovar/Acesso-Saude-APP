<script setup>
import { ref } from 'vue'

const emit = defineEmits(['login'])

const username = ref('')
const password = ref('')
const rememberMe = ref(false)

const handleLogin = () => {
  emit('login', { name: 'João Silva', plan: 'Acesso Saúde Mais Premium', active: true })
}
</script>

<template>
  <div class="login-wrapper">
    <div class="login-card">
      <div class="login-header">
        <div class="logo-container">
          <img src="/logo.png" alt="Acesso Saúde" class="login-logo" />
        </div>
        <h2>Área do Cliente</h2>
        <p>Acesse seus benefícios e acompanhe sua assinatura</p>
      </div>

      <form @submit.prevent="handleLogin" class="login-form">
        <div class="form-group">
          <label class="form-label" for="username">CPF ou E-mail</label>
          <div class="input-icon-wrapper">
            <i class="ph ph-user"></i>
            <input 
              v-model="username" 
              type="text" 
              id="username" 
              placeholder="Digite seu CPF ou e-mail" 
              class="form-control with-icon" 
              required 
            />
          </div>
        </div>

        <div class="form-group">
          <label class="form-label" for="password">Senha</label>
          <div class="input-icon-wrapper">
            <i class="ph ph-lock"></i>
            <input 
              v-model="password" 
              type="password" 
              id="password" 
              placeholder="Digite sua senha" 
              class="form-control with-icon" 
              required 
            />
          </div>
        </div>

        <div class="login-options">
          <label class="checkbox-label">
            <input v-model="rememberMe" type="checkbox" />
            Lembrar-me
          </label>
          <a href="#" class="forgot-password" @click.prevent>Esqueci minha senha</a>
        </div>

        <button type="submit" class="btn btn-secondary btn-full">
          <span>Entrar</span>
          <i class="ph ph-sign-in"></i>
        </button>
      </form>
    </div>
  </div>
</template>

<style scoped>
.login-wrapper {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: radial-gradient(circle at top right, var(--primary-light), var(--bg-gray));
  padding: 16px;
  width: 100%;
}

.login-card {
  background: white;
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-lg);
  max-width: 440px;
  width: 100%;
  border: 1px solid var(--border-color);
  overflow: hidden;
}

.login-header {
  text-align: center;
  padding: 32px 32px 16px;
}

.logo-container {
  background: var(--bg-sidebar); /* Fundo verde escuro para dar contraste com a logo branca */
  padding: 16px;
  border-radius: var(--radius-md);
  margin-bottom: 24px;
  display: inline-flex;
  justify-content: center;
  align-items: center;
  width: 100%;
}

.login-logo {
  max-height: 48px;
  display: block;
}

.login-header h2 {
  font-size: 22px;
  color: var(--secondary);
  margin-bottom: 8px;
}

.login-header p {
  font-size: 14px;
  color: var(--text-gray);
}

.login-form {
  padding: 0 32px 32px;
}

.input-icon-wrapper {
  position: relative;
}

.input-icon-wrapper i {
  position: absolute;
  left: 14px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--text-light-gray);
  font-size: 18px;
}

.form-control.with-icon {
  padding-left: 44px;
}

.login-options {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
  font-size: 13px;
}

.checkbox-label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  color: var(--text-gray);
}

.forgot-password {
  color: var(--secondary);
  font-weight: 500;
}

.forgot-password:hover {
  color: var(--primary-hover);
  text-decoration: underline;
}
</style>
