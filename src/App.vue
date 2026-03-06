<template>
  <div id="app">
    <header class="app-header">
      <h1>Kanban Board</h1>
    </header>
    <main class="board"></main>
  </div>
</template>

<script setup>

</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

#app {
  font-family: 'Inter', system-ui, sans-serif;
  min-height: 100vh;
  background: #f4f5f7;
  color: #172b4d;
}

.app-header {
  background: #0079bf;
  color: white;
  padding: 1rem 2rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.app-header h1 {
  font-size: 1.5rem;
  font-weight: 600;
}

.board {
  padding: 1.5rem;
}
</style>
