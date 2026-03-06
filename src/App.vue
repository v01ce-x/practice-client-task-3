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
            @move="handleMoveTask"
            @complete="handleCompleteTask"
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

    <ReturnReasonModal
      v-model="showReturnModal"
      @submit="handleReturnTask"
    />
  </div>
</template>

<script setup>
import {ref, computed, watch, onMounted} from 'vue'
import {COLUMN_ORDER, COLUMNS} from './constants/kanban'
import ReturnReasonModal from './components/ReturnReasonModal.vue'
import TaskCard from './components/TaskCard.vue'
import CreateTaskModal from './components/CreateTaskModal.vue'
import EditTaskModal from './components/EditTaskModal.vue'

const STORAGE_KEY = 'kanban-tasks-vя1'
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

const showReturnModal = ref(false)
const taskToReturn = ref(null)

const checkDeadlineStatus = (task) => {
  if (!task.deadline) return 'on-time'

  const deadline = new Date(task.deadline)
  const now = new Date()

  if (deadline < now) {
    return 'overdue'
  }
  return 'on-time'
}

const handleMoveTask = (taskId, direction) => {
  const task = tasks.value.find(t => t.id === taskId)
  if (!task) return

  if (direction === 'forward') {
    if (task.column === 'planned') {
      task.column = 'in-progress'
    } else if (task.column === 'in-progress') {
      task.column = 'testing'
    } else if (task.column === 'testing') {
      task.column = 'done'
      task.status = checkDeadlineStatus(task)
    }
    task.updatedAt = new Date().toISOString()
  } else if (direction === 'back' && task.column === 'testing') {
    taskToReturn.value = task
    showReturnModal.value = true
  }
}

const handleReturnTask = (reason) => {
  if (taskToReturn.value) {
    taskToReturn.value.column = 'in-progress'
    taskToReturn.value.returnReason = reason
    taskToReturn.value.updatedAt = new Date().toISOString()
    taskToReturn.value = null
  }
}

const handleCompleteTask = (taskId) => {
  const task = tasks.value.find(t => t.id === taskId)
  if (!task) return

  if (confirm('Завершить задачу и переместить в "Выполнено"?')) {
    task.column = 'done'
    task.status = checkDeadlineStatus(task)
    task.updatedAt = new Date().toISOString()
  }
}

onMounted(() => {
  const saved = localStorage.getItem(STORAGE_KEY)
  if (saved) {
    try {
      tasks.value = JSON.parse(saved)
    } catch (e) {
      console.error('Ошибка загрузки данных:', e)
      tasks.value = []
    }
  }
})

watch(tasks, (newTasks) => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(newTasks))
}, { deep: true })
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

.btn-clear {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.3);
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.85rem;
  transition: background 0.2s;
}

.btn-clear:hover {
  background: rgba(255, 255, 255, 0.3);
}

.column-highlight {
  animation: highlight-pulse 1s ease;
}

@keyframes highlight-pulse {
  0%, 100% { background: #ebecf0; }
  50% { background: #d4e5f7; }
}
</style>
