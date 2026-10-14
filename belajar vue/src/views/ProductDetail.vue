<script setup>
import { ref, computed } from 'vue'
import { useRoute, RouterLink } from 'vue-router'
import { products, useStore } from '../store/index.js'

const route = useRoute()
const { addToCart, toggleWishlist, isWishlisted } = useStore()

const product = computed(() => products.find(p => p.id === Number(route.params.id)))
const related = computed(() => products.filter(p => p.category === product.value?.category && p.id !== product.value?.id).slice(0, 4))

const activeImg = ref(0)
const selectedColor = ref(null)
const selectedSize = ref(null)
const qty = ref(1)

const formatPrice = (p) => 'Rp ' + p.toLocaleString('id-ID')
const discount = (orig, price) => Math.round((1 - price / orig) * 100)
const stars = (r) => '★'.repeat(Math.floor(r)) + (r % 1 ? '½' : '')

const handleAdd = () => {
  if (!product.value) return
  addToCart(product.value, qty.value, selectedColor.value || product.value.colors[0], selectedSize.value || product.value.sizes[0])
}

const tabs = ['Deskripsi', 'Ulasan', 'Pengiriman']
const activeTab = ref('Deskripsi')

const reviews = [
  { name: 'Raka S.', rating: 5, date: '15 Sep 2026', comment: 'Kualitas luar biasa! Bahannya enak banget di kulit, potongannya pas. Sangat rekomendasi.' },
  { name: 'Dewi A.', rating: 5, date: '10 Sep 2026', comment: 'Sudah beli 3x di sini. Selalu puas dengan kualitas dan kecepatannya pengiriman.' },
  { name: 'Budi P.', rating: 4, date: '5 Sep 2026', comment: 'Produknya bagus, sesuai foto. Minus sedikit di jahitan tapi overall oke.' },
]
</script>

<template>
  <div v-if="product" class="detail-page">
    <!-- Breadcrumb -->
    <div class="breadcrumb-bar">
      <div class="container">
        <nav class="breadcrumb">
          <RouterLink to="/">Beranda</RouterLink>
          <span>›</span>
          <RouterLink to="/products">Produk</RouterLink>
          <span>›</span>
          <span class="breadcrumb-current">{{ product.name }}</span>
        </nav>
      </div>
    </div>

    <!-- Main Section -->
    <section class="container detail-grid">
      <!-- Gallery -->
      <div class="gallery">
        <div class="gallery-main">
          <img :src="product.images[activeImg]" :alt="product.name" class="gallery-img" />
          <button
            class="wishlist-fab"
            :class="{ active: isWishlisted(product.id) }"
            @click="toggleWishlist(product.id)"
          >{{ isWishlisted(product.id) ? '♥' : '♡' }}</button>
          <span class="gallery-badge badge badge-purple">{{ product.badge }}</span>
        </div>
        <div class="gallery-thumbs">
          <button
            v-for="(img, i) in product.images"
            :key="i"
            class="thumb"
            :class="{ active: i === activeImg }"
            @click="activeImg = i"
          >
            <img :src="img" :alt="`${product.name} ${i + 1}`" />
          </button>
        </div>
      </div>

      <!-- Info Panel -->
      <div class="info-panel">
        <p class="info-category">{{ product.category }}</p>
        <h1 class="info-name serif">{{ product.name }}</h1>

        <!-- Rating & Stats -->
        <div class="info-stats">
          <div class="stat-group">
            <span class="stars-lg">{{ stars(product.rating) }}</span>
            <span class="stat-val">{{ product.rating }}</span>
            <span class="stat-sep">·</span>
            <span class="stat-label">{{ product.reviews }} ulasan</span>
          </div>
          <div class="stat-sep-bar"></div>
          <span class="stat-label">{{ product.sold.toLocaleString('id-ID') }} terjual</span>
          <div class="stat-sep-bar"></div>
          <span :class="product.stock > 10 ? 'in-stock' : 'low-stock'">
            {{ product.stock > 10 ? '✓ Stok tersedia' : `⚠ Stok terbatas (${product.stock} tersisa)` }}
          </span>
        </div>

        <!-- Pricing -->
        <div class="info-pricing">
          <span class="info-price">{{ formatPrice(product.price) }}</span>
          <div class="price-row">
            <span class="info-orig">{{ formatPrice(product.originalPrice) }}</span>
            <span class="discount-tag">Hemat {{ discount(product.originalPrice, product.price) }}%</span>
          </div>
        </div>

        <!-- Color selector -->
        <div class="selector-group">
          <p class="selector-label">
            Warna: <span class="selector-val">{{ selectedColor || product.colors[0] }}</span>
          </p>
          <div class="color-list">
            <button
              v-for="color in product.colors"
              :key="color"
              class="color-chip"
              :class="{ active: (selectedColor || product.colors[0]) === color }"
              @click="selectedColor = color"
            >{{ color }}</button>
          </div>
        </div>

        <!-- Size selector -->
        <div class="selector-group" v-if="product.sizes.length > 1 || product.sizes[0] !== 'One Size'">
          <p class="selector-label">
            Ukuran: <span class="selector-val">{{ selectedSize || product.sizes[0] }}</span>
          </p>
          <div class="size-list">
            <button
              v-for="size in product.sizes"
              :key="size"
              class="size-chip"
              :class="{ active: (selectedSize || product.sizes[0]) === size }"
              @click="selectedSize = size"
            >{{ size }}</button>
          </div>
        </div>

        <!-- Qty & CTA -->
        <div class="action-row">
          <div class="qty-ctrl">
            <button @click="qty = Math.max(1, qty - 1)">−</button>
            <span>{{ qty }}</span>
            <button @click="qty = Math.min(product.stock, qty + 1)">+</button>
          </div>
          <button class="btn btn-primary flex-1" @click="handleAdd">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M6 2L3 6v14a2 2 0 002 2h14a2 2 0 002-2V6l-3-4z"/><line x1="3" y1="6" x2="21" y2="6"/><path d="M16 10a4 4 0 01-8 0"/></svg>
            Tambah ke Keranjang
          </button>
          <RouterLink to="/cart" class="btn btn-outline" @click="handleAdd">Beli Sekarang</RouterLink>
        </div>

        <!-- Tags -->
        <div class="tags">
          <span v-for="tag in product.tags" :key="tag" class="tag">#{{ tag }}</span>
        </div>
      </div>
    </section>

    <!-- Tabs -->
    <section class="tabs-section">
      <div class="container">
        <div class="tabs">
          <button
            v-for="tab in tabs"
            :key="tab"
            class="tab"
            :class="{ active: activeTab === tab }"
            @click="activeTab = tab"
          >{{ tab }}</button>
        </div>
        <div class="tab-content">
          <div v-if="activeTab === 'Deskripsi'" class="tab-pane">
            <p>{{ product.description }}</p>
            <ul class="feature-list">
              <li>Bahan premium berkualitas tinggi</li>
              <li>Tersedia dalam berbagai pilihan warna</li>
              <li>Garansi kualitas 30 hari</li>
              <li>Gratis ongkir untuk pembelian di atas Rp 200.000</li>
            </ul>
          </div>
          <div v-else-if="activeTab === 'Ulasan'" class="tab-pane">
            <div class="review-summary">
              <div class="review-score">
                <span class="score-num">{{ product.rating }}</span>
                <div class="stars" style="font-size: 24px; color: var(--gold);">★★★★★</div>
                <span style="color: var(--text-3); font-size: 13px;">{{ product.reviews }} ulasan</span>
              </div>
              <div class="review-bars">
                <div v-for="n in [5,4,3,2,1]" :key="n" class="review-bar-row">
                  <span class="bar-label">{{ n }}★</span>
                  <div class="bar-track"><div class="bar-fill" :style="{ width: (n === 5 ? 68 : n === 4 ? 20 : n === 3 ? 8 : 3) + '%' }"></div></div>
                  <span class="bar-pct">{{ n === 5 ? 68 : n === 4 ? 20 : n === 3 ? 8 : 3 }}%</span>
                </div>
              </div>
            </div>
            <div class="review-list">
              <div v-for="r in reviews" :key="r.name" class="review-card">
                <div class="review-header">
                  <div class="reviewer-avatar">{{ r.name[0] }}</div>
                  <div>
                    <p class="reviewer-name">{{ r.name }}</p>
                    <p class="review-date">{{ r.date }}</p>
                  </div>
                  <div class="review-stars">
                    <span v-for="i in r.rating" :key="i" style="color: var(--gold);">★</span>
                  </div>
                </div>
                <p class="review-comment">{{ r.comment }}</p>
              </div>
            </div>
          </div>
          <div v-else class="tab-pane">
            <div class="ship-info">
              <div class="ship-row">
                <span class="ship-icon">🚚</span>
                <div>
                  <p class="ship-title">Pengiriman Reguler</p>
                  <p class="ship-desc">2-5 hari kerja — Gratis untuk pembelian di atas Rp 200.000</p>
                </div>
                <span class="ship-price">Rp 15.000</span>
              </div>
              <div class="ship-row">
                <span class="ship-icon">⚡</span>
                <div>
                  <p class="ship-title">Pengiriman Express</p>
                  <p class="ship-desc">Tiba hari yang sama atau besok (khusus Jabodetabek)</p>
                </div>
                <span class="ship-price">Rp 35.000</span>
              </div>
              <div class="ship-row">
                <span class="ship-icon">↩️</span>
                <div>
                  <p class="ship-title">Kebijakan Retur</p>
                  <p class="ship-desc">Retur gratis dalam 30 hari jika produk tidak sesuai deskripsi</p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Related Products -->
    <section class="section">
      <div class="container">
        <h2 class="serif" style="font-size: 1.8rem; margin-bottom: 32px;">Produk Serupa</h2>
        <div class="related-grid">
          <RouterLink
            v-for="rp in related"
            :key="rp.id"
            :to="`/products/${rp.id}`"
            class="product-card card"
          >
            <div class="product-img-wrap">
              <img :src="rp.images[0]" :alt="rp.name" class="product-img" loading="lazy" />
            </div>
            <div class="product-info">
              <h3 class="product-name">{{ rp.name }}</h3>
              <p class="price">{{ formatPrice(rp.price) }}</p>
            </div>
          </RouterLink>
        </div>
      </div>
    </section>
  </div>

  <!-- 404 -->
  <div v-else class="not-found">
    <span>😕</span>
    <h2>Produk tidak ditemukan</h2>
    <RouterLink to="/products" class="btn btn-primary">Kembali ke Produk</RouterLink>
  </div>
</template>

<style scoped>
.breadcrumb-bar {
  background: var(--bg-1);
  border-bottom: 1px solid var(--border);
  padding: 14px 0;
}
.breadcrumb {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 13px;
  color: var(--text-3);
}
.breadcrumb a:hover { color: var(--accent-2); }
.breadcrumb-current { color: var(--text-2); }

.detail-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 64px;
  padding-top: 48px;
  padding-bottom: 48px;
  align-items: start;
}

/* Gallery */
.gallery-main {
  position: relative;
  border-radius: var(--radius-xl);
  overflow: hidden;
  aspect-ratio: 4/5;
}
.gallery-img { width: 100%; height: 100%; object-fit: cover; }
.wishlist-fab {
  position: absolute;
  top: 16px;
  right: 16px;
  width: 44px;
  height: 44px;
  border-radius: 50%;
  background: rgba(0,0,0,0.5);
  backdrop-filter: blur(8px);
  border: 1px solid var(--border);
  color: white;
  font-size: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all var(--transition-fast);
}
.wishlist-fab:hover, .wishlist-fab.active { background: rgba(239,68,68,0.2); border-color: var(--red); color: var(--red); }
.gallery-badge { position: absolute; top: 16px; left: 16px; }
.gallery-thumbs { display: flex; gap: 10px; margin-top: 12px; }
.thumb {
  width: 72px;
  height: 72px;
  border-radius: var(--radius-sm);
  overflow: hidden;
  border: 2px solid transparent;
  transition: border-color var(--transition-fast);
  flex-shrink: 0;
}
.thumb img { width: 100%; height: 100%; object-fit: cover; }
.thumb.active { border-color: var(--accent); }

/* Info */
.info-category { font-size: 12px; color: var(--accent-2); text-transform: uppercase; letter-spacing: 1.5px; font-weight: 600; margin-bottom: 10px; }
.info-name { font-size: 2rem; margin-bottom: 20px; line-height: 1.2; }
.info-stats { display: flex; align-items: center; gap: 12px; flex-wrap: wrap; margin-bottom: 24px; }
.stat-group { display: flex; align-items: center; gap: 6px; }
.stars-lg { color: var(--gold); font-size: 14px; }
.stat-val { font-weight: 700; font-size: 15px; }
.stat-label { color: var(--text-3); font-size: 13px; }
.stat-sep { color: var(--text-3); }
.stat-sep-bar { width: 1px; height: 14px; background: var(--border-strong); }
.in-stock { color: var(--green); font-size: 13px; font-weight: 500; }
.low-stock { color: var(--gold); font-size: 13px; font-weight: 500; }

.info-pricing { margin-bottom: 28px; }
.info-price { font-size: 2.2rem; font-weight: 800; display: block; margin-bottom: 6px; background: linear-gradient(135deg, #fff, var(--accent-2)); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; }
.price-row { display: flex; align-items: center; gap: 10px; }
.info-orig { color: var(--text-3); text-decoration: line-through; font-size: 15px; }

.selector-group { margin-bottom: 24px; }
.selector-label { font-size: 13px; color: var(--text-2); margin-bottom: 10px; font-weight: 500; }
.selector-val { color: var(--text-1); font-weight: 600; }
.color-list { display: flex; flex-wrap: wrap; gap: 8px; }
.color-chip {
  padding: 7px 14px;
  border-radius: 100px;
  font-size: 13px;
  background: var(--bg-3);
  border: 1px solid var(--border);
  color: var(--text-2);
  transition: all var(--transition-fast);
}
.color-chip:hover, .color-chip.active {
  border-color: var(--accent);
  color: var(--accent-2);
  background: rgba(124,58,237,0.1);
}
.size-list { display: flex; flex-wrap: wrap; gap: 8px; }
.size-chip {
  width: 44px;
  height: 44px;
  border-radius: var(--radius-sm);
  font-size: 13px;
  font-weight: 600;
  background: var(--bg-3);
  border: 1px solid var(--border);
  color: var(--text-2);
  transition: all var(--transition-fast);
  display: flex;
  align-items: center;
  justify-content: center;
}
.size-chip:hover, .size-chip.active {
  border-color: var(--accent);
  color: var(--accent-2);
  background: rgba(124,58,237,0.1);
}

.action-row { display: flex; gap: 12px; align-items: center; margin-bottom: 24px; flex-wrap: wrap; }
.qty-ctrl {
  display: flex;
  align-items: center;
  border: 1px solid var(--border);
  border-radius: var(--radius);
  overflow: hidden;
  flex-shrink: 0;
}
.qty-ctrl button {
  width: 40px;
  height: 48px;
  font-size: 20px;
  color: var(--text-2);
  background: var(--bg-3);
  transition: all var(--transition-fast);
}
.qty-ctrl button:hover { background: var(--surface-hover); color: var(--text-1); }
.qty-ctrl span { width: 48px; text-align: center; font-weight: 700; font-size: 15px; background: var(--bg-2); }
.flex-1 { flex: 1; justify-content: center; }

.tags { display: flex; flex-wrap: wrap; gap: 8px; }
.tag {
  padding: 4px 12px;
  border-radius: 100px;
  font-size: 12px;
  color: var(--text-3);
  background: var(--surface);
  border: 1px solid var(--border);
}

/* Tabs */
.tabs-section {
  background: var(--bg-1);
  border-top: 1px solid var(--border);
  border-bottom: 1px solid var(--border);
  padding: 0 0 48px;
}
.tabs {
  display: flex;
  gap: 0;
  border-bottom: 1px solid var(--border);
  margin-bottom: 40px;
}
.tab {
  padding: 16px 28px;
  font-size: 14px;
  font-weight: 600;
  color: var(--text-3);
  border-bottom: 2px solid transparent;
  transition: all var(--transition-fast);
  margin-bottom: -1px;
}
.tab:hover { color: var(--text-2); }
.tab.active { color: var(--accent-2); border-bottom-color: var(--accent); }
.tab-pane { color: var(--text-2); line-height: 1.8; }
.feature-list { margin-top: 20px; padding-left: 20px; display: flex; flex-direction: column; gap: 8px; }
.feature-list li { font-size: 14px; }

/* Reviews */
.review-summary {
  display: grid;
  grid-template-columns: 160px 1fr;
  gap: 40px;
  margin-bottom: 40px;
  padding: 32px;
  background: var(--bg-2);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}
.review-score { display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 8px; }
.score-num { font-size: 56px; font-weight: 800; }
.review-bars { display: flex; flex-direction: column; gap: 8px; justify-content: center; }
.review-bar-row { display: flex; align-items: center; gap: 12px; }
.bar-label { font-size: 13px; color: var(--text-3); width: 24px; }
.bar-track { flex: 1; height: 6px; background: var(--bg-3); border-radius: 3px; overflow: hidden; }
.bar-fill { height: 100%; background: linear-gradient(90deg, var(--accent), var(--accent-2)); border-radius: 3px; }
.bar-pct { font-size: 12px; color: var(--text-3); width: 36px; }
.review-list { display: flex; flex-direction: column; gap: 16px; }
.review-card { padding: 24px; background: var(--bg-2); border: 1px solid var(--border); border-radius: var(--radius); }
.review-header { display: flex; align-items: center; gap: 12px; margin-bottom: 12px; }
.reviewer-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--accent), var(--accent-2));
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  flex-shrink: 0;
}
.reviewer-name { font-weight: 600; font-size: 14px; }
.review-date { color: var(--text-3); font-size: 12px; }
.review-stars { margin-left: auto; }
.review-comment { font-size: 14px; line-height: 1.7; }

/* Shipping */
.ship-info { display: flex; flex-direction: column; gap: 2px; }
.ship-row {
  display: flex;
  align-items: flex-start;
  gap: 16px;
  padding: 20px;
  background: var(--bg-2);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  margin-bottom: 12px;
}
.ship-icon { font-size: 24px; flex-shrink: 0; }
.ship-title { font-weight: 600; margin-bottom: 4px; }
.ship-desc { color: var(--text-3); font-size: 13px; }
.ship-price { margin-left: auto; font-weight: 700; font-size: 14px; color: var(--accent-2); white-space: nowrap; }

/* Related */
.related-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 24px;
}
.product-card { cursor: pointer; }
.product-img-wrap { aspect-ratio: 3/4; overflow: hidden; }
.product-img { width: 100%; height: 100%; object-fit: cover; transition: transform 0.4s; }
.product-card:hover .product-img { transform: scale(1.05); }
.product-info { padding: 14px; }
.product-name { font-size: 14px; font-weight: 600; margin-bottom: 6px; }
.price { font-weight: 700; color: var(--accent-2); }

.not-found { text-align: center; padding: 120px 24px; display: flex; flex-direction: column; align-items: center; gap: 16px; }
.not-found span { font-size: 56px; }
.not-found h2 { font-size: 24px; }

@media (max-width: 768px) {
  .detail-grid { grid-template-columns: 1fr; gap: 32px; }
  .action-row { flex-direction: column; }
  .flex-1 { width: 100%; }
  .related-grid { grid-template-columns: repeat(2, 1fr); }
  .review-summary { grid-template-columns: 1fr; }
}
</style>
