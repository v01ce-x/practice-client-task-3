<template>
  <div id="app">
    <header class="app-header">
      <h1>Kanban Board</h1>
    </header>
    <main class="board">
      <section
        v-for="column in columns"
        :key="column.id"
        class="column"
        :data-column="column.id"
      >
        <h2 class="column-title">
          {{ column.title }}
          <span class="column-count">({{ getTasksCount(column.id) }})</span>

          <button
            v-if="column.id === 'planned'"
            class="btn-add-task"
            @click="openCreateModal"
          >
            + Добавить
          </button>
        </h2>
        <div class="task-list" :data-column="column.id">
          <TaskCard
            v-for="task in tasksByColumn[column.id]"
            :key="task.id"
            :task="task"
          />
        </div>
      </section>
    </main>

    <CreateTaskModal
      v-model="showCreateModal"
      @submit="handleCreateTask"
    />
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { COLUMNS } from './constants/kanban'
import TaskCard from './components/TaskCard.vue'
import CreateTaskModal from './components/CreateTaskModal.vue'

const columns = ref(COLUMNS)
const tasks = ref([])
const showCreateModal = ref(false)

const tasksByColumn = computed(() => {
  const result = {
    'planned': [],
    'in-progress': [],
    'testing': [],
    'done': []
  }
  tasks.value.forEach(task => {
    result[task.column].push(task)
  })
  return result
})

const getTasksCount = (columnId) => {
  return tasksByColumn.value[columnId].length
}

const generateId = () => {
  return Date.now().toString(36) + Math.random().toString(36).substr(2)
}

const openCreateModal = () => {
  showCreateModal.value = true
}

const handleCreateTask = ({ title, description, deadline }) => {
  const newTask = {
    id: generateId(),
    title,
    description,
    deadline,
    createdAt: new Date().toISOString(),
    updatedAt: null,
    column: 'planned'
  }

  tasks.value.push(newTask)
  showCreateModal.value = false
}
</script>

<style>
.column-title {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 0.5rem;
}

.btn-add-task {
  background: #0079bf;
  color: white;
  border: none;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s;
}

.btn-add-task:hover {
  background: #026aa7;
}
</style>
