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

const handleDifficultyChange = (difficulty: 'easy' | 'medium' | 'hard') => {
  console.log('Difficulty changed to:', difficulty)
  // Add your difficulty logic here
}
</script>

<template>
  <div class="relative w-full h-screen main">
    <div class="navigation">
      <TheNavigation @next="handleNext" @difficulty-change="handleDifficultyChange" />
    </div>
    <div class="container">
      <TresCanvas
          clear-color="#020420"
      >
        <TheExperience ref="experienceRef" />
      </TresCanvas>
    </div>
  </div>
</template>

<style scoped>
button {
  backdrop-filter: blur(10px);
  border: 2px solid rgba(255, 255, 255, 0.1);
}

.main {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: #020420;
  height: 100vh;
}
.navigation {
  height: 5rem;
  width: 40rem;
  border-radius: 1rem;
  backdrop-filter: blur(10px);
}
.container {
  width: 40rem;
  height: 40rem;
}
</style>