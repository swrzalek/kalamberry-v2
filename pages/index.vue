<script setup lang="ts">
// Page metadata
useHead({
  title: 'TresJS Card Deck',
  meta: [
    { name: 'description', content: 'A TresJS Nuxt application' }
  ]
})

const experienceRef = ref<{ nextCard: () => void } | null>(null)

const handleNext = () => {
  experienceRef.value?.nextCard()
}
</script>

<template>
  <div class="relative w-full h-screen">
    <TresCanvas
      clear-color="#020420"
      window-size
    >
      <TheExperience ref="experienceRef" />
    </TresCanvas>
    
    <!-- Next Button Overlay -->
    <div class="absolute bottom-8 left-1/2 -translate-x-1/2 z-10">
      <button
        @click="handleNext"
        class="px-8 py-4 bg-gradient-to-r from-purple-600 to-blue-600 hover:from-purple-700 hover:to-blue-700 text-white font-bold text-lg rounded-full shadow-lg hover:shadow-xl transform hover:scale-105 active:scale-95 transition-all duration-200 ease-out"
      >
        Next Card
      </button>
    </div>
  </div>
</template>

<style scoped>
button {
  backdrop-filter: blur(10px);
  border: 2px solid rgba(255, 255, 255, 0.1);
}
</style>