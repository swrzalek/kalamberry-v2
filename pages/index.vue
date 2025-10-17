<script setup lang="ts">
import wordsByDifficulty from '~/data/words.json'

// Page metadata
useHead({
  title: 'TresJS Card Deck',
  meta: [
    { name: 'description', content: 'A TresJS Nuxt application' },
    { name: 'viewport', content: 'width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no' }
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
.main {
  position: fixed;
  width: 100%;
  height: 100vh;
  background: #020420;
  overflow: hidden;
  overscroll-behavior: none;
}

.navigation {
  position: absolute;
  top: 1rem;
  left: 50%;
  transform: translateX(-50%);
  height: 5.5rem;
  width: min(42.5rem, calc(100vw - 2rem));
  border-radius: 1rem;
  z-index: 10;
  padding: 0 0.5rem;
}

@media (max-width: 768px) {
  .navigation {
    top: 0.25rem;
    height: 5rem;
    width: calc(100vw - 1rem);
  }
}

@media (max-width: 480px) {
  .navigation {
    top: 0.15rem;
    height: 4.75rem;
  }
}
</style>
