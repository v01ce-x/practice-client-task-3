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
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { COLUMNS } from './constants/kanban'
import TaskCard from './components/TaskCard.vue'

const columns = ref(COLUMNS)
const tasks = ref([])

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

tasks.value = [
  {
    id: '1',
    title: 'Разработать дизайн главной страницы',
    description: 'Создать макет в Figma с учетом новых требований',
    deadline: new Date(Date.now() + 3 * 24 * 60 * 60 * 1000).toISOString(),
    createdAt: new Date().toISOString(),
    updatedAt: null,
    column: 'planned'
  },
  {
    id: '2',
    title: 'Реализовать авторизацию',
    description: 'Добавить JWT токены и защиту роутов',
    deadline: new Date(Date.now() - 1 * 24 * 60 * 60 * 1000).toISOString(),
    createdAt: new Date(Date.now() - 2 * 24 * 60 * 60 * 1000).toISOString(),
    updatedAt: new Date().toISOString(),
    column: 'in-progress'
  }
]
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
  font-size: 0.9rem;
  font-weight: 600;
  color: #5e6c84;
  padding: 0.5rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.column-count {
  background: #dfe1e6;
  color: #5e6c84;
  padding: 0.125rem 0.5rem;
  border-radius: 4px;
  font-size: 0.75rem;
}

.task-list {
  flex: 1;
  min-height: 100px;
  padding: 0.25rem;
}
</style>
