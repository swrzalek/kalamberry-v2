<script setup lang="ts">
import { RoundedBoxGeometry } from 'three/examples/jsm/geometries/RoundedBoxGeometry.js'

// Card positions and rotations
const cards = [
  {
    position: [0, 0, 0] as [number, number, number],
    rotation: [Math.PI / 2, 0, 0] as [number, number, number],
    color: '#4ecdc4'
  },
  {
    position: [0, 0, -0.25] as [number, number, number],
    rotation: [Math.PI / 2, 0.15, 0] as [number, number, number],
    color: '#ff6b6b'
  }
]

// Create rounded box geometry for the card (width, thickness, height)
const roundedGeometry = new RoundedBoxGeometry(2.5, 0.2, 3.5, 3, 0.08)
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
  
  <!-- Cards with rounded corners -->
  <TresMesh
    v-for="(card, index) in cards"
    :key="index"
    :position="card.position"
    :rotation="card.rotation"
    cast-shadow
    :geometry="roundedGeometry"
  >
    <TresMeshStandardMaterial
      :color="card.color"
      :roughness="0.4"
      :metalness="0.2"
    />
  </TresMesh>
</template>