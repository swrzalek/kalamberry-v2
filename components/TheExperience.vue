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

// Reactive word assignments
const cardWords = ref([
  words[0],
  words[1],
  words[2]
])

const isAnimating = ref(false)
const animationProgress = ref(0)

// Easing function for smooth animation (ease-in-out)
const easeInOutCubic = (t: number): number => {
  return t < 0.5 
    ? 4 * t * t * t 
    : 1 - Math.pow(-2 * t + 2, 3) / 2
}

// Create rounded box geometry for the card (width, thickness, height)
const roundedGeometry = new RoundedBoxGeometry(2.5, 0.2, 3.5, 3, 0.08)

// Animation function
const nextCard = () => {
  console.log('nextCard called!', isAnimating.value)
  if (isAnimating.value) return
  
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
  
  console.log('Animating, progress:', progress.toFixed(2), 'card1:', !!card1Ref.value)
  
  // Animate the top card (card1)
  if (card1Ref.value) {
    if (rawProgress < 0.5) {
      // Phase 1: Move card out to the right (0 to 0.5)
      const phase1Raw = rawProgress * 2
      const phase1Progress = easeInOutCubic(phase1Raw)
      card1Ref.value.position.x = phase1Progress * 5
      card1Ref.value.position.z = phase1Progress * 2
      card1Ref.value.rotation.y = phase1Progress * 1
      card1Ref.value.rotation.z = phase1Progress * 0.3
    } else {
      // Phase 2: Move card to back from left (0.5 to 1)
      const phase2Raw = (rawProgress - 0.5) * 2
      const phase2Progress = easeInOutCubic(phase2Raw)
      card1Ref.value.position.x = 5 - phase2Progress * 10 + phase2Progress * 5 // right to left to center-back
      card1Ref.value.position.z = 2 - phase2Progress * 2.36 // move to back position
      // Smoothly transition to the back card's rotation angle (-0.15)
      card1Ref.value.rotation.y = 1 - phase2Progress * 1.15 // rotate back to -0.15
      card1Ref.value.rotation.z = 0.3 - phase2Progress * 0.3
    }
  }
  
  // Move other cards forward smoothly with easing
  if (card2Ref.value) {
    const targetZ = rawProgress < 0.5 ? -0.18 : -0.18 + easeInOutCubic((rawProgress - 0.5) * 2) * 0.18
    card2Ref.value.position.z += (targetZ - card2Ref.value.position.z) * delta * 10
    
    const targetRotY = rawProgress < 0.5 ? 0.15 : 0.15 - easeInOutCubic((rawProgress - 0.5) * 2) * 0.15
    card2Ref.value.rotation.y += (targetRotY - card2Ref.value.rotation.y) * delta * 10
  }
  
  if (card3Ref.value) {
    const targetZ = rawProgress < 0.5 ? -0.36 : -0.36 + easeInOutCubic((rawProgress - 0.5) * 2) * 0.18
    card3Ref.value.position.z += (targetZ - card3Ref.value.position.z) * delta * 10
    
    const targetRotY = rawProgress < 0.5 ? -0.15 : -0.15 + easeInOutCubic((rawProgress - 0.5) * 2) * 0.3
    card3Ref.value.rotation.y += (targetRotY - card3Ref.value.rotation.y) * delta * 10
  }
  
  // Reset animation when complete
  if (rawProgress >= 1) {
    // Set final positions and rotations
    if (card1Ref.value && cards[2]) {
      card1Ref.value.position.set(...cards[2].basePosition)
      card1Ref.value.rotation.set(...cards[2].baseRotation)
    }
    if (card2Ref.value && cards[0]) {
      card2Ref.value.position.set(...cards[0].basePosition)
      card2Ref.value.rotation.set(...cards[0].baseRotation)
    }
    if (card3Ref.value && cards[1]) {
      card3Ref.value.position.set(...cards[1].basePosition)
      card3Ref.value.rotation.set(...cards[1].baseRotation)
    }
    
    // Swap card references to maintain order
    const temp = card1Ref.value
    card1Ref.value = card2Ref.value
    card2Ref.value = card3Ref.value
    card3Ref.value = temp
    
    // Update card words array - shift words forward
    cardWords.value.shift() // Remove front card word
    // The card that went to the back gets the next new word
    cardWords.value.push(words[nextWordIndex.value])
    // Advance to next word
    nextWordIndex.value = (nextWordIndex.value + 1) % words.length
    
    isAnimating.value = false
    animationProgress.value = 0
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
      :position="cards[0]?.basePosition || [0, 0, 0]"
      :rotation="cards[0]?.baseRotation || [0, 0, 0]"
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
        :position="[0, 0, 0.15]"
        :rotation="[-Math.PI / 2, 0, 0]"
        transform
        :distance-factor="0.5"
      >
        <div class="card-text">
          {{ cardWords[0] }}
        </div>
      </Html>
    </TresGroup>
    
    <!-- Card 2 with text -->
    <TresGroup 
      ref="card2Ref"
      :position="cards[1]?.basePosition || [0, 0, -0.18]"
      :rotation="cards[1]?.baseRotation || [Math.PI / 2, 0.15, 0]"
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
        :position="[0, 0, 0.15]"
        :rotation="[-Math.PI / 2, 0, 0]"
        transform
        :distance-factor="0.5"
      >
        <div class="card-text">
          {{ cardWords[1] }}
        </div>
      </Html>
    </TresGroup>
    
    <!-- Card 3 with text -->
    <TresGroup 
      ref="card3Ref"
      :position="cards[2]?.basePosition || [0, 0, -0.36]"
      :rotation="cards[2]?.baseRotation || [Math.PI / 2, -0.15, 0]"
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
        :position="[0, 0, 0.15]"
        :rotation="[-Math.PI / 2, 0, 0]"
        transform
        :distance-factor="0.5"
      >
        <div class="card-text">
          {{ cardWords[2] }}
        </div>
      </Html>
    </TresGroup>
  </TresGroup>
</template>

<style scoped>
.card-text {
  font-size: 48px;
  font-weight: bold;
  color: white;
  text-shadow: 0 0 20px rgba(0,0,0,0.8), 0 0 40px rgba(0,0,0,0.6);
  pointer-events: none;
  user-select: none;
  white-space: nowrap;
}
</style>