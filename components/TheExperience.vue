<script setup lang="ts">
import { RoundedBoxGeometry } from 'three/examples/jsm/geometries/RoundedBoxGeometry.js'
import type { Group } from 'three'
import { Html } from '@tresjs/cientos'

const { onBeforeRender } = useLoop()

// Words to display on cards
const words = [
  'Hello',
  'World',
  'Vue',
  'TresJS',
  'Three.js',
  'Animation',
  'Cards',
  'Deck',
  'Shuffle',
  'Amazing'
]

// Track next word index to show (starts at 3 since we're showing 0,1,2 initially)
const nextWordIndex = ref(3)

// Card refs - using shallowRef as recommended by TresJS docs (now pointing to Groups)
const card1Ref = shallowRef<Group | null>(null)
const card2Ref = shallowRef<Group | null>(null)
const card3Ref = shallowRef<Group | null>(null)

// Card data with word indices
const cards = [
  { ref: card1Ref, color: '#4ecdc4', basePosition: [0, 0, 0] as [number, number, number], baseRotation: [Math.PI / 2, 0, 0] as [number, number, number], wordIndex: 0 },
  { ref: card2Ref, color: '#ff6b6b', basePosition: [0, 0, -0.18] as [number, number, number], baseRotation: [Math.PI / 2, 0.15, 0] as [number, number, number], wordIndex: 1 },
  { ref: card3Ref, color: '#f7b731', basePosition: [0, 0, -0.36] as [number, number, number], baseRotation: [Math.PI / 2, -0.15, 0] as [number, number, number], wordIndex: 2 }
]

// Track which card is in which position (front=0, middle=1, back=2)
const cardOrder = ref([0, 1, 2]) // indices into [card1Ref, card2Ref, card3Ref]
const allCardRefs = [card1Ref, card2Ref, card3Ref]

// Reactive word assignments
const cardWords = ref([
  words[0],
  words[1],
  words[2]
])

// Computed properties to get the word for each card based on its position in the order
const card1Word = computed(() => {
  const positionIndex = cardOrder.value.indexOf(0) // Find where card 0 is in the order
  return cardWords.value[positionIndex]
})
const card2Word = computed(() => {
  const positionIndex = cardOrder.value.indexOf(1)
  return cardWords.value[positionIndex]
})
const card3Word = computed(() => {
  const positionIndex = cardOrder.value.indexOf(2)
  return cardWords.value[positionIndex]
})

const isAnimating = ref(false)
const animationProgress = ref(0)
const showText = ref(true)

// Easing function for smooth animation (ease-in-out)
const easeInOutCubic = (t: number): number => {
  return t < 0.5 
    ? 4 * t * t * t 
    : 1 - Math.pow(-2 * t + 2, 3) / 2
}

// Create rounded box geometry for the card (width, thickness, height)
const roundedGeometry = new RoundedBoxGeometry(2.5, 0.2, 3.5, 3, 0.08)

// Initialize card positions on mount
onMounted(() => {
  if (card1Ref.value && cards[0]) {
    card1Ref.value.position.set(...cards[0].basePosition)
    card1Ref.value.rotation.set(...cards[0].baseRotation)
  }
  if (card2Ref.value && cards[1]) {
    card2Ref.value.position.set(...cards[1].basePosition)
    card2Ref.value.rotation.set(...cards[1].baseRotation)
  }
  if (card3Ref.value && cards[2]) {
    card3Ref.value.position.set(...cards[2].basePosition)
    card3Ref.value.rotation.set(...cards[2].baseRotation)
  }
})

// Animation function
const nextCard = () => {
  console.log('nextCard called!', isAnimating.value)
  if (isAnimating.value) return
  
  // Hide text immediately
  showText.value = false
  
  isAnimating.value = true
  animationProgress.value = 0
  
  console.log('Animation started')
}

// Animation loop using TresJS pattern
onBeforeRender(({ delta }) => {
  if (!isAnimating.value) return
  
  // Increment animation progress (frame-rate independent)
  animationProgress.value += delta * 1.5 // 1.5 = speed multiplier (slightly slower for smoother feel)
  
  const rawProgress = Math.min(animationProgress.value, 1)
  const progress = easeInOutCubic(rawProgress) // Apply easing
  
  console.log('Animating, progress:', progress.toFixed(2))
  
  // Get the card that's currently in front position
  const frontCardIndex = cardOrder.value[0]
  const frontCardRef = frontCardIndex !== undefined ? allCardRefs[frontCardIndex] : null
  
  // Animate the front card - arc around the side of the deck
  if (frontCardRef?.value) {
    if (rawProgress < 0.5) {
      // Phase 1: Move card to the right and slightly up (0 to 0.5)
      const phase1Raw = rawProgress * 2
      const phase1Progress = easeInOutCubic(phase1Raw)
      frontCardRef.value.position.x = phase1Progress * 5 // Move right
      frontCardRef.value.position.y = phase1Progress * 0.5 // Just slightly up
      frontCardRef.value.position.z = phase1Progress * 0.5 // Move forward a bit
      frontCardRef.value.rotation.y = phase1Progress * 1.2
      frontCardRef.value.rotation.z = phase1Progress * 0.3
    } else {
      // Phase 2: Continue around behind the deck (0.5 to 1)
      const phase2Raw = (rawProgress - 0.5) * 2
      const phase2Progress = easeInOutCubic(phase2Raw)
      // Continue arc around to the back, moving behind the deck
      frontCardRef.value.position.x = 5 - phase2Progress * 5 // Come back to center
      frontCardRef.value.position.y = 0.5 - phase2Progress * 0.5 // Return to level
      frontCardRef.value.position.z = 0.5 - phase2Progress * 0.86 // Go behind to z=-0.36
      // Smoothly transition to the back card's rotation angle (-0.15)
      frontCardRef.value.rotation.y = 1.2 - phase2Progress * 1.35 // rotate back to -0.15
      frontCardRef.value.rotation.z = 0.3 - phase2Progress * 0.3
    }
  }
  
  // Move other cards forward smoothly with easing
  const middleCardIndex = cardOrder.value[1]
  const backCardIndex = cardOrder.value[2]
  const middleCardRef = middleCardIndex !== undefined ? allCardRefs[middleCardIndex] : null
  const backCardRef = backCardIndex !== undefined ? allCardRefs[backCardIndex] : null
  
  if (middleCardRef?.value) {
    const targetZ = rawProgress < 0.5 ? -0.18 : -0.18 + easeInOutCubic((rawProgress - 0.5) * 2) * 0.18
    middleCardRef.value.position.z += (targetZ - middleCardRef.value.position.z) * delta * 10
    
    const targetRotY = rawProgress < 0.5 ? 0.15 : 0.15 - easeInOutCubic((rawProgress - 0.5) * 2) * 0.15
    middleCardRef.value.rotation.y += (targetRotY - middleCardRef.value.rotation.y) * delta * 10
  }
  
  if (backCardRef?.value) {
    const targetZ = rawProgress < 0.5 ? -0.36 : -0.36 + easeInOutCubic((rawProgress - 0.5) * 2) * 0.18
    backCardRef.value.position.z += (targetZ - backCardRef.value.position.z) * delta * 10
    
    const targetRotY = rawProgress < 0.5 ? -0.15 : -0.15 + easeInOutCubic((rawProgress - 0.5) * 2) * 0.3
    backCardRef.value.rotation.y += (targetRotY - backCardRef.value.rotation.y) * delta * 10
  }
  
  // Reset animation when complete
  if (rawProgress >= 1) {
    // Rotate the order: front card goes to back
    const frontCard = cardOrder.value.shift()!
    cardOrder.value.push(frontCard)
    
    // Set final positions for all cards based on new order
    cardOrder.value.forEach((cardIndex, positionIndex) => {
      const cardRef = cardIndex !== undefined ? allCardRefs[cardIndex] : null
      if (cardRef?.value && cards[positionIndex]) {
        cardRef.value.position.set(...cards[positionIndex].basePosition)
        cardRef.value.rotation.set(...cards[positionIndex].baseRotation)
      }
    })
    
    // Update card words array - shift words forward
    cardWords.value.shift() // Remove front card word
    // The card that went to the back gets the next new word
    cardWords.value.push(words[nextWordIndex.value])
    // Advance to next word
    nextWordIndex.value = (nextWordIndex.value + 1) % words.length
    
    isAnimating.value = false
    animationProgress.value = 0
    
    // Show text with a delay for smooth fade-in
    setTimeout(() => {
      showText.value = true
    }, 100)
    
    console.log('Animation complete')
  }
})

// Expose nextCard function and cardWords for the button and text display
defineExpose({ nextCard, cardWords })
</script>

<template>
  <!-- Close-up perspective camera, like card in front of face -->
  <TresPerspectiveCamera
    :position="[0, 0, 6]"
    :fov="45"
    :look-at="[0, 0, 0]"
  />
  <OrbitControls />
  
  <!-- Lighting for 3D depth -->
  <TresAmbientLight :intensity="0.6" />
  <TresDirectionalLight
    :position="[5, 8, 5]"
    :intensity="0.8"
    cast-shadow
  />
  <TresDirectionalLight
    :position="[-3, 5, -3]"
    :intensity="0.3"
  />
  
  <!-- Cards with rounded corners and text -->
  <TresGroup>
    <!-- Card 1 with text -->
    <TresGroup 
      ref="card1Ref"
    >
      <TresMesh
        cast-shadow
        :geometry="roundedGeometry"
      >
        <TresMeshStandardMaterial
          :color="cards[0]?.color || '#4ecdc4'"
          :roughness="0.4"
          :metalness="0.2"
        />
      </TresMesh>
      <Html
        v-if="card1Ref && Math.abs(card1Ref.position.z) < 0.1 && showText"
        :position="[0, 0, 0.15]"
        :rotation="[-Math.PI / 2, 0, 0]"
        transform
        :distance-factor="0.5"
      >
        <div class="card-text">
          {{ card1Word }}
        </div>
      </Html>
    </TresGroup>
    
    <!-- Card 2 with text -->
    <TresGroup 
      ref="card2Ref"
    >
      <TresMesh
        cast-shadow
        :geometry="roundedGeometry"
      >
        <TresMeshStandardMaterial
          :color="cards[1]?.color || '#ff6b6b'"
          :roughness="0.4"
          :metalness="0.2"
        />
      </TresMesh>
      <Html
        v-if="card2Ref && Math.abs(card2Ref.position.z) < 0.1 && showText"
        :position="[0, 0, 0.15]"
        :rotation="[-Math.PI / 2, 0, 0]"
        transform
        :distance-factor="0.5"
      >
        <div class="card-text">
          {{ card2Word }}
        </div>
      </Html>
    </TresGroup>
    
    <!-- Card 3 with text -->
    <TresGroup 
      ref="card3Ref"
    >
      <TresMesh
        cast-shadow
        :geometry="roundedGeometry"
      >
        <TresMeshStandardMaterial
          :color="cards[2]?.color || '#f7b731'"
          :roughness="0.4"
          :metalness="0.2"
        />
      </TresMesh>
      <Html
        v-if="card3Ref && Math.abs(card3Ref.position.z) < 0.1 && showText"
        :position="[0, 0, 0.15]"
        :rotation="[-Math.PI / 2, 0, 0]"
        transform
        :distance-factor="0.5"
      >
        <div class="card-text">
          {{ card3Word }}
        </div>
      </Html>
    </TresGroup>
  </TresGroup>
</template>

<style scoped>
.card-text {
  font-size: 140px;
  font-weight: bold;
  color: white;
  text-shadow: 0 0 20px rgba(0,0,0,0.8), 0 0 40px rgba(0,0,0,0.6);
  pointer-events: none;
  user-select: none;
  white-space: nowrap;
  animation: fadeIn 0.5s ease-in;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.8);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
</style>