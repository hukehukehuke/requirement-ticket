<script setup>
import TicketForm from './components/TicketForm.vue'
import { ref } from 'vue'

const tickets = ref([])

const handleSubmit = (ticket) => {
  tickets.value.unshift({
    ...ticket,
    status: 'pending',
    createdAt: new Date().toISOString()
  })
  console.log('工单列表:', tickets.value)
}
</script>

<template>
  <div class="app-container">
    <header class="app-header">
      <h1>🎯 AI需求助手</h1>
    </header>
    <main>
      <TicketForm @submit="handleSubmit" />
    </main>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
}

.app-container {
  min-height: 100vh;
  padding: 40px 20px;
}

.app-header {
  text-align: center;
  margin-bottom: 40px;
}

.app-header h1 {
  color: #fff;
  font-size: 32px;
  font-weight: 600;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

main {
  max-width: 800px;
  margin: 0 auto;
}
</style>
