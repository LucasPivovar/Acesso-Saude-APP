<script setup>
import { ref } from 'vue'
import LoginView from './components/LoginView.vue'
import DashboardView from './components/DashboardView.vue'

// Estado de Login simulado
const isLoggedIn = ref(false)
const currentUser = ref({
  name: 'João Silva',
  plan: 'Acesso Saúde Mais Premium',
  active: true
})

// Abas de navegação: 'home', 'perfil', 'financeiro', 'configuracoes'
const currentTab = ref('home')

// Modo de layout de visualização: 'desktop' ou 'pwa'
const layoutMode = ref('desktop')

// Dropdown de perfil no Desktop
const showDropdown = ref(false)

// Modal Global de Desenvolvimento
const showDevModal = ref(false)
const devModalData = ref({ title: '', message: '' })

const handleLogin = (userData) => {
  currentUser.value = userData
  isLoggedIn.value = true
  currentTab.value = 'home'
}

const handleLogout = () => {
  isLoggedIn.value = false
  currentTab.value = 'home'
  showDropdown.value = false
}

const handleUpdateUser = (updatedData) => {
  currentUser.value = updatedData
}

const navigateTo = (tab) => {
  currentTab.value = tab
  showDropdown.value = false
}

const openDevModal = (data) => {
  devModalData.value = data
  showDevModal.value = true
}
</script>

<template>
  <!-- Seletor PWA / Desktop flutuante -->
  <div class="mode-selector">
    <button 
      :class="['mode-btn', { active: layoutMode === 'desktop' }]" 
      @click="layoutMode = 'desktop'"
    >
      <i class="ph ph-monitor"></i>
      <span>Desktop</span>
    </button>
    <button 
      :class="['mode-btn', { active: layoutMode === 'pwa' }]" 
      @click="layoutMode = 'pwa'"
    >
      <i class="ph ph-device-mobile"></i>
      <span>PWA (Mobile)</span>
    </button>
  </div>

  <!-- Estado Deslogado: Tela de Login -->
  <LoginView v-if="!isLoggedIn" @login="handleLogin" />

  <!-- Estado Logado: Layouts Adaptativos -->
  <div v-else class="app-layout">
    
    <!-- VERSÃO DESKTOP -->
    <div v-if="layoutMode === 'desktop'" class="desktop-layout">
      <!-- Navbar Superior Desktop (Fundo escuro sem nav central) -->
      <header class="topbar-desktop">
        <div class="topbar-container">
          <div class="logo-area" @click="navigateTo('home')" style="cursor: pointer;">
            <img src="/logo.png" alt="Acesso Saúde Mais" class="brand-logo" />
          </div>

          <!-- Avatar com Dropdown de Opções (Navegação apenas pelo menu) -->
          <div class="user-profile-wrapper">
            <div class="user-profile-area" @click="showDropdown = !showDropdown">
              <div class="avatar-circle">
                {{ currentUser.name.split(' ').map(n => n[0]).join('') }}
              </div>
              <span class="profile-name">{{ currentUser.name }}</span>
              <i class="ph ph-caret-down"></i>
            </div>

            <!-- Menu Dropdown -->
            <div v-if="showDropdown" class="dropdown-menu">
              <div class="dropdown-header">
                <strong>{{ currentUser.name }}</strong>
                <span>{{ currentUser.plan }}</span>
              </div>
              <div class="dropdown-divider"></div>
              
              <button class="dropdown-item" @click="navigateTo('home')">
                <i class="ph ph-squares-four"></i> Visão Geral
              </button>
              <button class="dropdown-item" @click="navigateTo('perfil')">
                <i class="ph ph-user"></i> Meu Perfil
              </button>
              <button class="dropdown-item" @click="navigateTo('financeiro')">
                <i class="ph ph-credit-card"></i> Financeiro
              </button>
              <button class="dropdown-item" @click="navigateTo('configuracoes')">
                <i class="ph ph-gear"></i> Configurações
              </button>
              <div class="dropdown-divider"></div>
              <button class="dropdown-item text-red" @click="handleLogout">
                <i class="ph ph-sign-out"></i> Sair da Conta
              </button>
            </div>
          </div>
        </div>
      </header>

      <!-- Conteúdo Desktop -->
      <main class="desktop-main container">
        <DashboardView 
          :user="currentUser" 
          :layoutMode="'desktop'"
          :currentTab="currentTab"
          @updateUser="handleUpdateUser"
          @logout="handleLogout"
          @triggerDevModal="openDevModal"
        />
      </main>
    </div>

    <!-- VERSÃO PWA MOBILE (SIMULADOR) -->
    <div v-else class="pwa-layout">
      <div class="pwa-simulator">
        <!-- Status Bar Fake -->
        <div class="pwa-status-bar">
          <span class="fake-time">14:04</span>
          <div class="fake-icons">
            <i class="ph ph-wifi-high"></i>
            <i class="ph ph-battery-full"></i>
          </div>
        </div>

        <!-- Cabeçalho PWA -->
        <header class="pwa-header">
          <img src="/logo.png" alt="Acesso Saúde" class="pwa-logo" />
        </header>

        <!-- Corpo do PWA -->
        <main class="main-content">
          <DashboardView 
            :user="currentUser" 
            :layoutMode="'pwa'"
            :currentTab="currentTab"
            @updateUser="handleUpdateUser"
            @logout="handleLogout"
            @triggerDevModal="openDevModal"
          />
        </main>

        <!-- Menu de Navegação Inferior PWA -->
        <nav class="pwa-bottom-nav">
          <button 
            :class="['pwa-nav-item', { active: currentTab === 'home' }]"
            @click="navigateTo('home')"
          >
            <i class="ph ph-squares-four"></i>
            <span>Início</span>
          </button>
          <button 
            :class="['pwa-nav-item', { active: currentTab === 'perfil' }]"
            @click="navigateTo('perfil')"
          >
            <i class="ph ph-user"></i>
            <span>Perfil</span>
          </button>
          <button 
            :class="['pwa-nav-item', { active: currentTab === 'financeiro' }]"
            @click="navigateTo('financeiro')"
          >
            <i class="ph ph-credit-card"></i>
            <span>Finanças</span>
          </button>
          <button 
            :class="['pwa-nav-item', { active: currentTab === 'configuracoes' }]"
            @click="navigateTo('configuracoes')"
          >
            <i class="ph ph-user-gear"></i>
            <span>Config</span>
          </button>
          <button 
            class="pwa-nav-item text-red"
            @click="handleLogout"
          >
            <i class="ph ph-sign-out"></i>
            <span>Sair</span>
          </button>
        </nav>
      </div>
    </div>

    <!-- MODAL CUSTOMIZADO: FUNCIONALIDADE EM DESENVOLVIMENTO -->
    <div v-if="showDevModal" class="custom-modal-overlay" @click.self="showDevModal = false">
      <div class="custom-modal-card">
        <i class="ph ph-info custom-modal-icon"></i>
        <h3 class="custom-modal-title">{{ devModalData.title }}</h3>
        <p class="custom-modal-text">{{ devModalData.message }}</p>
        <button class="btn btn-secondary btn-full" @click="showDevModal = false">Entendido</button>
      </div>
    </div>

  </div>
</template>

<style scoped>
.app-layout {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* Desktop Layout */
.desktop-layout {
  display: flex;
  flex-direction: column;
  flex-grow: 1;
}

.topbar-desktop {
  background: var(--bg-white); /* Fundo branco solicitado */
  color: var(--text-dark);
  border-bottom: 1px solid var(--border-color);
  position: sticky;
  top: 0;
  z-index: 1000;
  box-shadow: var(--shadow-sm);
}

.topbar-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 16px 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.brand-logo {
  max-height: 40px;
  display: block;
  /* Removido o filtro invertido para manter as cores originais no fundo branco */
}

/* Dropdown */
.user-profile-wrapper {
  position: relative;
}

.user-profile-area {
  display: flex;
  align-items: center;
  gap: 12px;
  cursor: pointer;
  padding: 8px 16px;
  border-radius: var(--radius-full);
  transition: var(--transition);
  background: var(--bg-gray);
  border: 1px solid var(--border-color);
}

.user-profile-area:hover {
  background: var(--border-color);
}

.avatar-circle {
  width: 32px;
  height: 32px;
  background: var(--primary-light); /* Fundo verde-claro suave */
  color: var(--secondary);          /* Iniciais em teal escuro */
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  font-size: 12px;
}

.profile-name {
  font-size: 14px;
  font-weight: 600;
  color: #0f172a !important; /* Cor do texto João Silva alterada para preto */
}

.dropdown-header {
  padding: 12px 16px 6px;
  display: flex;
  flex-direction: column;
}

.dropdown-header strong {
  font-size: 14px;
  color: var(--text-dark);
}

.dropdown-header span {
  font-size: 11px;
  color: var(--text-gray);
}

.text-red {
  color: #ef4444 !important;
}

.desktop-main {
  padding: 40px 24px;
  flex-grow: 1;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

/* PWA Simulator */
.pwa-layout {
  background: #0f172a;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  width: 100%;
}

.pwa-status-bar {
  background: var(--bg-sidebar);
  color: rgba(255, 255, 255, 0.8);
  padding: 8px 24px;
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  font-weight: 600;
}

.fake-icons {
  display: flex;
  gap: 6px;
  align-items: center;
}

.pwa-header {
  background: var(--bg-white);
  padding: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-bottom: 1px solid var(--border-color);
}

.pwa-logo {
  max-height: 36px;
  /* Removido o filtro invertido para manter as cores originais no fundo branco */
}
</style>
