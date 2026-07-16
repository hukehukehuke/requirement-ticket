<template>
  <div class="ticket-form">
    <div class="form-header">
      <h2>创建需求工单</h2>
      <p class="subtitle">填写以下信息来提交一个新的需求</p>
    </div>

    <form @submit.prevent="handleSubmit" class="form-content">
      <div class="form-group">
        <label for="title">工单标题 <span class="required">*</span></label>
        <input
          id="title"
          v-model="form.title"
          type="text"
          placeholder="请输入工单标题，简明扼要描述需求"
          :class="{ 'error': errors.title }"
        />
        <span v-if="errors.title" class="error-message">{{ errors.title }}</span>
      </div>

      <div class="form-row">
        <div class="form-group">
          <label for="category">需求分类 <span class="required">*</span></label>
          <select id="category" v-model="form.category" :class="{ 'error': errors.category }">
            <option value="">请选择分类</option>
            <option value="feature">功能需求</option>
            <option value="improvement">优化改进</option>
            <option value="bug">缺陷修复</option>
            <option value="other">其他</option>
          </select>
          <span v-if="errors.category" class="error-message">{{ errors.category }}</span>
        </div>

        <div class="form-group">
          <label for="priority">优先级 <span class="required">*</span></label>
          <select id="priority" v-model="form.priority" :class="{ 'error': errors.priority }">
            <option value="">请选择优先级</option>
            <option value="low">低</option>
            <option value="medium">中</option>
            <option value="high">高</option>
            <option value="urgent">紧急</option>
          </select>
          <span v-if="errors.priority" class="error-message">{{ errors.priority }}</span>
        </div>
      </div>

      <div class="form-group">
        <label for="description">详细描述 <span class="required">*</span></label>
        <textarea
          id="description"
          v-model="form.description"
          rows="6"
          placeholder="请详细描述需求背景、功能点、预期效果等..."
          :class="{ 'error': errors.description }"
        ></textarea>
        <span v-if="errors.description" class="error-message">{{ errors.description }}</span>
      </div>

      <div class="form-group">
        <label for="attachments">附件</label>
        <div class="file-upload" @click="triggerFileInput">
          <input
            ref="fileInput"
            type="file"
            multiple
            @change="handleFileChange"
            style="display: none"
          />
          <div class="upload-icon">📎</div>
          <p>点击上传文件或拖拽到此处</p>
          <span class="upload-hint">支持图片、文档、压缩包等文件</span>
        </div>
        <div v-if="form.attachments.length > 0" class="file-list">
          <div v-for="(file, index) in form.attachments" :key="index" class="file-item">
            <span class="file-name">{{ file.name }}</span>
            <span class="file-size">({{ formatFileSize(file.size) }})</span>
            <button type="button" class="remove-file" @click="removeFile(index)">×</button>
          </div>
        </div>
      </div>

      <div class="form-group">
        <label for="contact">联系方式</label>
        <input
          id="contact"
          v-model="form.contact"
          type="text"
          placeholder="请输入邮箱或手机号（选填）"
        />
      </div>

      <div class="form-actions">
        <button type="button" class="btn btn-ai" @click="openAiDialog">
          <span class="ai-button-icon" aria-hidden="true">✦</span>
          AI对话
        </button>
        <button type="button" class="btn btn-secondary" @click="resetForm">重置</button>
        <button type="submit" class="btn btn-primary" :disabled="isSubmitting">
          {{ isSubmitting ? '提交中...' : '提交工单' }}
        </button>
      </div>
    </form>

    <div
      v-if="isAiDialogOpen"
      class="ai-dialog-backdrop"
      role="dialog"
      aria-modal="true"
      aria-labelledby="ai-dialog-title"
    >
      <div class="ai-dialog">
        <div class="ai-dialog-header">
          <h3 id="ai-dialog-title">AI需求助手</h3>
          <button
            type="button"
            class="ai-dialog-close"
            :disabled="!isAiReady"
            @click="endAiDialog"
          >
            结束对话
          </button>
        </div>
        <div class="ai-dialog-body">
          <div v-if="!isAiReady" class="ai-dialog-loading">正在加载AI对话...</div>
          <iframe
            v-if="isAiDialogOpen"
            :key="iframeKey"
            ref="aiIframe"
            class="ai-dialog-iframe"
            :src="aiChatSrc"
            title="AI需求对话"
            allow="clipboard-read; clipboard-write; fullscreen"
          ></iframe>
        </div>
      </div>
    </div>

    <div v-if="submitSuccess" class="success-modal">
      <div class="success-content">
        <div class="success-icon">✓</div>
        <h3>提交成功！</h3>
        <p>您的需求工单已提交，工单号：<strong>{{ ticketId }}</strong></p>
        <button class="btn btn-primary" @click="closeSuccess">确定</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { nextTick, onBeforeUnmount, onMounted, ref, reactive, computed } from 'vue'

const emit = defineEmits(['submit'])

const form = reactive({
  title: '',
  category: '',
  priority: '',
  description: '',
  attachments: [],
  contact: ''
})

const errors = reactive({
  title: '',
  category: '',
  priority: '',
  description: ''
})

const fileInput = ref(null)
const isSubmitting = ref(false)
const submitSuccess = ref(false)
const ticketId = ref('')
const aiIframe = ref(null)
const isAiDialogOpen = ref(false)
const isAiReady = ref(false)
const iframeKey = ref(0)
const AI_CHAT_URL = import.meta.env.VITE_AI_CHAT_URL || 'http://localhost:3000'

// Generate a unique URL for each conversation to ensure fresh iframe
const aiChatSrc = computed(() => {
  const baseUrl = AI_CHAT_URL.endsWith('/') ? AI_CHAT_URL.slice(0, -1) : AI_CHAT_URL
  return `${baseUrl}?t=${Date.now()}&r=${Math.random().toString(36).substring(2, 8)}`
})

const isExpectedAiOrigin = (origin) => {
  try {
    return origin === new URL(AI_CHAT_URL, window.location.href).origin
  } catch {
    return false
  }
}

const handleAiMessage = async (event) => {
  if (event.source !== aiIframe.value?.contentWindow || !isExpectedAiOrigin(event.origin)) return
  if (!event.data || typeof event.data !== 'object') return

  if (event.data.type === 'ready') {
    isAiReady.value = true
    // Send init command to start the conversation
    aiIframe.value?.contentWindow?.postMessage({ type: 'init', payload: {} }, event.origin)
    return
  }

  // Handle conversationEnded event from iframe
  if (event.data.type === 'conversationEnded') {
    const transcript = event.data.payload?.content?.trim()
    if (transcript) {
      form.description = form.description.trim()
        ? `${form.description.trim()}\n\n${transcript}`
        : transcript
      errors.description = ''
    }
    closeAiDialog()
    await nextTick()
    document.querySelector('#description')?.focus()
    return
  }
}

const openAiDialog = () => {
  // Increment key to force iframe recreation for fresh conversation
  iframeKey.value++
  isAiReady.value = false
  isAiDialogOpen.value = true
}

const closeAiDialog = () => {
  isAiDialogOpen.value = false
  isAiReady.value = false
}

const endAiDialog = () => {
  if (!isAiReady.value || !aiIframe.value?.contentWindow) return
  // Send endConversation to get the transcript from iframe
  aiIframe.value.contentWindow.postMessage({ type: 'endConversation' }, new URL(AI_CHAT_URL, window.location.href).origin)
}

onMounted(() => window.addEventListener('message', handleAiMessage))
onBeforeUnmount(() => window.removeEventListener('message', handleAiMessage))

const triggerFileInput = () => {
  fileInput.value?.click()
}

const handleFileChange = (event) => {
  const files = Array.from(event.target.files)
  form.attachments.push(...files)
}

const removeFile = (index) => {
  form.attachments.splice(index, 1)
}

const formatFileSize = (bytes) => {
  if (bytes < 1024) return bytes + ' B'
  if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB'
  return (bytes / (1024 * 1024)).toFixed(1) + ' MB'
}

const validate = () => {
  errors.title = form.title.trim() ? '' : '请输入工单标题'
  errors.category = form.category ? '' : '请选择需求分类'
  errors.priority = form.priority ? '' : '请选择优先级'
  errors.description = form.description.trim() ? '' : '请输入详细描述'

  return form.title.trim() && form.category && form.priority && form.description.trim()
}

const handleSubmit = async () => {
  if (!validate()) return

  isSubmitting.value = true

  await new Promise(resolve => setTimeout(resolve, 1000))

  ticketId.value = 'TKT-' + Date.now().toString().slice(-8)

  emit('submit', { ...form, ticketId: ticketId.value })

  submitSuccess.value = true
  isSubmitting.value = false
}

const resetForm = () => {
  form.title = ''
  form.category = ''
  form.priority = ''
  form.description = ''
  form.attachments = []
  form.contact = ''
  errors.title = ''
  errors.category = ''
  errors.priority = ''
  errors.description = ''
}

const closeSuccess = () => {
  submitSuccess.value = false
  resetForm()
}
</script>

<style scoped>
.ticket-form {
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
  padding: 32px;
  max-width: 800px;
  margin: 0 auto;
}

.form-header {
  text-align: center;
  margin-bottom: 32px;
  padding-bottom: 24px;
  border-bottom: 1px solid #eee;
}

.form-header h2 {
  margin: 0 0 8px;
  font-size: 24px;
  color: #1a1a2e;
}

.subtitle {
  color: #666;
  margin: 0;
  font-size: 14px;
}

.form-content {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.form-group label {
  font-weight: 500;
  color: #333;
  font-size: 14px;
}

.required {
  color: #e74c3c;
}

.form-group input,
.form-group select,
.form-group textarea {
  padding: 12px 16px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 14px;
  transition: all 0.3s;
  font-family: inherit;
}

.form-group input:focus,
.form-group select:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #4f46e5;
  box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1);
}

.form-group input.error,
.form-group select.error,
.form-group textarea.error {
  border-color: #e74c3c;
}

.error-message {
  color: #e74c3c;
  font-size: 12px;
}

.form-group textarea {
  resize: vertical;
  min-height: 120px;
}

.file-upload {
  border: 2px dashed #ddd;
  border-radius: 8px;
  padding: 32px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s;
}

.file-upload:hover {
  border-color: #4f46e5;
  background: rgba(79, 70, 229, 0.02);
}

.upload-icon {
  font-size: 32px;
  margin-bottom: 8px;
}

.upload-icon ~ p {
  margin: 0;
  color: #666;
}

.upload-hint {
  font-size: 12px;
  color: #999;
}

.file-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-top: 12px;
}

.file-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  background: #f5f5f5;
  border-radius: 6px;
  font-size: 13px;
}

.file-name {
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.file-size {
  color: #999;
}

.remove-file {
  background: none;
  border: none;
  color: #999;
  cursor: pointer;
  font-size: 18px;
  padding: 0 4px;
}

.remove-file:hover {
  color: #e74c3c;
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 12px;
  padding-top: 24px;
  border-top: 1px solid #eee;
}

.btn {
  padding: 12px 32px;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s;
  border: none;
}

.btn-primary {
  background: linear-gradient(135deg, #4f46e5 0%, #7c3aed 100%);
  color: #fff;
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(79, 70, 229, 0.4);
}

.btn-primary:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

.btn-secondary {
  background: #f5f5f5;
  color: #666;
}

.btn-secondary:hover {
  background: #eee;
}

.btn-ai {
  display: inline-flex;
  align-items: center;
  gap: 7px;
  background: linear-gradient(135deg, #2563eb 0%, #4f46e5 100%);
  color: #fff;
  box-shadow: 0 4px 12px rgba(79, 70, 229, 0.24);
}

.btn-ai:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(79, 70, 229, 0.34);
}

.ai-button-icon {
  font-size: 16px;
  line-height: 1;
}

.ai-dialog-backdrop {
  position: fixed;
  inset: 0;
  z-index: 1200;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
  background: rgba(15, 23, 42, 0.58);
  backdrop-filter: blur(3px);
}

.ai-dialog {
  display: flex;
  flex-direction: column;
  width: min(1100px, 94vw);
  height: min(820px, 90vh);
  overflow: hidden;
  background: #fff;
  border-radius: 16px;
  box-shadow: 0 24px 70px rgba(15, 23, 42, 0.35);
}

.ai-dialog-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 24px;
  padding: 14px 18px 14px 22px;
  border-bottom: 1px solid #e5e7eb;
}

.ai-dialog-header h3 {
  margin: 0;
  color: #111827;
  font-size: 17px;
}

.ai-dialog-header p {
  margin: 3px 0 0;
  color: #6b7280;
  font-size: 12px;
}

.ai-dialog-close {
  min-width: 88px;
  height: 36px;
  padding: 0 14px;
  border: 0;
  border-radius: 9px;
  background: #4f46e5;
  color: #fff;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
}

.ai-dialog-close:hover {
  background: #4338ca;
}

.ai-dialog-close:disabled {
  opacity: 0.55;
  cursor: wait;
}

.ai-dialog-body {
  position: relative;
  flex: 1;
  min-height: 0;
  background: #e2e8f0;
}

.ai-dialog-loading {
  position: absolute;
  inset: 0;
  z-index: 1;
  display: grid;
  place-items: center;
  color: #64748b;
  font-size: 14px;
  background: #f8fafc;
}

.ai-dialog-iframe {
  display: block;
  width: 100%;
  height: 100%;
  border: 0;
  background: #fff;
}

.success-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.success-content {
  background: #fff;
  border-radius: 16px;
  padding: 40px;
  text-align: center;
  max-width: 400px;
}

.success-icon {
  width: 64px;
  height: 64px;
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
  color: #fff;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 32px;
  margin: 0 auto 20px;
}

.success-content h3 {
  margin: 0 0 12px;
  font-size: 20px;
  color: #1a1a2e;
}

.success-content p {
  margin: 0 0 24px;
  color: #666;
}

@media (max-width: 640px) {
  .form-row {
    grid-template-columns: 1fr;
  }

  .ticket-form {
    padding: 20px;
    margin: 16px;
  }

  .form-actions {
    flex-wrap: wrap;
  }

  .form-actions .btn {
    flex: 1 1 130px;
    padding-inline: 18px;
  }

  .ai-dialog-backdrop {
    padding: 0;
  }

  .ai-dialog {
    width: 100vw;
    height: 100vh;
    border-radius: 0;
  }
}
</style>
