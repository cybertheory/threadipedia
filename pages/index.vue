<script setup lang="ts">
import { Textarea } from '@/components/ui/textarea'
import { onMounted, ref } from 'vue'


interface FloatingElement {
  id: number
  icon: string
  x: number
  y: number
  speed: number
  scale: number
  rotation: number
}

const floatingElements = ref<FloatingElement[]>([])

const router = useRouter()
const userInput = ref('')
const messages = ref<string[]>([])
const isLoading = ref(false)

const setupAnimation = () => {
  const icons = ['📚', '🔍', '💡', '✏️', '📖', '🎓', '🗞️', '📝']
  floatingElements.value = Array.from({ length: 15 }, (_, i) => ({
    id: i,
    icon: icons[Math.floor(Math.random() * icons.length)],
    x: Math.random() * window.innerWidth,
    y: Math.random() * window.innerHeight,
    speed: 0.5 + Math.random() * 2,
    scale: 0.5 + Math.random() * 1.5,
    rotation: Math.random() * 360
  }))

  animateElements()
}

const animateElements = () => {
  floatingElements.value = floatingElements.value.map(element => {
    element.y -= element.speed
    element.rotation += 0.5
    if (element.y < -50) {
      element.y = window.innerHeight + 50
      element.x = Math.random() * window.innerWidth
    }
    return element
  })
  requestAnimationFrame(animateElements)
}

const navigateToChat = async () => {
  if (!userInput.value.trim()) return
  isLoading.value = true
  messages.value.push(userInput.value)
  
  try {
    const response = await fetch('https://api-fragrant-bush-7881.fly.dev/answer', {
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
    
    const chatMessages = useState<string[]>('chatMessages')
    chatMessages.value = messages.value
    
    navigateTo('/chat')
  } catch (error) {
    alert('Failed to get response. Please try again.')
    console.error('Error:', error)
  } finally {
    isLoading.value = false
  }
}





onMounted(() => {
  setupAnimation()
})


</script>

<template>
  <div class="bg-slate-50 overflow-hidden w-screen h-screen justify-center items-center flex-col relative">
    <!-- Floating Elements -->
    <div v-for="element in floatingElements" :key="element.id" class="floating-element absolute"
      :style="{
        left: `${element.x}px`,
        top: `${element.y}px`,
        transform: `scale(${element.scale}) rotate(${element.rotation}deg)`,
        transition: 'transform 0.5s ease'
      }">
      {{ element.icon }}
    </div>

    <!-- Original Content -->
    <div class="flex text-md font-bold text-slate-950 p-4 z-10 relative">
      Threadipedia
    </div>
    <div class="flex z-10 relative">
      <h1 class="text-4xl font-bold text-slate-950 text-center w-screen mt-40">The AI deep research agent for
        <span class="text-blue-700 underline"><a href="https://en.wikipedia.org">Wikipedia</a></span>
      </h1>
    </div>
    <div class="flex relative z-10">
      <Textarea 
       v-model="userInput"
      class="w-screen m-10 rounded-lg justify-center items-center bg-white border-slate-950 border-2 p-2"
        placeholder="Ask any question e.g Who built the statue of liberty?" />
        <button 
    @click="navigateToChat"
    class="absolute bottom-12 right-12 bg-blue-700 text-white px-4 py-2 rounded-lg hover:bg-blue-800 min-w-[80px] flex items-center justify-center"
    :disabled="isLoading"
  >
    <span v-if="!isLoading">Ask</span>
    <div v-else class="w-6 h-6 border-2 border-white border-t-transparent rounded-full animate-spin"></div>
  </button>
    </div>
    <p class="w-screen text-center z-10 relative">*unofficial client for wikipedia</p>
  </div>
</template>
<style scoped>
.floating-element {
  font-size: 2rem;
  opacity: 0.2;
  pointer-events: none;
  z-index: 1;
}

.floating-element:hover {
  opacity: 0.4;
}
</style>

