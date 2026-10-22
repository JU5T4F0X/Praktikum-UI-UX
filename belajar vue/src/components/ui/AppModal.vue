<script setup>
defineProps({
  title: {
    type: String,
    default: 'Konfirmasi'
  },
  message: {
    type: String,
    default: 'Apakah kamu yakin?'
  },
  confirmLabel: {
    type: String,
    default: 'Ya, Lanjutkan'
  },
  confirmVariant: {
    type: String,
    default: 'danger' // 'danger' | 'primary'
  }
});

const emit = defineEmits(['confirm', 'close']);
</script>

<template>
  <Transition name="modal-fade">
    <div class="modal-overlay" @click.self="emit('close')">
      <div class="modal-box">
        <div class="modal-header">
          <h3>
            <span v-if="confirmVariant === 'danger'">⚠️</span>
            <span v-else>⚠️</span>
            {{ title }}
          </h3>
          <button class="modal-close" @click="emit('close')" aria-label="Close">✕</button>
        </div>
        <div class="modal-body">
          <p>{{ message }}</p>
        </div>
        <div class="modal-footer">
          <button class="btn btn-secondary" @click="emit('close')">Batal</button>
          <button
            :class="['btn', confirmVariant === 'danger' ? 'btn-danger' : 'btn-primary']"
            @click="emit('confirm')"
          >
            {{ confirmLabel }}
          </button>
        </div>
      </div>
    </div>
  </Transition>
</template>

<style scoped>
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
  padding: 1rem;
}

.modal-box {
  background: white;
  border-radius: 16px;
  width: 100%;
  max-width: 440px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.25);
  overflow: hidden;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem 1.5rem 0;
}

.modal-header h3 {
  font-size: 1.15rem;
  font-weight: 700;
  color: var(--text-main);
  margin: 0;
}

.modal-close {
  background: none;
  border: none;
  font-size: 1rem;
  color: var(--text-muted);
  cursor: pointer;
  padding: 4px 8px;
  border-radius: 4px;
  transition: background 0.2s;
}

.modal-close:hover {
  background: var(--bg-light);
}

.modal-body {
  padding: 1rem 1.5rem 1.5rem;
}

.modal-body p {
  color: var(--text-muted);
  font-size: 0.95rem;
  line-height: 1.6;
  margin: 0;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 0.75rem;
  padding: 1rem 1.5rem;
  border-top: 1px solid var(--border-color);
  background: var(--bg-light);
}

.btn {
  padding: 0.5rem 1.25rem;
  border-radius: 8px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  border: none;
  transition: opacity 0.2s;
}

.btn:hover { opacity: 0.85; }

.btn-secondary {
  background: white;
  color: var(--text-main);
  border: 1.5px solid var(--border-color);
}

.btn-danger {
  background: #e74c3c;
  color: white;
}

.btn-primary {
  background: var(--primary);
  color: white;
}

/* Fade transition */
.modal-fade-enter-active,
.modal-fade-leave-active {
  transition: opacity 0.2s ease;
}
.modal-fade-enter-from,
.modal-fade-leave-to {
  opacity: 0;
}
</style>