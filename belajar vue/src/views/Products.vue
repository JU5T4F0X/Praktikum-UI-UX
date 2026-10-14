<script setup>
import { ref, computed } from 'vue'
import { RouterLink } from 'vue-router'
import { products, categories, useStore } from '../store/index.js'

const { addToCart, toggleWishlist, isWishlisted } = useStore()

const search = ref('')
const activeCategory = ref('Semua')
const sortBy = ref('popular')
const priceMax = ref(2000000)

const formatPrice = (p) => 'Rp ' + p.toLocaleString('id-ID')
const discount = (orig, price) => Math.round((1 - price / orig) * 100)
const stars = (r) => '★'.repeat(Math.floor(r)) + (r % 1 >= 0.5 ? '★' : '')

const filtered = computed(() => {
  let list = [...products]
  if (activeCategory.value !== 'Semua') list = list.filter(p => p.category === activeCategory.value)
  if (search.value) list = list.filter(p => p.name.toLowerCase().includes(search.value.toLowerCase()))
  list = list.filter(p => p.price <= priceMax.value)
  if (sortBy.value === 'popular') list.sort((a, b) => b.sold - a.sold)
  else if (sortBy.value === 'price-asc') list.sort((a, b) => a.price - b.price)
  else if (sortBy.value === 'price-desc') list.sort((a, b) => b.price - a.price)
  else if (sortBy.value === 'rating') list.sort((a, b) => b.rating - a.rating)
  return list
})

const handleAddToCart = (product) => {
  addToCart(product, 1, product.colors[0], product.sizes[0])
}
</script>

<template>
  <div class="products-page">
    <!-- Page Header -->
    <div class="page-hero">
      <div class="container">
        <p class="section-label">Katalog</p>
        <h1 class="page-title serif">Semua Produk</h1>
        <p class="page-sub">{{ filtered.length }} produk ditemukan</p>
      </div>
    </div>

    <div class="container products-layout">
      <!-- Sidebar Filters -->
      <aside class="sidebar">
        <div class="sidebar-block">
          <h3 class="sidebar-title">Kategori</h3>
          <div class="filter-list">
            <button
              v-for="cat in categories"
              :key="cat"
              class="filter-btn"
              :class="{ active: cat === activeCategory }"
              @click="activeCategory = cat"
            >
              <span>{{ cat }}</span>
              <span class="filter-count">
                {{ cat === 'Semua' ? products.length : products.filter(p => p.category === cat).length }}
              </span>
            </button>
          </div>
        </div>

        <div class="sidebar-block">
          <h3 class="sidebar-title">Harga Maks.</h3>
          <input type="range" v-model="priceMax" min="100000" max="2000000" step="50000" class="range-slider" />
          <div class="range-labels">
            <span>Rp 100K</span>
            <span class="range-val">{{ formatPrice(priceMax) }}</span>
          </div>
        </div>

        <div class="sidebar-block">
          <h3 class="sidebar-title">Urutkan</h3>
          <div class="filter-list">
            <button
              v-for="s in [
                { val: 'popular', label: 'Terpopuler' },
                { val: 'rating', label: 'Rating Tertinggi' },
                { val: 'price-asc', label: 'Harga Terendah' },
                { val: 'price-desc', label: 'Harga Tertinggi' },
              ]"
              :key="s.val"
              class="filter-btn"
              :class="{ active: sortBy === s.val }"
              @click="sortBy = s.val"
            >{{ s.label }}</button>
          </div>
        </div>
      </aside>

      <!-- Main Grid -->
      <div class="main-area">
        <!-- Search & Sort bar -->
        <div class="toolbar">
          <div class="search-wrap">
            <svg class="search-icon" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="8"/><path d="M21 21l-4.35-4.35"/></svg>
            <input v-model="search" class="input search-input" placeholder="Cari produk..." />
          </div>
          <p class="results-count">{{ filtered.length }} produk</p>
        </div>

        <!-- Empty state -->
        <div v-if="filtered.length === 0" class="empty-state">
          <span class="empty-icon">🔍</span>
          <h3>Produk tidak ditemukan</h3>
          <p>Coba ubah filter atau kata kunci pencarian</p>
          <button class="btn btn-outline" @click="search = ''; activeCategory = 'Semua'">Reset Filter</button>
        </div>

        <!-- Grid -->
        <div v-else class="grid">
          <RouterLink
            v-for="product in filtered"
            :key="product.id"
            :to="`/products/${product.id}`"
            class="product-card card"
          >
            <div class="product-img-wrap">
              <img :src="product.images[0]" :alt="product.name" class="product-img" loading="lazy" />
              <div class="product-overlay">
                <button
                  class="wishlist-btn"
                  :class="{ active: isWishlisted(product.id) }"
                  @click.prevent="toggleWishlist(product.id)"
                >{{ isWishlisted(product.id) ? '♥' : '♡' }}</button>
                <button class="quick-add btn btn-primary" @click.prevent="handleAddToCart(product)">
                  + Keranjang
                </button>
              </div>
              <span class="product-badge badge" :class="product.badge === 'Flash Sale' ? 'badge-red' : 'badge-purple'">
                {{ product.badge }}
              </span>
            </div>
            <div class="product-info">
              <p class="product-category">{{ product.category }}</p>
              <h3 class="product-name">{{ product.name }}</h3>
              <div class="product-rating">
                <span class="stars">{{ stars(product.rating) }}</span>
                <span class="rating-text">{{ product.rating }} ({{ product.reviews }})</span>
              </div>
              <div class="product-pricing">
                <span class="price">{{ formatPrice(product.price) }}</span>
                <span class="price-orig">{{ formatPrice(product.originalPrice) }}</span>
                <span class="discount-tag">-{{ discount(product.originalPrice, product.price) }}%</span>
              </div>
            </div>
          </RouterLink>
        </div>
      </div>
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
.page-title { font-size: 2.4rem; margin-bottom: 8px; }
.page-sub { color: var(--text-3); font-size: 14px; }

.products-layout {
  display: grid;
  grid-template-columns: 240px 1fr;
  gap: 40px;
  padding-top: 40px;
  padding-bottom: 80px;
  align-items: start;
}

/* Sidebar */
.sidebar { position: sticky; top: 80px; }
.sidebar-block {
  background: var(--bg-2);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 24px;
  margin-bottom: 16px;
}
.sidebar-title {
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 1.5px;
  text-transform: uppercase;
  color: var(--text-3);
  margin-bottom: 16px;
}
.filter-list { display: flex; flex-direction: column; gap: 4px; }
.filter-btn {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 9px 12px;
  border-radius: var(--radius-sm);
  font-size: 14px;
  color: var(--text-2);
  transition: all var(--transition-fast);
  text-align: left;
}
.filter-btn:hover { background: var(--surface); color: var(--text-1); }
.filter-btn.active { background: rgba(124,58,237,0.15); color: var(--accent-2); border: 1px solid rgba(124,58,237,0.2); }
.filter-count {
  font-size: 11px;
  background: var(--bg-3);
  color: var(--text-3);
  padding: 1px 7px;
  border-radius: 100px;
}

.range-slider {
  width: 100%;
  accent-color: var(--accent);
  margin-bottom: 8px;
}
.range-labels { display: flex; justify-content: space-between; font-size: 12px; color: var(--text-3); }
.range-val { color: var(--accent-2); font-weight: 600; }

/* Toolbar */
.toolbar {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 24px;
}
.search-wrap { position: relative; flex: 1; }
.search-icon {
  position: absolute;
  left: 14px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--text-3);
}
.search-input { padding-left: 40px; }
.results-count { color: var(--text-3); font-size: 14px; white-space: nowrap; }

/* Product Grid */
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}
.product-card { cursor: pointer; }
.product-img-wrap {
  position: relative;
  overflow: hidden;
  aspect-ratio: 3/4;
}
.product-img { width: 100%; height: 100%; object-fit: cover; transition: transform 0.5s ease; }
.product-card:hover .product-img { transform: scale(1.06); }
.product-overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(to top, rgba(0,0,0,0.7) 0%, transparent 60%);
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  padding: 16px;
  opacity: 0;
  transition: opacity var(--transition);
}
.product-card:hover .product-overlay { opacity: 1; }
.wishlist-btn {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: var(--surface);
  border: 1px solid var(--border);
  color: var(--text-2);
  font-size: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  backdrop-filter: blur(8px);
  transition: all var(--transition-fast);
  flex-shrink: 0;
}
.wishlist-btn:hover, .wishlist-btn.active { color: var(--red); border-color: var(--red); }
.quick-add { font-size: 12px; padding: 8px 14px; }
.product-badge { position: absolute; top: 12px; left: 12px; }
.product-info { padding: 16px; }
.product-category { font-size: 11px; color: var(--text-3); text-transform: uppercase; letter-spacing: 1px; margin-bottom: 6px; }
.product-name { font-size: 14px; font-weight: 600; margin-bottom: 8px; line-height: 1.4; }
.product-rating { display: flex; align-items: center; gap: 6px; margin-bottom: 10px; }
.stars { font-size: 11px; color: var(--gold); }
.rating-text { font-size: 11px; color: var(--text-3); }
.product-pricing { display: flex; align-items: center; gap: 6px; flex-wrap: wrap; }
.price { font-weight: 700; font-size: 14px; }
.price-orig { font-size: 12px; color: var(--text-3); text-decoration: line-through; }

/* Empty state */
.empty-state {
  text-align: center;
  padding: 80px 40px;
  color: var(--text-2);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}
.empty-icon { font-size: 48px; }
.empty-state h3 { font-size: 20px; font-weight: 600; color: var(--text-1); }
.empty-state p { color: var(--text-3); }

@media (max-width: 768px) {
  .products-layout { grid-template-columns: 1fr; }
  .sidebar { position: static; display: flex; gap: 12px; overflow-x: auto; padding-bottom: 8px; }
  .sidebar-block { min-width: 200px; }
  .grid { grid-template-columns: repeat(2, 1fr); gap: 14px; }
}
</style>
