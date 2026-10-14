<script setup>
import { ref } from 'vue'
import { RouterLink } from 'vue-router'
import { useStore } from '../store/index.js'

const { cart, cartTotal } = useStore()

const formatPrice = (p) => 'Rp ' + p.toLocaleString('id-ID')
const shipping = 15000
const total = () => cartTotal.value + (cartTotal.value >= 200000 ? 0 : shipping)

const step = ref(1)
const form = ref({
  name: '', email: '', phone: '',
  address: '', city: '', province: '', zip: '',
  payment: 'transfer',
})
const isSubmitted = ref(false)

const steps = ['Pengiriman', 'Pembayaran', 'Konfirmasi']

const nextStep = () => {
  if (step.value < 3) step.value++
}
const prevStep = () => {
  if (step.value > 1) step.value--
}
const submitOrder = () => {
  isSubmitted.value = true
}

const orderNum = () => 'STY-' + Math.random().toString(36).substring(2, 8).toUpperCase()
const num = orderNum()
</script>

<template>
  <div class="checkout-page">
    <div class="page-hero">
      <div class="container">
        <p class="section-label">Pembayaran</p>
        <h1 class="page-title serif">Checkout</h1>
      </div>
    </div>

    <!-- Success State -->
    <div v-if="isSubmitted" class="success-state">
      <div class="success-card">
        <div class="success-icon">✓</div>
        <h2 class="success-title serif">Pesanan Diterima!</h2>
        <p class="success-sub">Terima kasih sudah berbelanja di Stylo. Pesananmu sedang diproses.</p>
        <div class="order-info">
          <div class="order-row">
            <span>Nomor Pesanan</span>
            <strong>{{ num }}</strong>
          </div>
          <div class="order-row">
            <span>Estimasi Tiba</span>
            <strong>2-5 Hari Kerja</strong>
          </div>
          <div class="order-row">
            <span>Total Dibayar</span>
            <strong style="color: var(--accent-2);">{{ formatPrice(total()) }}</strong>
          </div>
        </div>
        <RouterLink to="/" class="btn btn-primary" style="width: 100%; justify-content: center;">
          Kembali Berbelanja
        </RouterLink>
      </div>
    </div>

    <!-- Checkout Form -->
    <template v-else>
      <div class="container checkout-layout">
        <div class="checkout-main">
          <!-- Step Indicator -->
          <div class="step-indicator">
            <div
              v-for="(s, i) in steps"
              :key="s"
              class="step-item"
              :class="{ active: i + 1 === step, done: i + 1 < step }"
            >
              <div class="step-circle">
                <span v-if="i + 1 < step">✓</span>
                <span v-else>{{ i + 1 }}</span>
              </div>
              <span class="step-label">{{ s }}</span>
              <div v-if="i < steps.length - 1" class="step-line" :class="{ done: i + 1 < step }"></div>
            </div>
          </div>

          <!-- Step 1: Shipping -->
          <div v-if="step === 1" class="form-section">
            <h2 class="form-title">Informasi Pengiriman</h2>
            <div class="form-grid">
              <div class="form-group span-2">
                <label>Nama Lengkap *</label>
                <input v-model="form.name" class="input" placeholder="Masukkan nama lengkap" />
              </div>
              <div class="form-group">
                <label>Email *</label>
                <input v-model="form.email" type="email" class="input" placeholder="email@kamu.com" />
              </div>
              <div class="form-group">
                <label>Nomor HP *</label>
                <input v-model="form.phone" class="input" placeholder="+62 8xx xxxx xxxx" />
              </div>
              <div class="form-group span-2">
                <label>Alamat Lengkap *</label>
                <input v-model="form.address" class="input" placeholder="Nama jalan, nomor, RT/RW" />
              </div>
              <div class="form-group">
                <label>Kota *</label>
                <input v-model="form.city" class="input" placeholder="Jakarta Selatan" />
              </div>
              <div class="form-group">
                <label>Provinsi *</label>
                <input v-model="form.province" class="input" placeholder="DKI Jakarta" />
              </div>
              <div class="form-group">
                <label>Kode Pos *</label>
                <input v-model="form.zip" class="input" placeholder="12345" />
              </div>
            </div>
          </div>

          <!-- Step 2: Payment -->
          <div v-if="step === 2" class="form-section">
            <h2 class="form-title">Metode Pembayaran</h2>
            <div class="payment-options">
              <label
                v-for="opt in [
                  { val: 'transfer', label: 'Transfer Bank', icon: '🏧', desc: 'BCA, Mandiri, BNI, BRI' },
                  { val: 'card', label: 'Kartu Kredit/Debit', icon: '💳', desc: 'Visa, Mastercard, JCB' },
                  { val: 'gopay', label: 'GoPay', icon: '💚', desc: 'Saldo GoPay' },
                  { val: 'dana', label: 'DANA', icon: '💙', desc: 'Saldo DANA' },
                  { val: 'ovo', label: 'OVO', icon: '💜', desc: 'Saldo OVO' },
                ]"
                :key="opt.val"
                class="payment-option"
                :class="{ active: form.payment === opt.val }"
              >
                <input type="radio" v-model="form.payment" :value="opt.val" style="display: none;" />
                <span class="pay-option-icon">{{ opt.icon }}</span>
                <div class="pay-option-info">
                  <p class="pay-option-title">{{ opt.label }}</p>
                  <p class="pay-option-desc">{{ opt.desc }}</p>
                </div>
                <div class="pay-check" :class="{ active: form.payment === opt.val }">
                  <span v-if="form.payment === opt.val">✓</span>
                </div>
              </label>
            </div>
          </div>

          <!-- Step 3: Confirmation -->
          <div v-if="step === 3" class="form-section">
            <h2 class="form-title">Konfirmasi Pesanan</h2>
            <div class="confirm-section">
              <h3 class="confirm-label">Alamat Pengiriman</h3>
              <div class="confirm-box">
                <p class="confirm-name">{{ form.name || 'Nama Lengkap' }}</p>
                <p class="confirm-detail">{{ form.phone || '—' }}</p>
                <p class="confirm-detail">{{ form.address || '—' }}</p>
                <p class="confirm-detail">{{ [form.city, form.province, form.zip].filter(Boolean).join(', ') || '—' }}</p>
              </div>
            </div>
            <div class="confirm-section">
              <h3 class="confirm-label">Metode Pembayaran</h3>
              <div class="confirm-box">
                <p>{{ { transfer: '🏧 Transfer Bank', card: '💳 Kartu Kredit/Debit', gopay: '💚 GoPay', dana: '💙 DANA', ovo: '💜 OVO' }[form.payment] }}</p>
              </div>
            </div>
            <div class="confirm-section">
              <h3 class="confirm-label">Item yang Dipesan</h3>
              <div v-for="item in cart" :key="item.key" class="confirm-item">
                <img :src="item.images[0]" :alt="item.name" class="confirm-img" />
                <div class="confirm-item-info">
                  <p class="confirm-item-name">{{ item.name }}</p>
                  <p class="confirm-item-var" v-if="item.color">{{ item.color }}{{ item.size && item.size !== 'One Size' ? ` · ${item.size}` : '' }}</p>
                  <p class="confirm-item-qty">{{ item.qty }}x {{ formatPrice(item.price) }}</p>
                </div>
                <p class="confirm-item-total">{{ formatPrice(item.price * item.qty) }}</p>
              </div>
            </div>
          </div>

          <!-- Navigation Buttons -->
          <div class="form-nav">
            <button v-if="step > 1" class="btn btn-outline" @click="prevStep">← Kembali</button>
            <div v-else></div>
            <button v-if="step < 3" class="btn btn-primary" @click="nextStep">
              Lanjutkan →
            </button>
            <button v-else class="btn btn-primary" @click="submitOrder">
              ✓ Konfirmasi & Bayar
            </button>
          </div>
        </div>

        <!-- Order Summary Sidebar -->
        <aside class="checkout-summary">
          <div class="summary-card">
            <h3 class="summary-title">Ringkasan</h3>
            <div class="summary-items">
              <div v-for="item in cart" :key="item.key" class="summary-item">
                <div class="summary-item-img-wrap">
                  <img :src="item.images[0]" :alt="item.name" class="summary-item-img" />
                  <span class="item-qty-badge">{{ item.qty }}</span>
                </div>
                <div class="summary-item-info">
                  <p class="summary-item-name">{{ item.name }}</p>
                  <p class="summary-item-var" v-if="item.color">{{ item.color }}</p>
                </div>
                <p class="summary-item-price">{{ formatPrice(item.price * item.qty) }}</p>
              </div>
            </div>
            <div class="summary-divider"></div>
            <div class="summary-rows">
              <div class="summary-row">
                <span>Subtotal</span>
                <span>{{ formatPrice(cartTotal.value) }}</span>
              </div>
              <div class="summary-row">
                <span>Ongkir</span>
                <span :class="cartTotal.value >= 200000 ? 'free-text' : ''">
                  {{ cartTotal.value >= 200000 ? 'GRATIS' : formatPrice(shipping) }}
                </span>
              </div>
            </div>
            <div class="summary-divider"></div>
            <div class="summary-total">
              <span>Total</span>
              <span class="total-val">{{ formatPrice(total()) }}</span>
            </div>
          </div>
        </aside>
      </div>
    </template>
  </div>
</template>

<style scoped>
.page-hero {
  background: var(--bg-1);
  border-bottom: 1px solid var(--border);
  padding: 48px 0 40px;
}
.section-label {
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: var(--accent-2);
  margin-bottom: 8px;
}
.page-title { font-size: 2.4rem; }

/* Success */
.success-state {
  display: flex;
  justify-content: center;
  padding: 80px 24px;
}
.success-card {
  background: var(--bg-2);
  border: 1px solid var(--border);
  border-radius: var(--radius-xl);
  padding: 48px;
  max-width: 480px;
  width: 100%;
  text-align: center;
}
.success-icon {
  width: 72px;
  height: 72px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--green), #059669);
  color: white;
  font-size: 32px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 24px;
  box-shadow: 0 8px 24px rgba(16,185,129,0.3);
}
.success-title { font-size: 2rem; margin-bottom: 12px; }
.success-sub { color: var(--text-2); margin-bottom: 32px; line-height: 1.7; }
.order-info {
  background: var(--bg-3);
  border-radius: var(--radius);
  padding: 20px;
  margin-bottom: 28px;
  text-align: left;
}
.order-row { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid var(--border); font-size: 14px; }
.order-row:last-child { border-bottom: none; }
.order-row span { color: var(--text-3); }

/* Layout */
.checkout-layout {
  display: grid;
  grid-template-columns: 1fr 360px;
  gap: 40px;
  padding-top: 40px;
  padding-bottom: 80px;
  align-items: start;
}

/* Step indicator */
.step-indicator {
  display: flex;
  align-items: center;
  margin-bottom: 40px;
}
.step-item { display: flex; align-items: center; flex: 1; }
.step-item:last-child { flex: none; }
.step-circle {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  border: 2px solid var(--border);
  background: var(--bg-2);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 13px;
  font-weight: 700;
  color: var(--text-3);
  flex-shrink: 0;
  transition: all var(--transition);
}
.step-item.active .step-circle {
  border-color: var(--accent);
  background: rgba(124,58,237,0.15);
  color: var(--accent-2);
}
.step-item.done .step-circle {
  border-color: var(--green);
  background: rgba(16,185,129,0.15);
  color: var(--green);
}
.step-label {
  font-size: 13px;
  color: var(--text-3);
  margin-left: 10px;
  font-weight: 500;
  white-space: nowrap;
}
.step-item.active .step-label { color: var(--text-1); font-weight: 600; }
.step-line {
  flex: 1;
  height: 2px;
  background: var(--border);
  margin: 0 12px;
  transition: background var(--transition);
}
.step-line.done { background: var(--green); }

/* Form */
.form-section {
  background: var(--bg-2);
  border: 1px solid var(--border);
  border-radius: var(--radius-xl);
  padding: 36px;
  margin-bottom: 20px;
}
.form-title { font-size: 1.3rem; font-weight: 700; margin-bottom: 28px; }
.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}
.form-group { display: flex; flex-direction: column; gap: 8px; }
.form-group.span-2 { grid-column: 1 / -1; }
.form-group label { font-size: 13px; font-weight: 500; color: var(--text-2); }

/* Payment options */
.payment-options { display: flex; flex-direction: column; gap: 10px; }
.payment-option {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 16px 20px;
  background: var(--bg-3);
  border: 2px solid var(--border);
  border-radius: var(--radius);
  cursor: pointer;
  transition: all var(--transition-fast);
}
.payment-option:hover { border-color: var(--border-strong); }
.payment-option.active { border-color: var(--accent); background: rgba(124,58,237,0.08); }
.pay-option-icon { font-size: 28px; flex-shrink: 0; }
.pay-option-title { font-weight: 600; font-size: 14px; }
.pay-option-desc { font-size: 12px; color: var(--text-3); }
.pay-check {
  margin-left: auto;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  border: 2px solid var(--border);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  color: white;
  flex-shrink: 0;
  transition: all var(--transition-fast);
}
.pay-check.active { background: var(--accent); border-color: var(--accent); }

/* Confirmation */
.confirm-section { margin-bottom: 28px; }
.confirm-label {
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 1.5px;
  text-transform: uppercase;
  color: var(--text-3);
  margin-bottom: 12px;
}
.confirm-box {
  padding: 16px;
  background: var(--bg-3);
  border-radius: var(--radius);
  border: 1px solid var(--border);
}
.confirm-name { font-weight: 600; margin-bottom: 4px; }
.confirm-detail { color: var(--text-2); font-size: 14px; margin-bottom: 2px; }
.confirm-item {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 14px;
  background: var(--bg-3);
  border-radius: var(--radius);
  border: 1px solid var(--border);
  margin-bottom: 8px;
}
.confirm-img { width: 60px; height: 60px; border-radius: var(--radius-sm); object-fit: cover; flex-shrink: 0; }
.confirm-item-name { font-weight: 600; font-size: 14px; }
.confirm-item-var { font-size: 12px; color: var(--text-3); }
.confirm-item-qty { font-size: 12px; color: var(--text-3); }
.confirm-item-total { margin-left: auto; font-weight: 700; white-space: nowrap; }

/* Nav buttons */
.form-nav {
  display: flex;
  justify-content: space-between;
  gap: 16px;
}

/* Summary sidebar */
.checkout-summary { position: sticky; top: 80px; }
.summary-card {
  background: var(--bg-2);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 24px;
}
.summary-title { font-size: 16px; font-weight: 700; margin-bottom: 20px; }
.summary-items { display: flex; flex-direction: column; gap: 14px; margin-bottom: 20px; }
.summary-item { display: flex; align-items: center; gap: 12px; }
.summary-item-img-wrap { position: relative; flex-shrink: 0; }
.summary-item-img { width: 56px; height: 56px; border-radius: var(--radius-sm); object-fit: cover; }
.item-qty-badge {
  position: absolute;
  top: -6px;
  right: -6px;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: var(--accent);
  color: white;
  font-size: 10px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
}
.summary-item-name { font-size: 13px; font-weight: 600; line-height: 1.4; }
.summary-item-var { font-size: 11px; color: var(--text-3); }
.summary-item-price { margin-left: auto; font-size: 13px; font-weight: 600; white-space: nowrap; }
.summary-divider { height: 1px; background: var(--border); margin: 16px 0; }
.summary-rows { display: flex; flex-direction: column; gap: 10px; }
.summary-row { display: flex; justify-content: space-between; font-size: 13px; color: var(--text-2); }
.free-text { color: var(--green); font-weight: 700; }
.summary-total { display: flex; justify-content: space-between; align-items: center; }
.summary-total span:first-child { font-weight: 600; }
.total-val { font-size: 18px; font-weight: 800; color: var(--accent-2); }

@media (max-width: 768px) {
  .checkout-layout { grid-template-columns: 1fr; }
  .checkout-summary { position: static; }
  .form-grid { grid-template-columns: 1fr; }
  .form-group.span-2 { grid-column: auto; }
}
</style>
