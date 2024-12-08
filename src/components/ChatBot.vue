<template>
  <div class="chatbot-container max-w-2xl mx-auto p-6 bg-gray-50 rounded-lg shadow-md">
    <!-- Title -->
    <h2 class="text-3xl font-bold mb-6 text-center text-gray-800">AI Chatbot</h2>

    <!-- Chat window -->
    <div class="chat-window overflow-y-auto bg-white p-4 h-96 border border-gray-300 rounded mb-4">
      <div v-for="(msg, index) in messages" :key="index" class="mb-3">
        <div
          :class="{
            'flex justify-start': msg.sender === 'user',
            'flex justify-end': msg.sender === 'bot',
          }"
        >
          <div
            :class="{
              'bg-blue-500 text-white px-4 py-2 rounded-lg shadow': msg.sender === 'user',
              'bg-gray-300 text-gray-900 px-4 py-2 rounded-lg shadow': msg.sender === 'bot',
            }"
            class="max-w-xs border border-gray-300"
          >
            {{ msg.text }}
          </div>
        </div>
      </div>
      <!-- Loading indicator -->
      <div v-if="isLoading" class="flex justify-end">
        <div class="bg-gray-300 text-gray-900 px-4 py-2 rounded-lg shadow max-w-xs border border-gray-300">
          Přemýšlím...
        </div>
      </div>
    </div>

    <!-- Input and send button -->
    <div class="chat-input flex items-center">
      <input
        v-model="userMessage"
        type="text"
        placeholder="Napište zprávu..."
        class="flex-grow p-3 border border-gray-300 rounded-l-md focus:outline-none focus:ring focus:ring-blue-400"
        @keyup.enter="sendMessage"
      />
      <button
        class="bg-blue-600 text-white px-5 py-3 rounded-r-md hover:bg-blue-700 transition-colors flex-shrink-0"
        @click="sendMessage"
      >
        Odeslat
      </button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      userMessage: '', // Message typed by the user
      messages: [],    // Stores conversation history
      isLoading: false, // Indicates if bot is "thinking"
    };
  },
  methods: {
    // Sends user message and fetches bot response
    async sendMessage() {
      if (!this.userMessage.trim()) return; // Prevent empty messages

      // Add user's message to the chat
      this.messages.push({ sender: 'user', text: this.userMessage });

      // Show loading indicator
      this.isLoading = true;

      // Fetch bot response
      const botReply = await this.fetchResponse(this.userMessage);

      // Hide loading indicator
      this.isLoading = false;

      // Add bot's reply to the chat
      this.messages.push({ sender: 'bot', text: botReply.reply });

      // Clear input field
      this.userMessage = '';
    },

    // Fetches response from the xAI API
    async fetchResponse(message) {
      const API_URL = 'https://api.x.ai/v1/chat/completions';
      const apiKey = 'xai-hGeKJ18KruztcCgF50YJBRBBTktzXVDZ5tnTa3vpE1xbrbGinUqZrP0pYENN7G1YAwDF8kdCHV0htevx'; // Replace with your xAI API key

      const headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${apiKey}`,
      };

      const data = {
        model: 'grok-beta',
        messages: [
          { role: 'system', content: 'You are Grok, a chatbot inspired by the Hitchhiker\'s Guide to the Galaxy.' },
          { role: 'user', content: message },
        ],
      };

      try {
        const response = await axios.post(API_URL, data, { headers });
        console.log('API Response:', response.data);
        return { reply: response.data.choices[0].message.content };
      } catch (error) {
        console.error('Error communicating with API:', error);
        return { reply: 'Omlouvám se, došlo k chybě při zpracování.' };
      }
    },
  },
};
</script>

<style scoped>
.chat-window {
  scroll-behavior: smooth;
}

.chat-input {
  align-items: center;
  flex-wrap: nowrap;
}

.bg-blue-500 {
  background-color: #405391c2;
  display: flex;
  border-radius: 1.25rem;
  padding: 10px;
}

.bg-blue-600 {
  background-color: #1E3A8A;
}

.bg-gray-300 {
  background-color: #8c96a5;
  display: flex;
  border-radius: 1.25rem;
  padding: 10px;
}

.text-gray-900 {
  color: #1F2937;
}

.text-white {
  color: #FFFFFF;
}
</style>
