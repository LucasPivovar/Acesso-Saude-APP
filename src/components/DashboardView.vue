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

const emit = defineEmits(['updateUser', 'logout', 'triggerDevModal'])

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
  background: rgba(16, 66, 70, 0.85);
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
