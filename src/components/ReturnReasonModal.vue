<template>
  <Teleport to="body">
    <div v-if="modelValue" class="modal-overlay" @click.self="emit('update:modelValue', false)">
      <div class="modal">
        <header class="modal-header">
          <h3>Возврат задачи</h3>
          <button class="modal-close" @click="emit('update:modelValue', false)">&times;</button>
        </header>

        <form @submit.prevent="handleSubmit" class="modal-form">
          <div class="form-group">
            <label for="reason">Причина возврата *</label>
            <textarea
              id="reason"
              v-model="reason"
              rows="4"
              required
              maxlength="300"
              placeholder="Опишите, что нужно исправить..."
            ></textarea>
          </div>

          <div class="modal-actions">
            <button type="button" class="btn-secondary" @click="emit('update:modelValue', false)">
              Отмена
            </button>
            <button type="submit" class="btn-primary" :disabled="!reason.trim()">
              Вернуть задачу
            </button>
          </div>
        </form>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  modelValue: {
    type: Boolean,
    required: true
  }
})

const emit = defineEmits(['update:modelValue', 'submit'])

const reason = ref('')

const handleSubmit = () => {
  if (!reason.value.trim()) return

  emit('submit', reason.value.trim())
  reason.value = ''
  emit('update:modelValue', false)
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 1rem;
}

.modal {
  background: white;
  border-radius: 8px;
  width: 100%;
  max-width: 480px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 1.25rem;
  border-bottom: 1px solid #dfe1e6;
}

.modal-header h3 {
  font-size: 1.1rem;
  font-weight: 600;
  color: #172b4d;
  margin: 0;
}

.modal-close {
  background: none;
  border: none;
  font-size: 1.5rem;
  color: #6b778c;
  cursor: pointer;
  padding: 0.25rem;
}

.modal-form {
  padding: 1.25rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
}

.form-group label {
  font-size: 0.85rem;
  font-weight: 500;
  color: #5e6c84;
}

.form-group textarea {
  padding: 0.5rem 0.75rem;
  border: 2px solid #dfe1e6;
  border-radius: 4px;
  font-size: 0.95rem;
  font-family: inherit;
  resize: vertical;
  min-height: 100px;
}

.form-group textarea:focus {
  outline: none;
  border-color: #ff5630;
}

.modal-actions {
  display: flex;
  gap: 0.75rem;
}

.btn-primary,
.btn-secondary {
  flex: 1;
  padding: 0.625rem 1rem;
  border: none;
  border-radius: 4px;
  font-size: 0.9rem;
  font-weight: 500;
  cursor: pointer;
}

.btn-primary {
  background: #ff5630;
  color: white;
}

.btn-primary:hover:not(:disabled) {
  background: #de350b;
}

.btn-primary:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.btn-secondary {
  background: #f4f5f7;
  color: #5e6c84;
}

.btn-secondary:hover {
  background: #dfe1e6;
}
</style>
