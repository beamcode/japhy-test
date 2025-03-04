<template>
  <div
    class="relative perspective-1000 h-screen overflow-hidden w-[800px] items-center flex justify-center"
  >
    <img
      src="../../src/assets/run-dog-run.gif"
      alt="Running Dog"
      class="absolute bottom-8 object-cover h-[50px]"
    />
    <div
      v-for="(card, index) in cards"
      :key="card.id"
      :ref="(el) => (cardRefs[index] = el)"
      class="card absolute transition-transform duration-500 flip-3d cursor-pointer"
      :style="getCardStyle(index)"
      @click="pickCard(index)"
    >
      <div
        class="absolute size-full inset-0 rounded-lg backface-hidden flex items-center justify-center bg-gray-300"
      >
        <img src="../../src/assets/card-back.png" alt="Front" class="w-full h-full object-cover" />
      </div>
      <div
        class="absolute size-full inset-0 rounded-lg backface-hidden flex items-center justify-center rotate-y-180 bg-cover bg-center"
        :class="{
          'bg-[url(../../src/assets/win-joker.png)]':
            showResult && pickedCard === index && isWinner,
          'bg-[url(../../src/assets/lose-joker.png)]':
            showResult && pickedCard === index && !isWinner,
        }"
      ></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue"
import confetti from "canvas-confetti"

// cards array
const cards = ref([{ id: 0 }, { id: 1 }, { id: 2 }])
const cardRefs = ref([])

// states
const pickedCard = ref(null)
const correctCard = ref(null)
const cardFlipped = ref(false)
const showResult = ref(false)
const isWinner = ref(false)
const canPick = ref(true)

// anims
const isShuffling = ref(false)
const offsets = [-130, 0, 130]

// creepy sound loop
const audio = new Audio(new URL("../../src/assets/creepy.mp3", import.meta.url))
audio.loop = true
// Ensure the audio truly loops by restarting it when it ends
audio.addEventListener("ended", () => {
  audio.currentTime = 0
  audio.play()
})

// flag to ensure audio starts only once
const audioStarted = ref(false)

// cards style
const getCardStyle = (index) => {
  const flipDeg = cardFlipped.value && pickedCard.value === index ? 180 : 0
  const offsetX = isShuffling.value ? 0 : offsets[index]
  return {
    left: "50%",
    top: "42%",
    transform: `translate(-50%, -50%) translateX(${offsetX}px) rotateY(${flipDeg}deg)`,
  }
}

// api call for the random number
const nextRandom = ref(null)
const fetchRandomCard = async () => {
  try {
    const response = await fetch(
      "https://www.random.org/integers/?num=1&min=0&max=2&col=1&base=10&format=plain&rnd=new"
    )
    nextRandom.value = parseInt(await response.text(), 10)
  } catch (e) {
    nextRandom.value = Math.floor(Math.random() * 3)
  }
}

onMounted(() => {
  fetchRandomCard()
})

// card picker
const pickCard = (index) => {
  if (!canPick.value) return
  canPick.value = false

  if (!audioStarted.value) {
    audio.play()
    audioStarted.value = true
  }

  correctCard.value = nextRandom.value
  pickedCard.value = index
  isWinner.value = index === correctCard.value
  showResult.value = true
  cardFlipped.value = true

  if (isWinner.value) {
    const rect = cardRefs.value[index].getBoundingClientRect()
    confetti({
      particleCount: 100,
      spread: 70,
      origin: {
        x: (rect.left + rect.width / 2) / window.innerWidth,
        y: (rect.top + rect.height / 2) / window.innerHeight,
      },
    })
  }

  setTimeout(
    () => {
      cardFlipped.value = false
      setTimeout(() => {
        resetGame()
      }, 500)
    },
    isWinner.value ? 3000 : 1000
  )
}

const resetGame = () => {
  pickedCard.value = null
  correctCard.value = null
  showResult.value = false
  isWinner.value = false

  fetchRandomCard().finally(() => {
    canPick.value = true
  })

  startShuffle()
}

const startShuffle = () => {
  isShuffling.value = true
  setTimeout(() => {
    isShuffling.value = false
  }, 1000)
}
</script>

<style scoped>
.perspective-1000 {
  perspective: 1000px;
}
.flip-3d {
  transform-style: preserve-3d;
}
.backface-hidden {
  backface-visibility: hidden;
}
.card {
  width: 96px;
  height: 144px;
}
</style>
