<template>
  <div
    v-if="showStartButton"
    class="flex justify-center items-center h-screen overflow-hidden w-screen mb-30"
  >
    <button @click="handleStart" class="px-6 py-3 bg-blue-500 text-white rounded">Play Game</button>
  </div>

  <div
    v-else
    class="relative perspective-1000 h-screen overflow-hidden w-screen flex justify-center items-center"
  >
    <img :src="runDogRun" alt="Running Dog" class="absolute bottom-8 object-cover h-[50px]" />

    <div
      v-for="(card, index) in cards"
      :key="index"
      class="card"
      :style="cardStyle(index)"
      ref="cardRefs"
    >
      <div
        class="card-inner transition-transform duration-700 ease-out"
        :class="{ flipped: cardFlipped(index) }"
        @click="pickCard(index)"
      >
        <div class="card-face card-front">
          <img :src="cardBack" class="w-full h-full object-cover" />
        </div>

        <div class="card-face card-back">
          <img :src="card.isWinner ? winJoker : loseJoker" class="w-full h-full object-cover" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue"
import confetti from "canvas-confetti"

import winJoker from "../assets/win-joker.jpg"
import loseJoker from "../assets/lose-joker.jpg"
import cardBack from "../assets/card-back.png"
import runDogRun from "../assets/run-dog-run.gif"

const showStartButton = ref(true)
const canPick = ref(false)
const firstGame = ref(true)

const cards = ref([
  { id: 0, isWinner: false },
  { id: 1, isWinner: false },
  { id: 2, isWinner: false },
])

const flippedStates = ref([false, false, false])

const positions = ref([
  { x: 0, y: 0 },
  { x: 0, y: 0 },
  { x: 0, y: 0 },
])

const nextRandom = ref(null)

const audio = new Audio(new URL("../assets/creepy.mp3", import.meta.url))
audio.loop = true

function preloadImages() {
  ;[winJoker, loseJoker].forEach((src) => {
    const img = new Image()
    img.src = src
  })
}

function shuffle(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[array[i], array[j]] = [array[j], array[i]]
  }
}

const wait = (ms) => new Promise((r) => setTimeout(r, ms))

function cardFlipped(index) {
  return flippedStates.value[index]
}

function randomTriangle() {
  const leftX = -80 - Math.random() * 60
  const rightX = 80 + Math.random() * 60
  const topY = -40 - Math.random() * 20
  const bottomY = 40 + Math.random() * 20

  return [
    { x: leftX, y: bottomY },
    { x: 0, y: topY },
    { x: rightX, y: bottomY },
  ]
}

function randomScramble() {
  return Array.from({ length: 3 }, () => ({
    x: (Math.random() - 0.5) * 300,
    y: (Math.random() - 0.5) * 120,
  }))
}

async function fetchWinner() {
  try {
    const res = await fetch(
      "https://www.random.org/integers/?num=1&min=0&max=2&col=1&base=10&format=plain&rnd=new"
    )
    nextRandom.value = parseInt(await res.text(), 10)
  } catch {
    nextRandom.value = Math.floor(Math.random() * 3)
  }
}

async function shuffleAnimation() {
  positions.value = randomTriangle()
  await wait(800)

  positions.value = randomScramble()
  await wait(800)

  positions.value = [
    { x: -130, y: 0 },
    { x: 0, y: 0 },
    { x: 130, y: 0 },
  ]
  await wait(800)
}

async function firstGameReveal() {
  canPick.value = false

  await fetchWinner()
  cards.value.forEach((c) => (c.isWinner = false))
  cards.value[nextRandom.value].isWinner = true

  flippedStates.value = [true, true, true]
  await wait(2000)

  flippedStates.value = [false, false, false]
  await wait(700)

  shuffle(cards.value)
  await shuffleAnimation()
  canPick.value = true
}

async function newRound() {
  canPick.value = false
  flippedStates.value = [false, false, false]

  await fetchWinner()
  cards.value.forEach((c) => (c.isWinner = false))
  cards.value[nextRandom.value].isWinner = true

  shuffle(cards.value)
  await shuffleAnimation()
  canPick.value = true
}

function pickCard(index) {
  if (!canPick.value) return
  canPick.value = false

  flippedStates.value = flippedStates.value.map(() => true)

  if (cards.value[index].isWinner) {
    const cardEl = document.getElementsByClassName("card")[index]
    const rect = cardEl.getBoundingClientRect()

    confetti({
      particleCount: 80,
      spread: 70,
      origin: {
        x: (rect.left + rect.width / 2) / window.innerWidth,
        y: (rect.top + rect.height / 2) / window.innerHeight,
      },
    })
  }

  setTimeout(async () => {
    flippedStates.value = [false, false, false]
    await wait(700)
    newRound()
  }, 1500)
}

async function handleStart() {
  showStartButton.value = false
  audio.play()

  positions.value = [
    { x: -130, y: 0 },
    { x: 0, y: 0 },
    { x: 130, y: 0 },
  ]

  if (firstGame.value) {
    firstGame.value = false
    await firstGameReveal()
  } else {
    newRound()
  }
}

onMounted(() => {
  preloadImages()
})

function cardStyle(index) {
  const { x, y } = positions.value[index]
  return {
    position: "absolute",
    width: "96px",
    height: "144px",
    left: "50%",
    top: "42%",
    transform: `translate(-50%, -50%) translateX(${x}px) translateY(${y}px)`,
  }
}
</script>

<style scoped>
.perspective-1000 {
  perspective: 1000px;
}

.card-inner {
  width: 100%;
  height: 100%;
  transform-style: preserve-3d;
  position: relative;
  cursor: pointer;
}

.card-inner.flipped {
  transform: rotateY(180deg);
}

.card-face {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 6px;
}

.card-front {
  transform: rotateY(0deg);
  z-index: 2;
}

.card-back {
  transform: rotateY(180deg);
}

.card {
  transition: transform 0.8s ease-in-out;
  border-radius: 6px;
  overflow: hidden;
}
</style>
