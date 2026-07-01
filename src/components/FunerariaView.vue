<script setup>
import { ref } from 'vue'

const showSSODemo = ref(false)

const handleSSO = () => {
  showSSODemo.value = true
  setTimeout(() => {
    showSSODemo.value = false
    alert('Autenticação de Assistência Funerária executada via SSO. Central de Atendimento acionada.')
  }, 2500)
}
</script>

<template>
  <div class="benefit-page">
    <div class="benefit-hero">
      <div class="hero-text">
        <span class="badge badge-warning">Segurança Familiar</span>
        <h1>Assistência Funerária</h1>
        <p>Apoio completo e acolhimento nos momentos mais difíceis, garantindo tranquilidade e dignidade à sua família.</p>
      </div>
      <div class="hero-icon-container">
        <i class="ph ph-heartbeat"></i>
      </div>
    </div>

    <div class="benefit-content">
      <div class="main-info">
        <h2>Coberturas Incluídas</h2>
        <p>A assistência oferece cobertura completa em âmbito nacional, assumindo toda a burocracia e custos de serviços necessários de forma sensível e ágil.</p>
        
        <div class="coverage-grid">
          <div class="coverage-item">
            <i class="ph ph-shield-check"></i>
            <div>
              <h3>Sepultamento ou Cremação</h3>
              <p>Custos e trâmites para o ritual fúnebre inclusos.</p>
            </div>
          </div>
          <div class="coverage-item">
            <i class="ph ph-truck"></i>
            <div>
              <h3>Traslado de Corpo</h3>
              <p>Mapeamento e transporte interestadual ou intermunicipal.</p>
            </div>
          </div>
          <div class="coverage-item">
            <i class="ph ph-scales"></i>
            <div>
              <h3>Assessoria Jurídica</h3>
              <p>Orientação e auxílio com trâmites de certidão e cartório.</p>
            </div>
          </div>
          <div class="coverage-item">
            <i class="ph ph-house"></i>
            <div>
              <h3>Capela e Ornamentação</h3>
              <p>Aluguel de sala de velório, coroas de flores e preparação básica.</p>
            </div>
          </div>
        </div>
      </div>

      <div class="sidebar-info">
        <div class="info-card">
          <h3>Central de Emergência</h3>
          <p class="support-subtitle">Atendimento imediato 24h:</p>
          <div class="phone-box">
            <i class="ph ph-phone"></i>
            <span class="phone-number">0800 555 1234</span>
          </div>
          <div class="phone-box">
            <i class="ph ph-whatsapp-logo"></i>
            <span class="phone-number">(11) 99999-8888</span>
          </div>
          <button @click="handleSSO" class="btn btn-secondary btn-full margin-top">
            <i class="ph ph-lock-key-open"></i>
            Acessar Benefício (SSO)
          </button>
        </div>
      </div>
    </div>

    <!-- SSO Demo Overlay -->
    <div v-if="showSSODemo" class="sso-overlay">
      <div class="sso-modal">
        <div class="spinner"></div>
        <h3>Verificando Cobertura...</h3>
        <p>Integrando com a Central de Assistência Funerária Parceira.</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.benefit-page {
  display: flex;
  flex-direction: column;
  gap: 32px;
}

.benefit-hero {
  background: linear-gradient(135deg, #104246 0%, #0d2a2c 100%);
  color: white;
  border-radius: var(--radius-lg);
  padding: 40px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.hero-text {
  max-width: 600px;
}

.hero-text h1 {
  color: white;
  font-size: 32px;
  margin: 12px 0;
}

.hero-text p {
  font-size: 16px;
  opacity: 0.9;
}

.hero-icon-container {
  font-size: 96px;
  opacity: 0.15;
}

.benefit-content {
  display: grid;
  grid-template-columns: 1.8fr 1.2fr;
  gap: 32px;
}

@media (max-width: 992px) {
  .benefit-content {
    grid-template-columns: 1fr;
  }
}

.main-info {
  background: white;
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  padding: 32px;
}

.main-info h2 {
  font-size: 22px;
  margin-bottom: 16px;
  color: var(--secondary);
}

.main-info p {
  font-size: 15px;
  line-height: 1.6;
  margin-bottom: 24px;
}

.coverage-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 24px;
}

.coverage-item {
  display: flex;
  gap: 16px;
  align-items: flex-start;
}

.coverage-item i {
  font-size: 24px;
  color: var(--primary-hover);
  background: var(--primary-light);
  padding: 10px;
  border-radius: var(--radius-sm);
  flex-shrink: 0;
}

.coverage-item h3 {
  font-size: 15px;
  margin-bottom: 4px;
  color: var(--text-dark);
}

.coverage-item p {
  font-size: 13px;
  color: var(--text-gray);
  margin-bottom: 0;
}

.sidebar-info .info-card {
  background: white;
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  padding: 32px;
}

.sidebar-info h3 {
  font-size: 18px;
  margin-bottom: 8px;
  color: var(--secondary);
}

.support-subtitle {
  font-size: 13px;
  color: var(--text-gray);
  margin-bottom: 16px;
}

.phone-box {
  display: flex;
  align-items: center;
  gap: 12px;
  background: var(--bg-gray);
  padding: 14px;
  border-radius: var(--radius-sm);
  margin-bottom: 12px;
  color: var(--secondary);
  font-weight: 600;
}

.phone-box i {
  font-size: 20px;
}

.margin-top {
  margin-top: 20px;
}

/* SSO Demo Modal */
.sso-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
}

.sso-modal {
  background: white;
  padding: 40px;
  border-radius: var(--radius-lg);
  text-align: center;
  max-width: 400px;
  width: 90%;
  box-shadow: var(--shadow-lg);
}

.spinner {
  width: 50px;
  height: 50px;
  border: 5px solid var(--border-color);
  border-top-color: var(--secondary);
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 24px;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.sso-modal h3 {
  font-size: 18px;
  margin-bottom: 8px;
  color: var(--secondary);
}

.sso-modal p {
  font-size: 14px;
  color: var(--text-gray);
}
</style>
