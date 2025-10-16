<script setup lang="ts">
import wordsByDifficulty from '~/data/words.json'

// Page metadata
useHead({
  title: 'TresJS Card Deck',
  meta: [
    { name: 'description', content: 'A TresJS Nuxt application' }
  ]
})

// State management
const currentDifficulty = ref<'easy' | 'medium' | 'hard'>('medium')
const currentWords = computed(() => wordsByDifficulty[currentDifficulty.value])

const experienceRef = ref<{ nextCard: () => void } | null>(null)

const handleNext = () => {
  experienceRef.value?.nextCard()
}

const handleDifficultyChange = (difficulty: 'easy' | 'medium' | 'hard') => {
  currentDifficulty.value = difficulty
}
</script>

<template>
  <div class="main">
    <div class="navigation">
      <TheNavigation @next="handleNext" @difficulty-change="handleDifficultyChange" />
    </div>
    <TresCanvas clear-color="#020420">
      <TheExperience ref="experienceRef" :words="currentWords" />
    </TresCanvas>
  </div>
</template>

<style scoped>
button {
  backdrop-filter: blur(10px);
  border: 2px solid rgba(255, 255, 255, 0.1);
}

.main {
  position: relative;
  width: 100%;
  height: 100vh;
  background: #020420;
}

.navigation {
  position: absolute;
  top: 1rem;
  left: 50%;
  transform: translateX(-50%);
  height: 5rem;
  width: 40rem;
  border-radius: 1rem;
  z-index: 10;
}
</style>
