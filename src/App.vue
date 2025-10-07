<script setup lang="ts">
import { ref } from 'vue'
import axios from 'axios'

interface Message {
  text: string
  isUser: boolean
}

const messages = ref<Message[]>([])
const userInput = ref('')
const isLoading = ref(false)

const sendMessage = async () => {
  if (!userInput.value.trim()) return

  // Add user message to chat
  messages.value.push({ text: userInput.value, isUser: true })

  isLoading.value = true
  try {
    const response = await axios.post('https://unity.app.n8n.cloud/webhook/api/chat', {
      message: userInput.value  // JSON payload: { message: "user input" }
    })
    // Assume response.data.reply contains the AI's reply; adjust if different
    // messages.value.push({ text: response.data.reply || 'No response from AI', isUser: false }) //Back to reply
    messages.value.push({ text: response.data.contents?.[0]?.parts?.[0]?.reply || 'No response from AI', isUser: false })
  } catch (error) {
    messages.value.push({ text: 'Error connecting to AI: ' + (error as Error).message, isUser: false })
  } finally {
    isLoading.value = false
    userInput.value = ''
  }
}
</script>

<template>
  <div class="container text-center mt-5">
    <h1>Micro-Assistant Teacher</h1>
    <p>Click the button to start chatting with the AI teacher.</p>
    <button class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#chatModal">Start Chat</button>
  </div>

  <!-- Chat Modal -->
  <div class="modal fade" id="chatModal" tabindex="-1" aria-labelledby="chatModalLabel" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered modal-lg">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="chatModalLabel">Chat with AI Teacher</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
        </div>
        <div class="modal-body">
          <div class="chat-window border rounded p-3 mb-3" style="height: 300px; overflow-y: auto;">
            <div v-for="(msg, index) in messages" :key="index" :class="['mb-2', msg.isUser ? 'text-end' : 'text-start']">
              <span :class="['badge', msg.isUser ? 'bg-primary' : 'bg-secondary']">{{ msg.text }}</span>
            </div>
            <div v-if="isLoading" class="text-center">Thinking...</div>
          </div>
          <div class="input-group">
            <input v-model="userInput" type="text" class="form-control" placeholder="Type your question..." @keyup.enter="sendMessage" />
            <button class="btn btn-primary" @click="sendMessage" :disabled="isLoading">Send</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Optional custom styles if needed, but Bootstrap handles most */
.chat-window {
  background-color: #f8f9fa;
}
</style>