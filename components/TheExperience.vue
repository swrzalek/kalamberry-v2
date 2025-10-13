<script setup lang="ts">
import * as THREE from 'three'

const { onBeforeRender } = useLoop()

// Card data
const cardColors = [
  '#ff6b6b', // Red
  '#4ecdc4', // Teal
  '#45b7d1', // Blue
  '#f7b731', // Yellow
  '#5f27cd', // Purple
  '#00d2d3', // Cyan
  '#ff9ff3', // Pink
  '#54a0ff', // Light Blue
  '#48dbfb', // Sky Blue
  '#feca57', // Orange
]

interface Card {
  id: number
  color: string
  position: [number, number, number]
  rotation: [number, number, number]
  targetPosition: [number, number, number]
  isAnimating: boolean
}

const cards = ref<Card[]>([])

// Initialize cards
const initCards = () => {
  cards.value = cardColors.map((color, index) => ({
    id: index,
    color,
    position: [0, index * 0.05, 0],
    rotation: [0, 0, 0],
    targetPosition: [0, index * 0.05, 0],
    isAnimating: false,
  }))
}

initCards()

const isTransitioning = ref(false)

// Move top card to bottom
const nextCard = () => {
  if (isTransitioning.value) return
  
  isTransitioning.value = true
  const topCard = cards.value[cards.value.length - 1]
  
  if (!topCard) return

  // Animate the top card out and around
  topCard.isAnimating = true
  topCard.targetPosition = [5, 2, 0]
  
  setTimeout(() => {
    // Move to bottom of array (back of deck)
    const movedCard = cards.value.pop()!
    cards.value.unshift(movedCard)
    
    // Reset position immediately
    movedCard.position = [0, 0, 0]
    movedCard.targetPosition = [0, 0, 0]
    movedCard.isAnimating = false
    
    // Update positions for all cards
    cards.value.forEach((card, index) => {
      card.targetPosition = [0, index * 0.05, 0]
    })
    
    isTransitioning.value = false
  }, 600)
}

// Animation loop
onBeforeRender(({ delta }) => {
  cards.value.forEach((card) => {
    // Smoothly interpolate to target position
    const lerpFactor = card.isAnimating ? 8 : 10
    card.position[0] += (card.targetPosition[0] - card.position[0]) * delta * lerpFactor
    card.position[1] += (card.targetPosition[1] - card.position[1]) * delta * lerpFactor
    card.position[2] += (card.targetPosition[2] - card.position[2]) * delta * lerpFactor
  })
})

// Expose nextCard function
defineExpose({ nextCard })
</script>

<template>
  <TresPerspectiveCamera :position="[0, 3, 8]" />
  <OrbitControls />
  
  <!-- Lighting -->
  <TresAmbientLight :intensity="0.5" />
  <TresDirectionalLight
    :position="[5, 5, 5]"
    :intensity="1"
    cast-shadow
  />
  <TresPointLight
    :position="[-5, 5, 5]"
    :intensity="0.5"
    color="#4ecdc4"
  />
  
  <!-- Cards -->
  <TresGroup>
    <TresMesh
      v-for="card in cards"
      :key="card.id"
      :position="card.position"
      :rotation="card.rotation"
      cast-shadow
      receive-shadow
    >
      <!-- Card geometry: 2.5 wide, 3.5 tall, 0.05 thick -->
      <TresBoxGeometry :args="[2.5, 3.5, 0.05]" />
      <TresMeshStandardMaterial
        :color="card.color"
        :roughness="0.3"
        :metalness="0.1"
      />
    </TresMesh>
  </TresGroup>
  
  <!-- Ground plane for shadows -->
  <TresMesh
    :position="[0, -0.1, 0]"
    :rotation="[-Math.PI / 2, 0, 0]"
    receive-shadow
  >
    <TresPlaneGeometry :args="[20, 20]" />
    <TresMeshStandardMaterial
      color="#1a1a2e"
      :roughness="0.8"
    />
  </TresMesh>
  
  <TresGridHelper :args="[10, 10, 0x333333, 0x333333]" />
</template>