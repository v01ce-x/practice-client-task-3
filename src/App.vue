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
            @edit="openEditModal"
            @delete="handleDeleteTask"
          />
        </div>
      </section>
    </main>

    <CreateTaskModal
      v-model="showCreateModal"
      @submit="handleCreateTask"
    />

    <EditTaskModal
      v-model="showEditModal"
      :task="editingTask"
      @submit="handleEditTask"
    />
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { COLUMNS } from './constants/kanban'
import TaskCard from './components/TaskCard.vue'
import CreateTaskModal from './components/CreateTaskModal.vue'
import EditTaskModal from './components/EditTaskModal.vue'

const columns = ref(COLUMNS)
const tasks = ref([])
const showCreateModal = ref(false)
const showEditModal = ref(false)
const editingTask = ref(null)

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

const openEditModal = (task) => {
  editingTask.value = task
  showEditModal.value = true
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

const handleEditTask = ({ id, title, description, deadline }) => {
  const taskIndex = tasks.value.findIndex(t => t.id === id)
  if (taskIndex !== -1) {
    tasks.value[taskIndex] = {
      ...tasks.value[taskIndex],
      title,
      description,
      deadline,
      updatedAt: new Date().toISOString()
    }
  }
  showEditModal.value = false
}

const handleDeleteTask = (taskId) => {
  if (confirm('Вы уверены, что хотите удалить эту задачу?')) {
    tasks.value = tasks.value.filter(t => t.id !== taskId)
  }
}
</script>

<style>
.board {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1rem;
  padding: 1.5rem;
  min-height: calc(100vh - 70px);
  align-items: start;
}

.column {
  background: #ebecf0;
  border-radius: 8px;
  padding: 0.75rem;
  min-height: 500px;
  display: flex;
  flex-direction: column;
}

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
