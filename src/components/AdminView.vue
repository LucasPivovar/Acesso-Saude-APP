<script setup>
import { ref, computed, onMounted } from 'vue'

const props = defineProps({
  layoutMode: {
    type: String,
    default: 'desktop'
  }
})

const emit = defineEmits(['triggerDevModal'])

// Aba ativa do Admin: 'usuarios' ou 'financeiro'
const activeAdminTab = ref('usuarios')

// Dados do LocalStorage ou mockados inicialmente
const users = ref([])
const config = ref({
  planIndividualPrice: 79.90,
  planIndividualMmn: 20.00,
  planFamilyPrice: 129.90,
  planFamilyMmn: 50.00,
  percentages: [40, 20, 20, 10, 10],
  modules: {
    health: { label: 'Telemedicina', price: 10.00, icon: 'ph-first-aid' },
    clube: { label: 'Clube de Descontos', price: 5.00, icon: 'ph-tag' },
    pet: { label: 'Veterinário (Pet)', price: 15.00, icon: 'ph-dog' },
    funeral: { label: 'Auxílio Funerário', price: 8.00, icon: 'ph-skull' }
  }
})

// Controles de Modais
const showRegisterModal = ref(false)
const showConfigModal = ref(false)
const showTreeModal = ref(false)
const selectedTreeUser = ref(null)
const editingUser = ref(null)

// Histórico Financeiro do Admin (Renovações e Comissões)
const billingHistory = ref([
  { id: 1, user: 'João Silva', plan: 'Família', value: 129.90, status: 'Renovado (Pago)', date: '01/07/2026', commissionMmn: 50.00 },
  { id: 2, user: 'Carlos Silva', plan: 'Individual', value: 79.90, status: 'Renovado (Pago)', date: '01/07/2026', commissionMmn: 20.00 },
  { id: 3, user: 'Marina Costa', plan: 'Individual', value: 79.90, status: 'Pendente', date: '01/07/2026', commissionMmn: 0.00 },
  { id: 4, user: 'Ana Martins', plan: 'Família', value: 129.90, status: 'Renovado (Pago)', date: '20/06/2026', commissionMmn: 50.00 },
  { id: 5, user: 'Pedro Santos', plan: 'Individual', value: 79.90, status: 'Atrasado', date: '10/06/2026', commissionMmn: 0.00 }
])

// Inicialização dos dados com migração e limpeza de versões antigas do localStorage
const initData = () => {
  const savedUsers = localStorage.getItem('acesso_saude_users')
  const savedConfig = localStorage.getItem('acesso_saude_config')

  if (savedConfig) {
    try {
      const parsed = JSON.parse(savedConfig)
      if (!parsed.planIndividualPrice || !parsed.percentages) {
        localStorage.removeItem('acesso_saude_config')
        localStorage.removeItem('acesso_saude_users')
      }
    } catch (e) {
      localStorage.removeItem('acesso_saude_config')
      localStorage.removeItem('acesso_saude_users')
    }
  }

  const freshUsers = localStorage.getItem('acesso_saude_users')
  const freshConfig = localStorage.getItem('acesso_saude_config')

  if (freshConfig) {
    config.value = JSON.parse(freshConfig)
  } else {
    localStorage.setItem('acesso_saude_config', JSON.stringify(config.value))
  }

  if (freshUsers) {
    users.value = JSON.parse(freshUsers)
  } else {
    // Mock inicial de usuários
    users.value = [
      {
        id: 1,
        name: 'João Silva',
        email: 'joao.silva@email.com',
        cpf: '111.222.333-44',
        plan: 'Família',
        level: 'Sem Nível (Diretor)',
        referredBy: 'Nenhum',
        access: { health: true, clube: true, pet: true, funeral: true },
        status: 'ativo',
        date: '01/01/2026'
      },
      {
        id: 2,
        name: 'Carlos Silva',
        email: 'carlos@email.com',
        cpf: '222.333.444-55',
        plan: 'Individual',
        level: '1º Nível',
        referredBy: 'João Silva',
        access: { health: true, clube: true, pet: false, funeral: false },
        status: 'ativo',
        date: '03/06/2026'
      },
      {
        id: 3,
        name: 'Marina Costa',
        email: 'marina@email.com',
        cpf: '333.444.555-66',
        plan: 'Individual',
        level: '1º Nível',
        referredBy: 'João Silva',
        access: { health: true, clube: false, pet: false, funeral: false },
        status: 'pendente',
        date: '28/05/2026'
      },
      {
        id: 4,
        name: 'Ana Martins',
        email: 'ana@email.com',
        cpf: '444.555.666-77',
        plan: 'Família',
        level: '2º Nível',
        referredBy: 'Carlos Silva',
        access: { health: true, clube: true, pet: true, funeral: false },
        status: 'ativo',
        date: '20/05/2026'
      },
      {
        id: 5,
        name: 'Pedro Santos',
        email: 'pedro@email.com',
        cpf: '555.666.777-88',
        plan: 'Individual',
        level: '3º Nível',
        referredBy: 'Ana Martins',
        access: { health: false, clube: true, pet: false, funeral: false },
        status: 'inativo',
        date: '10/05/2026'
      }
    ]
    localStorage.setItem('acesso_saude_users', JSON.stringify(users.value))
  }
}

onMounted(() => {
  initData()
})

const saveToLocalStorage = () => {
  localStorage.setItem('acesso_saude_users', JSON.stringify(users.value))
  localStorage.setItem('acesso_saude_config', JSON.stringify(config.value))
}

// Cálculo do preço dinâmico de um usuário com base nas configurações
const calculatePrice = (userAccess, planName) => {
  let base = 19.90
  if (planName === 'Individual') base = config.value.planIndividualPrice
  else if (planName === 'Família') base = config.value.planFamilyPrice
  else if (planName === 'Bronze') base = 29.90
  else if (planName === 'Acesso Saúde Mais Premium') base = 99.90

  let total = base
  if (userAccess.health) total += config.value.modules.health.price
  if (userAccess.clube) total += config.value.modules.clube.price
  if (userAccess.pet) total += config.value.modules.pet.price
  if (userAccess.funeral) total += config.value.modules.funeral.price
  return total
}

// Cálculo recursivo de quanto o usuário está recebendo de comissão em 5 níveis
const calculateUserCommission = (user) => {
  let total = 0
  const lvl1 = users.value.filter(u => u.referredBy === user.name)
  lvl1.forEach(u1 => {
    const mmnVal = u1.plan === 'Família' ? config.value.planFamilyMmn : (u1.plan === 'Individual' ? config.value.planIndividualMmn : 0)
    total += (mmnVal * config.value.percentages[0]) / 100
    
    const lvl2 = users.value.filter(u => u.referredBy === u1.name)
    lvl2.forEach(u2 => {
      const mmnVal2 = u2.plan === 'Família' ? config.value.planFamilyMmn : (u2.plan === 'Individual' ? config.value.planIndividualMmn : 0)
      total += (mmnVal2 * config.value.percentages[1]) / 100
      
      const lvl3 = users.value.filter(u => u.referredBy === u2.name)
      lvl3.forEach(u3 => {
        const mmnVal3 = u3.plan === 'Família' ? config.value.planFamilyMmn : (u3.plan === 'Individual' ? config.value.planIndividualMmn : 0)
        total += (mmnVal3 * config.value.percentages[2]) / 100
        
        const lvl4 = users.value.filter(u => u.referredBy === u3.name)
        lvl4.forEach(u4 => {
          const mmnVal4 = u4.plan === 'Família' ? config.value.planFamilyMmn : (u4.plan === 'Individual' ? config.value.planIndividualMmn : 0)
          total += (mmnVal4 * config.value.percentages[3]) / 100
          
          const lvl5 = users.value.filter(u => u.referredBy === u4.name)
          lvl5.forEach(u5 => {
            const mmnVal5 = u5.plan === 'Família' ? config.value.planFamilyMmn : (u5.plan === 'Individual' ? config.value.planIndividualMmn : 0)
            total += (mmnVal5 * config.value.percentages[4]) / 100
          })
        })
      })
    })
  })
  
  return total
}

// Retorna uma lista aninhada contendo a rede abaixo de um determinado usuário (Estrutura de Árvore/Pirâmide)
const getReferralTree = (userName, currentLevel = 1) => {
  if (currentLevel > 5) return []
  const directChildren = users.value.filter(u => u.referredBy === userName)
  
  return directChildren.map(child => {
    return {
      name: child.name,
      plan: child.plan,
      level: child.level,
      status: child.status,
      depth: currentLevel,
      children: getReferralTree(child.name, currentLevel + 1)
    }
  })
}

// Detalha os padrinhos que receberam a comissão de uma determinada renovação
const getCommissionReceivers = (userName, planName) => {
  const receivers = []
  let currentUser = users.value.find(u => u.name === userName)
  
  if (!currentUser) return receivers
  
  let currentReferrerName = currentUser.referredBy
  let depth = 0
  const mmnVal = planName === 'Família' ? config.value.planFamilyMmn : (planName === 'Individual' ? config.value.planIndividualMmn : 0)
  
  while (currentReferrerName && currentReferrerName !== 'Nenhum' && depth < 5) {
    const parent = users.value.find(u => u.name === currentReferrerName)
    const pct = config.value.percentages[depth]
    const gain = (mmnVal * pct) / 100
    
    receivers.push({
      level: depth + 1,
      name: currentReferrerName,
      gain: gain
    })
    
    if (parent) {
      currentReferrerName = parent.referredBy
    } else {
      break
    }
    depth++
  }
  
  return receivers
}

// Formulário de Cadastro
const newUser = ref({
  name: '',
  email: '',
  cpf: '',
  plan: 'Individual',
  level: '1º Nível',
  referredBy: 'João Silva',
  access: {
    health: true,
    clube: true,
    pet: false,
    funeral: false
  },
  status: 'ativo'
})

// Cálculo em tempo real do preço do novo usuário
const newUserPrice = computed(() => {
  return calculatePrice(newUser.value.access, newUser.value.plan)
})

const registerUser = () => {
  if (!newUser.value.name || !newUser.value.email || !newUser.value.cpf) {
    emit('triggerDevModal', {
      title: 'Campos Obrigatórios',
      message: 'Por favor, preencha todos os campos cadastrais obrigatórios.'
    })
    return
  }

  const createdUser = {
    id: Date.now(),
    ...newUser.value,
    access: { ...newUser.value.access },
    date: new Date().toLocaleDateString('pt-BR')
  }

  users.value.push(createdUser)
  saveToLocalStorage()

  // Adiciona histórico financeiro inicial simulado
  billingHistory.value.unshift({
    id: Date.now(),
    user: createdUser.name,
    plan: createdUser.plan,
    value: calculatePrice(createdUser.access, createdUser.plan),
    status: 'Renovado (Pago)',
    date: new Date().toLocaleDateString('pt-BR'),
    commissionMmn: createdUser.plan === 'Família' ? config.value.planFamilyMmn : (createdUser.plan === 'Individual' ? config.value.planIndividualMmn : 0.00)
  })

  emit('triggerDevModal', {
    title: 'Usuário Cadastrado!',
    message: `O usuário ${createdUser.name} foi criado com sucesso com acesso aos módulos configurados.`
  })

  // Reset formulário e fecha modal
  newUser.value = {
    name: '',
    email: '',
    cpf: '',
    plan: 'Individual',
    level: '1º Nível',
    referredBy: 'João Silva',
    access: { health: true, clube: true, pet: false, funeral: false },
    status: 'ativo'
  }

  showRegisterModal.value = false
}

// Excluir usuário
const deleteUser = (id) => {
  if (confirm('Tem certeza que deseja remover este usuário?')) {
    const userToDelete = users.value.find(u => u.id === id)
    if (userToDelete) {
      billingHistory.value = billingHistory.value.filter(b => b.user !== userToDelete.name)
    }
    users.value = users.value.filter(u => u.id !== id)
    saveToLocalStorage()
    emit('triggerDevModal', {
      title: 'Usuário Removido',
      message: 'O cadastro do usuário foi removido permanentemente do sistema.'
    })
  }
}

// Edição rápida de acessos
const openEditAccess = (user) => {
  editingUser.value = JSON.parse(JSON.stringify(user))
}

const saveEditAccess = () => {
  const idx = users.value.findIndex(u => u.id === editingUser.value.id)
  if (idx !== -1) {
    users.value[idx] = editingUser.value
    saveToLocalStorage()
    editingUser.value = null
    emit('triggerDevModal', {
      title: 'Usuário Atualizado',
      message: 'Os dados e acessos do usuário foram salvos com sucesso.'
    })
  }
}

// Salvar Regras de Comissão e Preços
const saveRules = () => {
  saveToLocalStorage()
  showConfigModal.value = false
  emit('triggerDevModal', {
    title: 'Regras Atualizadas!',
    message: 'Regras de planos, módulos e distribuição de comissões em 5 níveis foram salvas no sistema.'
  })
}

const openTreeModal = (user) => {
  selectedTreeUser.value = user
  showTreeModal.value = true
}

// Filtro de Busca
const searchFilter = ref('')
const filteredUsers = computed(() => {
  return users.value.filter(u => {
    return u.name.toLowerCase().includes(searchFilter.value.toLowerCase()) ||
           u.email.toLowerCase().includes(searchFilter.value.toLowerCase()) ||
           u.cpf.includes(searchFilter.value)
  })
})
</script>

<template>
  <div class="admin-panel" :class="[layoutMode]">
    
    <!-- Header Admin -->
    <header class="admin-header card animated-item" style="animation-delay: 0s;">
      <div class="header-main">
        <div>
          <span class="admin-badge"><i class="ph ph-shield-check" style="margin-right: 4px;"></i> PAINEL DO ADMINISTRADOR</span>
          <h2>Usuários, Preços e Comissões</h2>
          <p>Visão geral da rede de afiliados e controle de mensalidades baseadas em acessos.</p>
        </div>
        
        <div class="action-buttons-header">
          <button class="btn btn-secondary" @click="showRegisterModal = true">
            <i class="ph ph-user-plus"></i> Cadastrar Usuário
          </button>
          <button class="btn btn-outline" @click="showConfigModal = true">
            <i class="ph ph-sliders"></i> Regras & Comissões
          </button>
        </div>
      </div>
    </header>

    <!-- Navegação de Abas do Admin -->
    <div class="admin-tabs-nav">
      <button 
        :class="['admin-tab-btn', { active: activeAdminTab === 'usuarios' }]" 
        @click="activeAdminTab = 'usuarios'"
      >
        <i class="ph ph-users"></i> Gestão de Usuários & Rede
      </button>
      <button 
        :class="['admin-tab-btn', { active: activeAdminTab === 'financeiro' }]" 
        @click="activeAdminTab = 'financeiro'"
      >
        <i class="ph ph-chart-line-up"></i> Histórico Financeiro (Renovações)
      </button>
    </div>

    <!-- ABA 1: GESTÃO DE USUÁRIOS -->
    <div v-if="activeAdminTab === 'usuarios'" class="tab-content-admin">
      <!-- Barra de Pesquisa -->
      <div class="card search-card animated-item" style="animation-delay: 0.05s; padding: 16px; margin-bottom: 20px; display: flex; gap: 12px; align-items: center;">
        <i class="ph ph-magnifying-glass text-gray"></i>
        <input 
          v-model="searchFilter"
          type="text" 
          placeholder="Buscar usuário por nome, email ou CPF..." 
          class="form-control"
          style="border: none; padding: 4px; font-size: 14px; width: 100%; outline: none;"
        />
      </div>

      <!-- LISTA DE USUÁRIOS: DESKTOP -->
      <div v-if="layoutMode === 'desktop'" class="admin-content animated-item" style="animation-delay: 0.1s;">
        <div class="card" style="padding: 0; overflow-x: auto;">
          <table class="admin-table">
            <thead>
              <tr>
                <th>Usuário</th>
                <th>Plano</th>
                <th>Nível Rede</th>
                <th>Indicado Por</th>
                <th>Acessos Ativos</th>
                <th>Mensalidade</th>
                <th>Comissão Recebida (MMN)</th>
                <th>Status</th>
                <th>Ações</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="u in filteredUsers" :key="u.id">
                <td>
                  <div class="user-info-col">
                    <strong>{{ u.name }}</strong>
                    <span>{{ u.email }} • CPF: {{ u.cpf }}</span>
                  </div>
                </td>
                <td>{{ u.plan }}</td>
                <td>
                  <span class="badge badge-level">{{ u.level }}</span>
                </td>
                <td>
                  <span v-if="u.referredBy !== 'Nenhum'" class="referred-badge">
                    <i class="ph ph-arrow-bend-down-right"></i> {{ u.referredBy }}
                  </span>
                  <span v-else class="text-gray">-</span>
                </td>
                <td>
                  <div class="modules-badges">
                    <span :class="['module-badge-icon', { active: u.access.health }]" :title="'Telemedicina: ' + (u.access.health ? 'Ativo' : 'Inativo')">
                      <i class="ph ph-first-aid"></i>
                    </span>
                    <span :class="['module-badge-icon', { active: u.access.clube }]" :title="'Clube de Descontos: ' + (u.access.clube ? 'Ativo' : 'Inativo')">
                      <i class="ph ph-tag"></i>
                    </span>
                    <span :class="['module-badge-icon', { active: u.access.pet }]" :title="'Veterinário (Pet): ' + (u.access.pet ? 'Ativo' : 'Inativo')">
                      <i class="ph ph-dog"></i>
                    </span>
                    <span :class="['module-badge-icon', { active: u.access.funeral }]" :title="'Auxílio Funerário: ' + (u.access.funeral ? 'Ativo' : 'Inativo')">
                      <i class="ph ph-skull"></i>
                    </span>
                  </div>
                </td>
                <td class="price-col">
                  R$ {{ calculatePrice(u.access, u.plan).toFixed(2) }}
                </td>
                <td style="font-weight: bold; color: #16a34a;">
                  R$ {{ calculateUserCommission(u).toFixed(2) }}
                </td>
                <td>
                  <span :class="['status-pill', u.status]">{{ u.status }}</span>
                </td>
                <td>
                  <div class="actions-buttons">
                    <button class="btn-action-edit" title="Editar Usuário" @click="openEditAccess(u)">
                      <i class="ph ph-pencil-simple"></i>
                    </button>
                    <button class="btn-action-tree" title="Visualizar Rede MMN" @click="openTreeModal(u)" style="color: #7c3aed; background: transparent; border: none; cursor: pointer; font-size: 16px; padding: 6px; border-radius: var(--radius-sm);">
                      <i class="ph ph-tree-structure"></i>
                    </button>
                    <button class="btn-action-delete" title="Excluir Usuário" @click="deleteUser(u.id)">
                      <i class="ph ph-trash"></i>
                    </button>
                  </div>
                </td>
              </tr>
              <tr v-if="filteredUsers.length === 0">
                <td colspan="9" style="text-align: center; padding: 24px; color: var(--text-gray);">
                  Nenhum usuário correspondente encontrado.
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- LISTA DE USUÁRIOS: MOBILE / PWA -->
      <div v-else class="admin-content-mobile animated-item" style="animation-delay: 0.1s;">
        <div v-for="u in filteredUsers" :key="u.id" class="mobile-user-card card">
          <div class="mobile-card-header">
            <div class="user-avatar-mini">{{ u.name.split(' ').map(n=>n[0]).join('') }}</div>
            <div class="mobile-card-title">
              <strong>{{ u.name }}</strong>
              <span>{{ u.email }}</span>
            </div>
            <span :class="['status-pill', u.status]">{{ u.status }}</span>
          </div>

          <div class="mobile-card-details">
            <div class="detail-row">
              <span class="label">CPF:</span>
              <span>{{ u.cpf }}</span>
            </div>
            <div class="detail-row">
              <span class="label">Plano:</span>
              <strong>{{ u.plan }}</strong>
            </div>
            <div class="detail-row">
              <span class="label">Rede:</span>
              <span class="badge badge-level">{{ u.level }}</span>
            </div>
            <div class="detail-row">
              <span class="label">Indicador:</span>
              <span v-if="u.referredBy !== 'Nenhum'" class="referred-badge">
                {{ u.referredBy }}
              </span>
              <span v-else>-</span>
            </div>
            <div class="detail-row">
              <span class="label">Preço:</span>
              <strong class="price-col">R$ {{ calculatePrice(u.access, u.plan).toFixed(2) }}</strong>
            </div>
            <div class="detail-row">
              <span class="label">Comissão MMN:</span>
              <strong style="color: #16a34a;">R$ {{ calculateUserCommission(u).toFixed(2) }}</strong>
            </div>
            <div class="detail-row" style="border: none; padding-top: 10px;">
              <span class="label">Serviços:</span>
              <div class="modules-badges">
                <span :class="['module-badge-icon', { active: u.access.health }]" :title="'Telemedicina: ' + (u.access.health ? 'Ativo' : 'Inativo')">
                  <i class="ph ph-first-aid"></i>
                </span>
                <span :class="['module-badge-icon', { active: u.access.clube }]" :title="'Clube de Descontos: ' + (u.access.clube ? 'Ativo' : 'Inativo')">
                  <i class="ph ph-tag"></i>
                </span>
                <span :class="['module-badge-icon', { active: u.access.pet }]" :title="'Veterinário (Pet): ' + (u.access.pet ? 'Ativo' : 'Inativo')">
                  <i class="ph ph-dog"></i>
                </span>
                <span :class="['module-badge-icon', { active: u.access.funeral }]" :title="'Auxílio Funerário: ' + (u.access.funeral ? 'Ativo' : 'Inativo')">
                  <i class="ph ph-skull"></i>
                </span>
              </div>
            </div>
          </div>

          <div class="mobile-card-actions" style="display: flex; gap: 8px; flex-wrap: wrap; border-top: 1px solid var(--border-color); padding-top: 12px;">
            <button class="btn btn-outline btn-sm" @click="openEditAccess(u)" style="flex: 1; min-width: 70px; display: flex; align-items: center; justify-content: center; gap: 4px;">
              <i class="ph ph-pencil-simple"></i> Editar
            </button>
            <button class="btn btn-outline btn-sm" @click="openTreeModal(u)" style="flex: 1; min-width: 70px; color: #7c3aed; border-color: #7c3aed; display: flex; align-items: center; justify-content: center; gap: 4px;">
              <i class="ph ph-tree-structure"></i> Rede
            </button>
            <button class="btn btn-outline btn-sm text-red" @click="deleteUser(u.id)" style="flex: 1; min-width: 70px; display: flex; align-items: center; justify-content: center; gap: 4px;">
              <i class="ph ph-trash"></i> Excluir
            </button>
          </div>
        </div>

        <div v-if="filteredUsers.length === 0" class="card" style="text-align: center; padding: 24px; color: var(--text-gray);">
          Nenhum usuário correspondente encontrado.
        </div>
      </div>
    </div>

    <!-- ABA 2: HISTÓRICO FINANCEIRO / RENOVAÇÕES -->
    <div v-else class="tab-content-admin animated-item" style="animation-delay: 0s;">
      <!-- Grid de métricas rápidas de faturamento -->
      <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 16px; margin-bottom: 24px;">
        <div class="rules-col-box" style="text-align: center; padding: 16px;">
          <span style="font-size: 11px; color: var(--text-gray); font-weight: 700; display:block; margin-bottom: 4px;">FATURAMENTO BRUTO</span>
          <strong style="font-size: 24px; color: var(--secondary);">R$ {{ billingHistory.filter(b=>b.status.includes('Pago')).reduce((a,b)=>a+b.value, 0).toFixed(2) }}</strong>
        </div>
        <div class="rules-col-box" style="text-align: center; padding: 16px; border-color: #fbcfe8;">
          <span style="font-size: 11px; color: var(--text-gray); font-weight: 700; display:block; margin-bottom: 4px;">COMISSÕES DISTRIBUÍDAS</span>
          <strong style="font-size: 24px; color: #db2777;">R$ {{ billingHistory.filter(b=>b.status.includes('Pago')).reduce((a,b)=>a+b.commissionMmn, 0).toFixed(2) }}</strong>
        </div>
        <div class="rules-col-box" style="text-align: center; padding: 16px; border-color: #bbf7d0;">
          <span style="font-size: 11px; color: var(--text-gray); font-weight: 700; display:block; margin-bottom: 4px;">RECEITA LÍQUIDA DA EMPRESA</span>
          <strong style="font-size: 24px; color: #16a34a;">R$ {{ (billingHistory.filter(b=>b.status.includes('Pago')).reduce((a,b)=>a+b.value, 0) - billingHistory.filter(b=>b.status.includes('Pago')).reduce((a,b)=>a+b.commissionMmn, 0)).toFixed(2) }}</strong>
        </div>
      </div>

      <div class="card" style="padding: 0; overflow-x: auto;">
        <table class="admin-table">
          <thead>
            <tr>
              <th>Usuário</th>
              <th>Plano</th>
              <th>Valor Cobrado</th>
              <th>Data Vencimento/Pago</th>
              <th>Status Renovação</th>
              <th>MMN Distribuído</th>
              <th>Padrinhos Beneficiados (Detalhamento MMN)</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="b in billingHistory" :key="b.id">
              <td>
                <strong>{{ b.user }}</strong>
              </td>
              <td>{{ b.plan }}</td>
              <td class="price-col">R$ {{ b.value.toFixed(2) }}</td>
              <td>{{ b.date }}</td>
              <td>
                <span :class="['status-pill', b.status.includes('Pago') ? 'ativo' : (b.status === 'Pendente' ? 'pendente' : 'inativo')]">
                  {{ b.status }}
                </span>
              </td>
              <td style="font-weight: bold; color: #db2777;">R$ {{ b.commissionMmn.toFixed(2) }}</td>
              <td>
                <div v-if="b.commissionMmn > 0 && getCommissionReceivers(b.user, b.plan).length > 0" style="display:flex; flex-direction:column; gap:4px; font-size: 11px;">
                  <div v-for="recv in getCommissionReceivers(b.user, b.plan)" :key="recv.name" style="background:#fdf2f8; border:1px solid #fbcfe8; padding: 4px 8px; border-radius: 4px; display:inline-flex; align-items:center; justify-content:space-between; gap:10px;">
                    <span>Nível {{ recv.level }}: <strong>{{ recv.name }}</strong></span>
                    <strong style="color: #db2777;">+ R$ {{ recv.gain.toFixed(2) }}</strong>
                  </div>
                </div>
                <span v-else class="text-gray" style="font-size:11px;">Nenhuma comissão distribuída</span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- MODAL 1: CADASTRAR USUÁRIO -->
    <div v-if="showRegisterModal" class="custom-modal-overlay" @click.self="showRegisterModal = false">
      <div class="custom-modal-card modal-large">
        <div class="modal-header-container">
          <h3><i class="ph ph-user-plus"></i> Novo Cadastro</h3>
          <button class="btn-close-modal" @click="showRegisterModal = false">✕</button>
        </div>

        <div class="modal-grid-content">
          <form @submit.prevent="registerUser" class="admin-form">
            <div class="form-row">
              <div class="form-group">
                <label class="form-label">Nome Completo</label>
                <input v-model="newUser.name" type="text" placeholder="Ex: Maria Oliveira" class="form-control" required />
              </div>
              <div class="form-group">
                <label class="form-label">E-mail</label>
                <input v-model="newUser.email" type="email" placeholder="Ex: maria@email.com" class="form-control" required />
              </div>
            </div>

            <div class="form-row">
              <div class="form-group">
                <label class="form-label">CPF</label>
                <input v-model="newUser.cpf" type="text" placeholder="Ex: 000.000.000-00" class="form-control" required />
              </div>
              <div class="form-group">
                <label class="form-label">Plano Comercial</label>
                <select v-model="newUser.plan" class="form-control">
                  <option value="Bronze">Bronze (Mais Simples)</option>
                  <option value="Individual">Individual (MMN R$ 79,90)</option>
                  <option value="Família">Família (MMN R$ 129,90)</option>
                  <option value="Acesso Saúde Mais Premium">Acesso Saúde Mais Premium</option>
                </select>
              </div>
            </div>

            <div class="form-row">
              <div class="form-group">
                <label class="form-label">Nível no Programa de Afiliados</label>
                <select v-model="newUser.level" class="form-control">
                  <option value="Sem Nível (Diretor)">Sem Nível (Diretor Comercial)</option>
                  <option value="1º Nível">1º Nível (Indicação Direta)</option>
                  <option value="2º Nível">2º Nível</option>
                  <option value="3º Nível">3º Nível</option>
                  <option value="4º Nível">4º Nível</option>
                  <option value="5º Nível">5º Nível</option>
                </select>
              </div>
              <div class="form-group">
                <label class="form-label">Indicado Por (Quem o trouxe)</label>
                <select v-model="newUser.referredBy" class="form-control">
                  <option value="Nenhum">Nenhum (Direto da empresa)</option>
                  <option v-for="u in users" :key="u.id" :value="u.name">{{ u.name }}</option>
                </select>
              </div>
            </div>

            <h4 style="margin: 20px 0 10px; color: var(--secondary);">Módulos e Benefícios Ativos</h4>
            <div class="access-checklist scrollable-checklist">
              <label class="checklist-item">
                <input type="checkbox" v-model="newUser.access.health" />
                <div class="checklist-content">
                  <span class="checklist-title">Telemedicina</span>
                  <span class="checklist-price">+ R$ {{ config.modules.health.price.toFixed(2) }}/mês</span>
                </div>
              </label>

              <label class="checklist-item">
                <input type="checkbox" v-model="newUser.access.clube" />
                <div class="checklist-content">
                  <span class="checklist-title">Clube de Descontos</span>
                  <span class="checklist-price">+ R$ {{ config.modules.clube.price.toFixed(2) }}/mês</span>
                </div>
              </label>

              <label class="checklist-item">
                <input type="checkbox" v-model="newUser.access.pet" />
                <div class="checklist-content">
                  <span class="checklist-title">Veterinário (Pet)</span>
                  <span class="checklist-price">+ R$ {{ config.modules.pet.price.toFixed(2) }}/mês</span>
                </div>
              </label>

              <label class="checklist-item">
                <input type="checkbox" v-model="newUser.access.funeral" />
                <div class="checklist-content">
                  <span class="checklist-title">Auxílio Funerário</span>
                  <span class="checklist-price">+ R$ {{ config.modules.funeral.price.toFixed(2) }}/mês</span>
                </div>
              </label>
            </div>

            <button type="submit" class="btn btn-secondary btn-full" style="margin-top: 20px;">
              <i class="ph ph-user-plus"></i> Salvar e Cadastrar
            </button>
          </form>

          <!-- Precificação ao lado -->
          <div class="modal-price-aside">
            <h4>Resumo do Preço</h4>
            <div class="price-summary-list">
              <div class="summary-row">
                <span>Preço do Plano Base</span>
                <strong>
                  R$ {{ (newUser.plan === 'Individual' ? config.planIndividualPrice : (newUser.plan === 'Família' ? config.planFamilyPrice : (newUser.plan === 'Bronze' ? 29.90 : 99.90))).toFixed(2) }}
                </strong>
              </div>
              <div v-if="newUser.access.health" class="summary-row">
                <span>+ Telemedicina</span>
                <strong>R$ {{ config.modules.health.price.toFixed(2) }}</strong>
              </div>
              <div v-if="newUser.access.clube" class="summary-row">
                <span>+ Clube</span>
                <strong>R$ {{ config.modules.clube.price.toFixed(2) }}</strong>
              </div>
              <div v-if="newUser.access.pet" class="summary-row">
                <span>+ Pet</span>
                <strong>R$ {{ config.modules.pet.price.toFixed(2) }}</strong>
              </div>
              <div v-if="newUser.access.funeral" class="summary-row">
                <span>+ Funerário</span>
                <strong>R$ {{ config.modules.funeral.price.toFixed(2) }}</strong>
              </div>
            </div>
            <div class="total-price-badge">
              <span style="font-size: 11px; color: var(--text-gray);">Total Mensal</span>
              <strong>R$ {{ newUserPrice.toFixed(2) }}</strong>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- MODAL 2: CONFIGURAÇÃO DE REGRAS E COMISSÕES -->
    <div v-if="showConfigModal" class="custom-modal-overlay" @click.self="showConfigModal = false">
      <div class="custom-modal-card modal-large">
        <div class="modal-header-container">
          <h3><i class="ph ph-sliders"></i> Regras de Preços & Comissões MMN</h3>
          <button class="btn-close-modal" @click="showConfigModal = false">✕</button>
        </div>

        <div class="modal-grid-content two-columns">
          <!-- Coluna 1: Preços dos Módulos -->
          <div class="rules-col-box">
            <h4>Preços dos Módulos Adicionais</h4>
            <form @submit.prevent="saveRules" class="admin-form">
              <div class="form-group">
                <label class="form-label">Telemedicina</label>
                <div class="input-icon-wrapper">
                  <span class="currency-prefix">R$</span>
                  <input v-model.number="config.modules.health.price" type="number" step="0.01" class="form-control with-icon" style="padding-left: 36px;" />
                </div>
              </div>
              <div class="form-group">
                <label class="form-label">Clube de Descontos</label>
                <div class="input-icon-wrapper">
                  <span class="currency-prefix">R$</span>
                  <input v-model.number="config.modules.clube.price" type="number" step="0.01" class="form-control with-icon" style="padding-left: 36px;" />
                </div>
              </div>
              <div class="form-group">
                <label class="form-label">Veterinário (Pet)</label>
                <div class="input-icon-wrapper">
                  <span class="currency-prefix">R$</span>
                  <input v-model.number="config.modules.pet.price" type="number" step="0.01" class="form-control with-icon" style="padding-left: 36px;" />
                </div>
              </div>
              <div class="form-group">
                <label class="form-label">Auxílio Funerário</label>
                <div class="input-icon-wrapper">
                  <span class="currency-prefix">R$</span>
                  <input v-model.number="config.modules.funeral.price" type="number" step="0.01" class="form-control with-icon" style="padding-left: 36px;" />
                </div>
              </div>
            </form>
          </div>

          <!-- Coluna 2: Configurações dos Planos MMN -->
          <div class="rules-col-box">
            <h4>Valores MMN & Distribuição (%)</h4>
            <form @submit.prevent="saveRules" class="admin-form">
              <div class="form-row">
                <div class="form-group">
                  <label class="form-label">Individual - Preço</label>
                  <input v-model.number="config.planIndividualPrice" type="number" step="0.01" class="form-control" />
                </div>
                <div class="form-group">
                  <label class="form-label">Individual - Comissão</label>
                  <input v-model.number="config.planIndividualMmn" type="number" step="0.01" class="form-control" />
                </div>
              </div>
              <div class="form-row">
                <div class="form-group">
                  <label class="form-label">Familiar - Preço</label>
                  <input v-model.number="config.planFamilyPrice" type="number" step="0.01" class="form-control" />
                </div>
                <div class="form-group">
                  <label class="form-label">Familiar - Comissão</label>
                  <input v-model.number="config.planFamilyMmn" type="number" step="0.01" class="form-control" />
                </div>
              </div>
              <h5 style="margin: 10px 0 5px; color: var(--secondary);">Percentuais de Distribuição (5 Níveis)</h5>
              <div style="display: grid; grid-template-columns: repeat(5, 1fr); gap: 6px;">
                <div v-for="(pct, idx) in config.percentages" :key="idx" class="form-group">
                  <label class="form-label" style="font-size: 10px; text-align: center;">Nível {{ idx + 1 }} (%)</label>
                  <input v-model.number="config.percentages[idx]" type="number" class="form-control" style="padding: 4px; text-align: center;" />
                </div>
              </div>
            </form>
          </div>
        </div>

        <!-- Tabelas demonstrativas das comissões idênticas às imagens -->
        <h4 style="margin: 24px 0 12px; color: var(--secondary); text-align: center;">Demonstrativo de Comissão por Nível</h4>
        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px;">
          <!-- Tabela Familiar -->
          <div class="rules-col-box">
            <h5 style="margin: 0 0 10px; color: var(--secondary); text-align: center; font-size: 14px;">Plano Familiar (MMN: R$ {{ config.planFamilyMmn.toFixed(2) }})</h5>
            <table class="admin-table mini-table">
              <thead>
                <tr>
                  <th style="padding: 8px;">Nível</th>
                  <th style="padding: 8px;">%</th>
                  <th style="padding: 8px;">Familiar R$/pessoa</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(pct, idx) in config.percentages" :key="idx">
                  <td style="padding: 8px;"><strong>{{ idx + 1 }}</strong></td>
                  <td style="padding: 8px;">{{ pct }}%</td>
                  <td style="padding: 8px; font-weight: bold; color: #16a34a;">R$ {{ (config.planFamilyMmn * pct / 100).toFixed(2) }}</td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- Tabela Individual -->
          <div class="rules-col-box">
            <h5 style="margin: 0 0 10px; color: var(--secondary); text-align: center; font-size: 14px;">Plano Individual (MMN: R$ {{ config.planIndividualMmn.toFixed(2) }})</h5>
            <table class="admin-table mini-table">
              <thead>
                <tr>
                  <th style="padding: 8px;">Nível</th>
                  <th style="padding: 8px;">%</th>
                  <th style="padding: 8px;">Individual R$/pessoa</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(pct, idx) in config.percentages" :key="idx">
                  <td style="padding: 8px;"><strong>{{ idx + 1 }}</strong></td>
                  <td style="padding: 8px;">{{ pct }}%</td>
                  <td style="padding: 8px; font-weight: bold; color: #16a34a;">R$ {{ (config.planIndividualMmn * pct / 100).toFixed(2) }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <div style="margin-top: 24px; text-align: right;">
          <button class="btn btn-outline" @click="showConfigModal = false" style="margin-right: 12px;">Cancelar</button>
          <button class="btn btn-secondary" @click="saveRules">Salvar Tudo</button>
        </div>
      </div>
    </div>

    <!-- MODAL 3: VISUALIZAR REDE MMN (PIRÂMIDE / ÁRVORE) -->
    <div v-if="showTreeModal" class="custom-modal-overlay" @click.self="showTreeModal = false">
      <div class="custom-modal-card modal-large" style="max-width: 700px;">
        <div class="modal-header-container">
          <h3><i class="ph ph-tree-structure"></i> Rede de Afiliados (Árvore MMN)</h3>
          <button class="btn-close-modal" @click="showTreeModal = false">✕</button>
        </div>
        
        <div v-if="selectedTreeUser">
          <div style="background: #faf5ff; border-left: 4px solid #7c3aed; padding: 16px; border-radius: var(--radius-sm); margin-bottom: 24px;">
            <strong style="font-size: 16px; color: #4c1d95;">Usuário Raiz: {{ selectedTreeUser.name }}</strong>
            <div style="font-size: 13px; color: #6b21a8; margin-top: 4px; display:flex; justify-content:space-between;">
              <span>Plano: {{ selectedTreeUser.plan }} • Nível: {{ selectedTreeUser.level }}</span>
              <strong>Total Comissão Recebida: R$ {{ calculateUserCommission(selectedTreeUser).toFixed(2) }}</strong>
            </div>
          </div>
          
          <h4 style="margin-bottom: 12px; color: var(--secondary);">Estrutura Hierárquica (Até 5 Níveis)</h4>
          
          <div class="referral-tree-container" style="background: white; border: 1px solid var(--border-color); border-radius: var(--radius-lg); padding: 20px; max-height: 400px; overflow-y: auto;">
            <div v-if="getReferralTree(selectedTreeUser.name).length > 0">
              <div v-for="node1 in getReferralTree(selectedTreeUser.name)" :key="node1.name" class="tree-node depth-1">
                <div class="tree-item-box">
                  <span class="tree-badge l1">1º Nível</span>
                  <strong>{{ node1.name }}</strong>
                  <span class="tree-plan-name">({{ node1.plan }})</span>
                  <span :class="['status-pill', node1.status]" style="font-size: 10px; padding: 1px 6px;">{{ node1.status }}</span>
                  <span class="tree-gain-share">+ R$ {{ ((node1.plan === 'Família' ? config.planFamilyMmn : config.planIndividualMmn) * config.percentages[0] / 100).toFixed(2) }}</span>
                </div>
                
                <!-- Nível 2 -->
                <div v-if="node1.children.length > 0" style="margin-left: 24px; border-left: 2px dashed #e2e8f0; padding-left: 16px; margin-top: 8px;">
                  <div v-for="node2 in node1.children" :key="node2.name" class="tree-node depth-2">
                    <div class="tree-item-box">
                      <span class="tree-badge l2">2º Nível</span>
                      <strong>{{ node2.name }}</strong>
                      <span class="tree-plan-name">({{ node2.plan }})</span>
                      <span :class="['status-pill', node2.status]" style="font-size: 10px; padding: 1px 6px;">{{ node2.status }}</span>
                      <span class="tree-gain-share">+ R$ {{ ((node2.plan === 'Família' ? config.planFamilyMmn : config.planIndividualMmn) * config.percentages[1] / 100).toFixed(2) }}</span>
                    </div>
                    
                    <!-- Nível 3 -->
                    <div v-if="node2.children.length > 0" style="margin-left: 24px; border-left: 2px dashed #e2e8f0; padding-left: 16px; margin-top: 8px;">
                      <div v-for="node3 in node2.children" :key="node3.name" class="tree-node depth-3">
                        <div class="tree-item-box">
                          <span class="tree-badge l3">3º Nível</span>
                          <strong>{{ node3.name }}</strong>
                          <span class="tree-plan-name">({{ node3.plan }})</span>
                          <span :class="['status-pill', node3.status]" style="font-size: 10px; padding: 1px 6px;">{{ node3.status }}</span>
                          <span class="tree-gain-share">+ R$ {{ ((node3.plan === 'Família' ? config.planFamilyMmn : config.planIndividualMmn) * config.percentages[2] / 100).toFixed(2) }}</span>
                        </div>
                        
                        <!-- Nível 4 -->
                        <div v-if="node3.children.length > 0" style="margin-left: 24px; border-left: 2px dashed #e2e8f0; padding-left: 16px; margin-top: 8px;">
                          <div v-for="node4 in node3.children" :key="node4.name" class="tree-node depth-4">
                            <div class="tree-item-box">
                              <span class="tree-badge l4">4º Nível</span>
                              <strong>{{ node4.name }}</strong>
                              <span class="tree-plan-name">({{ node4.plan }})</span>
                              <span :class="['status-pill', node4.status]" style="font-size: 10px; padding: 1px 6px;">{{ node4.status }}</span>
                              <span class="tree-gain-share">+ R$ {{ ((node4.plan === 'Família' ? config.planFamilyMmn : config.planIndividualMmn) * config.percentages[3] / 100).toFixed(2) }}</span>
                            </div>
                            
                            <!-- Nível 5 -->
                            <div v-if="node4.children.length > 0" style="margin-left: 24px; border-left: 2px dashed #e2e8f0; padding-left: 16px; margin-top: 8px;">
                              <div v-for="node5 in node4.children" :key="node5.name" class="tree-node depth-5">
                                <div class="tree-item-box">
                                  <span class="tree-badge l5">5º Nível</span>
                                  <strong>{{ node5.name }}</strong>
                                  <span class="tree-plan-name">({{ node5.plan }})</span>
                                  <span :class="['status-pill', node5.status]" style="font-size: 10px; padding: 1px 6px;">{{ node5.status }}</span>
                                  <span class="tree-gain-share">+ R$ {{ ((node5.plan === 'Família' ? config.planFamilyMmn : config.planIndividualMmn) * config.percentages[4] / 100).toFixed(2) }}</span>
                                </div>
                              </div>
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            <div v-else style="text-align: center; color: var(--text-gray); padding: 24px;">
              Este usuário ainda não possui indicações registradas na sua rede de afiliados.
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- MODAL DE EDIÇÃO COMPLETO -->
    <div v-if="editingUser" class="custom-modal-overlay" @click.self="editingUser = null">
      <div class="custom-modal-card modal-large">
        <div class="modal-header-container">
          <h3><i class="ph ph-pencil-simple"></i> Editar Usuário</h3>
          <button class="btn-close-modal" @click="editingUser = null">✕</button>
        </div>

        <div class="modal-grid-content">
          <!-- Coluna 1: Campos Cadastrais -->
          <form @submit.prevent="saveEditAccess" class="admin-form">
            <div class="form-row">
              <div class="form-group">
                <label class="form-label">Nome Completo</label>
                <input v-model="editingUser.name" type="text" class="form-control" required />
              </div>
              <div class="form-group">
                <label class="form-label">E-mail</label>
                <input v-model="editingUser.email" type="email" class="form-control" required />
              </div>
            </div>

            <div class="form-row">
              <div class="form-group">
                <label class="form-label">CPF</label>
                <input v-model="editingUser.cpf" type="text" class="form-control" required />
              </div>
              <div class="form-group">
                <label class="form-label">Plano Comercial</label>
                <select v-model="editingUser.plan" class="form-control">
                  <option value="Bronze">Bronze (Mais Simples)</option>
                  <option value="Individual">Individual (MMN R$ 79,90)</option>
                  <option value="Família">Família (MMN R$ 129,90)</option>
                  <option value="Acesso Saúde Mais Premium">Acesso Saúde Mais Premium</option>
                </select>
              </div>
            </div>

            <div class="form-row">
              <div class="form-group">
                <label class="form-label">Nível no Programa de Afiliados</label>
                <select v-model="editingUser.level" class="form-control">
                  <option value="Sem Nível (Diretor)">Sem Nível (Diretor Comercial)</option>
                  <option value="1º Nível">1º Nível (Indicação Direta)</option>
                  <option value="2º Nível">2º Nível</option>
                  <option value="3º Nível">3º Nível</option>
                  <option value="4º Nível">4º Nível</option>
                  <option value="5º Nível">5º Nível</option>
                </select>
              </div>
              <div class="form-group">
                <label class="form-label">Indicado Por (Quem o trouxe)</label>
                <select v-model="editingUser.referredBy" class="form-control">
                  <option value="Nenhum">Nenhum (Direto da empresa)</option>
                  <option v-for="u in users" :key="u.id" :value="u.name">{{ u.name }}</option>
                </select>
              </div>
            </div>

            <h4 style="margin: 20px 0 10px; color: var(--secondary);">Módulos e Benefícios Ativos</h4>
            <div class="access-checklist scrollable-checklist">
              <label class="checklist-item">
                <input type="checkbox" v-model="editingUser.access.health" />
                <div class="checklist-content">
                  <span class="checklist-title">Telemedicina</span>
                  <span class="checklist-price">+ R$ {{ config.modules.health.price.toFixed(2) }}/mês</span>
                </div>
              </label>

              <label class="checklist-item">
                <input type="checkbox" v-model="editingUser.access.clube" />
                <div class="checklist-content">
                  <span class="checklist-title">Clube de Descontos</span>
                  <span class="checklist-price">+ R$ {{ config.modules.clube.price.toFixed(2) }}/mês</span>
                </div>
              </label>

              <label class="checklist-item">
                <input type="checkbox" v-model="editingUser.access.pet" />
                <div class="checklist-content">
                  <span class="checklist-title">Veterinário (Pet)</span>
                  <span class="checklist-price">+ R$ {{ config.modules.pet.price.toFixed(2) }}/mês</span>
                </div>
              </label>

              <label class="checklist-item">
                <input type="checkbox" v-model="editingUser.access.funeral" />
                <div class="checklist-content">
                  <span class="checklist-title">Auxílio Funerário</span>
                  <span class="checklist-price">+ R$ {{ config.modules.funeral.price.toFixed(2) }}/mês</span>
                </div>
              </label>
            </div>

            <button type="submit" class="btn btn-secondary btn-full" style="margin-top: 20px;">
              <i class="ph ph-floppy-disk"></i> Salvar Alterações
            </button>
          </form>

          <!-- Coluna 2: Precificação ao lado -->
          <div class="modal-price-aside">
            <h4>Resumo da Cobrança</h4>
            <div class="price-summary-list">
              <div class="summary-row">
                <span>Preço do Plano Base</span>
                <strong>
                  R$ {{ (editingUser.plan === 'Individual' ? config.planIndividualPrice : (editingUser.plan === 'Família' ? config.planFamilyPrice : (editingUser.plan === 'Bronze' ? 29.90 : 99.90))).toFixed(2) }}
                </strong>
              </div>
              <div v-if="editingUser.access.health" class="summary-row">
                <span>+ Telemedicina</span>
                <strong>R$ {{ config.modules.health.price.toFixed(2) }}</strong>
              </div>
              <div v-if="editingUser.access.clube" class="summary-row">
                <span>+ Clube</span>
                <strong>R$ {{ config.modules.clube.price.toFixed(2) }}</strong>
              </div>
              <div v-if="editingUser.access.pet" class="summary-row">
                <span>+ Pet</span>
                <strong>R$ {{ config.modules.pet.price.toFixed(2) }}</strong>
              </div>
              <div v-if="editingUser.access.funeral" class="summary-row">
                <span>+ Funerário</span>
                <strong>R$ {{ config.modules.funeral.price.toFixed(2) }}</strong>
              </div>
            </div>
            <div class="total-price-badge">
              <span style="font-size: 11px; color: var(--text-gray);">Total Calculado</span>
              <strong>R$ {{ calculatePrice(editingUser.access, editingUser.plan).toFixed(2) }}</strong>
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<style scoped>
.admin-panel {
  width: 100%;
}

.admin-badge {
  background: var(--primary-light);
  color: var(--secondary);
  font-size: 11px;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: var(--radius-sm);
  display: inline-block;
  margin-bottom: 12px;
}

.admin-header {
  background: var(--bg-white);
  padding: 24px;
  margin-bottom: 24px;
  border: 1px solid var(--border-color);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-sm);
}

.header-main {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 16px;
}

.action-buttons-header {
  display: flex;
  flex-direction: column;
  gap: 10px;
  width: auto;
}

.action-buttons-header .btn {
  width: 100%;
  text-align: center;
  justify-content: center;
}

/* Navegação de Abas do Admin */
.admin-tabs-nav {
  display: flex;
  gap: 16px;
  margin-bottom: 20px;
  border-bottom: 1px solid var(--border-color);
  padding-bottom: 10px;
}

.admin-tab-btn {
  background: transparent;
  border: none;
  font-size: 15px;
  font-weight: 600;
  color: var(--text-gray);
  padding: 8px 16px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  border-bottom: 2px solid transparent;
  transition: var(--transition);
}

.admin-tab-btn.active {
  color: var(--secondary);
  border-bottom-color: var(--secondary);
}

.admin-tab-btn:hover {
  color: var(--secondary);
}

/* Tabela de Usuários */
.admin-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 14px;
}

.admin-table th {
  background: #f8fafc;
  padding: 14px 16px;
  text-align: left;
  font-weight: 600;
  color: var(--secondary);
  border-bottom: 1px solid var(--border-color);
}

.admin-table td {
  padding: 14px 16px;
  border-bottom: 1px solid var(--border-color);
  vertical-align: middle;
}

.user-info-col {
  display: flex;
  flex-direction: column;
}

.user-info-col strong {
  color: var(--text-dark);
  font-size: 14px;
}

.user-info-col span {
  font-size: 11px;
  color: var(--text-gray);
  margin-top: 2px;
}

.badge-level {
  background: #f1f5f9;
  color: var(--text-dark);
  border: 1px solid var(--border-color);
  font-weight: 500;
  font-size: 11px;
  padding: 2px 8px;
  border-radius: var(--radius-sm);
}

.referred-badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  font-size: 13px;
  color: var(--text-dark);
  background: #f0f9ff;
  border: 1px solid #bae6fd;
  padding: 2px 8px;
  border-radius: var(--radius-sm);
}

.modules-badges {
  display: flex;
  gap: 6px;
}

.module-badge-icon {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f1f5f9;
  color: #94a3b8;
  font-size: 14px;
  transition: var(--transition);
}

.module-badge-icon.active {
  background: #dcfce7 !important;
  color: #16a34a !important;
}

.price-col {
  font-weight: 700;
  color: var(--secondary);
}

.status-pill {
  display: inline-block;
  font-size: 11px;
  font-weight: 700;
  padding: 2px 8px;
  border-radius: var(--radius-full);
  text-transform: capitalize;
}

.status-pill.ativo {
  background: #dcfce7;
  color: #15803d;
}

.status-pill.pendente {
  background: #fef3c7;
  color: #d97706;
}

.status-pill.inativo {
  background: #fee2e2;
  color: #b91c1c;
}

.actions-buttons {
  display: flex;
  gap: 8px;
}

.btn-action-edit, .btn-action-delete {
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 6px;
  border-radius: var(--radius-sm);
  transition: var(--transition);
  font-size: 16px;
}

.btn-action-edit {
  color: #0284c7;
}

.btn-action-edit:hover {
  background: #e0f2fe;
}

.btn-action-delete {
  color: #ef4444;
}

.btn-action-delete:hover {
  background: #fee2e2;
}

/* Modais Específicos */
.modal-large {
  max-width: 850px !important;
  width: 90% !important;
  max-height: 90vh;
  overflow-y: auto;
  padding: 24px !important;
}

.modal-header-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid var(--border-color);
  padding-bottom: 14px;
  margin-bottom: 20px;
}

.modal-header-container h3 {
  color: var(--secondary);
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 0;
}

.btn-close-modal {
  background: transparent;
  border: none;
  font-size: 20px;
  color: var(--text-gray);
  cursor: pointer;
  transition: var(--transition);
}

.btn-close-modal:hover {
  color: #ef4444;
}

.modal-grid-content {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 24px;
}

.modal-grid-content.two-columns {
  grid-template-columns: 1fr 1fr;
}

@media (max-width: 768px) {
  .modal-grid-content, .modal-grid-content.two-columns {
    grid-template-columns: 1fr;
  }
}

.modal-price-aside {
  background: linear-gradient(135deg, var(--secondary-light), #f8fafc);
  border: 1px solid var(--border-color);
  border-radius: var(--radius-lg);
  padding: 20px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.modal-price-aside h4 {
  margin-top: 0;
  margin-bottom: 16px;
  color: var(--secondary);
}

.total-price-badge {
  border-top: 2px dashed var(--border-color);
  padding-top: 16px;
  margin-top: 20px;
  text-align: center;
  display: flex;
  flex-direction: column;
}

.total-price-badge strong {
  font-size: 28px;
  color: var(--secondary);
}

.rules-col-box {
  background: #f8fafc;
  padding: 16px;
  border-radius: var(--radius-lg);
  border: 1px solid var(--border-color);
}

.rules-col-box h4 {
  margin-top: 0;
  margin-bottom: 16px;
  color: var(--secondary);
  border-bottom: 1px solid var(--border-color);
  padding-bottom: 8px;
}

.mini-table {
  font-size: 12px;
}

.mini-table th {
  padding: 8px 12px;
}

.mini-table td {
  padding: 8px 12px;
}

/* Árvore Hierárquica MMN style */
.referral-tree-container {
  font-family: inherit;
}

.tree-node {
  margin-bottom: 12px;
  transition: var(--transition);
}

.tree-item-box {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  background: #f8fafc;
  border: 1px solid var(--border-color);
  padding: 8px 16px;
  border-radius: var(--radius-sm);
  font-size: 13px;
  color: var(--text-dark);
}

.tree-badge {
  font-size: 10px;
  font-weight: 700;
  padding: 2px 6px;
  border-radius: var(--radius-sm);
  color: white;
}

.tree-badge.l1 { background: #3b82f6; }
.tree-badge.l2 { background: #8b5cf6; }
.tree-badge.l3 { background: #ec4899; }
.tree-badge.l4 { background: #f59e0b; }
.tree-badge.l5 { background: #10b981; }

.tree-plan-name {
  font-size: 11px;
  color: var(--text-gray);
}

.tree-gain-share {
  font-weight: 700;
  color: #16a34a;
  margin-left: 8px;
}

/* Formulário e Checklist */
.admin-form {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 14px;
}

@media (max-width: 576px) {
  .form-row {
    grid-template-columns: 1fr;
  }
}

.access-checklist {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

@media (max-width: 576px) {
  .access-checklist {
    grid-template-columns: 1fr;
  }
}

.checklist-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px;
  background: white;
  border: 1px solid var(--border-color);
  border-radius: var(--radius-sm);
  cursor: pointer;
  transition: var(--transition);
}

.checklist-item:hover {
  background: #fafafa;
  border-color: #cbd5e1;
}

.checklist-item input[type="checkbox"] {
  width: 16px;
  height: 16px;
  accent-color: var(--secondary);
}

.checklist-content {
  display: flex;
  flex-direction: column;
  flex-grow: 1;
}

.checklist-title {
  font-size: 13px;
  font-weight: 600;
  color: var(--text-dark);
}

.checklist-price {
  font-size: 10px;
  color: var(--text-gray);
}

.price-summary-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.summary-row {
  display: flex;
  justify-content: space-between;
  font-size: 13px;
  color: var(--text-dark);
}

.summary-row span {
  color: var(--text-gray);
}

.input-icon-wrapper {
  position: relative;
  width: 100%;
}

.currency-prefix {
  position: absolute;
  left: 10px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 13px;
  font-weight: 600;
  color: var(--text-gray);
  z-index: 5;
}

/* LISTAGEM MOBILE (Cards) */
.admin-content-mobile {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.mobile-user-card {
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.mobile-card-header {
  display: flex;
  align-items: center;
  gap: 12px;
  border-bottom: 1px solid var(--border-color);
  padding-bottom: 12px;
}

.mobile-card-title {
  display: flex;
  flex-direction: column;
  flex-grow: 1;
}

.mobile-card-title strong {
  font-size: 15px;
  color: var(--text-dark);
}

.mobile-card-title span {
  font-size: 11px;
  color: var(--text-gray);
}

.mobile-card-details {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.detail-row {
  display: flex;
  justify-content: space-between;
  font-size: 13px;
  padding-bottom: 6px;
  border-bottom: 1px dashed #f1f5f9;
}

.detail-row .label {
  color: var(--text-gray);
  font-weight: 500;
}

.mobile-card-actions {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  border-top: 1px solid var(--border-color);
  padding-top: 12px;
}

/* Alinhamento dos modais e inputs */
.custom-modal-card {
  text-align: left !important;
}

.form-label {
  text-align: left !important;
  display: block;
}

.form-control {
  text-align: left !important;
}
</style>
