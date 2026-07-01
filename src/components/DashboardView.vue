<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps({
  user: {
    type: Object,
    required: true
  },
  layoutMode: {
    type: String,
    default: 'desktop'
  },
  currentTab: {
    type: String,
    default: 'home'
  }
})

const emit = defineEmits(['updateUser', 'logout', 'triggerDevModal', 'changeTab'])

// Carrossel Contínuo
const activeSlide = ref(0)
const slides = ref([
  {
    title: 'Telemedicina Integrada',
    description: 'Consulte médicos especialistas e clínicos gerais sem filas e sem agendamento direto do seu celular.',
    image: '/saude_banner.png',
    tag: 'Destaque'
  },
  {
    title: 'Clube de Descontos & Lojas',
    description: 'Até 50% de desconto em farmácias, cinemas, e-commerce e lojas parceiras de todo o Brasil.',
    image: '/clube_banner.png',
    tag: 'Economia'
  }
])

const nextSlide = () => {
  activeSlide.value = (activeSlide.value + 1) % slides.value.length
}

onMounted(() => {
  setInterval(nextSlide, 6000)
})

// Modais de Redirecionamento
const activeRedirect = ref(null)
const showRedirectModal = ref(false)

const triggerRedirect = (benefitName) => {
  activeRedirect.value = benefitName
  showRedirectModal.value = true
  setTimeout(() => {
    showRedirectModal.value = false
    emit('triggerDevModal', {
      title: 'Redirecionamento Concluído',
      message: `Você foi direcionado de forma segura e autenticada para o portal da ${benefitName}.`
    })
  }, 2000)
}

// Configurações do Perfil
const name = ref(props.user.name)
const email = ref('joao.silva@email.com')
const phone = ref('(11) 99999-7777')
const password = ref('********')

const saveProfile = () => {
  emit('updateUser', { ...props.user, name: name.value })
  emit('triggerDevModal', {
    title: 'Perfil Salvo',
    message: 'Seus dados de cadastro foram atualizados no sistema!'
  })
}

// Carteirinha Digital
const showCardModal = ref(false)

const recentActivities = ref([
  { desc: 'Consulta de Telemedicina com Clínico Geral', date: '25/06/2026 às 14:32', type: 'health' },
  { desc: 'Cupom Droga Raia resgatado (35% OFF)', date: '18/06/2026 às 09:15', type: 'clube' },
  { desc: 'Atendimento Veterinário Preventivo', date: '04/06/2026 às 11:00', type: 'pet' }
])

// Programa de Indicações (Afiliação)
import { computed } from 'vue'
const activeRefTab = ref('visaoGeral')
const refSearchName = ref('')
const refStatusFilter = ref('todos')
const refLevelFilter = ref('todos')

const rawReferrals = ref([
  { name: 'Carlos Silva', email: 'carlos@email.com', level: '1º Nível', status: 'ativo', date: '03/06/2026', gain: 'R$ 10,00' },
  { name: 'Marina Costa', email: 'marina@email.com', level: '1º Nível', status: 'pendente', date: '28/05/2026', gain: '-' },
  { name: 'João Pereira', email: 'joao@email.com', level: '1º Nível', status: 'ativo', date: '25/05/2026', gain: 'R$ 10,00' },
  { name: 'Ana Martins', email: 'ana@email.com', level: '2º Nível', status: 'ativo', date: '20/05/2026', gain: 'R$ 10,00' },
  { name: 'Pedro Santos', email: 'pedro@email.com', level: '1º Nível', status: 'inativo', date: '10/05/2026', gain: 'R$ 0,00' }
])

const filteredReferrals = computed(() => {
  return rawReferrals.value.filter(item => {
    const matchName = item.name.toLowerCase().includes(refSearchName.value.toLowerCase())
    const matchStatus = refStatusFilter.value === 'todos' || item.status === refStatusFilter.value
    const matchLevel = refLevelFilter.value === 'todos' || item.level.includes(refLevelFilter.value)
    return matchName && matchStatus && matchLevel
  })
})

const copyLink = (text) => {
  if (navigator.clipboard) {
    navigator.clipboard.writeText(text)
  }
  emit('triggerDevModal', {
    title: 'Link Copiado!',
    message: 'Link de afiliação copiado para sua área de transferência com sucesso.'
  })
}

// Lógica de Gerador de Links de Checkout
const showCreateLinkModal = ref(false)
const selectedPlan = ref('Bronze')
const refCodeInput = ref('joao-silva-123')
const selectedPayment = ref('ambos')

const userLinks = ref([
  {
    name: 'Checkout - Plano Bronze (Promocional)',
    planType: 'Bronze',
    desc: 'Link de checkout promocional',
    price: 'R$ 29,90/mês',
    payment: 'Cartão ou PIX',
    url: 'https://checkout.acessosaude.com/plano-bronze?ref=joao-silva-123',
    cliques: 845,
    conversoes: 25,
    comissao: 'R$ 2.450,00',
    status: 'Ativo'
  },
  {
    name: 'Checkout - Plano Individual',
    planType: 'Individual',
    desc: 'Link de checkout oficial da landing page',
    price: 'R$ 39,90/mês',
    payment: 'Cartão ou PIX',
    url: 'https://checkout.acessosaude.com/plano-individual?ref=joao-silva-123',
    cliques: 482,
    conversoes: 14,
    comissao: 'R$ 1.120,50',
    status: 'Ativo'
  },
  {
    name: 'Checkout - Plano Família',
    planType: 'Família',
    desc: 'Link de checkout familiar (Até 4 Vidas)',
    price: 'R$ 84,90/mês',
    payment: 'Cartão ou PIX',
    url: 'https://checkout.acessosaude.com/plano-familia?ref=joao-silva-123',
    cliques: 143,
    conversoes: 3,
    comissao: 'R$ 277,00',
    status: 'Ativo'
  }
])

const generateNewLink = () => {
  let planPrice = 'R$ 29,90/mês'
  if (selectedPlan.value === 'Individual') planPrice = 'R$ 39,90/mês'
  if (selectedPlan.value === 'Família') planPrice = 'R$ 84,90/mês'

  let paymentMethod = 'Cartão ou PIX'
  if (selectedPayment.value === 'cartao') paymentMethod = 'Apenas Cartão'
  if (selectedPayment.value === 'pix') paymentMethod = 'Apenas PIX'

  const refCode = refCodeInput.value || 'joao-silva-123'
  const finalUrl = `https://checkout.acessosaude.com/plano-${selectedPlan.value.toLowerCase()}?ref=${refCode}`

  userLinks.value.unshift({
    name: `Checkout - Plano ${selectedPlan.value}`,
    planType: selectedPlan.value,
    desc: `Link de checkout para indicação do Plano ${selectedPlan.value}`,
    price: planPrice,
    payment: paymentMethod,
    url: finalUrl,
    cliques: 0,
    conversoes: 0,
    comissao: 'R$ 0,00',
    status: 'Ativo'
  })

  showCreateLinkModal.value = false
  emit('triggerDevModal', {
    title: 'Link Gerado!',
    message: `Seu link de indicação para o Plano ${selectedPlan.value} foi gerado e adicionado à lista.`
  })
}

// Lógica de Tela de Checkout Simulado Real
const showCheckoutModal = ref(false)
const checkoutPlan = ref(null)
const checkoutSelectedPlanType = ref('Individual')
const checkoutStep = ref(1) // 1: preencher dados, 2: sucesso
const checkoutName = ref('')
const checkoutEmail = ref('')
const checkoutCpf = ref('')
const checkoutPhone = ref('')
const checkoutPaymentMethod = ref('card')

const checkoutCardNumber = ref('')
const checkoutCardName = ref('')
const checkoutCardExpiry = ref('')
const checkoutCardCvv = ref('')

const formatCPF = (e) => {
  let value = e.target.value.replace(/\D/g, '')
  if (value.length > 11) value = value.slice(0, 11)
  value = value.replace(/(\d{3})(\d)/, '$1.$2')
  value = value.replace(/(\d{3})(\d)/, '$1.$2')
  value = value.replace(/(\d{3})(\d{1,2})$/, '$1-$2')
  checkoutCpf.value = value
}

const formatPhone = (e) => {
  let value = e.target.value.replace(/\D/g, '')
  if (value.length > 11) value = value.slice(0, 11)
  value = value.replace(/^(\d{2})(\d)/g, '($1) $2')
  value = value.replace(/(\d{5})(\d)/, '$1-$2')
  checkoutPhone.value = value
}

const formatCardNumber = (e) => {
  let value = e.target.value.replace(/\D/g, '')
  if (value.length > 16) value = value.slice(0, 16)
  value = value.replace(/(\d{4})(?=\d)/g, '$1 ')
  checkoutCardNumber.value = value
}

const formatCardExpiry = (e) => {
  let value = e.target.value.replace(/\D/g, '')
  if (value.length > 4) value = value.slice(0, 4)
  if (value.length > 2) {
    value = value.replace(/(\d{2})(\d)/, '$1/$2')
  }
  checkoutCardExpiry.value = value
}

const formatCardCvv = (e) => {
  let value = e.target.value.replace(/\D/g, '')
  if (value.length > 4) value = value.slice(0, 4)
  checkoutCardCvv.value = value
}

const formatName = (e) => {
  let value = e.target.value
  checkoutName.value = value.replace(/(^\w|\s\w)/g, m => m.toUpperCase())
}

const formatEmail = (e) => {
  checkoutEmail.value = e.target.value.toLowerCase().replace(/\s/g, '')
}

const openCheckout = (linkItem) => {
  checkoutPlan.value = linkItem || {
    name: 'Checkout - Plano Individual',
    planType: 'Individual',
    desc: 'Link de checkout oficial da landing page',
    price: 'R$ 39,90/mês',
    url: 'https://checkout.acessosaude.com/plano-individual?ref=joao-silva-123'
  }
  checkoutSelectedPlanType.value = checkoutPlan.value.planType
  checkoutStep.value = 1
  checkoutName.value = ''
  checkoutEmail.value = ''
  checkoutCpf.value = ''
  checkoutPhone.value = ''
  checkoutCardNumber.value = ''
  checkoutCardName.value = ''
  checkoutCardExpiry.value = ''
  checkoutCardCvv.value = ''
  showCheckoutModal.value = true
}

const updateCheckoutPlanDetails = () => {
  let planPrice = 'R$ 39,90/mês'
  let planUrl = 'https://checkout.acessosaude.com/plano-individual?ref=joao-silva-123'
  let planDesc = 'Link de checkout oficial da landing page'
  
  if (checkoutSelectedPlanType.value === 'Bronze') {
    planPrice = 'R$ 29,90/mês'
    planUrl = 'https://checkout.acessosaude.com/plano-bronze?ref=joao-silva-123'
    planDesc = 'Link de checkout promocional'
  } else if (checkoutSelectedPlanType.value === 'Família') {
    planPrice = 'R$ 84,90/mês'
    planUrl = 'https://checkout.acessosaude.com/plano-familia?ref=joao-silva-123'
    planDesc = 'Link de checkout familiar (Até 4 Vidas)'
  }

  checkoutPlan.value = {
    name: `Checkout - Plano ${checkoutSelectedPlanType.value}`,
    planType: checkoutSelectedPlanType.value,
    desc: planDesc,
    price: planPrice,
    url: planUrl
  }
}

const finishCheckout = () => {
  checkoutStep.value = 2
  
  if (checkoutPlan.value) {
    let found = userLinks.value.find(l => l.url === checkoutPlan.value.url)
    if (!found) {
      const planPriceVal = parseFloat(checkoutPlan.value.price.replace('R$ ', '').replace('/mês', '').replace(',', '.'))
      const newComm = planPriceVal * 0.1
      
      userLinks.value.unshift({
        name: checkoutPlan.value.name,
        planType: checkoutPlan.value.planType,
        desc: checkoutPlan.value.desc,
        price: checkoutPlan.value.price,
        payment: 'Cartão ou PIX',
        url: checkoutPlan.value.url,
        cliques: 1,
        conversoes: 1,
        comissao: 'R$ ' + newComm.toLocaleString('pt-BR', { minimumFractionDigits: 2, maximumFractionDigits: 2 }),
        status: 'Ativo'
      })
    } else {
      found.conversoes++
      const currentComm = parseFloat(found.comissao.replace('R$ ', '').replace('.', '').replace(',', '.'))
      const planValue = parseFloat(found.price.replace('R$ ', '').replace('/mês', '').replace(',', '.'))
      const newComm = currentComm + (planValue * 0.1)
      found.comissao = 'R$ ' + newComm.toLocaleString('pt-BR', { minimumFractionDigits: 2, maximumFractionDigits: 2 })
    }
  }
}

// MODAIS INTERATIVOS DE LINKS
const showShareModal = ref(false)
const showReportModal = ref(false)
const showEditLinkModal = ref(false)
const selectedLink = ref(null)

// Parâmetros de Edição
const editLinkName = ref('')
const editLinkPlanType = ref('Individual')
const editLinkRefCode = ref('joao-silva-123')
const editLinkStatus = ref('Ativo')

const openShare = (linkItem) => {
  selectedLink.value = linkItem
  // Copiar link automaticamente
  if (navigator.clipboard) {
    navigator.clipboard.writeText(linkItem.url)
  }
  showShareModal.value = true
}

const openReport = (linkItem) => {
  selectedLink.value = linkItem
  showReportModal.value = true
}

const openEdit = (linkItem) => {
  selectedLink.value = linkItem
  editLinkName.value = linkItem.name
  editLinkPlanType.value = linkItem.planType
  const match = linkItem.url.match(/ref=([^&]+)/)
  editLinkRefCode.value = match ? match[1] : 'joao-silva-123'
  editLinkStatus.value = linkItem.status
  showEditLinkModal.value = true
}

const saveEditedLink = () => {
  if (selectedLink.value) {
    const found = userLinks.value.find(l => l.url === selectedLink.value.url)
    if (found) {
      let planPrice = 'R$ 29,90/mês'
      if (editLinkPlanType.value === 'Individual') planPrice = 'R$ 39,90/mês'
      if (editLinkPlanType.value === 'Família') planPrice = 'R$ 84,90/mês'

      found.name = editLinkName.value
      found.planType = editLinkPlanType.value
      found.price = planPrice
      found.status = editLinkStatus.value
      found.url = `https://checkout.acessosaude.com/plano-${editLinkPlanType.value.toLowerCase()}?ref=${editLinkRefCode.value}`
      found.desc = `Link de checkout para indicação do Plano ${editLinkPlanType.value}`
    }
  }
  showEditLinkModal.value = false
  emit('triggerDevModal', {
    title: 'Link Atualizado!',
    message: 'As alterações do link de indicação foram salvas com sucesso.'
  })
}
</script>

<template>
  <div class="dashboard-wrapper" :class="[layoutMode]">
    
    <!-- ABA 1: VISÃO GERAL (HOME) -->
    <div v-if="currentTab === 'home'" class="tab-content">
      <header class="welcome-section animated-item" style="animation-delay: 0s;">
        <div class="welcome-text">
          <h1>Olá, {{ user.name }}</h1>
          <p>Confira o andamento da sua conta e acesse seus benefícios de saúde.</p>
        </div>
        <div class="plan-pill">
          <span class="badge badge-success">Assinatura Ativa</span>
          <span class="plan-name">{{ user.plan }}</span>
        </div>
      </header>

      <!-- Slider Ampliado Contínuo (Sem piscar em branco) -->
      <section class="banner-slider animated-item" style="animation-delay: 0.1s;">
        <div class="slider-track" :style="{ transform: `translateX(-${activeSlide * 100}%)` }">
          <div 
            v-for="(slide, idx) in slides" 
            :key="idx" 
            class="slide-item"
            :style="{ backgroundImage: `linear-gradient(rgba(16, 66, 70, 0.9), rgba(16, 66, 70, 0.75)), url(${slide.image})` }"
          >
            <div class="slide-content">
              <span class="badge badge-warning">{{ slide.tag }}</span>
              <h2>{{ slide.title }}</h2>
              <p>{{ slide.description }}</p>
              <button class="btn btn-primary" @click="triggerRedirect(slide.title)">Acessar Benefício</button>
            </div>
          </div>
        </div>
        <div class="slide-indicator-container">
          <span 
            v-for="(s, idx) in slides" 
            :key="idx" 
            :class="['indicator-dot', { active: activeSlide === idx }]"
            @click="activeSlide = idx"
          ></span>
        </div>
      </section>

      <!-- Grade de Métricas -->
      <section class="metrics-grid animated-item" style="animation-delay: 0.2s;">
        <div class="metric-card card">
          <div class="metric-header">
            <i class="ph ph-piggy-bank"></i>
            <span>Economia Acumulada</span>
          </div>
          <h3>R$ 145,90</h3>
          <p>Economizados com descontos em lojas e farmácias este mês</p>
        </div>
        <div class="metric-card card">
          <div class="metric-header">
            <i class="ph ph-video-camera"></i>
            <span>Telemedicina</span>
          </div>
          <h3>Ilimitada</h3>
          <p>Consultas virtuais 24h sem custo adicional</p>
        </div>
        <div class="metric-card card">
          <div class="metric-header">
            <i class="ph ph-users"></i>
            <span>Dependentes</span>
          </div>
          <h3>3 Ativos</h3>
          <p>Incluídos na cobertura da sua conta</p>
        </div>
      </section>

      <!-- Painel Principal de Dois Lados -->
      <div class="dashboard-grid">
        <!-- Lado Esquerdo: Atalhos com entrada staggered -->
        <div class="left-side">
          <h2 class="section-title animated-item" style="animation-delay: 0.3s;">Atalhos Rápidos de Benefícios</h2>
          <div class="shortcuts-list">
            
            <div class="shortcut-card animated-item" style="animation-delay: 0.4s;" @click="triggerRedirect('Telemedicina')">
              <div class="shortcut-icon icon-teal">
                <i class="ph ph-first-aid"></i>
              </div>
              <div class="shortcut-details">
                <h3>Telemedicina</h3>
                <p>Consultas de clínico geral ou especialista por vídeo 24h</p>
              </div>
              <i class="ph ph-caret-right action-arrow"></i>
            </div>

            <div class="shortcut-card animated-item" style="animation-delay: 0.5s;" @click="triggerRedirect('Assistência Funerária')">
              <div class="shortcut-icon icon-pink">
                <i class="ph ph-heartbeat"></i>
              </div>
              <div class="shortcut-details">
                <h3>Assistência Funerária</h3>
                <p>Amparo familiar e suporte de emergência nacional</p>
              </div>
              <i class="ph ph-caret-right action-arrow"></i>
            </div>

            <div class="shortcut-card animated-item" style="animation-delay: 0.6s;" @click="triggerRedirect('Veterinário 24h')">
              <div class="shortcut-icon icon-green">
                <i class="ph ph-paw-print"></i>
              </div>
              <div class="shortcut-details">
                <h3>Veterinário 24h</h3>
                <p>Orientação e pronto-socorro veterinário a qualquer hora</p>
              </div>
              <i class="ph ph-caret-right action-arrow"></i>
            </div>

            <div class="shortcut-card animated-item" style="animation-delay: 0.7s;" @click="triggerRedirect('Clube de Descontos')">
              <div class="shortcut-icon icon-orange">
                <i class="ph ph-tag"></i>
              </div>
              <div class="shortcut-details">
                <h3>Clube de Descontos</h3>
                <p>Economia em farmácias, cinemas, lazer e lojas parceiras</p>
              </div>
              <i class="ph ph-caret-right action-arrow"></i>
            </div>

            <div class="shortcut-card animated-item" style="animation-delay: 0.8s;" @click="emit('changeTab', 'indicacoes')">
              <div class="shortcut-icon icon-teal" style="background: #e0f2fe; color: #0369a1;">
                <i class="ph ph-users-three"></i>
              </div>
              <div class="shortcut-details">
                <h3>Programa de Indicações</h3>
                <p>Indique amigos e ganhe descontos e comissões em dinheiro</p>
              </div>
              <i class="ph ph-caret-right action-arrow"></i>
            </div>

          </div>
        </div>

        <!-- Lado Direito: Carteirinha Digital e Histórico -->
        <div class="right-side">
          <!-- Carteirinha Digital -->
          <h2 class="section-title animated-item" style="animation-delay: 0.35s;">Carteirinha Digital</h2>
          <div class="digital-card-preview card animated-item" style="animation-delay: 0.45s;" @click="showCardModal = true">
            <div class="dcard-header">
              <!-- Logotipo da carteirinha em container com contraste -->
              <div class="dcard-logo-box">
                <img src="/logo.png" alt="Logo Acesso Saúde" class="dcard-logo" />
              </div>
              <span class="badge badge-success">Premium</span>
            </div>
            <div class="dcard-body">
              <h3>{{ user.name }}</h3>
              <p class="dcard-plan">Plano: {{ user.plan }}</p>
              <p class="dcard-cpf">CPF: 123.***.***-00</p>
            </div>
            <div class="dcard-footer">
              <span>Clique para ver QR Code</span>
              <i class="ph ph-qr-code"></i>
            </div>
          </div>

          <!-- Atividades Recentes com Margem Aumentada -->
          <div class="activities-wrapper">
            <h2 class="section-title animated-item" style="animation-delay: 0.55s;">Atividades Recentes</h2>
            <div class="activities-card card animated-item" style="animation-delay: 0.65s;">
              <div class="activity-item" v-for="(act, idx) in recentActivities" :key="idx">
                <div class="activity-dot" :class="act.type"></div>
                <div class="activity-text">
                  <p>{{ act.desc }}</p>
                  <span>{{ act.date }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ABA 2: MEU PERFIL -->
    <div v-if="currentTab === 'perfil'" class="tab-content animated-item" style="animation-delay: 0s;">
      <header class="tab-header">
        <h2>Meu Perfil</h2>
        <p>Atualize seus dados pessoais e de contato para comunicação.</p>
      </header>
      <div class="card form-card">
        <form @submit.prevent="saveProfile" class="profile-form">
          <div class="form-row">
            <div class="form-group flex-1">
              <label class="form-label">Nome Completo</label>
              <input v-model="name" type="text" class="form-control" />
            </div>
            <div class="form-group flex-1">
              <label class="form-label">CPF (Inalterável)</label>
              <input type="text" class="form-control" value="123.456.789-00" disabled style="background:#f1f5f9; cursor:not-allowed;" />
            </div>
          </div>
          <div class="form-row">
            <div class="form-group flex-1">
              <label class="form-label">E-mail</label>
              <input v-model="email" type="email" class="form-control" />
            </div>
            <div class="form-group flex-1">
              <label class="form-label">Celular</label>
              <input v-model="phone" type="text" class="form-control" />
            </div>
          </div>
          <button type="submit" class="btn btn-secondary">Salvar Perfil</button>
        </form>
      </div>
    </div>

    <!-- ABA 3: FINANCEIRO (Layout Melhorado e Profissional) -->
    <div v-if="currentTab === 'financeiro'" class="tab-content animated-item" style="animation-delay: 0s;">
      <header class="tab-header">
        <h2>Financeiro & Faturamento</h2>
        <p>Gerencie sua assinatura, consulte boletos/PIX ativos e histórico de cobranças.</p>
      </header>

      <div class="financial-grid">
        <!-- Detalhes do Plano e Ações Rápidas -->
        <div class="card financial-main-card">
          <div class="billing-header">
            <div>
              <span class="badge badge-success">Assinatura Ativa</span>
              <h3 class="plan-title">{{ user.plan }}</h3>
            </div>
            <div class="price-block">
              <span class="currency">R$</span>
              <span class="price-val">29,90</span>
              <span class="period">/mês</span>
            </div>
          </div>

          <div class="billing-details-list">
            <div class="detail-row">
              <span>Método de pagamento padrão:</span>
              <strong>PIX Autopagamento</strong>
            </div>
            <div class="detail-row">
              <span>Próxima cobrança automática:</span>
              <strong>01/08/2026</strong>
            </div>
            <div class="detail-row">
              <span>Cadastrado em:</span>
              <strong>15/03/2026</strong>
            </div>
          </div>

          <div class="billing-actions">
            <button class="btn btn-primary" @click="emit('triggerDevModal', { title: 'Segunda Via', message: 'Gerando a 2ª via da fatura do mês atual...' })">
              <i class="ph ph-barcode"></i> Gerar Boleto
            </button>
            <button class="btn btn-outline" @click="emit('triggerDevModal', { title: 'PIX Copia e Cola', message: 'Gerando chave de pagamento PIX...' })">
              <i class="ph ph-qr-code"></i> Pagar via PIX
            </button>
          </div>
        </div>

        <!-- Histórico e faturamento anteriores -->
        <div class="card financial-history-card">
          <h3>Histórico de Mensalidades</h3>
          <div class="invoices-list-v2">
            <div class="invoice-item-v2">
              <div class="inv-info">
                <i class="ph ph-check-circle text-green icon-large"></i>
                <div>
                  <strong>Mensalidade Julho/2026</strong>
                  <span>Pago em 01/07/2026 às 08:24 via PIX</span>
                </div>
              </div>
              <div class="inv-value">
                <span>R$ 29,90</span>
                <button class="btn-receipt-download" @click="emit('triggerDevModal', { title: 'Recibo de Pagamento', message: 'Fazendo o download do recibo oficial da mensalidade de Julho...' })">
                  <i class="ph ph-download"></i> Recibo
                </button>
              </div>
            </div>

            <div class="invoice-item-v2">
              <div class="inv-info">
                <i class="ph ph-check-circle text-green icon-large"></i>
                <div>
                  <strong>Mensalidade Junho/2026</strong>
                  <span>Pago em 01/06/2026 às 10:15 via Cartão</span>
                </div>
              </div>
              <div class="inv-value">
                <span>R$ 29,90</span>
                <button class="btn-receipt-download" @click="emit('triggerDevModal', { title: 'Recibo de Pagamento', message: 'Fazendo o download do recibo oficial da mensalidade de Junho...' })">
                  <i class="ph ph-download"></i> Recibo
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ABA 4: CONFIGURAÇÕES (Enriquecida) -->
    <div v-if="currentTab === 'configuracoes'" class="tab-content animated-item" style="animation-delay: 0s;">
      <header class="tab-header">
        <h2>Configurações da Conta</h2>
        <p>Gerencie seus dados de acesso, e-mail, notificações e privacidade.</p>
      </header>

      <div class="config-grid">
        <!-- Alterar E-mail -->
        <div class="card config-section-card">
          <h3>Dados de Acesso (E-mail)</h3>
          <p class="config-desc">Atualize o e-mail que você utiliza para fazer login e receber avisos.</p>
          <form @submit.prevent="emit('triggerDevModal', { title: 'E-mail Atualizado', message: 'Um link de validação foi enviado para seu novo endereço de e-mail.' })" class="config-form">
            <div class="form-group">
              <label class="form-label">E-mail Atual</label>
              <input type="email" class="form-control" value="joao.silva@email.com" disabled style="background:#f1f5f9; cursor:not-allowed;" />
            </div>
            <div class="form-group">
              <label class="form-label">Novo E-mail</label>
              <input v-model="email" type="email" class="form-control" placeholder="digite o novo e-mail" required />
            </div>
            <button type="submit" class="btn btn-secondary">Alterar E-mail</button>
          </form>
        </div>

        <!-- Alterar Senha -->
        <div class="card config-section-card">
          <h3>Segurança (Senha)</h3>
          <p class="config-desc">Mantenha sua conta protegida alterando sua senha regularmente.</p>
          <form @submit.prevent="emit('triggerDevModal', { title: 'Senha Atualizada', message: 'Sua senha de acesso foi modificada com sucesso!' })" class="config-form">
            <div class="form-group">
              <label class="form-label">Senha Atual</label>
              <input type="password" class="form-control" placeholder="••••••••" required />
            </div>
            <div class="form-group">
              <label class="form-label">Nova Senha</label>
              <input type="password" class="form-control" required />
            </div>
            <button type="submit" class="btn btn-outline">Atualizar Senha</button>
          </form>
        </div>

        <!-- Preferências de Notificação -->
        <div class="card config-section-card">
          <h3>Notificações</h3>
          <p class="config-desc">Escolha quais canais você deseja utilizar para receber alertas de consultas e faturamento.</p>
          <div class="notification-options">
            <div class="notification-row">
              <div class="notif-text">
                <strong>Notificações por E-mail</strong>
                <p>Receba faturas e lembretes médicos por e-mail</p>
              </div>
              <label class="switch">
                <input type="checkbox" checked />
                <span class="slider"></span>
              </label>
            </div>
            <div class="notification-row">
              <div class="notif-text">
                <strong>Alertas de WhatsApp</strong>
                <p>Receba receitas e confirmações de consultas pelo WhatsApp</p>
              </div>
              <label class="switch">
                <input type="checkbox" checked />
                <span class="slider"></span>
              </label>
            </div>
            <div class="notification-row">
              <div class="notif-text">
                <strong>SMS de Emergência</strong>
                <p>Receba avisos urgentes no celular via SMS</p>
              </div>
              <label class="switch">
                <input type="checkbox" />
                <span class="slider"></span>
              </label>
            </div>
          </div>
        </div>

        <!-- Privacidade e LGPD -->
        <div class="card config-section-card">
          <h3>Privacidade & Segurança Adicional</h3>
          <p class="config-desc">Gerencie a segurança dos seus dados de saúde e conta.</p>
          <div class="notification-options">
            <div class="notification-row">
              <div class="notif-text">
                <strong>Autenticação em Duas Etapas (2FA)</strong>
                <p>Exigir código extra no celular para realizar login</p>
              </div>
              <label class="switch">
                <input type="checkbox" />
                <span class="slider"></span>
              </label>
            </div>
            <div class="notification-row">
              <div class="notif-text">
                <strong>Anonimizar Dados de Saúde</strong>
                <p>Compartilhar dados de forma anônima para fins de pesquisas médicas</p>
              </div>
              <label class="switch">
                <input type="checkbox" checked />
                <span class="slider"></span>
              </label>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ABA 5: PROGRAMA DE INDICAÇÕES -->
    <div v-if="currentTab === 'indicacoes'" class="tab-content animated-item" style="animation-delay: 0s;">
      
      <!-- Sub-Abas do Programa de Indicações -->
      <nav class="referral-tabs">
        <button 
          :class="['ref-tab-btn', { active: activeRefTab === 'visaoGeral' }]"
          @click="activeRefTab = 'visaoGeral'"
        >
          <i class="ph ph-squares-four"></i> Visão Geral
        </button>
        <button 
          :class="['ref-tab-btn', { active: activeRefTab === 'indicados' }]"
          @click="activeRefTab = 'indicados'"
        >
          <i class="ph ph-users"></i> Meus Indicados
        </button>
        <button 
          :class="['ref-tab-btn', { active: activeRefTab === 'financeiroRef' }]"
          @click="activeRefTab = 'financeiroRef'"
        >
          <i class="ph ph-hand-coins"></i> Financeiro
        </button>
        <button 
          :class="['ref-tab-btn', { active: activeRefTab === 'links' }]"
          @click="activeRefTab = 'links'"
        >
          <i class="ph ph-link"></i> Meus Links
        </button>
      </nav>

      <!-- SUB-ABA 1: VISÃO GERAL -->
      <div v-if="activeRefTab === 'visaoGeral'" class="ref-sub-content">
        <header class="tab-header animated-item" style="animation-delay: 0.05s;">
          <h2>Seu Programa de Indicações</h2>
          <p>Acompanhe seus ganhos, indicados e performance em tempo real.</p>
        </header>

        <!-- Cards de Resumo de Ganhos/Indicados -->
        <section class="metrics-grid">
          <div class="metric-card card animated-item" style="animation-delay: 0.1s;">
            <div class="metric-header">
              <i class="ph ph-coins" style="color: #15803d; background: #dcfce7; padding: 8px; border-radius: var(--radius-sm);"></i>
              <span>GANHOS TOTAIS</span>
            </div>
            <h3>R$ 3.847,50</h3>
            <p>Desde que começou</p>
          </div>
          <div class="metric-card card animated-item" style="animation-delay: 0.15s;">
            <div class="metric-header">
              <i class="ph ph-calendar" style="color: #1d4ed8; background: #dbeafe; padding: 8px; border-radius: var(--radius-sm);"></i>
              <span>ESTE MÊS</span>
            </div>
            <h3>R$ 487,50</h3>
            <p>Será descontado da fatura</p>
          </div>
          <div class="metric-card card animated-item" style="animation-delay: 0.2s;">
            <div class="metric-header">
              <i class="ph ph-users" style="color: #6d28d9; background: #ede9fe; padding: 8px; border-radius: var(--radius-sm);"></i>
              <span>TOTAL INDICADOS</span>
            </div>
            <h3>42</h3>
            <p>Pessoas na sua rede</p>
          </div>
          <div class="metric-card card animated-item" style="animation-delay: 0.25s;">
            <div class="metric-header">
              <i class="ph ph-trend-up" style="color: #b45309; background: #fef3c7; padding: 8px; border-radius: var(--radius-sm);"></i>
              <span>TAXA DE ATIVAÇÃO</span>
            </div>
            <h3>88%</h3>
            <p>Pessoas ativas usando</p>
          </div>
        </section>

        <!-- Grid de Crescimento e Ganhos por Nível -->
        <div class="dashboard-grid" style="margin-top: 24px;">
          <!-- Lado Esquerdo: Crescimento da Rede -->
          <div class="card animated-item" style="padding: 24px; animation-delay: 0.3s;">
            <h3 style="font-size: 18px; color: var(--secondary); margin-bottom: 20px;">Crescimento da Rede</h3>
            
            <div class="level-row">
              <div class="level-row-header">
                <div class="level-label">
                  <span class="level-badge lvl-1">1</span>
                  <span>1º Nível - Indicações diretas</span>
                </div>
                <span class="level-count">15 pessoas</span>
              </div>
              <div class="progress-track">
                <div class="progress-bar lvl-1" style="width: 100%;"></div>
              </div>
              <span class="level-footer">100% ativas • R$ 150,00/mês</span>
            </div>

            <div class="level-row">
              <div class="level-row-header">
                <div class="level-label">
                  <span class="level-badge lvl-2">2</span>
                  <span>2º Nível - Indicações indiretas</span>
                </div>
                <span class="level-count">22 pessoas</span>
              </div>
              <div class="progress-track">
                <div class="progress-bar lvl-2" style="width: 82%;"></div>
              </div>
              <span class="level-footer">82% ativas • R$ 220,00/mês</span>
            </div>

            <div class="level-row">
              <div class="level-row-header">
                <div class="level-label">
                  <span class="level-badge lvl-3">3</span>
                  <span>3º Nível - Indicações do 2º nível</span>
                </div>
                <span class="level-count">5 pessoas</span>
              </div>
              <div class="progress-track">
                <div class="progress-bar lvl-3" style="width: 60%;"></div>
              </div>
              <span class="level-footer">60% ativas • R$ 117,50/mês</span>
            </div>
          </div>

          <!-- Lado Direito: Ganhos por Nível -->
          <div class="card animated-item" style="padding: 24px; display: flex; flex-direction: column; gap: 16px; animation-delay: 0.35s;">
            <h3 style="font-size: 18px; color: var(--secondary);">Ganhos por Nível</h3>
            
            <div style="background: #eff6ff; border-left: 4px solid #2563eb; padding: 12px 16px; border-radius: var(--radius-sm); display: flex; justify-content: space-between; align-items: center;">
              <div>
                <strong style="color: #1e3a8a; font-size: 14px; display:block;">1º Nível</strong>
                <span style="font-size: 12px; color: #1e40af;">15 pessoas × R$ 10</span>
              </div>
              <strong style="color: #1d4ed8;">R$ 150</strong>
            </div>

            <div style="background: #faf5ff; border-left: 4px solid #7c3aed; padding: 12px 16px; border-radius: var(--radius-sm); display: flex; justify-content: space-between; align-items: center;">
              <div>
                <strong style="color: #4c1d95; font-size: 14px; display:block;">2º Nível</strong>
                <span style="font-size: 12px; color: #5b21b6;">22 pessoas × R$ 10</span>
              </div>
              <strong style="color: #6d28d9;">R$ 220</strong>
            </div>

            <div style="background: #fff7ed; border-left: 4px solid #ea580c; padding: 12px 16px; border-radius: var(--radius-sm); display: flex; justify-content: space-between; align-items: center;">
              <div>
                <strong style="color: #7c2d12; font-size: 14px; display:block;">3º Nível</strong>
                <span style="font-size: 12px; color: #9a3412;">5 pessoas × R$ 23,50</span>
              </div>
              <strong style="color: #c2410c;">R$ 117,50</strong>
            </div>

            <div style="background: #f0fdf4; padding: 16px; border-radius: var(--radius-sm); text-align: center; border: 1px solid #bbf7d0; margin-top: auto;">
              <span style="font-size: 12px; color: var(--text-gray); display:block; margin-bottom: 4px;">Total este mês</span>
              <strong style="font-size: 24px; color: #166534;">R$ 487,50</strong>
            </div>
          </div>
        </div>

        <!-- Últimas Indicações -->
        <div class="card animated-item" style="padding: 24px; margin-top: 24px; animation-delay: 0.4s;">
          <h3 style="font-size: 18px; color: var(--secondary); margin-bottom: 16px;">Últimas Indicações</h3>
          <div class="activities-list" style="display: flex; flex-direction: column; gap: 16px;">
            <div class="activity-item" style="border-bottom: 1px solid var(--border-color); padding-bottom: 12px;">
              <div class="user-avatar-mini">CS</div>
              <div>
                <strong style="color: var(--text-dark); display:block; font-size: 14px;">Carlos Silva</strong>
                <span style="font-size: 12px; color: var(--text-gray);">Indicado há 3 dias • 1º Nível • <span class="badge badge-success" style="font-size:10px; padding: 2px 6px;">Ativo</span></span>
              </div>
            </div>
            <div class="activity-item" style="border-bottom: 1px solid var(--border-color); padding-bottom: 12px;">
              <div class="user-avatar-mini">MC</div>
              <div>
                <strong style="color: var(--text-dark); display:block; font-size: 14px;">Marina Costa</strong>
                <span style="font-size: 12px; color: var(--text-gray);">Indicada há 5 dias • 1º Nível • <span class="badge badge-warning" style="font-size:10px; padding: 2px 6px;">Pendente</span></span>
              </div>
            </div>
            <div class="activity-item">
              <div class="user-avatar-mini">JP</div>
              <div>
                <strong style="color: var(--text-dark); display:block; font-size: 14px;">João Pereira</strong>
                <span style="font-size: 12px; color: var(--text-gray);">Indicado há 7 dias • 1º Nível • <span class="badge badge-success" style="font-size:10px; padding: 2px 6px;">Ativo</span></span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- SUB-ABA 2: MEUS INDICADOS -->
      <div v-if="activeRefTab === 'indicados'" class="ref-sub-content">
        <header class="tab-header animated-item" style="animation-delay: 0.05s;">
          <h2>Meus Indicados</h2>
          <p>Lista completa de todas as pessoas que você indicou.</p>
        </header>

        <!-- Filtros -->
        <div class="card animated-item" style="padding: 16px; margin-bottom: 24px; display: flex; gap: 12px; align-items: center; flex-wrap: wrap; animation-delay: 0.1s;">
          <input 
            v-model="refSearchName" 
            type="text" 
            placeholder="Buscar por nome..." 
            class="form-control" 
            style="flex: 1; min-width: 200px;" 
          />
          <select v-model="refStatusFilter" class="form-control" style="width: auto; min-width: 150px;">
            <option value="todos">Todos os status</option>
            <option value="ativo">Ativo</option>
            <option value="pendente">Pendente</option>
            <option value="inativo">Inativo</option>
          </select>
          <select v-model="refLevelFilter" class="form-control" style="width: auto; min-width: 150px;">
            <option value="todos">Todos os níveis</option>
            <option value="1">1º Nível</option>
            <option value="2">2º Nível</option>
            <option value="3">3º Nível</option>
          </select>
          <button class="btn btn-outline" @click="refSearchName = ''; refStatusFilter = 'todos'; refLevelFilter = 'todos';">
            Limpar Filtros
          </button>
        </div>

        <!-- Tabela -->
        <div class="card animated-item" style="overflow-x: auto; padding: 0; animation-delay: 0.15s;">
          <table class="referral-table">
            <thead>
              <tr>
                <th>Nome</th>
                <th>Email</th>
                <th>Nível</th>
                <th>Status</th>
                <th>Data</th>
                <th>Ganho/Mês</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(refItem, index) in filteredReferrals" :key="index" class="animated-item" :style="{ 'animation-delay': (0.2 + index * 0.05) + 's' }">
                <td>
                  <div class="referral-user">
                    <div class="user-avatar-mini">{{ refItem.name.split(' ').map(n=>n[0]).join('') }}</div>
                    <strong style="color: var(--text-dark);">{{ refItem.name }}</strong>
                  </div>
                </td>
                <td>{{ refItem.email }}</td>
                <td><span class="badge badge-outline" style="font-size:11px;">{{ refItem.level }}</span></td>
                <td>
                  <span :class="['status-badge-ref', refItem.status]">
                    {{ refItem.status.charAt(0).toUpperCase() + refItem.status.slice(1) }}
                  </span>
                </td>
                <td>{{ refItem.date }}</td>
                <td :style="{ color: refItem.gain !== '-' && refItem.gain !== 'R$ 0,00' ? '#16a34a' : 'inherit', fontWeight: 'bold' }">
                  {{ refItem.gain }}
                </td>
              </tr>
              <tr v-if="filteredReferrals.length === 0">
                <td colspan="6" style="text-align: center; padding: 24px; color: var(--text-gray);">
                  Nenhum indicado encontrado com os filtros aplicados.
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- SUB-ABA 3: FINANCEIRO -->
      <div v-if="activeRefTab === 'financeiroRef'" class="ref-sub-content">
        <header class="tab-header animated-item" style="animation-delay: 0.05s;">
          <h2>Histórico Financeiro</h2>
          <p>Acompanhe todas as suas comissões e ganhos obtidos através do programa.</p>
        </header>

        <section class="metrics-grid">
          <div class="metric-card card animated-item" style="animation-delay: 0.1s;">
            <div class="metric-header">
              <i class="ph ph-hand-coins" style="color: #15803d; background: #dcfce7; padding: 8px; border-radius: var(--radius-sm);"></i>
              <span>GANHOS ACUMULADOS</span>
            </div>
            <h3>R$ 3.847,50</h3>
            <p>Desde janeiro de 2026</p>
          </div>
          <div class="metric-card card animated-item" style="animation-delay: 0.15s;">
            <div class="metric-header">
              <i class="ph ph-ticket" style="color: #1d4ed8; background: #dbeafe; padding: 8px; border-radius: var(--radius-sm);"></i>
              <span>DESCONTOS APLICADOS</span>
            </div>
            <h3>R$ 2.847,50</h3>
            <p>Já abatidos nas faturas</p>
          </div>
          <div class="metric-card card animated-item" style="animation-delay: 0.2s;">
            <div class="metric-header">
              <i class="ph ph-clock" style="color: #6d28d9; background: #ede9fe; padding: 8px; border-radius: var(--radius-sm);"></i>
              <span>SALDO A RECEBER</span>
            </div>
            <h3>R$ 1.000,00</h3>
            <p>A ser pago nos próximos 2 meses</p>
          </div>
        </section>

        <!-- Lista de Ganhos por Mês -->
        <div class="card animated-item" style="padding: 24px; margin-top: 24px; animation-delay: 0.25s;">
          <h3 style="font-size: 18px; color: var(--secondary); margin-bottom: 20px;">Ganhos por Mês</h3>
          <div class="invoices-list-v2">
            
            <div class="invoice-item-v2 animated-item" style="animation-delay: 0.3s;">
              <div class="inv-info">
                <i class="ph ph-calendar text-teal icon-large"></i>
                <div>
                  <strong>Julho 2026</strong>
                  <span>Comissão calculada • Mês atual</span>
                </div>
              </div>
              <div class="inv-value">
                <span style="color: #16a34a;">R$ 487,50</span>
                <span class="badge badge-success" style="font-size: 11px;">Será descontado</span>
              </div>
            </div>

            <div class="invoice-item-v2 animated-item" style="animation-delay: 0.35s;">
              <div class="inv-info">
                <i class="ph ph-calendar text-teal icon-large"></i>
                <div>
                  <strong>Junho 2026</strong>
                  <span>Abatido na fatura de Junho</span>
                </div>
              </div>
              <div class="inv-value">
                <span style="color: #16a34a;">R$ 450,00</span>
                <span class="badge badge-outline" style="font-size: 11px;">Descontado</span>
              </div>
            </div>

            <div class="invoice-item-v2 animated-item" style="animation-delay: 0.4s;">
              <div class="inv-info">
                <i class="ph ph-calendar text-teal icon-large"></i>
                <div>
                  <strong>Maio 2026</strong>
                  <span>Abatido na fatura de Maio</span>
                </div>
              </div>
              <div class="inv-value">
                <span style="color: #16a34a;">R$ 420,00</span>
                <span class="badge badge-outline" style="font-size: 11px;">Descontado</span>
              </div>
            </div>

          </div>
        </div>
      </div>

      <!-- SUB-ABA 4: MEUS LINKS (Dinâmico) -->
      <div v-if="activeRefTab === 'links'" class="ref-sub-content">
        <header class="tab-header animated-item" style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 24px; flex-wrap: wrap; gap: 16px; animation-delay: 0.05s;">
          <div>
            <h2>Seus Links de Indicação</h2>
            <p style="margin-bottom: 0;">Crie, gerencie e acompanhe a performance dos seus links de divulgação.</p>
          </div>
          <button class="btn btn-secondary" @click="openCheckout(null)">
            <i class="ph ph-plus"></i> Criar Novo Link
          </button>
        </header>

        <div v-for="(linkItem, index) in userLinks" :key="index" class="card link-sharing-card animated-item" :style="{ 'animation-delay': (0.15 + index * 0.1) + 's' }">
          <div style="display:flex; justify-content:space-between; align-items:flex-start; margin-bottom: 12px; flex-wrap: wrap; gap: 8px;">
            <div>
              <h4 style="font-size: 17px; font-weight: 700; color: var(--secondary); margin-bottom: 2px;">{{ linkItem.name }}</h4>
              <p style="font-size: 12px; color: var(--text-gray); margin-bottom: 0;">{{ linkItem.desc }}</p>
            </div>
            <div style="text-align: right;">
              <span class="badge badge-success" style="font-size: 11px; margin-bottom: 4px; display: inline-block;">{{ linkItem.status }}</span>
              <div style="font-size: 12px; color: var(--text-gray);">Pagamento: <strong>{{ linkItem.payment }}</strong></div>
            </div>
          </div>

          <div class="link-input-group">
            <input type="text" class="link-input" :value="linkItem.url" readonly />
            <button class="btn btn-primary" @click="copyLink(linkItem.url)">
              <i class="ph ph-copy"></i> Copiar
            </button>
          </div>

          <div class="sharing-metrics">
            <div class="sharing-metric-box">
              <span>Valor do Plano</span>
              <strong style="color: var(--secondary);">{{ linkItem.price }}</strong>
            </div>
            <div class="sharing-metric-box">
              <span>Cliques</span>
              <strong>{{ linkItem.cliques }}</strong>
            </div>
            <div class="sharing-metric-box">
              <span>Conversões</span>
              <strong>{{ linkItem.conversoes }}</strong>
            </div>
            <div class="sharing-metric-box">
              <span>Comissão Gerada</span>
              <strong style="color: #16a34a;">{{ linkItem.comissao }}</strong>
            </div>
          </div>

          <div class="sharing-actions" style="display:flex; gap:12px; flex-wrap:wrap;">
            <button class="btn btn-outline" style="flex:1; min-width: 120px;" @click="openShare(linkItem)"><i class="ph ph-share"></i> Compartilhar</button>
            <button class="btn btn-outline" style="flex:1; min-width: 120px;" @click="openReport(linkItem)"><i class="ph ph-chart-bar"></i> Ver Relatório</button>
            <button class="btn btn-outline" style="flex:1; min-width: 120px;" @click="openEdit(linkItem)"><i class="ph ph-note-pencil"></i> Editar</button>
          </div>
        </div>
      </div>

    </div>

    <!-- TELA DE CHECKOUT REAL SIMULADO -->
    <div v-if="showCheckoutModal" class="checkout-overlay" @click.self="showCheckoutModal = false">
      <div class="checkout-box">
        <header class="checkout-header">
          <h3><i class="ph ph-shield-check text-green"></i> Checkout Seguro - Acesso Saúde</h3>
          <div class="modal-close" @click="showCheckoutModal = false" style="position:static; cursor:pointer;"><i class="ph ph-x"></i></div>
        </header>

        <div class="checkout-body">
          <!-- Passo 1: Informações e Pagamento -->
          <div v-if="checkoutStep === 1" class="checkout-grid">
            <div class="checkout-left">
              <form @submit.prevent="finishCheckout">
                <div class="checkout-section-title">1. Plano de Assinatura</div>
                <div class="form-group" style="margin-bottom: 20px;">
                  <label class="form-label" style="color: var(--text-dark) !important;">Selecione o Plano</label>
                  <select v-model="checkoutSelectedPlanType" class="form-control" @change="updateCheckoutPlanDetails" required>
                    <option value="Bronze">Plano Bronze (Promocional) — R$ 29,90/mês</option>
                    <option value="Individual">Plano Individual — R$ 39,90/mês</option>
                    <option value="Família">Plano Família — R$ 84,90/mês</option>
                  </select>
                </div>

                <div class="checkout-section-title">2. Informações Pessoais</div>
                <div class="form-group" style="margin-bottom: 12px;">
                  <label class="form-label" style="color: var(--text-dark) !important;">Nome Completo</label>
                  <input v-model="checkoutName" type="text" class="form-control" placeholder="Digite seu nome completo" @input="formatName" required />
                </div>
                <div class="form-group" style="margin-bottom: 12px;">
                  <label class="form-label" style="color: var(--text-dark) !important;">E-mail</label>
                  <input v-model="checkoutEmail" type="email" class="form-control" placeholder="nome@exemplo.com" @input="formatEmail" required />
                </div>
                <div style="display:grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 20px;">
                  <div class="form-group">
                    <label class="form-label" style="color: var(--text-dark) !important;">CPF</label>
                    <input v-model="checkoutCpf" type="text" class="form-control" placeholder="000.000.000-00" @input="formatCPF" required />
                  </div>
                  <div class="form-group">
                    <label class="form-label" style="color: var(--text-dark) !important;">Celular</label>
                    <input v-model="checkoutPhone" type="tel" class="form-control" placeholder="(00) 00000-0000" @input="formatPhone" required />
                  </div>
                </div>

                <div class="checkout-section-title">3. Forma de Pagamento</div>
                <div class="payment-methods-select">
                  <div :class="['pay-select-card', { active: checkoutPaymentMethod === 'card' }]" @click="checkoutPaymentMethod = 'card'">
                    <i class="ph ph-credit-card"></i> Cartão de Crédito
                  </div>
                  <div :class="['pay-select-card', { active: checkoutPaymentMethod === 'pix' }]" @click="checkoutPaymentMethod = 'pix'">
                    <i class="ph ph-qr-code"></i> PIX à Vista
                  </div>
                </div>

                <!-- Formulário de Cartão -->
                <div v-if="checkoutPaymentMethod === 'card'" style="display:flex; flex-direction:column; gap:12px;">
                  <div class="form-group">
                    <label class="form-label">Número do Cartão</label>
                    <input v-model="checkoutCardNumber" type="text" class="form-control" placeholder="4444 5555 6666 7777" @input="formatCardNumber" required />
                  </div>
                  <div class="form-group">
                    <label class="form-label">Nome do Titular (como no cartão)</label>
                    <input v-model="checkoutCardName" type="text" class="form-control" placeholder="JOAO H SILVA" required />
                  </div>
                  <div style="display:grid; grid-template-columns: 1fr 1fr; gap: 12px;">
                    <div class="form-group">
                      <label class="form-label">Validade</label>
                      <input v-model="checkoutCardExpiry" type="text" class="form-control" placeholder="MM/AA" @input="formatCardExpiry" required />
                    </div>
                    <div class="form-group">
                      <label class="form-label">CVV</label>
                      <input v-model="checkoutCardCvv" type="text" class="form-control" placeholder="123" @input="formatCardCvv" required />
                    </div>
                  </div>
                </div>

                <!-- Formulário de Pix -->
                <div v-else style="background: var(--bg-gray); padding: 16px; border-radius: var(--radius-sm); border: 1px dashed var(--border-color); text-align: center;">
                  <i class="ph ph-qr-code" style="font-size: 80px; color: var(--secondary); display:block; margin-bottom: 8px;"></i>
                  <strong style="color: var(--secondary); display:block; margin-bottom: 4px;">PIX Copia e Cola disponível após confirmar</strong>
                  <p style="font-size: 12px; color: var(--text-gray); margin-bottom: 0;">Liberação imediata dos seus benefícios de telemedicina.</p>
                </div>

                <button type="submit" class="btn btn-secondary btn-full" style="margin-top: 24px; font-weight: 700; height: 48px;">
                  Confirmar e Ativar Plano
                </button>
              </form>
            </div>

            <!-- Coluna Direita: Resumo do Pedido -->
            <div class="checkout-right" v-if="checkoutPlan">
              <div class="order-summary-box">
                <h4 style="font-size: 15px; color: var(--secondary); margin-bottom: 16px; border-bottom: 1px solid var(--border-color); padding-bottom: 8px;">Resumo do Pedido</h4>
                <div class="summary-row">
                  <span>Plano Selecionado:</span>
                  <strong>{{ checkoutPlan.planType }}</strong>
                </div>
                <div class="summary-row">
                  <span>Periodicidade:</span>
                  <strong>Mensal</strong>
                </div>
                <div class="summary-row">
                  <span>Forma de Pagamento:</span>
                  <strong>{{ checkoutPaymentMethod === 'card' ? 'Cartão de Crédito' : 'PIX' }}</strong>
                </div>
                <div class="summary-row total">
                  <span>Total a Pagar:</span>
                  <span>{{ checkoutPlan.price }}</span>
                </div>

                <div style="margin-top: 20px; text-align: center; font-size: 12px; color: var(--text-gray);">
                  <i class="ph ph-lock-key" style="margin-right: 4px;"></i> Ambiente 100% criptografado e seguro.
                </div>
              </div>
            </div>
          </div>

          <!-- Passo 2: Sucesso -->
          <div v-else class="checkout-success-view">
            <i class="ph ph-check-circle success-icon-check" style="font-size: 64px; color: var(--primary); display:block; margin-bottom: 16px;"></i>
            <h2 style="color: var(--secondary); font-size: 24px; font-weight: 800; margin-bottom: 8px;">Parabéns! Assinatura Confirmada!</h2>
            <p style="color: var(--text-gray); font-size: 14px; max-width: 500px; margin: 0 auto 24px; line-height: 1.6;">
              Obrigado, <strong>{{ checkoutName }}</strong>! Seu plano <strong>{{ checkoutPlan?.planType }}</strong> foi ativado com sucesso. Os dados de login e instruções de telemedicina foram enviados para <strong>{{ checkoutEmail }}</strong>.
            </p>
            <button class="btn btn-secondary" style="min-width: 180px;" @click="showCheckoutModal = false">
              Voltar ao Portal
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- MODAL DE COMPARTILHAMENTO -->
    <div v-if="showShareModal && selectedLink" class="sso-overlay-custom" @click.self="showShareModal = false">
      <div class="card-details-modal" style="max-width: 480px;">
        <div class="modal-close" @click="showShareModal = false"><i class="ph ph-x"></i></div>
        <h3 style="font-size: 20px; color: #ffffff !important; margin-bottom: 8px;">Compartilhar Link</h3>
        <p style="font-size: 13px; color: #cbd5e1 !important; margin-bottom: 20px;">
          O link foi copiado para sua área de transferência! Escolha um canal abaixo para enviar:
        </p>

        <div style="display:flex; flex-direction:column; gap:16px;">
          <!-- Campo Copiar Link -->
          <div class="link-input-group">
            <input type="text" class="link-input" :value="selectedLink.url" readonly />
            <button class="btn btn-primary" @click="copyLink(selectedLink.url)">
              <i class="ph ph-copy"></i> Copiar
            </button>
          </div>

          <!-- Canais Rápidos -->
          <div style="display:grid; grid-template-columns: 1fr 1fr; gap:12px;">
            <a :href="`https://api.whatsapp.com/send?text=Olá! Veja as vantagens do Acesso Saúde: ${selectedLink.url}`" target="_blank" class="btn btn-outline" style="display:flex; align-items:center; justify-content:center; gap:8px; text-decoration:none; background:#22c55e; color:white; border-color:#22c55e; font-weight:600;">
              <i class="ph ph-whatsapp-logo" style="font-size: 20px;"></i> WhatsApp
            </a>
            <a :href="`mailto:?subject=Indicação Acesso Saúde&body=Olá! Conheça o plano do Acesso Saúde que estou te indicando: ${selectedLink.url}`" class="btn btn-outline" style="display:flex; align-items:center; justify-content:center; gap:8px; text-decoration:none; color:white; font-weight:600;">
              <i class="ph ph-envelope" style="font-size: 20px;"></i> E-mail
            </a>
          </div>

          <!-- QR Code Simulado -->
          <div style="background: white; border-radius: var(--radius-md); padding: 16px; border: 1px solid var(--border-color); text-align: center; margin-top: 8px;">
            <i class="ph ph-qr-code" style="font-size: 100px; color: #0f172a !important; display:block; margin: 0 auto 8px;"></i>
            <span style="font-size: 12px; color: #475569 !important; font-weight:600;">QR Code de Afiliado</span>
          </div>
        </div>
      </div>
    </div>

    <!-- MODAL DE RELATÓRIO DO LINK -->
    <div v-if="showReportModal && selectedLink" class="sso-overlay-custom" @click.self="showReportModal = false">
      <div class="card-details-modal" style="max-width: 520px;">
        <div class="modal-close" @click="showReportModal = false"><i class="ph ph-x"></i></div>
        <h3 style="font-size: 20px; color: #ffffff !important; margin-bottom: 4px;">Relatório de Desempenho</h3>
        <p style="font-size: 13px; color: #cbd5e1 !important; margin-bottom: 20px;">
          Métricas de conversão para: <strong>{{ selectedLink.name }}</strong>
        </p>

        <div style="display:flex; flex-direction:column; gap:20px;">
          <!-- Grid de Métricas -->
          <div style="display:grid; grid-template-columns: 1fr 1fr; gap:12px;">
            <div style="background:var(--bg-gray); padding:16px; border-radius:var(--radius-md); border:1px solid var(--border-color); text-align:center;">
              <span style="font-size: 12px; color: #475569 !important; display:block; margin-bottom: 4px; font-weight:500;">Cliques Totais</span>
              <strong style="font-size: 24px; color: #0f172a !important;">{{ selectedLink.cliques }}</strong>
            </div>
            <div style="background:var(--bg-gray); padding:16px; border-radius:var(--radius-md); border:1px solid var(--border-color); text-align:center;">
              <span style="font-size: 12px; color: #475569 !important; display:block; margin-bottom: 4px; font-weight:500;">Conversões</span>
              <strong style="font-size: 24px; color: #16a34a;">{{ selectedLink.conversoes }}</strong>
            </div>
          </div>

          <!-- Detalhamento de Comissões -->
          <div style="background:var(--bg-gray); padding:16px; border-radius:var(--radius-md); border:1px solid var(--border-color);">
            <div style="display:flex; justify-content:space-between; margin-bottom: 8px;">
              <span style="font-size: 13px; color: #475569 !important; font-weight:500;">Comissão Gerada:</span>
              <strong style="font-size: 14px; color: #16a34a;">{{ selectedLink.comissao }}</strong>
            </div>
            <div style="display:flex; justify-content:space-between;">
              <span style="font-size: 13px; color: #475569 !important; font-weight:500;">Taxa de Conversão:</span>
              <strong style="font-size: 14px; color: #0f172a !important;">{{ selectedLink.cliques > 0 ? ((selectedLink.conversoes / selectedLink.cliques) * 100).toFixed(1) : '0.0' }}%</strong>
            </div>
          </div>

          <!-- Histórico de Conversões Simuladas -->
          <div>
            <h4 style="font-size: 14px; color: #f1f5f9 !important; margin-bottom: 12px; font-weight:600;">Histórico de Clientes Convertidos</h4>
            <div v-if="selectedLink.conversoes > 0" style="display:flex; flex-direction:column; gap:8px; max-height:160px; overflow-y:auto; padding-right:4px;">
              <div style="display:flex; justify-content:space-between; align-items:center; background:rgba(255,255,255,0.05); padding:8px 12px; border-radius:var(--radius-sm); font-size:12px;">
                <span style="color:white; font-weight:500;">Carlos Silva</span>
                <span style="color:#94a3b8 !important;">Há 3 dias • Ativo</span>
              </div>
              <div v-if="selectedLink.conversoes > 1" style="display:flex; justify-content:space-between; align-items:center; background:rgba(255,255,255,0.05); padding:8px 12px; border-radius:var(--radius-sm); font-size:12px;">
                <span style="color:white; font-weight:500;">Ana Martins</span>
                <span style="color:#94a3b8 !important;">Há 5 dias • Ativo</span>
              </div>
              <div v-if="selectedLink.conversoes > 2" style="display:flex; justify-content:space-between; align-items:center; background:rgba(255,255,255,0.05); padding:8px 12px; border-radius:var(--radius-sm); font-size:12px;">
                <span style="color:white; font-weight:500;">Simulação Cliente #{{ selectedLink.conversoes }}</span>
                <span style="color:#94a3b8 !important;">Hoje • Ativo</span>
              </div>
            </div>
            <p v-else style="font-size:12px; color:#cbd5e1 !important; text-align:center; padding:12px 0; border:1px dashed rgba(255,255,255,0.1); border-radius:var(--radius-sm);">
              Nenhuma conversão registrada para este link ainda.
            </p>
          </div>
        </div>
      </div>
    </div>

    <!-- MODAL DE EDIÇÃO DE LINK -->
    <div v-if="showEditLinkModal && selectedLink" class="sso-overlay-custom" @click.self="showEditLinkModal = false">
      <div class="card-details-modal" style="max-width: 480px;">
        <div class="modal-close" @click="showEditLinkModal = false"><i class="ph ph-x"></i></div>
        <h3 style="font-size: 20px; color: #ffffff !important; margin-bottom: 8px;">Editar Link de Indicação</h3>
        <p style="font-size: 13px; color: #cbd5e1 !important; margin-bottom: 20px;">
          Modifique as informações de identificação do seu link.
        </p>

        <form @submit.prevent="saveEditedLink" style="display:flex; flex-direction:column; gap:16px;">
          <div class="form-group">
            <label class="form-label">Nome do Link</label>
            <input v-model="editLinkName" type="text" class="form-control" required />
          </div>

          <div class="form-group">
            <label class="form-label">Plano Acesso Saúde</label>
            <select v-model="editLinkPlanType" class="form-control" required>
              <option value="Bronze">Plano Bronze (Promocional) — R$ 29,90/mês</option>
              <option value="Individual">Plano Individual — R$ 39,90/mês</option>
              <option value="Família">Plano Família — R$ 84,90/mês</option>
            </select>
          </div>

          <div class="form-group">
            <label class="form-label">Código de Indicação (Ref)</label>
            <input v-model="editLinkRefCode" type="text" class="form-control" required />
          </div>

          <div class="form-group">
            <label class="form-label">Status do Link</label>
            <select v-model="editLinkStatus" class="form-control" required>
              <option value="Ativo">Ativo</option>
              <option value="Inativo">Inativo</option>
            </select>
          </div>

          <div style="display:flex; gap:12px; margin-top: 12px;">
            <button type="button" class="btn btn-outline" style="flex:1;" @click="showEditLinkModal = false">Cancelar</button>
            <button type="submit" class="btn btn-secondary" style="flex:1;">Salvar Alterações</button>
          </div>
        </form>
      </div>
    </div>

    <!-- MODAL DE CARTEIRINHA DIGITAL DETALHADA -->
    <div v-if="showCardModal" class="sso-overlay-custom" @click.self="showCardModal = false">
      <div class="card-details-modal">
        <div class="modal-close" @click="showCardModal = false"><i class="ph ph-x"></i></div>
        <div class="digital-card-preview full-size">
          <div class="dcard-header">
            <div class="dcard-logo-box">
              <img src="/logo.png" alt="Logo" class="dcard-logo" />
            </div>
            <span class="badge badge-success">Premium</span>
          </div>
          <div class="dcard-body">
            <h3>{{ user.name }}</h3>
            <p class="dcard-plan">Plano: {{ user.plan }}</p>
            <p class="dcard-cpf">CPF: 123.456.789-00</p>
          </div>
          <div class="qr-code-area">
            <i class="ph ph-qr-code large-qr"></i>
            <p>Apresente este código para atendimento nos parceiros.</p>
          </div>
        </div>
      </div>
    </div>

    <!-- MODAL REDIRECIONAMENTO SIMULADO (Apenas você está sendo redirecionado...) -->
    <div v-if="showRedirectModal" class="sso-overlay-custom">
      <div class="sso-modal-box">
        <div class="loader-circle"></div>
        <h3>Você está sendo redirecionado...</h3>
        <p>Por favor, aguarde enquanto conectamos você com segurança ao portal da {{ activeRedirect }}.</p>
      </div>
    </div>

  </div>
</template>

<style scoped>
.dashboard-wrapper {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.welcome-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
  flex-wrap: wrap;
  gap: 16px;
}

.welcome-text h1 {
  font-size: 26px;
  color: var(--secondary);
}

.welcome-text p {
  font-size: 14px;
  color: var(--text-gray);
}

.plan-pill {
  background: white;
  padding: 12px 18px;
  border-radius: var(--radius-md);
  border: 1px solid var(--border-color);
  display: flex;
  align-items: center;
  gap: 12px;
}

.plan-name {
  font-size: 13px;
  font-weight: 600;
  color: var(--text-dark);
}

/* Slider Ampliado Contínuo */
.banner-slider {
  position: relative;
  border-radius: var(--radius-lg);
  overflow: hidden;
  height: 320px;
  box-shadow: var(--shadow-md);
  background-color: var(--bg-sidebar);
}

.slider-track {
  display: flex;
  height: 100%;
  width: 100%;
  transition: transform 0.6s cubic-bezier(0.16, 1, 0.3, 1);
}

.slide-item {
  min-width: 100%;
  height: 100%;
  background-size: cover;
  background-position: center;
  padding: 40px;
  display: flex;
  align-items: center;
  color: white;
  flex-shrink: 0;
}

.slide-content {
  max-width: 500px;
  display: flex;
  flex-direction: column;
  gap: 12px;
  align-items: flex-start;
}

.slide-content h2 {
  font-size: 28px;
  color: white;
  margin-top: 4px;
}

.slide-content p {
  font-size: 15px;
  opacity: 0.9;
  line-height: 1.6;
}

.slide-indicator-container {
  position: absolute;
  bottom: 20px;
  right: 20px;
  display: flex;
  gap: 8px;
  z-index: 10;
}

.indicator-dot {
  width: 8px;
  height: 8px;
  background: rgba(255, 255, 255, 0.4);
  border-radius: 50%;
  cursor: pointer;
  transition: var(--transition);
}

.indicator-dot.active {
  background: white;
  transform: scale(1.3);
}

/* Grade de Métricas */
.metrics-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 20px;
  margin: 12px 0;
}

.metric-card {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.metric-header {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 12px;
  color: var(--text-gray);
  font-weight: 500;
}

.metric-header i {
  font-size: 18px;
  color: var(--primary-hover);
}

.metric-card h3 {
  font-size: 24px;
  color: var(--secondary);
  margin: 4px 0 2px;
}

.metric-card p {
  font-size: 12px;
  color: var(--text-light-gray);
}

/* Layout Grid Principal */
.dashboard-grid {
  display: grid;
  grid-template-columns: 1.7fr 1.3fr;
  gap: 32px;
}

@media (max-width: 992px) {
  .dashboard-grid {
    grid-template-columns: 1fr;
  }
}

.section-title {
  font-size: 18px;
  margin-bottom: 16px;
  color: var(--secondary);
}

.shortcuts-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.shortcut-card {
  background: white;
  border: 1px solid var(--border-color);
  padding: 18px;
  border-radius: var(--radius-md);
  display: flex;
  align-items: center;
  gap: 16px;
  cursor: pointer;
  transition: var(--transition);
}

.shortcut-card:hover {
  transform: translateY(-2px);
  border-color: var(--primary);
  box-shadow: var(--shadow-md);
}

.shortcut-icon {
  width: 44px;
  height: 44px;
  border-radius: var(--radius-sm);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  flex-shrink: 0;
}

.icon-teal { background: #e8f2f3; color: var(--secondary); }
.icon-pink { background: #fdf2f8; color: #db2777; }
.icon-green { background: #eef7eb; color: var(--primary-hover); }
.icon-orange { background: #fffbeb; color: var(--accent); }

.shortcut-details {
  flex-grow: 1;
}

.shortcut-details h3 {
  font-size: 15px;
  color: var(--text-dark);
  margin-bottom: 2px;
}

.shortcut-details p {
  font-size: 12px;
  color: var(--text-gray);
  margin-bottom: 0;
}

.action-arrow {
  color: var(--text-light-gray);
  transition: var(--transition);
}

.shortcut-card:hover .action-arrow {
  transform: translateX(4px);
  color: var(--primary-hover);
}

/* Carteirinha Digital */
.digital-card-preview {
  background: linear-gradient(135deg, var(--secondary) 0%, #0d2a2c 100%);
  color: white;
  padding: 24px;
  border-radius: var(--radius-lg);
  display: flex;
  flex-direction: column;
  gap: 20px;
  cursor: pointer;
  transition: var(--transition);
  position: relative;
  overflow: hidden;
  box-shadow: var(--shadow-md);
}

.digital-card-preview::before {
  content: '';
  position: absolute;
  top: -50%;
  right: -20%;
  width: 250px;
  height: 250px;
  background: rgba(255, 255, 255, 0.03);
  border-radius: 50%;
}

.digital-card-preview:hover {
  transform: scale(1.02);
  box-shadow: var(--shadow-lg);
}

.dcard-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.dcard-logo-box {
  background: #ffffff; /* Fundo totalmente branco para realçar o logotipo */
  padding: 6px 12px;
  border-radius: var(--radius-sm);
  display: flex;
  align-items: center;
}

.dcard-logo {
  max-height: 24px;
}

.dcard-body h3 {
  color: white;
  font-size: 18px;
  margin-bottom: 6px;
}

.dcard-plan, .dcard-cpf {
  font-size: 13px;
  opacity: 0.8;
}

.dcard-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  padding-top: 12px;
  font-size: 12px;
  opacity: 0.8;
}

/* Margem Aumentada nas Atividades Recentes */
.activities-wrapper {
  margin-top: 40px;
}

.activities-card {
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.activity-item {
  display: flex;
  gap: 12px;
  align-items: flex-start;
}

.activity-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  margin-top: 6px;
  flex-shrink: 0;
}

.activity-dot.health { background: #3b82f6; }
.activity-dot.clube { background: #f59e0b; }
.activity-dot.pet { background: #10b981; }

.activity-text p {
  font-size: 13px;
  color: var(--text-dark);
}

.activity-text span {
  font-size: 11px;
  color: var(--text-light-gray);
}

/* Modal Carteirinha */
.card-details-modal {
  background: #1e293b;
  padding: 32px;
  border-radius: var(--radius-xl);
  max-width: 400px;
  width: 90%;
  box-shadow: var(--shadow-lg);
  position: relative;
  color: #f8fafc;
}

.card-details-modal h3 {
  color: #ffffff !important;
}

.card-details-modal h4 {
  color: #f1f5f9 !important;
}

.card-details-modal p {
  color: #cbd5e1 !important;
}

.card-details-modal span {
  color: #94a3b8 !important;
}

.card-details-modal .form-label,
.card-details-modal label {
  color: #f1f5f9 !important;
  font-weight: 500;
}

.card-details-modal .form-control {
  background: #0f172a !important;
  color: #ffffff !important;
  border: 1px solid #334155 !important;
}

.card-details-modal .form-control:focus {
  border-color: var(--primary) !important;
  box-shadow: 0 0 0 2px rgba(22, 163, 74, 0.2) !important;
}

.modal-close {
  position: absolute;
  top: 16px;
  right: 16px;
  color: white;
  font-size: 20px;
  cursor: pointer;
}

.digital-card-preview.full-size {
  background: linear-gradient(135deg, var(--secondary) 0%, #0d2a2c 100%);
  cursor: default;
}

.digital-card-preview.full-size:hover {
  transform: none;
}

.qr-code-area {
  background: white;
  border-radius: var(--radius-md);
  padding: 20px;
  text-align: center;
  color: var(--text-dark);
  margin-top: 12px;
}

.large-qr {
  font-size: 140px;
  color: var(--text-dark);
}

.qr-code-area p {
  font-size: 11px;
  margin-top: 10px;
  color: var(--text-gray);
}

/* SSO Modals */
.sso-overlay-custom {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(15, 23, 42, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 99999;
}

.sso-modal-box {
  background: white;
  padding: 40px;
  border-radius: var(--radius-lg);
  text-align: center;
  max-width: 400px;
  width: 90%;
  box-shadow: var(--shadow-lg);
}

.loader-circle {
  width: 48px;
  height: 48px;
  border: 4px solid var(--border-color);
  border-top-color: var(--primary);
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 20px;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* Formulários */
.form-row {
  display: flex;
  gap: 16px;
  flex-wrap: wrap;
}

.flex-1 {
  flex: 1;
  min-width: 200px;
}

/* Melhorias do Financeiro */
.financial-grid {
  display: grid;
  grid-template-columns: 1.2fr 1.8fr;
  gap: 28px;
}

@media (max-width: 992px) {
  .financial-grid {
    grid-template-columns: 1fr;
  }
}

.financial-main-card {
  padding: 32px;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.billing-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  flex-wrap: wrap;
  gap: 16px;
  border-bottom: 1px solid var(--border-color);
  padding-bottom: 20px;
}

.plan-title {
  font-size: 20px;
  color: var(--secondary);
  margin-top: 8px;
}

.price-block {
  display: flex;
  align-items: baseline;
  color: var(--secondary);
}

.currency {
  font-size: 14px;
  font-weight: 600;
  margin-right: 2px;
}

.price-val {
  font-size: 32px;
  font-weight: 800;
}

.period {
  font-size: 13px;
  color: var(--text-gray);
  margin-left: 2px;
}

.billing-details-list {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.detail-row {
  display: flex;
  justify-content: space-between;
  font-size: 14px;
}

.detail-row span {
  color: var(--text-gray);
}

.detail-row strong {
  color: var(--text-dark);
}

.billing-actions {
  display: flex;
  gap: 12px;
}

.billing-actions .btn {
  flex: 1;
}

/* Histórico de Faturamento */
.financial-history-card {
  padding: 32px;
}

.financial-history-card h3 {
  font-size: 18px;
  color: var(--secondary);
  margin-bottom: 20px;
}

.invoices-list-v2 {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.invoice-item-v2 {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px;
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  background: var(--bg-gray);
  flex-wrap: wrap;
  gap: 16px;
}

.inv-info {
  display: flex;
  align-items: center;
  gap: 16px;
}

.icon-large {
  font-size: 28px;
}

.inv-info strong {
  display: block;
  font-size: 15px;
  color: var(--text-dark);
}

.inv-info span {
  font-size: 12px;
  color: var(--text-gray);
}

.inv-value {
  display: flex;
  align-items: center;
  gap: 20px;
}

.inv-value span {
  font-size: 16px;
  font-weight: 700;
  color: var(--text-dark);
}

.btn-receipt-download {
  background: transparent;
  border: 1px solid var(--primary-hover);
  color: var(--primary-hover);
  padding: 6px 12px;
  border-radius: var(--radius-sm);
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  transition: var(--transition);
}

.btn-receipt-download:hover {
  background: var(--primary-light);
}

/* PWA Overrides */
.pwa {
  padding: 16px;
}

.pwa .banner-slider {
  height: 240px;
}

.pwa .slide-item {
  padding: 24px;
}

.pwa .slide-content h2 {
  font-size: 20px;
}

.pwa .slide-content p {
  font-size: 13px;
}

.pwa .dashboard-grid {
  grid-template-columns: 1fr;
  gap: 20px;
}

.pwa .financial-grid {
  grid-template-columns: 1fr;
}

/* Config Grid & Sections */
.config-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: 24px;
  margin-top: 20px;
}

.config-section-card {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.config-section-card h3 {
  font-size: 16px;
  color: var(--secondary);
}

.config-desc {
  font-size: 13px;
  color: var(--text-gray);
  line-height: 1.5;
}

.notification-options {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.notification-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  padding-bottom: 12px;
  border-bottom: 1px dashed var(--border-color);
}

.notification-row:last-child {
  border-bottom: none;
}

.notif-text strong {
  display: block;
  font-size: 14px;
  color: var(--text-dark);
}

.notif-text p {
  font-size: 12px;
  color: var(--text-gray);
}
</style>
