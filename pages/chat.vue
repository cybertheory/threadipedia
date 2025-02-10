<script setup lang="ts">
import { Textarea } from '@/components/ui/textarea'
import { ref } from 'vue'


interface RouteState {
  messages?: string[]
}

const route = useRoute()
const messages = useState<string[]>('chatMessages', () => [])



// const messages = ref<string[]>(defineProps<{ initialMessages: string[] }>().initialMessages)
const newMessage = ref('')

const formatMessage = (text: string) => {
  return text.replace(/\[([^\]]+)\]\(([^)]+)\)/g, '<a href="$2" target="_blank" class="text-blue-600 hover:underline">$1</a>')
}

const sendMessage = async () => {
  if (!newMessage.value.trim()) return
  
  messages.value.push(newMessage.value)
  
  try {
    const response = await fetch('http://localhost:8000/answer', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        msg: messages.value
      })
    })
    
    const data = await response.json()
    messages.value.push(data.message)
    newMessage.value = ''
  } catch (error) {
    console.error('Error:', error)
  }
}
</script>

<template>
  <div class="flex flex-col h-screen bg-slate-50">
    <div class="text-md font-bold text-slate-950 p-4">
      Threadipedia
    </div>
    <div class="flex-1 overflow-y-auto p-4">
      <div v-for="(message, index) in messages" :key="index" 
           class="mb-4 p-3 rounded-lg" 
           :class="index % 2 === 0 ? 'bg-blue-100 ml-auto' : 'bg-white'"
           v-html="formatMessage(message)">
      </div>
    </div>
    
    <div class="p-4 border-t flex gap-2">
      <Textarea 
        v-model="newMessage"
        class="flex-1 rounded-lg bg-white border-slate-950 border-2 p-2"
        placeholder="Ask a follow-up question..."
      />
      <button 
        @click="sendMessage"
        class="bg-blue-700 text-white px-4 py-2 rounded-lg hover:bg-blue-800"
      >
        Send
      </button>
    </div>
  </div>
</template>
