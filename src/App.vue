<template>
  <!-- Header -->
  <div class="header">
    <div class="chat-header">
      <img class="logo" src="./assets/logo.svg" alt="logo">
      <h1>MantiBot</h1>
    </div>
  </div>

  <!-- Chat -->
  <div class="chat-window">
    <div v-for="(message, index) in chatMessages" :key="index" class="chat-message">
      <div :class="message.role">
        <vue-markdown :source="message.content" />
      </div>
    </div>
  </div>

  <!-- Input Mensajes -->
  <div class="input-container">
    <input v-model="userInput" type="text" placeholder="Escribe un mensaje..." @keydown.enter="sendMessage" />
    <button @click="sendMessage" :disabled="isLoading">Enviar</button>
  </div>
</template>

<script setup>
import { computed, ref } from 'vue';
import OpenAI from "openai";
import VueMarkdown from 'vue-markdown-render'

const mantibot = new OpenAI({
  apiKey: import.meta.env.VITE_OPENAI_KEY,
  dangerouslyAllowBrowser: true
})

// Pautas que debe seguir la IA
const systemInfo = {
  role: "system",
  content: "Eres un asistente de IA experto en salud mental y emocional; experto en los estudios de Janis Leann Whitlock y Matthew K. Nock; y tu nombre es MantiBot. Eres un bot amigable que brinda consejos. Si te hacen una pregunta que no está relacionada con la salud mental o emocional, responde 'Lo siento, pero tu pregunta está más allá de mis funciones'. Si alguien te saluda, preséntate con tu nombre y mencionales cuál es tu uso. Cuando saludes o termines una sesión haz uso de algunos emojis."
}
const messages = ref([systemInfo])
const chatMessages = computed(() => {
  return messages.value.filter((message) => message.role !== 'system')
})

const userInput = ref('')
const isLoading = ref(false)

const sendMessage = async () => {
  if (userInput.value.trim() === '') return;

  // Agregar mensaje de usuario al chat
  messages.value.push({ content: userInput.value, role: 'user' });
  
  // Limpiar el valor del input
  userInput.value = '';

  // Simulación de chatbot pensando.
  messages.value.push({ content: '...', role: 'assistant' });

  // Realizanción de la consulta a openai
  isLoading.value = true
  const completion = await mantibot.chat.completions.create({
    messages: messages.value,
    model: "gpt-4o-mini-2024-07-18",
  });
  messages.value[messages.value.length - 1] = completion.choices[0].message;
  isLoading.value = false
  console.log(completion)
  console.log(completion.choices[0]);
}

</script>
