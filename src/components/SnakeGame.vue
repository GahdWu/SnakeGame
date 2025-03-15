<script setup>
import {onMounted, onUnmounted, ref} from 'vue'

//地图尺寸
const mapWidth = 400
const mapHeight = 400
//方块尺寸
const unitWidth = 20
const unitHeight = 20

//方向
const DIRECTION_UP = 1
const DIRECTION_DOWN = 2
const DIRECTION_LEFT = 3
const DIRECTION_RIGHT = 4

const gameCanvas = ref(null)
const context = ref(null)

const isGameOver = ref(true)
const snake = ref([{x: 10, y: 10}])
const food = ref([])
const score = ref(0)
const direction = ref(DIRECTION_LEFT)
const forbidDirection = ref(DIRECTION_RIGHT)
const intervalId = ref(0)
const gameLoopId = ref(0)
const frameInterval = ref(500)
const level = ref(0)
const levels = ref([
  {level: 1, interval: 500, score: 1},
  {level: 2, interval: 450, score: 20},
  {level: 3, interval: 400, score: 30},
  {level: 4, interval: 350, score: 40},
  {level: 5, interval: 300, score: 50},
])

//挂载
onMounted(() => {
  initGame()
  window.addEventListener("keydown", onKeyDown)
})

//按下按键
const onKeyDown = (e) => {
  if (e.code === "KeyW" && forbidDirection.value !== DIRECTION_UP) {
    direction.value = DIRECTION_UP
  } else if (e.code === "KeyS" && forbidDirection.value !== DIRECTION_DOWN) {
    direction.value = DIRECTION_DOWN
  } else if (e.code === "KeyA" && forbidDirection.value !== DIRECTION_LEFT) {
    direction.value = DIRECTION_LEFT
  } else if (e.code === "KeyD" && forbidDirection.value !== DIRECTION_RIGHT) {
    direction.value = DIRECTION_RIGHT
  } else if (e.code === "Space" && isGameOver.value) {
    startGame()
  }
}

//卸载
onUnmounted(() => {
  window.removeEventListener("keydown", onKeyDown)
})

//初始化游戏
const initGame = () => {
  context.value = gameCanvas.value.getContext('2d')
}

const startGame = () => {
  console.log("start")
  snake.value = [{x: 10 * unitWidth, y: 10 * unitHeight}]
  score.value = 0
  frameInterval.value = 500
  food.value = []
  direction.value = DIRECTION_LEFT
  randFood()
  randFood()
  randFood()
  isGameOver.value = false
  updateGame()
}

const updateGame = () => {
  if (isGameOver.value) {
    return
  }

  moveSnake()
  checkCollision()
  drawGame()

  const levelInfo = levels.value[level.value]
  setTimeout(() => {
    gameLoopId.value = requestAnimationFrame(updateGame)
  }, levelInfo.interval)
}

const moveSnake = () => {
  if (direction.value === DIRECTION_RIGHT) {
    forbidDirection.value = DIRECTION_LEFT
  } else if (direction.value === DIRECTION_LEFT) {
    forbidDirection.value = DIRECTION_RIGHT
  } else if (direction.value === DIRECTION_UP) {
    forbidDirection.value = DIRECTION_DOWN
  } else if (direction.value === DIRECTION_DOWN) {
    forbidDirection.value = DIRECTION_UP
  }

  // console.log("move")
  let snakeHead = null
  let lastSnakeBody = {x: 0, y: 0}
  for (let i = 0; i < snake.value.length; i++) {
    const snakeBody = snake.value[i]
    if (i === 0) {
      lastSnakeBody.x = snakeBody.x
      lastSnakeBody.y = snakeBody.y
      snakeHead = snakeBody
      if (direction.value === DIRECTION_UP) {
        if (snakeBody.y - unitHeight < 0) {
          gameOver()
          return
        }
        snakeBody.y -= unitHeight
      }

      if (direction.value === DIRECTION_DOWN) {
        if (snakeBody.y + unitHeight >= mapHeight) {
          gameOver()
          return
        }
        snakeBody.y += unitHeight
      }

      if (direction.value === DIRECTION_LEFT) {
        if (snakeBody.x - unitWidth < 0) {
          gameOver()
          return;
        }
        snakeBody.x -= unitWidth
      }

      if (direction.value === DIRECTION_RIGHT) {
        if (snakeBody.x + unitWidth >= mapWidth) {
          gameOver()
          return
        }
        snakeBody.x += unitWidth
      }
    } else {
      const currX = snakeBody.x
      const currY = snakeBody.y
      snakeBody.x = lastSnakeBody.x
      snakeBody.y = lastSnakeBody.y
      lastSnakeBody.x = currX
      lastSnakeBody.y = currY
    }
  }

  //是否吃到食物
  for (let i = 0; i < food.value.length; i++) {
    if (food.value[i].x === snakeHead.x && food.value[i].y === snakeHead.y) {
      score.value += 1
      if (score.value >= levels.value[level.value].score) {
        if (level.value < levels.value.length) {
          level.value += 1
          gameOver()
          startGame()
          return
        }
      }
      // frameInterval.value = frameInterval.value < 100 ? 100 : frameInterval.value - 10
      snake.value.push({x: lastSnakeBody.x, y: lastSnakeBody.y})
      randFood()
      food.value.splice(i, 1)
      break
    }
  }
}

//随机食物
const randFood = () => {
  for (let i = 0; i < mapWidth * mapHeight; i++) {
    const x = Math.floor(Math.random() * ((mapWidth / unitWidth - 1))) * unitWidth
    const y = Math.floor(Math.random() * ((mapHeight - unitHeight) / unitHeight)) * unitHeight

    let isCover = false
    for (let j = 0; j < snake.value.length; j++) {
      if (snake.value[j].x === x && snake.value[j].y === y) {
        isCover = true
        break
      }
    }

    if (isCover) {
      continue
    }

    for (let j = 0; j < food.value.length; j++) {
      if (food.value[j].x === x && food.value[j].y === y) {
        isCover = true
        break
      }
    }

    if (isCover) {
      continue
    }

    food.value.push({x: x, y: y})
    break
  }
}

const checkCollision = () => {
  const snakeHead = snake.value[0]
  for (let i = 1; i < snake.value.length; i++) {
    if (snakeHead.x === snake.value[i].x && snakeHead.y === snake.value[i].y) {
      gameOver()
      return
    }
  }
}

const drawGame = () => {
  // console.log("draw")
  const cxt = context.value

  //清理画布
  cxt.clearRect(0, 0, mapWidth, mapHeight)

  let r = 0
  let g = 100
  let b = 0
  for (let i = 0; i < snake.value.length; i++) {
    const body = snake.value[i]
    cxt.fillStyle = `rgb(${r}, ${g}, ${b})`
    cxt.fillRect(body.x, body.y, unitWidth, unitHeight)

    g += 20
    if (g >= 230) {
      g = 230
      r += 20
      b += 20
      r = r > 230 ? 230 : r
      b = b > 230 ? 230 : b
    }
  }

  cxt.fillStyle = `red`
  for (let i = 0; i < food.value.length; i++) {
    cxt.fillRect(food.value[i].x, food.value[i].y, unitWidth, unitHeight)
  }
  // console.log("draw")
}

const gameOver = () => {
  isGameOver.value = true
  console.log("game over")
  if (intervalId.value > 0) {
    clearInterval(intervalId.value)
    intervalId.value = 0
  }

  if (gameLoopId.value > 0) {
    cancelAnimationFrame(gameLoopId.value)
    gameLoopId.value = 0
  }
}

</script>

<template>
  <div>
    <p v-if="isGameOver">Game Over!!!Press Space to start</p>
    <p v-else>Level:{{ levels[level].level }} Score:{{ score }}</p>
    <canvas ref="gameCanvas" :width="mapWidth" :height="mapHeight"></canvas>
  </div>
</template>

<style scoped>
canvas {
  border: 1px solid #181818;
  display: block;
  margin: 0 auto;
}

p {
  text-align: center;
  font-size: 20px;
}
</style>
