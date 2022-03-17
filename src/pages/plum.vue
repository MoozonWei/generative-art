<template>
  <div flex flex-col items-center justify-center>
    <canvas ref="el" :width="WIDTH" :height="HEIGHT" border="~ 4 [#888] opacity-40" />
    <div>
      <button class="btn m-3 text-sm mt-8" @click="generate">Generate</button>
      <button class="btn m-3 text-sm mt-8" @click="router.back()">Back</button>
    </div>
  </div>
</template>

<script setup lang="ts">
const router = useRouter()
const el = $ref<HTMLCanvasElement>()
const [WIDTH, HEIGHT] = [600, 600]
const ctx = $computed(() => el!.getContext('2d')!)
const THETA = () => 0.4 * Math.random()
const LENGTH = () => 10 * Math.random() - 5
const BASE_DEPTH = 5
const FRAMES_PER_SEC = 12

interface Point {
  x: number
  y: number
}

interface Branch {
  startPoint: Point
  length: number
  theta: number
}

function lineTo(p1: Point, p2: Point) {
  ctx.strokeStyle = 'rgba(128, 128, 128, 0.5)'
  ctx.beginPath() // start a new path
  ctx.moveTo(p1.x, p1.y) // move the pen to (30, 50)
  ctx.lineTo(p2.x, p2.y) // draw a line to (150, 100)
  ctx.stroke()
}

function getEndPoint(b: Branch) {
  const { startPoint, length, theta } = b
  return {
    x: startPoint.x + length * Math.cos(theta),
    y: startPoint.y + length * Math.sin(theta),
  }
}

function drawBrach(b: Branch) {
  const { startPoint } = b
  const endPoint = getEndPoint(b)
  lineTo(startPoint, endPoint)
}

function init() {
  const branch = {
    startPoint: { x: WIDTH / 2, y: HEIGHT },
    length: 10 + LENGTH(),
    theta: -Math.PI / 2,
  }
  step(branch, 1)
}

const pendingTask: Function[] = []

function step(b: Branch, depth: number) {
  const end = getEndPoint(b)
  drawBrach(b)

  if (depth < BASE_DEPTH || Math.random() < 0.5) {
    pendingTask.push(() => step({
      startPoint: end,
      length: 10 + LENGTH(),
      theta: b.theta - THETA(),
    }, depth + 1))
  }
  if (depth < BASE_DEPTH || Math.random() < 0.5) {
    pendingTask.push(() => step({
      startPoint: end,
      length: 10 + LENGTH(),
      theta: b.theta + THETA(),
    }, depth + 1))
  }
}

function frame() {
  const task = [...pendingTask]
  pendingTask.length = 0
  task.forEach((fn) => {
    fn()
  })
}

let framesCount = 0
function startFrame() {
  requestAnimationFrame(() => {
    if (framesCount % (60 / FRAMES_PER_SEC) === 0)
      frame()
    framesCount++
    startFrame()
  })
}

function generate() {
  window.history.go(0)
}

startFrame()

onMounted(() => {
  init()
})
</script>

<style scoped></style>
