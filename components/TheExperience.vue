<script setup lang="ts">
import { RoundedBoxGeometry } from 'three/examples/jsm/geometries/RoundedBoxGeometry.js'
import type { Group } from 'three'
import { Html } from '@tresjs/cientos'

// ============================================================================
// TYPES & INTERFACES
// ============================================================================

interface Vector3Tuple {
  x: number
  y: number
  z: number
}

interface CardConfig {
  ref: Ref<Group | null>
  color: string
  basePosition: [number, number, number]
  baseRotation: [number, number, number]
  wordIndex: number
}

interface AnimationPhaseResult {
  position: Vector3Tuple
  rotation: { y: number; z: number }
}

// ============================================================================
// CONSTANTS
// ============================================================================

const CARD_DIMENSIONS = {
  WIDTH: 2.5,
  THICKNESS: 0.2,
  HEIGHT: 3.5,
  SEGMENTS: 3,
  RADIUS: 0.08,
} as const

const CARD_SPACING = {
  MIDDLE: -0.18,
  BACK: -0.36,
} as const

const ANIMATION = {
  SPEED_MULTIPLIER: 1.5,
  TRANSITION_SMOOTH_FACTOR: 10,
  TEXT_FADE_OUT_DELAY: 500,
  TEXT_SHOW_PROGRESS: 0.6, // Show new text when card is 60% through animation
  PHASE_SPLIT: 0.5,
} as const

const CARD_POSITIONS = {
  FRONT: 0,
  MIDDLE: 1,
  BACK: 2,
} as const

const ARC_ANIMATION = {
  PHASE1: {
    X_DISTANCE: 6,
    Y_HEIGHT: 1.5,
    Z_FORWARD: 1,
    ROTATION_Y: 1.2,
    ROTATION_Z: 0.3,
  },
  PHASE2: {
    X_POWER: 1.5,
    Y_POWER: 2,
    Z_TOTAL: 1.36,
    ROTATION_Y_TOTAL: 1.35,
  },
} as const

const CARD_ROTATIONS = {
  BASE_X: Math.PI / 2,
  MIDDLE: 0.15,
  BACK: -0.15,
} as const

const CAMERA = {
  POSITION: [0, 0, 6] as [number, number, number],
  FOV: 50,
  LOOK_AT: [0, 0, 0] as [number, number, number],
} as const

const ORBIT_CONTROLS = {
  DAMPING_FACTOR: 0.05,
  ENABLE_DAMPING: true,
  ROTATION_SPEED: 0.5,
  MAX_POLAR_ANGLE: Math.PI / 2 + 0.3,
  MIN_POLAR_ANGLE: Math.PI / 2 - 0.3,
  MAX_AZIMUTH_ANGLE: 0.3,
  MIN_AZIMUTH_ANGLE: -0.3,
  ENABLE_ZOOM: false,
  ENABLE_PAN: false,
  SPRING_BACK_SPEED: 0.08,
} as const

const LIGHTING = {
  AMBIENT_INTENSITY: 0.8,
  PRIMARY_INTENSITY: 1.2,
  SECONDARY_INTENSITY: 0.5,
  PRIMARY_POSITION: [5, 8, 5] as [number, number, number],
  SECONDARY_POSITION: [-3, 5, -3] as [number, number, number],
} as const

const CARD_COLORS = {
  CARD_1: '#9333ea', // Rich Purple 600
  CARD_2: '#a855f7', // Rich Purple 500
  CARD_3: '#c084fc', // Rich Purple 400
} as const

const MATERIAL_PROPERTIES = {
  ROUGHNESS: 0.4,
  METALNESS: 0.2,
} as const

const HTML_TEXT = {
  POSITION: [0, 0, 0.15] as [number, number, number],
  ROTATION: [-Math.PI / 2, 0, 0] as [number, number, number],
  DISTANCE_FACTOR: 0.5,
  VISIBILITY_Z_THRESHOLD: 0.1,
} as const

const WORDS = [
  'Hello',
  'World',
  'Vue',
  'TresJS',
  'Three.js',
  'Animation',
  'Cards',
  'Deck',
  'Shuffle',
  'Amazing',
] as const

const INITIAL_VISIBLE_CARDS_COUNT = 3

// ============================================================================
// COMPOSABLES & UTILS
// ============================================================================

const { onBeforeRender } = useLoop()

/**
 * Easing function for smooth cubic ease-in-out animation
 * @param t - Progress value between 0 and 1
 * @returns Eased progress value
 */
const easeInOutCubic = (t: number): number => {
  return t < ANIMATION.PHASE_SPLIT
    ? 4 * t * t * t
    : 1 - Math.pow(-2 * t + 2, 3) / 2
}

/**
 * Normalizes raw progress to a value between 0 and 1
 */
const normalizeProgress = (rawProgress: number): number => {
  return Math.min(rawProgress, 1)
}

/**
 * Calculates phase 1 animation (card moving right and up)
 */
const calculatePhase1Animation = (rawProgress: number): AnimationPhaseResult => {
  const phase1Raw = rawProgress * 2
  const phase1Progress = easeInOutCubic(phase1Raw)

  return {
    position: {
      x: phase1Progress * ARC_ANIMATION.PHASE1.X_DISTANCE,
      y: phase1Progress * ARC_ANIMATION.PHASE1.Y_HEIGHT,
      z: phase1Progress * ARC_ANIMATION.PHASE1.Z_FORWARD,
    },
    rotation: {
      y: phase1Progress * ARC_ANIMATION.PHASE1.ROTATION_Y,
      z: phase1Progress * ARC_ANIMATION.PHASE1.ROTATION_Z,
    },
  }
}

/**
 * Calculates phase 2 animation (card arcing behind the deck)
 */
const calculatePhase2Animation = (rawProgress: number): AnimationPhaseResult => {
  const phase2Raw = (rawProgress - ANIMATION.PHASE_SPLIT) * 2
  const phase2Progress = easeInOutCubic(phase2Raw)

  const xArc = ARC_ANIMATION.PHASE1.X_DISTANCE - Math.pow(phase2Progress, ARC_ANIMATION.PHASE2.X_POWER) * ARC_ANIMATION.PHASE1.X_DISTANCE
  const yArc = ARC_ANIMATION.PHASE1.Y_HEIGHT * (1 - Math.pow(phase2Progress, ARC_ANIMATION.PHASE2.Y_POWER))
  const zArc = ARC_ANIMATION.PHASE1.Z_FORWARD - phase2Progress * ARC_ANIMATION.PHASE2.Z_TOTAL

  return {
    position: {
      x: xArc,
      y: yArc,
      z: zArc,
    },
    rotation: {
      y: ARC_ANIMATION.PHASE1.ROTATION_Y - phase2Progress * ARC_ANIMATION.PHASE2.ROTATION_Y_TOTAL,
      z: ARC_ANIMATION.PHASE1.ROTATION_Z - phase2Progress * ARC_ANIMATION.PHASE1.ROTATION_Z,
    },
  }
}

/**
 * Applies animation result to a card reference
 */
const applyAnimationToCard = (cardRef: Group, animation: AnimationPhaseResult): void => {
  cardRef.position.x = animation.position.x
  cardRef.position.y = animation.position.y
  cardRef.position.z = animation.position.z
  cardRef.rotation.y = animation.rotation.y
  cardRef.rotation.z = animation.rotation.z
}

/**
 * Smoothly interpolates a value towards a target
 */
const smoothLerp = (current: number, target: number, delta: number): number => {
  return current + (target - current) * delta * ANIMATION.TRANSITION_SMOOTH_FACTOR
}

// ============================================================================
// STATE MANAGEMENT
// ============================================================================

const nextWordIndex = ref(INITIAL_VISIBLE_CARDS_COUNT)
const isAnimating = ref(false)
const animationProgress = ref(0)
const showText = ref(true)
const showNewText = ref(false)

// Card refs - using shallowRef as recommended by TresJS docs
const card1Ref = shallowRef<Group | null>(null)
const card2Ref = shallowRef<Group | null>(null)
const card3Ref = shallowRef<Group | null>(null)

const allCardRefs = [card1Ref, card2Ref, card3Ref] as const

// Camera and controls refs
const cameraRef = shallowRef(null)
const orbitControlsRef = shallowRef(null)

// Track if user is currently interacting
const isInteracting = ref(false)

// Card configuration
const cards: CardConfig[] = [
  {
    ref: card1Ref,
    color: CARD_COLORS.CARD_1,
    basePosition: [0, 0, 0],
    baseRotation: [CARD_ROTATIONS.BASE_X, 0, 0],
    wordIndex: 0,
  },
  {
    ref: card2Ref,
    color: CARD_COLORS.CARD_2,
    basePosition: [0, 0, CARD_SPACING.MIDDLE],
    baseRotation: [CARD_ROTATIONS.BASE_X, CARD_ROTATIONS.MIDDLE, 0],
    wordIndex: 1,
  },
  {
    ref: card3Ref,
    color: CARD_COLORS.CARD_3,
    basePosition: [0, 0, CARD_SPACING.BACK],
    baseRotation: [CARD_ROTATIONS.BASE_X, CARD_ROTATIONS.BACK, 0],
    wordIndex: 2,
  },
]

// Track which card is in which position (front=0, middle=1, back=2)
const cardOrder = ref([
  CARD_POSITIONS.FRONT,
  CARD_POSITIONS.MIDDLE,
  CARD_POSITIONS.BACK,
])

// Reactive word assignments
const cardWords = ref<string[]>([WORDS[0], WORDS[1], WORDS[2]])

// ============================================================================
// COMPUTED PROPERTIES
// ============================================================================

const createCardWordComputed = (cardIndex: 0 | 1 | 2) => {
  return computed(() => {
    const positionIndex = cardOrder.value.indexOf(cardIndex)
    return cardWords.value[positionIndex] ?? ''
  })
}

const card1Word = createCardWordComputed(0)
const card2Word = createCardWordComputed(1)
const card3Word = createCardWordComputed(2)

/**
 * Computed properties to show text only on the front card
 * During animation (after text show point), show text on the middle card that's moving to front
 */
const showCard1Text = computed(() => {
  // During animation, show new text on middle card (incoming card)
  if (isAnimating.value && showNewText.value) {
    return cardOrder.value[CARD_POSITIONS.MIDDLE] === 0
  }
  
  // Normal state: show text on front card
  if (!isAnimating.value && showText.value) {
    return cardOrder.value[CARD_POSITIONS.FRONT] === 0
  }
  
  return false
})

const showCard2Text = computed(() => {
  // During animation, show new text on middle card (incoming card)
  if (isAnimating.value && showNewText.value) {
    return cardOrder.value[CARD_POSITIONS.MIDDLE] === 1
  }
  
  // Normal state: show text on front card
  if (!isAnimating.value && showText.value) {
    return cardOrder.value[CARD_POSITIONS.FRONT] === 1
  }
  
  return false
})

const showCard3Text = computed(() => {
  // During animation, show new text on middle card (incoming card)
  if (isAnimating.value && showNewText.value) {
    return cardOrder.value[CARD_POSITIONS.MIDDLE] === 2
  }
  
  // Normal state: show text on front card
  if (!isAnimating.value && showText.value) {
    return cardOrder.value[CARD_POSITIONS.FRONT] === 2
  }
  
  return false
})

// ============================================================================
// GEOMETRY
// ============================================================================

const roundedGeometry = new RoundedBoxGeometry(
  CARD_DIMENSIONS.WIDTH,
  CARD_DIMENSIONS.THICKNESS,
  CARD_DIMENSIONS.HEIGHT,
  CARD_DIMENSIONS.SEGMENTS,
  CARD_DIMENSIONS.RADIUS,
)

// ============================================================================
// CARD ANIMATION LOGIC
// ============================================================================

/**
 * Initializes all cards to their base positions
 */
const initializeCardPositions = (): void => {
  cards.forEach((card, index) => {
    if (card.ref.value) {
      card.ref.value.position.set(...card.basePosition)
      card.ref.value.rotation.set(...card.baseRotation)
    }
  })
}

/**
 * Animates the front card through its arc path
 */
const animateFrontCard = (frontCardRef: Group, rawProgress: number): void => {
  const animation = rawProgress < ANIMATION.PHASE_SPLIT
    ? calculatePhase1Animation(rawProgress)
    : calculatePhase2Animation(rawProgress)

  applyAnimationToCard(frontCardRef, animation)
}

/**
 * Animates the middle card moving forward
 */
const animateMiddleCard = (middleCardRef: Group, rawProgress: number, delta: number): void => {
  const targetZ = rawProgress < ANIMATION.PHASE_SPLIT
    ? CARD_SPACING.MIDDLE
    : CARD_SPACING.MIDDLE + easeInOutCubic((rawProgress - ANIMATION.PHASE_SPLIT) * 2) * Math.abs(CARD_SPACING.MIDDLE)

  const targetRotY = rawProgress < ANIMATION.PHASE_SPLIT
    ? CARD_ROTATIONS.MIDDLE
    : CARD_ROTATIONS.MIDDLE - easeInOutCubic((rawProgress - ANIMATION.PHASE_SPLIT) * 2) * CARD_ROTATIONS.MIDDLE

  middleCardRef.position.z = smoothLerp(middleCardRef.position.z, targetZ, delta)
  middleCardRef.rotation.y = smoothLerp(middleCardRef.rotation.y, targetRotY, delta)
}

/**
 * Animates the back card moving forward
 */
const animateBackCard = (backCardRef: Group, rawProgress: number, delta: number): void => {
  const targetZ = rawProgress < ANIMATION.PHASE_SPLIT
    ? CARD_SPACING.BACK
    : CARD_SPACING.BACK + easeInOutCubic((rawProgress - ANIMATION.PHASE_SPLIT) * 2) * Math.abs(CARD_SPACING.MIDDLE)

  const targetRotY = rawProgress < ANIMATION.PHASE_SPLIT
    ? CARD_ROTATIONS.BACK
    : CARD_ROTATIONS.BACK + easeInOutCubic((rawProgress - ANIMATION.PHASE_SPLIT) * 2) * (CARD_ROTATIONS.MIDDLE + Math.abs(CARD_ROTATIONS.BACK))

  backCardRef.position.z = smoothLerp(backCardRef.position.z, targetZ, delta)
  backCardRef.rotation.y = smoothLerp(backCardRef.rotation.y, targetRotY, delta)
}

/**
 * Resets animation and updates card order
 */
const completeAnimation = (): void => {
    // Rotate the order: front card goes to back
    const frontCard = cardOrder.value.shift()!
    cardOrder.value.push(frontCard)
    
    // Set final positions for all cards based on new order
    cardOrder.value.forEach((cardIndex, positionIndex) => {
    const cardRef = allCardRefs[cardIndex]
      if (cardRef?.value && cards[positionIndex]) {
        cardRef.value.position.set(...cards[positionIndex].basePosition)
        cardRef.value.rotation.set(...cards[positionIndex].baseRotation)
      }
    })
    
    // Update card words array AFTER card order changes
    cardWords.value.shift()
    const nextWord = WORDS[nextWordIndex.value]
    if (nextWord) {
      cardWords.value.push(nextWord)
    }
    nextWordIndex.value = (nextWordIndex.value + 1) % WORDS.length

  // Reset animation state
    isAnimating.value = false
    animationProgress.value = 0
    showNewText.value = false
    showText.value = true
}

/**
 * Gets the card ref at a specific position in the order
 */
const getCardAtPosition = (position: number): Group | null => {
  const cardIndex = cardOrder.value[position]
  return cardIndex !== undefined ? allCardRefs[cardIndex]?.value ?? null : null
}

// ============================================================================
// LIFECYCLE HOOKS
// ============================================================================

/**
 * Handles when user starts dragging
 */
const onControlsStart = (): void => {
  isInteracting.value = true
}

/**
 * Handles when user stops dragging
 */
const onControlsEnd = (): void => {
  isInteracting.value = false
}

onMounted(() => {
  initializeCardPositions()

  // Set up orbit controls event listeners after next tick
  nextTick(() => {
    if (orbitControlsRef.value) {
      const controls = orbitControlsRef.value as any
      if (controls.addEventListener) {
        controls.addEventListener('start', onControlsStart)
        controls.addEventListener('end', onControlsEnd)
      }
    }
  })
})

onUnmounted(() => {
  // Clean up event listeners
  if (orbitControlsRef.value) {
    const controls = orbitControlsRef.value as any
    if (controls.removeEventListener) {
      controls.removeEventListener('start', onControlsStart)
      controls.removeEventListener('end', onControlsEnd)
    }
  }
})

/**
 * Springs the camera back to default position
 */
const springCameraBack = (delta: number): void => {
  if (!cameraRef.value || isInteracting.value) return

  const camera = cameraRef.value as any
  const targetPosition = CAMERA.POSITION

  // Smoothly interpolate camera position back to default
  camera.position.x += (targetPosition[0] - camera.position.x) * ORBIT_CONTROLS.SPRING_BACK_SPEED
  camera.position.y += (targetPosition[1] - camera.position.y) * ORBIT_CONTROLS.SPRING_BACK_SPEED
  camera.position.z += (targetPosition[2] - camera.position.z) * ORBIT_CONTROLS.SPRING_BACK_SPEED

  // Reset the orbit controls target if it exists
  if (orbitControlsRef.value) {
    const controls = orbitControlsRef.value as any
    if (controls.target) {
      controls.target.x += (0 - controls.target.x) * ORBIT_CONTROLS.SPRING_BACK_SPEED
      controls.target.y += (0 - controls.target.y) * ORBIT_CONTROLS.SPRING_BACK_SPEED
      controls.target.z += (0 - controls.target.z) * ORBIT_CONTROLS.SPRING_BACK_SPEED
    }
  }
}

/**
 * Main animation loop
 */
onBeforeRender(({ delta }) => {
  // Always apply spring-back effect when not interacting
  springCameraBack(delta)

  if (!isAnimating.value) return

  animationProgress.value += delta * ANIMATION.SPEED_MULTIPLIER
  const rawProgress = normalizeProgress(animationProgress.value)

  // Show new text on middle card when it's moving into view
  if (rawProgress >= ANIMATION.TEXT_SHOW_PROGRESS && !showNewText.value) {
    showNewText.value = true
  }

  // Animate each card based on its position
  const frontCard = getCardAtPosition(CARD_POSITIONS.FRONT)
  const middleCard = getCardAtPosition(CARD_POSITIONS.MIDDLE)
  const backCard = getCardAtPosition(CARD_POSITIONS.BACK)

  if (frontCard) {
    animateFrontCard(frontCard, rawProgress)
  }

  if (middleCard) {
    animateMiddleCard(middleCard, rawProgress, delta)
  }

  if (backCard) {
    animateBackCard(backCard, rawProgress, delta)
  }

  if (rawProgress >= 1) {
    completeAnimation()
  }
})

// ============================================================================
// PUBLIC API
// ============================================================================

/**
 * Triggers the next card animation
 */
const nextCard = (): void => {
  if (isAnimating.value) return

  // Delay hiding the old text
  setTimeout(() => {
    showText.value = false
  }, ANIMATION.TEXT_FADE_OUT_DELAY)
  
  isAnimating.value = true
  animationProgress.value = 0
  showNewText.value = false
}

defineExpose({ nextCard, cardWords })
</script>

<template>
  <!-- Close-up perspective camera, like card in front of face -->
  <TresPerspectiveCamera
    ref="cameraRef"
    :position="CAMERA.POSITION"
    :fov="CAMERA.FOV"
    :look-at="CAMERA.LOOK_AT"
  />
  <OrbitControls
    ref="orbitControlsRef"
    :enable-damping="ORBIT_CONTROLS.ENABLE_DAMPING"
    :damping-factor="ORBIT_CONTROLS.DAMPING_FACTOR"
    :rotation-speed="ORBIT_CONTROLS.ROTATION_SPEED"
    :max-polar-angle="ORBIT_CONTROLS.MAX_POLAR_ANGLE"
    :min-polar-angle="ORBIT_CONTROLS.MIN_POLAR_ANGLE"
    :max-azimuth-angle="ORBIT_CONTROLS.MAX_AZIMUTH_ANGLE"
    :min-azimuth-angle="ORBIT_CONTROLS.MIN_AZIMUTH_ANGLE"
    :enable-zoom="ORBIT_CONTROLS.ENABLE_ZOOM"
    :enable-pan="ORBIT_CONTROLS.ENABLE_PAN"
  />
  
  <!-- Lighting for 3D depth -->
  <TresAmbientLight :intensity="LIGHTING.AMBIENT_INTENSITY" />
  <TresDirectionalLight
    :position="LIGHTING.PRIMARY_POSITION"
    :intensity="LIGHTING.PRIMARY_INTENSITY"
    cast-shadow
  />
  <TresDirectionalLight
    :position="LIGHTING.SECONDARY_POSITION"
    :intensity="LIGHTING.SECONDARY_INTENSITY"
  />
  
  <!-- Cards with rounded corners and text -->
  <TresGroup>
    <!-- Card 1 with text -->
    <TresGroup ref="card1Ref">
      <TresMesh cast-shadow :geometry="roundedGeometry">
        <TresMeshStandardMaterial
          :color="cards[0]?.color || CARD_COLORS.CARD_1"
          :roughness="MATERIAL_PROPERTIES.ROUGHNESS"
          :metalness="MATERIAL_PROPERTIES.METALNESS"
        />
      </TresMesh>
      <Html
        v-if="showCard1Text"
        :position="HTML_TEXT.POSITION"
        :rotation="HTML_TEXT.ROTATION"
        transform
        :distance-factor="HTML_TEXT.DISTANCE_FACTOR"
      >
        <div class="card-text">
          {{ card1Word }}
        </div>
      </Html>
    </TresGroup>
    
    <!-- Card 2 with text -->
    <TresGroup ref="card2Ref">
      <TresMesh cast-shadow :geometry="roundedGeometry">
        <TresMeshStandardMaterial
          :color="cards[1]?.color || CARD_COLORS.CARD_2"
          :roughness="MATERIAL_PROPERTIES.ROUGHNESS"
          :metalness="MATERIAL_PROPERTIES.METALNESS"
        />
      </TresMesh>
      <Html
        v-if="showCard2Text"
        :position="HTML_TEXT.POSITION"
        :rotation="HTML_TEXT.ROTATION"
        transform
        :distance-factor="HTML_TEXT.DISTANCE_FACTOR"
      >
        <div class="card-text">
          {{ card2Word }}
        </div>
      </Html>
    </TresGroup>
    
    <!-- Card 3 with text -->
    <TresGroup ref="card3Ref">
      <TresMesh cast-shadow :geometry="roundedGeometry">
        <TresMeshStandardMaterial
          :color="cards[2]?.color || CARD_COLORS.CARD_3"
          :roughness="MATERIAL_PROPERTIES.ROUGHNESS"
          :metalness="MATERIAL_PROPERTIES.METALNESS"
        />
      </TresMesh>
      <Html
        v-if="showCard3Text"
        :position="HTML_TEXT.POSITION"
        :rotation="HTML_TEXT.ROTATION"
        transform
        :distance-factor="HTML_TEXT.DISTANCE_FACTOR"
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
  font-size: 11rem;
  font-weight: 700;
  color: white;
  pointer-events: none;
  user-select: none;
  white-space: nowrap;
  animation: fadeIn 0.3s cubic-bezier(0.16, 1, 0.3, 1);
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Helvetica Neue', sans-serif;
  letter-spacing: 0.02em;
  text-shadow: 
    0 2px 10px rgba(0, 0, 0, 0.3),
    0 4px 20px rgba(0, 0, 0, 0.2),
    0 8px 40px rgba(0, 0, 0, 0.1);
  line-height: 1.1;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.95) translateY(10px);
    filter: blur(4px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
    filter: blur(0);
  }
}
</style>