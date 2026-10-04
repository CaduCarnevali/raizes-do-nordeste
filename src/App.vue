<script setup>
import { computed, nextTick, onMounted, ref } from 'vue'

const steps = [
  { id: 'menu', label: 'Cardápio' },
  { id: 'identification', label: 'Identificação' },
  { id: 'privacy', label: 'Privacidade' },
  { id: 'payment', label: 'Pagamento' },
  { id: 'tracking', label: 'Acompanhar' },
]

const orderStatuses = ['Pedido recebido', 'Em preparo', 'Pronto para retirada', 'Pedido retirado']
const POINT_VALUE = 0.05

const units = ref([])
const menu = ref([])
const promotions = ref([])
const customers = ref([])
const loading = ref(true)
const loadError = ref('')
const selectedUnit = ref(null)
const currentStep = ref('menu')
const selectedCategory = ref('Todos')
const cart = ref([])
const user = ref(null)
const authMode = ref('login')
const loginForm = ref({ email: 'cliente@raizes.com.br', password: 'raizes123' })
const loginError = ref('')
const registerForm = ref({ name: '', email: '', password: '', confirmPassword: '' })
const registerError = ref('')
const privacyAccepted = ref(false)
const marketingAccepted = ref(false)
const paymentCard = ref({
  number: '4111 1111 1111 1111',
  holder: 'Marina Oliveira',
  expiry: '12/30',
  cvv: '123',
})
const paymentState = ref('idle')
const usePoints = ref(false)
const orderStatus = ref(0)
const orderNumber = ref('')
const toast = ref('')

const money = new Intl.NumberFormat('pt-BR', {
  style: 'currency',
  currency: 'BRL',
})

const filteredMenu = computed(() => {
  if (!selectedUnit.value) return []
  return menu.value.filter((item) => {
    const available = item.units.includes(selectedUnit.value.id)
    const matchesCategory = selectedCategory.value === 'Todos' || item.category === selectedCategory.value
    return available && matchesCategory
  })
})

const categories = computed(() => {
  const categoryOrder = ['Sanduíches', 'Pratos', 'Acompanhamentos', 'Bebidas', 'Sobremesas']
  const availableCategories = new Set(menu.value.map((item) => item.category))
  return ['Todos', ...categoryOrder.filter((category) => availableCategories.has(category))]
})
const cartCount = computed(() => cart.value.reduce((total, item) => total + item.quantity, 0))
const subtotal = computed(() => cart.value.reduce((total, item) => total + item.price * item.quantity, 0))
const activeStepIndex = computed(() => steps.findIndex((step) => step.id === currentStep.value))
const currentOrderRecord = computed(() => user.value?.orders?.find((order) => order.id === orderNumber.value))
const applicablePoints = computed(() => {
  if (paymentState.value === 'approved' && currentOrderRecord.value) {
    return currentOrderRecord.value.pointsUsed || 0
  }
  if (!usePoints.value || !user.value || subtotal.value <= 0) return 0
  const pointsNeeded = Math.ceil(subtotal.value / POINT_VALUE - 1e-9)
  return Math.min(user.value.points, pointsNeeded)
})
const pointsDiscount = computed(() => Math.min(subtotal.value, roundCurrency(applicablePoints.value * POINT_VALUE)))
const orderTotal = computed(() => roundCurrency(subtotal.value - pointsDiscount.value))
const pointsEarned = computed(() => Math.floor(orderTotal.value))
const pointsCredit = computed(() => roundCurrency((user.value?.points || 0) * POINT_VALUE))

function roundCurrency(value) {
  return Math.round((value + Number.EPSILON) * 100) / 100
}

onMounted(async () => {
  try {
    const base = import.meta.env.BASE_URL
    const responses = await Promise.all([
      fetch(`${base}data/units.json`),
      fetch(`${base}data/menu.json`),
      fetch(`${base}data/promotions.json`),
      fetch(`${base}data/customers.json`),
    ])
    if (responses.some((response) => !response.ok)) throw new Error('Falha ao carregar dados')
    ;[units.value, menu.value, promotions.value, customers.value] = await Promise.all(
      responses.map((response) => response.json()),
    )
  } catch {
    loadError.value = 'Não foi possível carregar o cardápio. Tente atualizar a página.'
  } finally {
    loading.value = false
  }
})

async function chooseUnit(unit) {
  const previousUnit = selectedUnit.value
  const isSameUnit = previousUnit?.id === unit.id
  const hadCartItems = cart.value.length > 0

  if (isSameUnit) {
    document.querySelector('#menu')?.scrollIntoView({ behavior: 'smooth' })
    return
  }

  selectedUnit.value = unit
  selectedCategory.value = 'Todos'
  cart.value = []
  usePoints.value = false
  currentStep.value = 'menu'
  await nextTick()
  document.querySelector('#menu')?.scrollIntoView({ behavior: 'smooth' })

  if (previousUnit) {
    showToast(
      hadCartItems
        ? `Cardápio atualizado para ${unit.name}. O pedido anterior foi limpo.`
        : `Cardápio atualizado para ${unit.name}.`,
    )
  } else {
    showToast(`Cardápio de ${unit.name} carregado.`)
  }
}

function addToCart(product) {
  const existing = cart.value.find((item) => item.id === product.id)
  if (existing) existing.quantity += 1
  else cart.value.push({ ...product, quantity: 1 })
  showToast(`${product.name} adicionado ao pedido.`)
}

function addPromotionToCart(promotion) {
  addToCart({
    id: `promotion-${promotion.id}`,
    name: promotion.title,
    description: promotion.description,
    price: promotion.price,
    originalPrice: promotion.originalPrice,
    emoji: promotion.emoji || '🎁',
    promotion: true,
  })
}

function changeQuantity(productId, change) {
  const item = cart.value.find((product) => product.id === productId)
  if (!item) return
  item.quantity += change
  if (item.quantity <= 0) cart.value = cart.value.filter((product) => product.id !== productId)
}

function showToast(message) {
  toast.value = message
  window.setTimeout(() => {
    if (toast.value === message) toast.value = ''
  }, 2400)
}

function startCheckout() {
  if (!cart.value.length) return
  currentStep.value = user.value
    ? (user.value.privacyConsent ? 'payment' : 'privacy')
    : 'identification'
  scrollToFlow()
}

function continueOrder() {
  if (!cart.value.length) return

  if (currentStep.value === 'menu') {
    currentStep.value = user.value
      ? (user.value.privacyConsent ? 'payment' : 'privacy')
      : 'identification'
  }

  const cartPanel = document.querySelector('#cartPanel')
  if (cartPanel?.classList.contains('show')) {
    cartPanel.addEventListener('hidden.bs.offcanvas', scrollToFlow, { once: true })
    // O clique também aciona o fechamento pelo Bootstrap. Em alguns navegadores,
    // o evento pode ocorrer antes de o listener acima ser registrado; o fallback
    // garante que a etapa atual permaneça visível após a animação do painel.
    window.setTimeout(scrollToFlow, 450)
  } else {
    scrollToFlow()
  }
}

function login() {
  const normalizedEmail = loginForm.value.email.trim().toLowerCase()
  const account = customers.value.find(
    (customer) => customer.email.toLowerCase() === normalizedEmail && customer.password === loginForm.value.password,
  )

  if (account) {
    const { password: _password, ...profile } = account
    user.value = { ...profile }
    privacyAccepted.value = Boolean(profile.privacyConsent)
    marketingAccepted.value = Boolean(profile.marketingConsent)
    paymentCard.value.holder = profile.name
    loginError.value = ''
    currentStep.value = profile.privacyConsent ? 'payment' : 'privacy'
    scrollToFlow()
  } else {
    loginError.value = 'E-mail ou senha incorretos. Verifique os dados e tente novamente.'
  }
}

function register() {
  registerError.value = ''
  const normalizedEmail = registerForm.value.email.trim().toLowerCase()

  if (customers.value.some((customer) => customer.email.toLowerCase() === normalizedEmail)) {
    registerError.value = 'Já existe uma conta cadastrada com este e-mail.'
    return
  }
  if (registerForm.value.password.length < 6) {
    registerError.value = 'A senha deve ter pelo menos 6 caracteres.'
    return
  }
  if (registerForm.value.password !== registerForm.value.confirmPassword) {
    registerError.value = 'As senhas informadas não coincidem.'
    return
  }
  const newAccount = {
    id: `customer-${Date.now()}`,
    name: registerForm.value.name,
    email: normalizedEmail,
    password: registerForm.value.password,
    points: 0,
    privacyConsent: false,
    privacyAcceptedAt: null,
    marketingConsent: false,
    orders: [],
  }
  customers.value.push(newAccount)
  const { password: _password, ...profile } = newAccount
  user.value = { ...profile }
  privacyAccepted.value = false
  marketingAccepted.value = false
  paymentCard.value.holder = profile.name
  currentStep.value = 'privacy'
  scrollToFlow()
}

function syncCurrentAccount() {
  if (!user.value) return
  const account = customers.value.find((customer) => customer.id === user.value.id)
  if (account) {
    account.points = user.value.points
    account.privacyConsent = Boolean(user.value.privacyConsent)
    account.privacyAcceptedAt = user.value.privacyAcceptedAt || null
    account.marketingConsent = Boolean(user.value.marketingConsent)
    account.orders = (user.value.orders || []).map((order) => ({
      ...order,
      unit: order.unit ? { ...order.unit } : null,
      items: order.items.map((item) => ({ ...item })),
    }))
  }
}

function switchAuthMode(mode) {
  authMode.value = mode
  loginError.value = ''
  registerError.value = ''
}

function continueToPayment() {
  if (!privacyAccepted.value) return
  user.value.privacyConsent = true
  user.value.privacyAcceptedAt ||= new Date().toISOString()
  user.value.marketingConsent = marketingAccepted.value
  syncCurrentAccount()
  currentStep.value = 'payment'
  scrollToFlow()
}

function applyPoints() {
  if (!user.value?.points || subtotal.value <= 0) return
  usePoints.value = true
  showToast(`${applicablePoints.value} pontos aplicados: ${money.format(pointsDiscount.value)} de desconto.`)
}

function removePoints() {
  usePoints.value = false
  showToast('Desconto em pontos removido.')
}

function applyPointsFromAccount() {
  if (!user.value?.points || !cart.value.length || currentStep.value === 'tracking') return
  usePoints.value = true
  if (currentStep.value === 'menu') {
    currentStep.value = user.value.privacyConsent ? 'payment' : 'privacy'
  }

  const ordersPanel = document.querySelector('#ordersPanel')
  if (ordersPanel?.classList.contains('show')) {
    ordersPanel.addEventListener('hidden.bs.offcanvas', scrollToFlow, { once: true })
  } else {
    scrollToFlow()
  }
  showToast(`${applicablePoints.value} pontos reservados para este pedido.`)
}

function pay() {
  const usedPoints = applicablePoints.value
  const discount = pointsDiscount.value
  const finalTotal = orderTotal.value
  const earnedPoints = pointsEarned.value
  const cardDigits = paymentCard.value.number.replace(/\D/g, '')
  paymentState.value = finalTotal === 0 || !cardDigits.endsWith('0000') ? 'approved' : 'declined'
  if (paymentState.value === 'approved') {
    orderNumber.value = `RN-${Math.floor(1000 + Math.random() * 9000)}`
    user.value.points = user.value.points - usedPoints + earnedPoints
    user.value.orders = [
      {
        id: orderNumber.value,
        date: new Date().toLocaleDateString('pt-BR'),
        subtotal: subtotal.value,
        pointsUsed: usedPoints,
        pointsDiscount: discount,
        total: finalTotal,
        status: orderStatuses[0],
        pointsEarned: earnedPoints,
        unit: {
          id: selectedUnit.value.id,
          name: selectedUnit.value.name,
          address: selectedUnit.value.address,
        },
        items: cart.value.map((item) => ({
          name: item.name,
          quantity: item.quantity,
          price: item.price,
        })),
      },
      ...(user.value.orders || []),
    ]
    syncCurrentAccount()
    window.setTimeout(() => {
      currentStep.value = 'tracking'
      scrollToFlow()
    }, 800)
  }
}

function retryPayment() {
  paymentState.value = 'idle'
  paymentCard.value.number = ''
}

function advanceOrder() {
  if (orderStatus.value < orderStatuses.length - 1) {
    orderStatus.value += 1
    const currentOrder = user.value?.orders?.find((order) => order.id === orderNumber.value)
    if (currentOrder) {
      currentOrder.status = orderStatuses[orderStatus.value]
      syncCurrentAccount()
    }
  }
}

async function beginNewOrder(initialItems = []) {
  cart.value = initialItems
  usePoints.value = false
  selectedCategory.value = 'Todos'
  currentStep.value = 'menu'
  paymentState.value = 'idle'
  orderStatus.value = 0
  orderNumber.value = ''
  paymentCard.value = {
    number: '4111 1111 1111 1111',
    holder: user.value?.name || '',
    expiry: '12/30',
    cvv: '123',
  }
  await nextTick()
  document.querySelector('#menu')?.scrollIntoView({ behavior: 'smooth' })
}

async function startNewOrder() {
  await beginNewOrder()
  showToast('Novo pedido iniciado. Seus pontos foram mantidos.')
}

function changeUnit() {
  selectedUnit.value = null
  currentStep.value = 'menu'
  selectedCategory.value = 'Todos'
  cart.value = []
  usePoints.value = false
  paymentCard.value = {
    number: '4111 1111 1111 1111',
    holder: user.value?.name || '',
    expiry: '12/30',
    cvv: '123',
  }
  paymentState.value = 'idle'
  orderStatus.value = 0
  orderNumber.value = ''
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

async function logout() {
  user.value = null
  authMode.value = 'login'
  privacyAccepted.value = false
  marketingAccepted.value = false
  await beginNewOrder()
  showToast('Você saiu da sua conta.')
}

function scrollToFlow() {
  window.setTimeout(() => document.querySelector('#fluxo')?.scrollIntoView({ behavior: 'smooth' }), 50)
}
</script>

<template>
  <div class="app-shell">
    <header class="site-header sticky-top">
      <nav class="navbar navbar-expand-lg" aria-label="Navegação principal">
        <div class="container py-2">
          <a class="navbar-brand d-flex align-items-center gap-2" href="#" @click.prevent="changeUnit">
            <span class="brand-mark" aria-hidden="true">RN</span>
            <span>
              <strong>Raízes</strong>
              <small>do Nordeste</small>
            </span>
          </a>
          <div class="d-flex align-items-center gap-2 ms-auto">
            <span v-if="selectedUnit" class="unit-pill d-none d-md-inline-flex">
              <span aria-hidden="true">●</span> {{ selectedUnit.name }}
            </span>
            <span v-if="user" class="account-pill d-none d-lg-inline-flex">
              <span>Olá, {{ user.name.split(' ')[0] }}</span>
              <small>{{ user.points }} pontos</small>
            </span>
            <button v-if="user" class="btn btn-account btn-sm" type="button" @click="logout">
              Sair
            </button>
            <button
              v-if="user"
              class="btn btn-account btn-sm"
              type="button"
              data-bs-toggle="offcanvas"
              data-bs-target="#ordersPanel"
              aria-controls="ordersPanel"
            >
              <span class="d-none d-md-inline">Meus </span>pedidos
            </button>
            <button v-if="selectedUnit" class="btn btn-outline-brand btn-sm" type="button" @click="changeUnit">
              Trocar unidade
            </button>
            <button
              class="btn cart-button position-relative"
              type="button"
              :disabled="!selectedUnit"
              data-bs-toggle="offcanvas"
              data-bs-target="#cartPanel"
              aria-controls="cartPanel"
            >
              Pedido
              <span v-if="cartCount" class="cart-count">{{ cartCount }}</span>
            </button>
          </div>
        </div>
      </nav>
    </header>

    <main>
      <section class="hero-section">
        <div class="container">
          <div class="row align-items-center g-5">
            <div class="col-lg-7">
              <span class="eyebrow">Sabor que conta histórias</span>
              <h1>Seu Nordeste favorito, pronto para retirar.</h1>
              <p class="hero-copy">
                Escolha a unidade, monte seu pedido e acompanhe cada etapa — rápido, simples e cheio de afeto.
              </p>
              <a class="btn btn-brand btn-lg" href="#unidades">Escolher unidade</a>
            </div>
            <div class="col-lg-5">
              <div class="hero-card">
                <div class="sun-shape"></div>
                <div class="hero-food" aria-label="Ilustração de refeição nordestina">🍔</div>
                <div class="hero-card-copy">
                  <span>Oferta da semana</span>
                  <strong>Combo Sabores do Sertão</strong>
                  <small>A partir de {{ money.format(46.9) }}</small>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>

      <section id="unidades" class="section-space">
        <div class="container">
          <div class="section-heading">
            <div>
              <span class="eyebrow">Onde você está?</span>
              <h2>Escolha sua unidade</h2>
            </div>
            <p>O cardápio e o tempo de preparo podem variar conforme a unidade.</p>
          </div>

          <div v-if="loading" class="loading-card" role="status">Carregando cardápio…</div>
          <div v-else-if="loadError" class="alert alert-danger" role="alert">{{ loadError }}</div>
          <div v-else class="row g-3">
            <div v-for="unit in units" :key="unit.id" class="col-md-6">
              <button
                class="unit-card w-100 text-start"
                :class="{ selected: selectedUnit?.id === unit.id }"
                type="button"
                @click="chooseUnit(unit)"
              >
                <span class="unit-icon" aria-hidden="true">⌂</span>
                <span class="flex-grow-1">
                  <strong>{{ unit.name }}</strong>
                  <small>{{ unit.address }}</small>
                  <span class="unit-meta">{{ unit.hours }} · Preparo: {{ unit.prepTime }}</span>
                </span>
                <span class="select-arrow" aria-hidden="true">→</span>
              </button>
            </div>
          </div>
        </div>
      </section>

      <template v-if="selectedUnit">
        <section class="promo-section">
          <div class="container">
            <article v-for="promotion in promotions" :key="promotion.id" class="promo-card">
              <div>
                <span class="promo-badge">{{ promotion.badge }}</span>
                <h2>{{ promotion.title }}</h2>
                <p>{{ promotion.description }}</p>
              </div>
              <div class="promo-actions">
                <div class="promo-price">
                  <del>{{ money.format(promotion.originalPrice) }}</del>
                  <strong>{{ money.format(promotion.price) }}</strong>
                  <small>Economize {{ money.format(promotion.originalPrice - promotion.price) }}</small>
                </div>
                <button class="btn btn-promo" type="button" @click="addPromotionToCart(promotion)">
                  <span aria-hidden="true">+</span> Adicionar combo
                </button>
              </div>
            </article>
          </div>
        </section>

        <section id="menu" class="section-space menu-section">
          <div class="container">
            <div class="section-heading align-items-end">
              <div>
                <span class="eyebrow">Feito na hora</span>
                <h2>Nosso cardápio</h2>
              </div>
              <div class="selected-unit-summary">
                <span>{{ selectedUnit.name }}</span>
                <small>{{ filteredMenu.length }} produtos disponíveis</small>
              </div>
            </div>

            <div class="category-tabs" role="tablist" aria-label="Categorias do cardápio">
              <button
                v-for="category in categories"
                :key="category"
                type="button"
                role="tab"
                :aria-selected="selectedCategory === category"
                :class="{ active: selectedCategory === category }"
                @click="selectedCategory = category"
              >
                {{ category }}
              </button>
            </div>

            <div class="row g-4 mt-1">
              <div v-for="product in filteredMenu" :key="product.id" class="col-md-6 col-xl-4">
                <article class="product-card h-100">
                  <div class="product-visual" aria-hidden="true">{{ product.emoji }}</div>
                  <div class="product-content">
                    <div class="d-flex justify-content-between gap-3">
                      <h3>{{ product.name }}</h3>
                      <div class="product-tags">
                        <span v-if="product.featured" class="favorite-tag">Popular</span>
                        <span v-if="product.units.length === 1" class="exclusive-tag">Exclusivo</span>
                      </div>
                    </div>
                    <p>{{ product.description }}</p>
                    <div class="product-footer">
                      <strong>{{ money.format(product.price) }}</strong>
                      <button class="btn btn-add" type="button" @click="addToCart(product)">
                        <span aria-hidden="true">+</span> Adicionar
                      </button>
                    </div>
                  </div>
                </article>
              </div>
            </div>
          </div>
        </section>

        <section id="fluxo" class="checkout-section section-space">
          <div class="container">
            <div class="flow-header">
              <span class="eyebrow">Finalização do pedido</span>
              <h2>Conclua seu pedido</h2>
              <p>Revise seus dados, escolha suas preferências e finalize o pagamento.</p>
            </div>

            <ol class="stepper" aria-label="Etapas do pedido">
              <li
                v-for="(step, index) in steps"
                :key="step.id"
                :class="{ active: currentStep === step.id, complete: activeStepIndex > index }"
              >
                <span>{{ index + 1 }}</span>
                <small>{{ step.label }}</small>
              </li>
            </ol>

            <div class="flow-card">
              <div v-if="currentStep === 'menu'" class="empty-flow text-center">
                <span aria-hidden="true">🧺</span>
                <h3>{{ cartCount ? 'Seu pedido está pronto para continuar' : 'Adicione itens ao pedido' }}</h3>
                <p v-if="cartCount">{{ cartCount }} item(ns) · {{ money.format(subtotal) }}</p>
                <p v-else>Escolha ao menos um produto no cardápio acima.</p>
                <button class="btn btn-brand" type="button" :disabled="!cartCount" @click="startCheckout">
                  Ir para identificação
                </button>
              </div>

              <div v-else-if="currentStep === 'identification'" class="form-panel">
                <div class="panel-title">
                  <span class="panel-icon" aria-hidden="true">👤</span>
                  <div>
                    <h3>{{ authMode === 'login' ? 'Entre na sua conta' : 'Crie sua conta' }}</h3>
                    <p>{{ authMode === 'login' ? 'Acesse seus pontos e acompanhe o pedido.' : 'Cadastre-se para acumular pontos e receber benefícios.' }}</p>
                  </div>
                </div>
                <div class="auth-tabs" role="tablist" aria-label="Acesso à conta">
                  <button type="button" role="tab" :aria-selected="authMode === 'login'" :class="{ active: authMode === 'login' }" @click="switchAuthMode('login')">Entrar</button>
                  <button type="button" role="tab" :aria-selected="authMode === 'register'" :class="{ active: authMode === 'register' }" @click="switchAuthMode('register')">Criar conta</button>
                </div>

                <form v-if="authMode === 'login'" @submit.prevent="login">
                  <div class="row g-3">
                    <div class="col-md-6">
                      <label class="form-label" for="email">E-mail</label>
                      <input id="email" v-model="loginForm.email" class="form-control form-control-lg" type="email" autocomplete="email" required />
                    </div>
                    <div class="col-md-6">
                      <label class="form-label" for="password">Senha</label>
                      <input id="password" v-model="loginForm.password" class="form-control form-control-lg" type="password" autocomplete="current-password" required />
                    </div>
                  </div>
                  <p v-if="loginError" class="form-error" role="alert">{{ loginError }}</p>
                  <button class="btn btn-brand mt-4" type="submit">Entrar e continuar</button>
                </form>

                <form v-else @submit.prevent="register">
                  <div class="row g-3">
                    <div class="col-12">
                      <label class="form-label" for="register-name">Nome completo</label>
                      <input id="register-name" v-model.trim="registerForm.name" class="form-control form-control-lg" type="text" autocomplete="name" required />
                    </div>
                    <div class="col-12">
                      <label class="form-label" for="register-email">E-mail</label>
                      <input id="register-email" v-model.trim="registerForm.email" class="form-control form-control-lg" type="email" autocomplete="email" required />
                    </div>
                    <div class="col-md-6">
                      <label class="form-label" for="register-password">Senha</label>
                      <input id="register-password" v-model="registerForm.password" class="form-control form-control-lg" type="password" minlength="6" autocomplete="new-password" required />
                    </div>
                    <div class="col-md-6">
                      <label class="form-label" for="register-confirm">Confirmar senha</label>
                      <input id="register-confirm" v-model="registerForm.confirmPassword" class="form-control form-control-lg" type="password" minlength="6" autocomplete="new-password" required />
                    </div>
                  </div>
                  <p v-if="registerError" class="form-error" role="alert">{{ registerError }}</p>
                  <button class="btn btn-brand mt-4" type="submit">Criar conta e continuar</button>
                </form>
              </div>

              <div v-else-if="currentStep === 'privacy'" class="form-panel">
                <div class="panel-title">
                  <span class="panel-icon" aria-hidden="true">🛡️</span>
                  <div>
                    <h3>Suas escolhas de privacidade</h3>
                    <p>Transparência e controle sobre o uso dos seus dados.</p>
                  </div>
                </div>
                <div class="privacy-box">
                  <h4>Aviso de privacidade</h4>
                  <p>
                    Nome e e-mail são usados para identificar sua conta, confirmar o pedido,
                    acompanhar a retirada e creditar benefícios do programa de fidelidade.
                  </p>
                </div>
                <label class="choice-card required-choice">
                  <input v-model="privacyAccepted" type="checkbox" />
                  <span>
                    <strong>Li o aviso e autorizo o uso dos dados para realizar meu pedido.</strong>
                    <small>Necessário para identificação, pagamento e retirada.</small>
                  </span>
                </label>
                <label class="choice-card">
                  <input v-model="marketingAccepted" type="checkbox" />
                  <span>
                    <strong>Quero receber promoções.</strong>
                    <small>Opcional. A recusa não impede o pedido.</small>
                  </span>
                </label>
                <button class="btn btn-brand mt-4" type="button" :disabled="!privacyAccepted" @click="continueToPayment">
                  Continuar para pagamento
                </button>
              </div>

              <form v-else-if="currentStep === 'payment'" class="form-panel" @submit.prevent="pay">
                <div class="panel-title">
                  <span class="panel-icon" aria-hidden="true">💳</span>
                  <div>
                    <h3>{{ orderTotal === 0 ? 'Confirmar pedido' : 'Pagamento' }}</h3>
                    <p>{{ orderTotal === 0 ? 'Seus pontos cobrem o valor total deste pedido.' : 'Use seus pontos e informe os dados do cartão para concluir.' }}</p>
                  </div>
                </div>
                <div class="payment-summary">
                  <div class="payment-summary-row">
                    <span>Subtotal</span>
                    <strong>{{ money.format(subtotal) }}</strong>
                  </div>
                  <div v-if="pointsDiscount" class="payment-summary-row discount">
                    <span>Desconto ({{ applicablePoints }} pontos)</span>
                    <strong>− {{ money.format(pointsDiscount) }}</strong>
                  </div>
                  <div class="payment-summary-row total">
                    <span>Total a pagar</span>
                    <strong>{{ money.format(orderTotal) }}</strong>
                  </div>
                </div>
                <div v-if="user.points > 0" class="points-payment-card">
                  <div>
                    <strong>{{ user.points }} pontos disponíveis</strong>
                    <small>Crédito de {{ money.format(pointsCredit) }} · cada ponto vale {{ money.format(POINT_VALUE) }}</small>
                  </div>
                  <button v-if="!usePoints" class="btn btn-sm btn-points" type="button" @click="applyPoints">Usar pontos</button>
                  <button v-else class="btn btn-sm btn-outline-secondary" type="button" @click="removePoints">Remover</button>
                </div>
                <template v-if="orderTotal > 0">
                  <div class="payment-method-title"><span aria-hidden="true">●</span> Cartão de crédito</div>
                  <div class="row g-3 mt-1">
                    <div class="col-12">
                      <label class="form-label" for="card-number">Número do cartão</label>
                      <input id="card-number" v-model="paymentCard.number" class="form-control form-control-lg" inputmode="numeric" autocomplete="cc-number" placeholder="0000 0000 0000 0000" maxlength="19" required />
                    </div>
                    <div class="col-12">
                      <label class="form-label" for="card-holder">Nome impresso no cartão</label>
                      <input id="card-holder" v-model.trim="paymentCard.holder" class="form-control form-control-lg text-uppercase" type="text" autocomplete="cc-name" required />
                    </div>
                    <div class="col-6">
                      <label class="form-label" for="card-expiry">Validade</label>
                      <input id="card-expiry" v-model="paymentCard.expiry" class="form-control form-control-lg" inputmode="numeric" autocomplete="cc-exp" placeholder="MM/AA" maxlength="5" required />
                    </div>
                    <div class="col-6">
                      <label class="form-label" for="card-cvv">CVV</label>
                      <input id="card-cvv" v-model="paymentCard.cvv" class="form-control form-control-lg" type="password" inputmode="numeric" autocomplete="cc-csc" placeholder="000" maxlength="4" required />
                    </div>
                  </div>
                </template>
                <div v-if="paymentState === 'declined'" class="result-card declined" role="alert">
                  <strong>Pagamento recusado</strong>
                  <span>Não foi possível autorizar a compra. Confira os dados ou tente outro cartão.</span>
                  <button type="button" class="btn btn-sm btn-outline-danger" @click="retryPayment">Usar outro cartão</button>
                </div>
                <div v-else-if="paymentState === 'approved'" class="result-card approved" role="status">
                  <strong>Pagamento aprovado!</strong>
                  <span>Seu pedido foi confirmado. Abrindo acompanhamento…</span>
                </div>
                <button v-if="paymentState !== 'approved'" class="btn btn-brand mt-4" type="submit">
                  {{ orderTotal === 0 ? 'Confirmar pedido' : `Pagar ${money.format(orderTotal)}` }}
                </button>
              </form>

              <div v-else-if="currentStep === 'tracking'" class="tracking-panel">
                <div class="success-heading">
                  <span aria-hidden="true">✓</span>
                  <div>
                    <p>Pagamento aprovado</p>
                    <h3>Pedido {{ orderNumber }}</h3>
                  </div>
                </div>
                <div class="order-unit mt-4">
                  <span aria-hidden="true">📍</span>
                  <div>
                    <small>Unidade do pedido</small>
                    <strong>{{ selectedUnit.name }}</strong>
                    <p>{{ selectedUnit.address }}</p>
                  </div>
                </div>
                <div class="tracking-grid">
                  <div>
                    <h4>Acompanhe seu pedido</h4>
                    <ol class="order-timeline">
                      <li v-for="(status, index) in orderStatuses" :key="status" :class="{ done: orderStatus >= index, current: orderStatus === index }">
                        <span></span>
                        <div>
                          <strong>{{ status }}</strong>
                          <small v-if="orderStatus === index">Status atualizado agora</small>
                        </div>
                      </li>
                    </ol>
                    <button v-if="orderStatus < 3" class="btn btn-brand" type="button" @click="advanceOrder">
                      Atualizar status
                    </button>
                    <div v-else class="order-complete-actions">
                      <span class="eyebrow">Pedido concluído</span>
                      <h4>Obrigado pela preferência!</h4>
                      <p>Faça um novo pedido e use seus pontos como desconto no pagamento.</p>
                      <div class="d-flex flex-column flex-sm-row gap-2">
                        <button class="btn btn-brand" type="button" @click="startNewOrder">
                          Fazer novo pedido
                        </button>
                      </div>
                    </div>
                  </div>
                  <aside class="loyalty-card">
                    <span class="eyebrow">Clube Raízes</span>
                    <h4>Você ganhou {{ currentOrderRecord?.pointsEarned || 0 }} pontos!</h4>
                    <strong>{{ user.points }} pontos no total</strong>
                    <small>Equivalem a {{ money.format(pointsCredit) }} em crédito para o próximo pedido.</small>
                  </aside>
                </div>
              </div>
            </div>
          </div>
        </section>
      </template>
    </main>

    <footer class="site-footer">
      <div class="container d-flex flex-column flex-md-row justify-content-between gap-3">
        <div>
          <strong>Raízes do Nordeste</strong>
          <p>Sabores regionais, pedidos rápidos e retirada sem fila.</p>
        </div>
        <div class="text-md-end">
          <span>© 2026 Raízes do Nordeste</span>
          <p>Privacidade · Atendimento</p>
        </div>
      </div>
    </footer>

    <div id="ordersPanel" class="offcanvas offcanvas-end" tabindex="-1" aria-labelledby="ordersTitle">
      <div class="offcanvas-header">
        <div>
          <span class="eyebrow">Sua conta</span>
          <h2 id="ordersTitle" class="offcanvas-title">Meus pedidos</h2>
        </div>
        <button type="button" class="btn-close" data-bs-dismiss="offcanvas" aria-label="Fechar"></button>
      </div>
      <div class="offcanvas-body">
        <section v-if="user" class="account-credit-card mb-4" aria-labelledby="creditTitle">
          <span class="eyebrow">Clube Raízes</span>
          <div class="account-credit-heading">
            <div>
              <h3 id="creditTitle">{{ user.points }} pontos</h3>
              <p>{{ money.format(pointsCredit) }} disponíveis em crédito</p>
            </div>
            <span aria-hidden="true">🪙</span>
          </div>
          <small>Cada R$ 1 pago gera 1 ponto. Cada ponto vale {{ money.format(POINT_VALUE) }} de desconto.</small>
          <button
            v-if="user.points > 0 && cart.length && currentStep !== 'tracking'"
            class="btn btn-points w-100 mt-3"
            type="button"
            data-bs-dismiss="offcanvas"
            @click="applyPointsFromAccount"
          >
            Aplicar pontos neste pedido
          </button>
          <small v-else-if="user.points > 0">O crédito poderá ser aplicado durante o pagamento do próximo pedido.</small>
        </section>
        <div v-if="!user?.orders?.length" class="cart-empty">
          <span aria-hidden="true">🧾</span>
          <strong>Nenhum pedido encontrado</strong>
          <p>Seus pedidos concluídos aparecerão aqui.</p>
        </div>
        <div v-else class="order-history-list">
          <article v-for="order in user.orders" :key="order.id" class="order-history-card">
            <div class="order-history-header">
              <div>
                <strong>{{ order.id }}</strong>
                <small>{{ order.date }}</small>
              </div>
              <span :class="{ completed: order.status === 'Pedido retirado' }">{{ order.status }}</span>
            </div>
            <div v-if="order.unit" class="order-unit compact mb-3">
              <span aria-hidden="true">📍</span>
              <div>
                <small>Unidade do pedido</small>
                <strong>{{ order.unit.name }}</strong>
                <p>{{ order.unit.address }}</p>
              </div>
            </div>
            <ul class="list-unstyled mb-3">
              <li v-for="item in order.items" :key="`${order.id}-${item.name}`">
                <span>{{ item.quantity }}× {{ item.name }}</span>
                <strong>{{ money.format(item.price * item.quantity) }}</strong>
              </li>
            </ul>
            <div v-if="order.pointsUsed" class="order-history-discount">
              <span>{{ order.pointsUsed }} pontos utilizados</span>
              <strong>− {{ money.format(order.pointsDiscount) }}</strong>
            </div>
            <div class="order-history-footer">
              <span>+{{ order.pointsEarned }} pontos</span>
              <strong>{{ money.format(order.total) }}</strong>
            </div>
          </article>
        </div>
      </div>
    </div>

    <div id="cartPanel" class="offcanvas offcanvas-end" tabindex="-1" aria-labelledby="cartTitle">
      <div class="offcanvas-header">
        <div>
          <span class="eyebrow">Resumo</span>
          <h2 id="cartTitle" class="offcanvas-title">Seu pedido</h2>
        </div>
        <button type="button" class="btn-close" data-bs-dismiss="offcanvas" aria-label="Fechar"></button>
      </div>
      <div class="offcanvas-body d-flex flex-column">
        <div v-if="!cart.length" class="cart-empty">
          <span aria-hidden="true">🧺</span>
          <strong>Seu pedido está vazio</strong>
          <p>Adicione itens do cardápio para continuar.</p>
        </div>
        <template v-else>
          <ul class="cart-list list-unstyled">
            <li v-for="item in cart" :key="item.id">
              <span class="cart-emoji" aria-hidden="true">{{ item.emoji }}</span>
              <div class="flex-grow-1">
                <strong>{{ item.name }}</strong>
                <small>{{ item.promotion ? 'Preço promocional · ' : '' }}{{ money.format(item.price) }}</small>
                <div class="quantity-control" aria-label="Controle de quantidade">
                  <button type="button" :aria-label="`Remover uma unidade de ${item.name}`" @click="changeQuantity(item.id, -1)">−</button>
                  <span>{{ item.quantity }}</span>
                  <button type="button" :aria-label="`Adicionar uma unidade de ${item.name}`" @click="changeQuantity(item.id, 1)">+</button>
                </div>
              </div>
              <strong>{{ money.format(item.price * item.quantity) }}</strong>
            </li>
          </ul>
          <div class="cart-total mt-auto">
            <span>Subtotal</span>
            <strong>{{ money.format(subtotal) }}</strong>
          </div>
          <div v-if="pointsDiscount" class="cart-discount">
            <span>{{ applicablePoints }} pontos</span>
            <strong>− {{ money.format(pointsDiscount) }}</strong>
          </div>
          <div v-if="pointsDiscount" class="cart-payable">
            <span>Total a pagar</span>
            <strong>{{ money.format(orderTotal) }}</strong>
          </div>
          <button class="btn btn-brand btn-lg w-100 mt-3" type="button" data-bs-dismiss="offcanvas" @click="continueOrder">
            Continuar pedido
          </button>
        </template>
      </div>
    </div>

    <div v-if="toast" class="app-toast" role="status">{{ toast }}</div>

    <button
      v-if="selectedUnit && cartCount"
      class="mobile-cart-bar d-lg-none"
      type="button"
      data-bs-toggle="offcanvas"
      data-bs-target="#cartPanel"
    >
      <span>{{ cartCount }} item(ns)</span>
      <strong>Ver pedido · {{ money.format(orderTotal) }}</strong>
    </button>
  </div>
</template>
