<template>
  <Teleport to="body">
    <div v-if="modelValue" class="modal-overlay" @click.self="emit('update:modelValue', false)">
      <div class="modal">
        <header class="modal-header">
          <h3>Новая задача</h3>
          <button class="modal-close" @click="emit('update:modelValue', false)">&times;</button>
        </header>

        <form @submit.prevent="handleSubmit" class="modal-form">
          <div class="form-group">
            <label for="title">Заголовок *</label>
            <input
              id="title"
              v-model="form.title"
              type="text"
              required
              maxlength="100"
              placeholder="Введите название задачи"
            >
          </div>

          <div class="form-group">
            <label for="description">Описание</label>
            <textarea
              id="description"
              v-model="form.description"
              rows="4"
              maxlength="500"
              placeholder="Детали задачи..."
            ></textarea>
          </div>

          <div class="form-group">
            <label for="deadline">Дедлайн *</label>
            <input
              id="deadline"
              v-model="form.deadline"
              type="datetime-local"
              required
              :min="minDate"
            >
          </div>

          <div class="modal-actions">
            <button type="button" class="btn-secondary" @click="emit('update:modelValue', false)">
              Отмена
            </button>
            <button type="submit" class="btn-primary" :disabled="!isValid">
              Создать задачу
            </button>
          </div>
        </form>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  modelValue: {
    type: Boolean,
    required: true
  }
})

const emit = defineEmits(['update:modelValue', 'submit'])

const form = ref({
  title: '',
  description: '',
  deadline: ''
})

const minDate = computed(() => {
  const now = new Date()
  now.setMinutes(now.getMinutes() - now.getTimezoneOffset())
  return now.toISOString().slice(0, 16)
})

const isValid = computed(() => {
  return form.value.title.trim().length > 0 && form.value.deadline !== ''
})

const handleSubmit = () => {
  if (!isValid.value) return

  emit('submit', {
    title: form.value.title.trim(),
    description: form.value.description.trim(),
    deadline: new Date(form.value.deadline).toISOString()
  })

  form.value = { title: '', description: '', deadline: '' }
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
  max-height: 90vh;
  overflow-y: auto;
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
  line-height: 1;
}

.modal-close:hover {
  color: #172b4d;
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

.form-group input,
.form-group textarea {
  padding: 0.5rem 0.75rem;
  border: 2px solid #dfe1e6;
  border-radius: 4px;
  font-size: 0.95rem;
  transition: border-color 0.2s;
  font-family: inherit;
}

.form-group input:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #0079bf;
}

.form-group textarea {
  resize: vertical;
  min-height: 80px;
}

.modal-actions {
  display: flex;
  gap: 0.75rem;
  margin-top: 0.5rem;
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
  transition: background 0.2s;
}

.btn-primary {
  background: #0079bf;
  color: white;
}

.btn-primary:hover:not(:disabled) {
  background: #026aa7;
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
