<script setup>
import TicketForm from './components/TicketForm.vue'
import { ref } from 'vue'

const tickets = ref([])
const showForm = ref(false)

const handleSubmit = (ticket) => {
  tickets.value.unshift({
    ...ticket,
    status: 'pending',
    createdAt: new Date().toISOString()
  })
  console.log('工单列表:', tickets.value)
}

const handleCreateNew = () => {
  showForm.value = true
}
</script>

<template>
  <div class="app-container">
    <!-- Landing Page -->
    <div v-if="!showForm" class="landing-page">
      <div class="landing-content">
        <div class="landing-icon">🎯</div>
        <h1>AI需求助手</h1>
        <p>智能需求分析与整理，让您的需求表达更清晰</p>
        <button class="btn btn-primary btn-large" @click="handleCreateNew">
          创建需求工单
        </button>
      </div>
    </div>

    <!-- Form Page -->
    <template v-else>
      <header class="app-header">
        <h1>🎯 AI需求助手</h1>
      </header>
      <main>
        <TicketForm @submit="handleSubmit" />
      </main>
    </template>
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

/* Landing Page Styles */
.landing-page {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: calc(100vh - 80px);
}

.landing-content {
  text-align: center;
  background: rgba(255, 255, 255, 0.95);
  padding: 60px 80px;
  border-radius: 24px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.15);
  max-width: 500px;
}

.landing-icon {
  font-size: 72px;
  margin-bottom: 24px;
}

.landing-content h1 {
  color: #1a1a2e;
  font-size: 36px;
  font-weight: 700;
  margin-bottom: 16px;
}

.landing-content p {
  color: #666;
  font-size: 16px;
  line-height: 1.6;
  margin-bottom: 32px;
}

.btn-large {
  padding: 16px 48px;
  font-size: 18px;
  border-radius: 12px;
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
