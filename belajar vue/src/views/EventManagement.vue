<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { useEvents } from '@/composables/useEvents';
import AppModal from '@/components/ui/AppModal.vue';

const router = useRouter();
const { activeEvents, completedEvents, createEvent, completeEvent, deleteEvent } = useEvents();

// ─── TAB STATE ───────────────────────────────────────────────
const currentTab = ref('active'); // 'active' | 'create' | 'completed'

// ─── FORM STATE ───────────────────────────────────────────────
const categories = ['Workshop', 'Meetup', 'Competition', 'Seminar', 'Conference'];

const form = ref({
  title: '',
  date: '',
  loc: '',
  cat: 'Workshop',
  quota: 100,
  price: 'Free',
  desc: '',
});

const formError = ref('');
const formSuccess = ref(false);

function resetForm() {
  form.value = { title: '', date: '', loc: '', cat: 'Workshop', quota: 100, price: 'Free', desc: '' };
  formError.value = '';
  formSuccess.value = false;
}

function handleSubmit() {
  if (!form.value.title || !form.value.date || !form.value.loc || !form.value.desc) {
    formError.value = 'Please fill in all required fields before publishing the event!';
    formSuccess.value = false;
    return;
  }
  createEvent({ ...form.value });
  formSuccess.value = true;
  formError.value = '';
  // Reset form dan pindah ke Active Events setelah delay singkat
  setTimeout(() => {
    resetForm();
    currentTab.value = 'active';
  }, 1800);
}

function handleCancel() {
  resetForm();
}

// ─── MODAL / DESTRUCTIVE ACTION ───────────────────────────────
const modalAction = ref(null); // null | { type: 'complete' | 'delete', id: Number }

const modalConfig = {
  complete: {
    title: 'Complete This Event?',
    message: 'This event will be moved to Completed Events and removed from the public catalog. This action cannot be undone.',
    confirmLabel: 'Yes, Complete It',
    confirmVariant: 'primary',
  },
  delete: {
    title: 'Delete This Event?',
    message: 'This event will be permanently deleted from the system. This action cannot be undone.',
    confirmLabel: 'Yes, Delete It',
    confirmVariant: 'danger',
  },
};

function openModal(type, id) {
  modalAction.value = { type, id };
}

function closeModal() {
  modalAction.value = null;
}

function handleConfirm() {
  if (!modalAction.value) return;
  const { type, id } = modalAction.value;
  if (type === 'complete') completeEvent(id);
  else if (type === 'delete') deleteEvent(id);
  closeModal();
}

function getModalConfig() {
  if (!modalAction.value) return modalConfig.delete;
  return modalConfig[modalAction.value.type];
}

// Format datetime-local value for display (e.g. "Oct 15, 2026 • 10:00 AM")
function formatEventDate(raw) {
  if (!raw) return '-';
  const d = new Date(raw);
  if (isNaN(d.getTime())) return raw;
  const monthNames = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
  const month = monthNames[d.getMonth()];
  const day = d.getDate();
  const year = d.getFullYear();
  let hours = d.getHours();
  const minutes = d.getMinutes().toString().padStart(2, '0');
  const ampm = hours >= 12 ? 'PM' : 'AM';
  hours = hours % 12;
  hours = hours ? hours : 12;
  const formattedHours = hours.toString().padStart(2, '0');
  return `${month} ${day}, ${year} • ${formattedHours}:${minutes} ${ampm}`;
}

function displayLocation(loc) {
  if (!loc) return '';
  const trimmed = loc.trim();
  return trimmed.startsWith('📍') ? trimmed : `📍 ${trimmed}`;
}
</script>

<template>
  <div class="event-management">

    <!-- Header -->
    <div class="page-header">
      <div>
        <h2>Event Management</h2>
        <p class="subtitle">Create, manage, and archive all events on the Gatherly platform.</p>
      </div>
      <button class="btn-back" @click="router.push('/dashboard')">
        ← Back to Dashboard
      </button>
    </div>

    <!-- Tab Navigation -->
    <div class="tab-nav">
      <button
        :class="['tab-btn', currentTab === 'active' ? 'tab-active' : '']"
        @click="currentTab = 'active'"
      >
        ⚡ Active Events ({{ activeEvents.length }})
      </button>
      <button
        :class="['tab-btn', 'tab-btn-create', currentTab === 'create' ? 'tab-active' : '']"
        @click="currentTab = 'create'"
      >
        + Create Event
      </button>
      <button
        :class="['tab-btn', currentTab === 'completed' ? 'tab-active' : '']"
        @click="currentTab = 'completed'"
      >
        ✅ Completed Events ({{ completedEvents.length }})
      </button>
    </div>

    <!-- ─── TAB 1: ACTIVE EVENTS ─────────────────────────────── -->
    <div v-if="currentTab === 'active'" class="tab-content">
      <!-- Empty state -->
      <div v-if="activeEvents.length === 0" class="empty-state">
        <span class="empty-icon">📋</span>
        <h3>No Active Events</h3>
        <p>Create your first event to start publishing to the public catalog.</p>
        <button class="btn btn-primary" @click="currentTab = 'create'">+ Create Event</button>
      </div>

      <!-- Event cards grid -->
      <div v-else class="events-grid">
        <div v-for="item in activeEvents" :key="item.id" class="event-card">
          <div class="card-header">
            <span class="category-badge">{{ item.cat }}</span>
            <span class="status-badge status-active">Active</span>
          </div>
          <h3 class="card-title">{{ item.title }}</h3>
          <div class="card-meta">
            <span>🗓️ {{ formatEventDate(item.date) }}</span>
            <span>{{ displayLocation(item.loc) }}</span>
            <span>👥 {{ item.quota }} Pax &nbsp;·&nbsp; 🎟️ {{ item.price || 'Free' }}</span>
          </div>
          <p v-if="item.desc" class="card-desc">{{ item.desc }}</p>
          <div class="card-actions">
            <button class="btn btn-complete" @click="openModal('complete', item.id)">
              ✓ Complete Event
            </button>
            <button class="btn btn-delete" @click="openModal('delete', item.id)">
              🗑 Delete
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- ─── TAB 2: CREATE EVENT ───────────────────────────────── -->
    <div v-else-if="currentTab === 'create'" class="tab-content">
      <div class="form-container">
        <div class="form-heading-block">
          <h3 class="form-heading">Create New Event Form</h3>
          <p class="form-subheading">Information entered here will be instantly published to the Gatherly public catalog.</p>
        </div>

        <!-- Error feedback -->
        <div v-if="formError" class="alert alert-error">
          ⚠️ {{ formError }}
        </div>

        <!-- Success feedback -->
        <div v-if="formSuccess" class="alert alert-success">
          🎉 Event published successfully and is now live!
        </div>

        <form @submit.prevent="handleSubmit" class="event-form">
          <!-- Event Title -->
          <div class="form-group">
            <label for="ev-title">Event Title *</label>
            <input
              id="ev-title"
              v-model="form.title"
              type="text"
              placeholder="e.g. Modern UI/UX Masterclass"
              :class="{ 'input-error': formError && !form.title }"
            />
          </div>

          <!-- Category + Date & Time -->
          <div class="form-row">
            <div class="form-group">
              <label for="ev-cat">Event Category *</label>
              <select id="ev-cat" v-model="form.cat">
                <option v-for="cat in categories" :key="cat" :value="cat">{{ cat }}</option>
              </select>
            </div>
            <div class="form-group">
              <label for="ev-date">Date &amp; Time *</label>
              <input
                id="ev-date"
                v-model="form.date"
                type="datetime-local"
                :class="{ 'input-error': formError && !form.date }"
              />
            </div>
          </div>

          <!-- Location + Quota -->
          <div class="form-row">
            <div class="form-group">
              <label for="ev-loc">Event Location *</label>
              <input
                id="ev-loc"
                v-model="form.loc"
                type="text"
                placeholder="e.g. Innovation Center, Jakarta"
                :class="{ 'input-error': formError && !form.loc }"
              />
            </div>
            <div class="form-group">
              <label for="ev-quota">Attendee Quota</label>
              <input id="ev-quota" v-model.number="form.quota" type="number" min="1" placeholder="100" />
            </div>
          </div>

          <!-- Description -->
          <div class="form-group">
            <label for="ev-desc">Event Description *</label>
            <textarea
              id="ev-desc"
              v-model="form.desc"
              rows="5"
              placeholder="Describe the agenda, speakers, and key takeaways..."
              :class="{ 'input-error': formError && !form.desc }"
            ></textarea>
          </div>

          <div class="form-actions">
            <button type="button" class="btn btn-cancel" @click="handleCancel">Cancel</button>
            <button type="submit" class="btn btn-publish">🚀 Publish Event Now</button>
          </div>
        </form>
      </div>
    </div>

    <!-- ─── TAB 3: COMPLETED EVENTS ──────────────────────────── -->
    <div v-else-if="currentTab === 'completed'" class="tab-content">
      <!-- Empty state -->
      <div v-if="completedEvents.length === 0" class="empty-state">
        <span class="empty-icon">🏁</span>
        <h3>No Completed Events</h3>
        <p>Events that are marked as completed will be archived here.</p>
      </div>

      <!-- Completed cards -->
      <div v-else class="events-grid">
        <div v-for="item in completedEvents" :key="item.id" class="event-card event-card-completed">
          <div class="card-header">
            <span class="category-badge">{{ item.cat }}</span>
            <span class="status-badge status-completed">Completed</span>
          </div>
          <h3 class="card-title card-title-crossed">{{ item.title }}</h3>
          <div class="card-meta-completed">
            <span>🗓️ {{ formatEventDate(item.date) }} &bull; {{ displayLocation(item.loc) }}</span>
          </div>
          <p v-if="item.desc" class="card-desc">{{ item.desc }}</p>
          <div class="card-archived-notice">
            <span class="archived-lock-icon">🔒</span>
            <span>This event has ended and is archived from the public catalog.</span>
          </div>
        </div>
      </div>
    </div>

    <!-- ─── MODAL KONFIRMASI ──────────────────────────────────── -->
    <AppModal
      v-if="modalAction"
      :title="getModalConfig().title"
      :message="getModalConfig().message"
      :confirm-label="getModalConfig().confirmLabel"
      :confirm-variant="getModalConfig().confirmVariant"
      @confirm="handleConfirm"
      @close="closeModal"
    />

  </div>
</template>

<style scoped>
.event-management {
  max-width: 1100px;
  margin: 0 auto;
}

/* ── Header ── */
.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: var(--space-8);
  flex-wrap: wrap;
  gap: var(--space-4);
}

.page-header h2 {
  font-size: 2rem;
  color: var(--text-main);
  margin-bottom: var(--space-1);
}

.subtitle {
  color: var(--text-muted);
  font-size: 1rem;
}

.btn-back {
  background: none;
  border: 1.5px solid var(--border-color);
  color: var(--text-muted);
  padding: var(--space-2) var(--space-4);
  border-radius: 8px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-back:hover {
  border-color: var(--primary);
  color: var(--primary);
}

/* ── Tabs ── */
.tab-nav {
  display: flex;
  gap: var(--space-2);
  margin-bottom: var(--space-8);
  flex-wrap: wrap;
}

.tab-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  background: white;
  border: 1.5px solid var(--border-color);
  border-radius: 50px;
  font-size: 0.9rem;
  font-weight: 600;
  color: var(--text-muted);
  cursor: pointer;
  transition: all 0.2s;
}

.tab-btn:hover {
  border-color: var(--primary);
  color: var(--primary);
}

.tab-btn-create:not(.tab-active) {
  color: var(--primary);
  border-color: rgba(102, 68, 255, 0.3);
}

.tab-active {
  background: var(--primary) !important;
  border-color: var(--primary) !important;
  color: white !important;
}

.tab-count {
  background: rgba(102, 68, 255, 0.12);
  color: var(--primary);
  font-size: 0.75rem;
  font-weight: 700;
  padding: 2px 8px;
  border-radius: 50px;
  min-width: 22px;
  text-align: center;
}

.tab-count-active {
  background: rgba(255, 255, 255, 0.25);
  color: white;
}

/* ── Tab Content ── */
.tab-content {
  min-height: 300px;
}

/* ── Empty State ── */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: var(--space-12) var(--space-4);
  text-align: center;
  gap: var(--space-3);
  background: var(--bg-light);
  border-radius: 16px;
  border: 2px dashed var(--border-color);
}

.empty-icon { font-size: 3rem; }

.empty-state h3 {
  font-size: 1.2rem;
  color: var(--text-main);
}

.empty-state p {
  color: var(--text-muted);
  font-size: 0.95rem;
}

/* ── Event Cards Grid ── */
.events-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: var(--space-6);
}

.event-card {
  background: white;
  border-radius: 16px;
  border: 1px solid var(--border-color);
  padding: var(--space-5);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.03);
  display: flex;
  flex-direction: column;
  gap: var(--space-3);
  transition: box-shadow 0.2s;
}

.event-card-completed {
  border-color: #e2e8f0;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.category-badge {
  background: rgba(102, 68, 255, 0.1);
  color: var(--primary);
  padding: 4px 10px;
  border-radius: 50px;
  font-size: 0.75rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.4px;
}

.status-badge {
  padding: 4px 10px;
  border-radius: 50px;
  font-size: 0.75rem;
  font-weight: 700;
}

.status-active {
  background: rgba(46, 204, 113, 0.15);
  color: #27ae60;
}

.status-completed {
  background: rgba(108, 117, 125, 0.15);
  color: #6c757d;
}

.card-title {
  font-size: 1.05rem;
  font-weight: 700;
  color: var(--text-main);
  margin: 0;
}

.card-meta {
  display: flex;
  flex-direction: column;
  gap: 4px;
  font-size: 0.85rem;
  color: var(--text-muted);
}

.card-desc {
  font-size: 0.9rem;
  color: var(--text-muted);
  line-height: 1.5;
  margin: 0;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.card-title-crossed {
  text-decoration: line-through;
  color: #8c98a4;
}

.card-meta-completed {
  font-size: 0.85rem;
  color: var(--text-muted);
}

.card-archived-notice {
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 10px 14px;
  font-size: 0.85rem;
  color: #64748b;
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: auto;
  line-height: 1.4;
}

.archived-lock-icon {
  font-size: 0.95rem;
  flex-shrink: 0;
}

.card-actions {
  display: flex;
  gap: var(--space-2);
  margin-top: auto;
  padding-top: var(--space-3);
  border-top: 1px solid var(--border-color);
  flex-wrap: wrap;
}

/* ── Buttons ── */
.btn {
  padding: 9px 18px;
  border-radius: 8px;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  border: none;
  transition: opacity 0.2s, transform 0.1s;
}

.btn:hover { opacity: 0.85; }
.btn:active { transform: scale(0.97); }

.btn-primary {
  background: var(--primary);
  color: white;
}

.btn-complete {
  background: rgba(46, 204, 113, 0.12);
  color: #27ae60;
  flex: 1;
}

.btn-delete {
  background: rgba(231, 76, 60, 0.1);
  color: #e74c3c;
}

.btn-cancel {
  background: white;
  color: var(--text-muted);
  border: 1.5px solid var(--border-color);
}

.btn-publish {
  background: var(--primary);
  color: white;
  padding: 10px 24px;
  font-size: 0.95rem;
}

/* ── Create Form ── */
.form-container {
  background: white;
  border-radius: 16px;
  border: 1px solid var(--border-color);
  padding: var(--space-8);
  max-width: 680px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.03);
}

.form-heading-block {
  margin-bottom: var(--space-6);
  padding-bottom: var(--space-5);
  border-bottom: 1px solid var(--border-color);
}

.form-heading {
  font-size: 1.4rem;
  color: var(--text-main);
  margin: 0 0 var(--space-1) 0;
}

.form-subheading {
  font-size: 0.9rem;
  color: var(--text-muted);
  margin: 0;
}

.alert {
  padding: 12px 16px;
  border-radius: 8px;
  font-size: 0.9rem;
  font-weight: 500;
  margin-bottom: var(--space-5);
}

.alert-success {
  background: #f0faf4;
  color: #27ae60;
  border: 1px solid #a8e6c1;
}

.alert-error {
  background: #fff5f5;
  color: #c0392b;
  border: 1px solid #f5b7b1;
}

.event-form {
  display: flex;
  flex-direction: column;
  gap: var(--space-5);
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: var(--space-4);
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-group label {
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--text-main);
}

.form-group input,
.form-group select,
.form-group textarea {
  padding: 10px 14px;
  border: 1.5px solid var(--border-color);
  border-radius: 8px;
  font-size: 0.95rem;
  color: var(--text-main);
  background: white;
  transition: border-color 0.2s, box-shadow 0.2s;
  font-family: inherit;
}

.form-group input:focus,
.form-group select:focus,
.form-group textarea:focus {
  outline: none;
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(102, 68, 255, 0.1);
}

.input-error {
  border-color: #e74c3c !important;
}

.input-error:focus {
  box-shadow: 0 0 0 3px rgba(231, 76, 60, 0.1) !important;
}

.form-group textarea {
  resize: vertical;
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  gap: var(--space-3);
  padding-top: var(--space-4);
  border-top: 1px solid var(--border-color);
}

/* ── Responsive ── */
@media (max-width: 640px) {
  .form-row { grid-template-columns: 1fr; }
  .events-grid { grid-template-columns: 1fr; }
  .page-header { flex-direction: column; align-items: flex-start; }
  .tab-nav { gap: var(--space-2); }
  .tab-btn { padding: 8px 14px; font-size: 0.85rem; }
}
</style>
