<script setup lang="ts">
const selectedDifficulty = ref<'easy' | 'medium' | 'hard'>('medium')

const emit = defineEmits<{
  next: []
  difficultyChange: [difficulty: 'easy' | 'medium' | 'hard']
}>()

const selectDifficulty = (difficulty: 'easy' | 'medium' | 'hard') => {
  selectedDifficulty.value = difficulty
  emit('difficultyChange', difficulty)
}

const handleNext = () => {
  emit('next')
}
</script>

<template>
  <nav class="nav-content">
    <!-- Difficulty group buttons -->
    <div class="button-group">
      <button
        class="difficulty-btn"
        :class="{ active: selectedDifficulty === 'easy' }"
        @click="selectDifficulty('easy')"
      >
        Easy
      </button>
      <button
        class="difficulty-btn"
        :class="{ active: selectedDifficulty === 'medium' }"
        @click="selectDifficulty('medium')"
      >
        Medium
      </button>
      <button
        class="difficulty-btn"
        :class="{ active: selectedDifficulty === 'hard' }"
        @click="selectDifficulty('hard')"
      >
        Hard
      </button>
    </div>

    <!-- Vertical divider -->
    <div class="divider"></div>

    <!-- Next button -->
    <button class="next-btn" @click="handleNext">
      Next
    </button>
  </nav>
</template>

<style scoped>
.nav-content {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: clamp(0.5rem, 3vw, 2rem);
  color: white;
}

.button-group {
  display: flex;
  gap: 0;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 0.75rem;
  padding: 0.25rem;
  backdrop-filter: blur(10px);
}

.difficulty-btn {
  padding: 0.75rem clamp(0.875rem, 2vw, 1.75rem);
  background: transparent;
  border: none;
  color: rgba(255, 255, 255, 0.7);
  font-size: clamp(0.8rem, 2vw, 1.05rem);
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
  border-radius: 0.5rem;
  position: relative;
  white-space: nowrap;
}

.difficulty-btn:hover {
  color: rgba(255, 255, 255, 0.95);
  background: rgba(255, 255, 255, 0.08);
}

.difficulty-btn.active {
  background: rgba(255, 255, 255, 0.15);
  color: white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.divider {
  width: 1px;
  height: 2.25rem;
  background: linear-gradient(
    to bottom,
    transparent,
    rgba(255, 255, 255, 0.2) 50%,
    transparent
  );
}

.next-btn {
  padding: 0.75rem clamp(1.25rem, 3vw, 2.25rem);
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  color: white;
  font-size: clamp(0.8rem, 2vw, 1.05rem);
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
  border-radius: 0.75rem;
  backdrop-filter: blur(10px);
  white-space: nowrap;
}

.next-btn:hover {
  background: rgba(255, 255, 255, 0.15);
  border-color: rgba(255, 255, 255, 0.3);
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.next-btn:active {
  transform: translateY(0);
}

@media (max-width: 768px) {
  .nav-content {
    gap: 0.65rem;
  }
  
  .button-group {
    padding: 0.3rem;
  }
  
  .difficulty-btn {
    padding: 0.75rem 1rem;
    font-size: 0.95rem;
  }
  
  .next-btn {
    padding: 0.75rem 1.35rem;
    font-size: 0.95rem;
  }
  
  .divider {
    height: 2rem;
  }
}

@media (max-width: 480px) {
  .nav-content {
    gap: 0.5rem;
  }
  
  .button-group {
    padding: 0.25rem;
  }
  
  .difficulty-btn {
    padding: 0.65rem 0.85rem;
    font-size: 0.875rem;
  }
  
  .next-btn {
    padding: 0.65rem 1.1rem;
    font-size: 0.875rem;
  }
  
  .divider {
    height: 1.85rem;
  }
}
</style>

