import { start } from 'repl';
<script setup lang="ts">
interface Point {
  x: number
  y: number
}

interface Branch {
  start: Point
  length: number
  angle: number
}

const el = $ref<HTMLCanvasElement>()
const ctx = $computed(() => el.getContext('2d')!)

const WIDTH = 600
const HEIGHT = 600
const pendingTasks: Function[] = []
let frameCounts = 0

function lineTo(p1: Point, p2: Point) {
  ctx.beginPath()
  ctx.moveTo(p1.x, p1.y)
  ctx.lineTo(p2.x, p2.y)
  ctx.stroke()
}

function getEndPoint(b: Branch) {
  return {
    x: b.start.x + b.length * Math.cos(b.angle),
    y: b.start.y + b.length * Math.sin(b.angle),
  }
}

function drawBranch(b: Branch) {
  lineTo(b.start, getEndPoint(b))
}

function step(b: Branch, depth = 0) {
  const end = getEndPoint(b)
  drawBranch(b)
  if (depth < 4 || Math.random() < 0.5) {
    pendingTasks.push(() => step({
      start: end,
      length: b.length + (Math.random() * 10 - 5),
      angle: b.angle - 0.3 * Math.random(),
    }, depth + 1))
  }
  if (depth < 4 || Math.random() < 0.5) {
    pendingTasks.push(() => step({
      start: end,
      length: b.length + (Math.random() * 10 - 5),
      angle: b.angle + 0.3 * Math.random(),
    }, depth + 1))
  }
}

function frame() {
  const tasks = [...pendingTasks]
  pendingTasks.length = 0
  tasks.forEach(fn => fn())
}

function startFrame() {
  requestAnimationFrame(() => {
    frameCounts += 1
    if (frameCounts % 3 === 0)
      frame()
    startFrame()
  })
}

startFrame()

function init() {
  ctx.strokeStyle = 'black'
  step({
    start: { x: WIDTH / 2, y: HEIGHT },
    length: 40,
    angle: -Math.PI / 2,
  })
}

onMounted(() => {
  init()
})
</script>

<template>
  <canvas ref="el" height="600" width="600" border />
</template>
