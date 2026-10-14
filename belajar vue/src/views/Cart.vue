<script setup>
import { RouterLink } from 'vue-router'
import { useStore } from '../store/index.js'

const { cart, cartCount, cartTotal, removeFromCart, updateQty } = useStore()

const formatPrice = (p) => 'Rp ' + p.toLocaleString('id-ID')
const shipping = 15000
const total = () => cartTotal.value + (cartTotal.value >= 200000 ? 0 : shipping)
const freeShipping = () => cartTotal.value >= 200000
const shippingProgress = () => Math.min((cartTotal.value / 200000) * 100, 100)
</script>

<template>
  <div class="cart-page">
    <div class="page-hero">
      <div class="container">
        <p class="section-label">Belanja Kamu</p>
        <h1 class="page-title serif">Keranjang</h1>
      </div>
    </div>

    <div class="container cart-layout">
      <!-- Empty Cart -->
      <div v-if="cart.length === 0" class="empty-cart">
        <div class="empty-icon">🛍️</div>
        <h2>Keranjangmu masih kosong</h2>
        <p>Yuk, temukan produk favoritmu dan mulai belanja!</p>
        <RouterLink to="/products" class="btn btn-primary">Jelajahi Produk</RouterLink>
      </div>

      <template v-else>
        <!-- Cart Items -->
        <div class="cart-items">
          <!-- Free shipping progress -->
          <div class="ship-progress-bar" v-if="!freeShipping()">
            <p class="ship-progress-label">
              Tambah <strong>{{ formatPrice(200000 - cartTotal.value) }}</strong> lagi untuk gratis ongkir! 🚚
            </p>
            <div class="progress-track">
              <div class="progress-fill" :style="{ width: shippingProgress() + '%' }"></div>
            </div>
          </div>
          <div class="ship-progress-bar ship-unlocked" v-else>
            <p>🎉 Selamat! Kamu mendapat <strong>gratis ongkir</strong>!</p>
          </div>

          <div v-for="item in cart" :key="item.key" class="cart-item">
            <RouterLink :to="`/products/${item.id}`" class="item-img-wrap">
              <img :src="item.images[0]" :alt="item.name" class="item-img" />
            </RouterLink>
            <div class="item-info">
              <p class="item-category">{{ item.category }}</p>
              <RouterLink :to="`/products/${item.id}`" class="item-name">{{ item.name }}</RouterLink>
              <div class="item-variants">
                <span v-if="item.color" class="variant-chip">{{ item.color }}</span>
                <span v-if="item.size && item.size !== 'One Size'" class="variant-chip">{{ item.size }}</span>
              </div>
              <p class="item-price">{{ formatPrice(item.price) }}</p>
            </div>
            <div class="item-actions">
              <div class="qty-ctrl">
                <button @click="updateQty(item.key, item.qty - 1)">−</button>
                <span>{{ item.qty }}</span>
                <button @click="updateQty(item.key, item.qty + 1)">+</button>
              </div>
              <p class="item-subtotal">{{ formatPrice(item.price * item.qty) }}</p>
              <button class="remove-btn" @click="removeFromCart(item.key)" title="Hapus">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="3 6 5 6 21 6"/><path d="M19 6v14a2 2 0 01-2 2H7a2 2 0 01-2-2V6m3 0V4a1 1 0 011-1h4a1 1 0 011 1v2"/></svg>
              </button>
            </div>
          </div>
        </div>

        <!-- Order Summary -->
        <aside class="cart-summary">
          <div class="summary-card">
            <h2 class="summary-title">Ringkasan Pesanan</h2>

            <div class="summary-rows">
              <div class="summary-row">
                <span>Subtotal ({{ cartCount }} item)</span>
                <span>{{ formatPrice(cartTotal.value) }}</span>
              </div>
              <div class="summary-row">
                <span>Ongkos Kirim</span>
                <span :class="freeShipping() ? 'free-ship' : ''">
                  {{ freeShipping() ? 'GRATIS' : formatPrice(shipping) }}
                </span>
              </div>
              <div class="summary-row voucher-row">
                <input class="input voucher-input" placeholder="Kode Voucher" />
                <button class="btn btn-outline" style="white-space: nowrap; padding: 10px 16px; font-size: 13px;">Pakai</button>
              </div>
            </div>

            <div class="summary-divider"></div>

            <div class="summary-total">
              <span>Total</span>
              <span class="total-val">{{ formatPrice(total()) }}</span>
            </div>

            <RouterLink to="/checkout" class="btn btn-primary" style="width: 100%; justify-content: center; margin-top: 20px; font-size: 15px; padding: 16px;">
              Lanjut ke Pembayaran →
            </RouterLink>

            <div class="secure-badge">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>
              Transaksi aman & terenkripsi
            </div>
          </div>

          <div class="payment-methods">
            <p class="pay-label">Metode Pembayaran</p>
            <div class="pay-icons">
              <span class="pay-icon">💳 Kartu Kredit</span>
              <span class="pay-icon">🏧 Transfer Bank</span>
              <span class="pay-icon">📱 E-Wallet</span>
            </div>
          </div>
        </aside>
      </template>
    </div>
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

.cart-layout {
  display: grid;
  grid-template-columns: 1fr 380px;
  gap: 40px;
  padding-top: 40px;
  padding-bottom: 80px;
  align-items: start;
}

/* Empty cart */
.empty-cart {
  grid-column: 1 / -1;
  text-align: center;
  padding: 100px 24px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}
.empty-icon { font-size: 64px; }
.empty-cart h2 { font-size: 24px; font-weight: 700; }
.empty-cart p { color: var(--text-3); font-size: 15px; }

/* Ship progress */
.ship-progress-bar {
  background: var(--bg-2);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 16px 20px;
  margin-bottom: 20px;
}
.ship-progress-bar.ship-unlocked {
  background: rgba(16,185,129,0.08);
  border-color: rgba(16,185,129,0.3);
  color: var(--green);
}
.ship-progress-label { font-size: 13px; color: var(--text-2); margin-bottom: 10px; }
.progress-track { height: 6px; background: var(--bg-3); border-radius: 3px; overflow: hidden; }
.progress-fill { height: 100%; background: linear-gradient(90deg, var(--accent), var(--accent-2)); border-radius: 3px; transition: width 0.5s ease; }

/* Cart items */
.cart-item {
  display: grid;
  grid-template-columns: 100px 1fr auto;
  gap: 20px;
  align-items: center;
  padding: 24px;
  background: var(--bg-2);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  margin-bottom: 12px;
  transition: border-color var(--transition-fast);
}
.cart-item:hover { border-color: var(--border-strong); }
.item-img-wrap { border-radius: var(--radius); overflow: hidden; aspect-ratio: 1; }
.item-img { width: 100%; height: 100%; object-fit: cover; }
.item-category { font-size: 11px; color: var(--text-3); text-transform: uppercase; letter-spacing: 1px; margin-bottom: 4px; }
.item-name { font-weight: 600; font-size: 15px; margin-bottom: 8px; display: block; }
.item-name:hover { color: var(--accent-2); }
.item-variants { display: flex; gap: 6px; margin-bottom: 8px; }
.variant-chip {
  padding: 3px 10px;
  border-radius: 100px;
  font-size: 11px;
  background: var(--bg-3);
  border: 1px solid var(--border);
  color: var(--text-3);
}
.item-price { font-size: 14px; color: var(--text-3); }
.item-actions { display: flex; flex-direction: column; align-items: flex-end; gap: 12px; }
.qty-ctrl {
  display: flex;
  align-items: center;
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  overflow: hidden;
}
.qty-ctrl button {
  width: 32px;
  height: 36px;
  font-size: 18px;
  color: var(--text-2);
  background: var(--bg-3);
  transition: all var(--transition-fast);
}
.qty-ctrl button:hover { background: var(--surface-hover); color: var(--text-1); }
.qty-ctrl span { width: 40px; text-align: center; font-weight: 700; font-size: 14px; background: var(--bg-2); height: 36px; display: flex; align-items: center; justify-content: center; }
.item-subtotal { font-weight: 700; font-size: 15px; }
.remove-btn {
  color: var(--text-3);
  padding: 6px;
  border-radius: var(--radius-sm);
  transition: all var(--transition-fast);
}
.remove-btn:hover { color: var(--red); background: var(--red-soft); }

/* Summary */
.cart-summary { position: sticky; top: 80px; }
.summary-card {
  background: var(--bg-2);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 28px;
  margin-bottom: 16px;
}
.summary-title { font-size: 18px; font-weight: 700; margin-bottom: 24px; }
.summary-rows { display: flex; flex-direction: column; gap: 14px; margin-bottom: 24px; }
.summary-row { display: flex; justify-content: space-between; align-items: center; font-size: 14px; color: var(--text-2); }
.free-ship { color: var(--green); font-weight: 700; }
.voucher-row { gap: 8px; }
.voucher-input { height: 44px; font-size: 13px; }
.summary-divider { height: 1px; background: var(--border); margin-bottom: 20px; }
.summary-total { display: flex; justify-content: space-between; align-items: center; }
.summary-total span:first-child { font-size: 16px; font-weight: 600; }
.total-val { font-size: 22px; font-weight: 800; color: var(--accent-2); }
.secure-badge {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  margin-top: 12px;
  font-size: 12px;
  color: var(--text-3);
}

.payment-methods {
  background: var(--bg-2);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 20px;
}
.pay-label { font-size: 12px; color: var(--text-3); margin-bottom: 12px; font-weight: 600; text-transform: uppercase; letter-spacing: 1px; }
.pay-icons { display: flex; flex-direction: column; gap: 8px; }
.pay-icon { font-size: 13px; color: var(--text-2); padding: 8px 0; border-bottom: 1px solid var(--border); }
.pay-icon:last-child { border-bottom: none; }

@media (max-width: 768px) {
  .cart-layout { grid-template-columns: 1fr; }
  .cart-summary { position: static; }
  .cart-item { grid-template-columns: 80px 1fr; }
  .item-actions { grid-column: 1 / -1; flex-direction: row; justify-content: space-between; align-items: center; }
}
</style>
