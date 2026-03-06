<template>
  <article class="task-card" :class="{ 'task-overdue': isOverdue, 'task-on-time': isOnTime }">
    <header class="task-header">
      <h3 class="task-title">{{ task.title }}</h3>
      <span v-if="task.status" class="task-status" :class="task.status">
        {{ task.status === 'on-time' ? '✓ В срок' : '⚠ Просрочено' }}
      </span>
    </header>

    <p v-if="task.description" class="task-description">{{ task.description }}</p>

    <footer class="task-footer">
      <div class="task-dates">
        <span class="task-created" :title="`Создано: ${formatDate(task.createdAt)}`">
          📅 {{ formatDate(task.createdAt) }}
        </span>
        <span v-if="task.updatedAt" class="task-edited" :title="`Изменено: ${formatDate(task.updatedAt)}`">
          ✏️ {{ formatDate(task.updatedAt) }}
        </span>
      </div>
      <div v-if="task.deadline" class="task-deadline" :class="{ 'deadline-warning': isDeadlineSoon }">
        ⏰ {{ formatDate(task.deadline) }}
      </div>
    </footer>

    <div v-if="task.returnReason" class="task-return-reason">
      <small>↩ Возврат: {{ task.returnReason }}</small>
    </div>

    <div class="task-actions" v-if="!isDoneColumn">
      <button class="btn-action btn-edit" @click="emit('edit', task)" title="Редактировать">
        ✏️
      </button>
      <button
        v-if="task.column === 'planned'"
        class="btn-action btn-delete"
        @click="emit('delete', task.id)"
        title="Удалить"
      >
        🗑️
      </button>

      <!-- Кнопки перемещения -->
      <button
        v-if="canMoveForward"
        class="btn-action btn-move"
        @click="emit('move', task.id, 'forward')"
        title="Переместить вперед"
      >
        ➡️
      </button>
      <button
        v-if="canMoveBack"
        class="btn-action btn-move"
        @click="emit('move', task.id, 'back')"
        title="Вернуть назад"
      >
        ⬅️
      </button>
    </div>
  </article>
</template>

<script setup>
import { computed } from 'vue'
import { COLUMN_ORDER } from '../constants/kanban.js'

const props = defineProps({
  task: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['edit', 'delete', 'move'])

const formatDate = (dateString) => {
  return new Date(dateString).toLocaleDateString('ru-RU', {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })
}

const isOverdue = computed(() => {
  if (props.task.column !== 'done') return false
  return props.task.status === 'overdue'
})

const isOnTime = computed(() => {
  if (props.task.column !== 'done') return false
  return props.task.status === 'on-time'
})

const isDeadlineSoon = computed(() => {
  if (!props.task.deadline || props.task.column === 'done') return false
  const deadline = new Date(props.task.deadline)
  const now = new Date()
  const diffDays = Math.ceil((deadline.getTime() - now.getTime()) / (1000 * 60 * 60 * 24))
  return diffDays <= 2 && diffDays >= 0
})

const isDoneColumn = computed(() => {
  return props.task.column === 'done'
})

const currentIndex = computed(() => {
  return COLUMN_ORDER.indexOf(props.task.column)
})

const canMoveForward = computed(() => {
  return currentIndex.value < COLUMN_ORDER.length - 1 && props.task.column !== 'testing'
})

const canMoveBack = computed(() => {
  return currentIndex.value > 0 && props.task.column === 'testing'
})
</script>

<style scoped>
.task-card {
  background: white;
  border-radius: 6px;
  padding: 0.75rem;
  margin-bottom: 0.5rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12);
  cursor: pointer;
  transition: box-shadow 0.2s, transform 0.1s;
}

.task-card:hover {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
  transform: translateY(-2px);
}

.task-overdue {
  border-left: 4px solid #ff5630;
  background: #ffebe6;
}

.task-on-time {
  border-left: 4px solid #36b37e;
  background: #e3fcef;
}

.task-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
}

.task-title {
  font-size: 0.95rem;
  font-weight: 500;
  color: #172b4d;
  margin: 0;
  line-height: 1.3;
}

.task-status {
  font-size: 0.7rem;
  padding: 0.125rem 0.375rem;
  border-radius: 4px;
  font-weight: 500;
  white-space: nowrap;
}

.task-status.on-time {
  background: #36b37e;
  color: white;
}

.task-status.overdue {
  background: #ff5630;
  color: white;
}

.task-description {
  font-size: 0.85rem;
  color: #5e6c84;
  margin: 0 0 0.75rem 0;
  line-height: 1.4;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.task-footer {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  font-size: 0.7rem;
  color: #6b778c;
}

.task-dates {
  display: flex;
  flex-direction: column;
  gap: 0.125rem;
}

.task-created,
.task-edited {
  display: flex;
  align-items: center;
  gap: 0.25rem;
}

.task-deadline {
  font-weight: 500;
}

.task-deadline.deadline-warning {
  color: #ffab00;
}

.task-return-reason {
  margin-top: 0.5rem;
  padding-top: 0.5rem;
  border-top: 1px solid #dfe1e6;
  color: #ff5630;
  font-size: 0.75rem;
}

.task-actions {
  display: flex;
  gap: 0.5rem;
  margin-top: 0.75rem;
  padding-top: 0.75rem;
  border-top: 1px solid #dfe1e6;
  justify-content: flex-end;
}

.btn-action {
  background: none;
  border: none;
  cursor: pointer;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  font-size: 1rem;
  transition: background 0.2s;
}

.btn-action:hover {
  background: #f4f5f7;
}

.btn-delete:hover {
  background: #ffebe6;
}

.btn-move {
  background: #e3fcef;
  color: #006644;
}

.btn-move:hover {
  background: #b3f5da;
}
</style>
