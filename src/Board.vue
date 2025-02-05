<script setup>
import { onMounted, ref } from 'vue'
import Cell from './components/Cell.vue'

const cell_size = ref('10')
const cells_per_side = Math.floor(500 / Number(cell_size.value))
const total_cells = cells_per_side * cells_per_side

const dead_cell_colour = ref('777777')
const live_cell_colour = ref('ffffff')

const board = ref([])
let initial_state = []

const interval_id = ref(-99)
const state_counter = ref(0)
const time_interval = ref(250)

const file_lines = ref([])

function initBoard() {
  replaceBoard([])
  for (let i = 0; i < cells_per_side; i++) {
    let row = []
    for (let j = 0; j < cells_per_side; j++) {
      row.push(0)
    }
    board.value.push(row)
  }
}

// function resetNeighbours() {
//   cell_neighbours = []
// }

// function setNeighbours(cell_id) {

//   resetNeighbours()

//   addNeighbour(cell_id, cell_id-cells_per_side-1)  // top left
//   addNeighbour(cell_id, cell_id-cells_per_side)  // top middle
//   addNeighbour(cell_id, cell_id-cells_per_side+1)  // top right

//   addNeighbour(cell_id, cell_id-1)  // middle left
//   addNeighbour(cell_id, cell_id+1)  // middle right

//   addNeighbour(cell_id, cell_id+cells_per_side-1)  // bottom left
//   addNeighbour(cell_id, cell_id+cells_per_side)  // bottom middle
//   addNeighbour(cell_id, cell_id+cells_per_side+1)  // bottom right

//   return cell_neighbours
// }

// function addNeighbour(cell_id, neighbour_id) {
//   if (neighbour_id > 0 && neighbour_id <= total_cells) {
//     if (cell_id % cells_per_side == 1) {
//       if (neighbour_id % cells_per_side > 0) {
//         cell_neighbours.push(neighbour_id)
//       }
//     }
//     else if (cell_id % cells_per_side == 0) {
//       if (neighbour_id % cells_per_side != 1) {
//         cell_neighbours.push(neighbour_id)
//       }
//     }
//     else {
//       cell_neighbours.push(neighbour_id)
//     }
//   }
// }

function idToIndices(id) {
  let c = (id - 1) % cells_per_side
  let r = (id - c - 1) / cells_per_side
  return [r, c]
}

function indicesToId(row_index, col_index) {
  return cells_per_side * row_index + col_index + 1
}

function toggleDeadAlive(id) {
  let [r, c] = idToIndices(id)
  console.log(r, c)
  board.value[r][c] = (board.value[r][c] + 1) % 2
}

function mod(a, b) {
  return ((a % b) + b) % b;
}

function countNeighbourhood(row, col) {
  let total = 0

  total += board.value[mod(row - 1, cells_per_side)][mod(col - 1, cells_per_side)]
  total += board.value[mod(row - 1, cells_per_side)][mod(col, cells_per_side)]
  total += board.value[mod(row - 1, cells_per_side)][mod(col + 1, cells_per_side)]

  total += board.value[mod(row, cells_per_side)][mod(col - 1, cells_per_side)]
  total += board.value[row][col]
  total += board.value[mod(row, cells_per_side)][mod(col + 1, cells_per_side)]

  total += board.value[mod(row + 1, cells_per_side)][mod(col - 1, cells_per_side)]
  total += board.value[mod(row + 1, cells_per_side)][mod(col, cells_per_side)]
  total += board.value[mod(row + 1, cells_per_side)][mod(col + 1, cells_per_side)]

  return total
}

function replaceBoard(new_board) {
  board.value = new_board
}

function replaceBoardRow(i, new_row) {
  board.value[i] = new_row
}

function getCellState(id) {
  if (!Array.isArray(board.value) || !board.value.length) {
    // console.log('Empty board!')
    return 0
  }
  let [r, c] = idToIndices(id)
  return board.value[r][c]
}

function getNextCellState(row, col) {
  // According to Wikipedia:
  // if sum of 9 neighbourhood cells is 3 -> live cell in middle
  // if sum is 4 -> maintain current middle state
  // every other sum -> dead cell in middle

  let total = countNeighbourhood(row, col)
  if (total == 3) {
    return 1
  }
  else if (total == 4) {
    return board.value[row][col]
  }
  else {
    return 0
  }
}

function getNextBoardState() {
  let next_board = []
  for (let i = 0; i < cells_per_side; i++) {
    let current_row = []
    for (let j = 0; j < cells_per_side; j++) {
      let next_cell = getNextCellState(i, j)
      current_row.push(next_cell)
    }
    next_board.push(current_row)
  }
  replaceBoard(next_board)
  state_counter.value++
}

function animationHandler(start) {
  if (state_counter.value == 0) {
    saveInitialState()
  }
  if (start) {
    interval_id.value = setInterval(getNextBoardState, time_interval.value)
  }
  else {
    clearInterval(interval_id.value)
    interval_id.value = -99
  }
}

function resetBoard() {
  animationHandler(false)
  state_counter.value = 0
  for (let i = 0; i < cells_per_side; i++) {
    for (let j = 0; j < cells_per_side; j++) {
      board.value[i][j] *= 0
    }
  }
}

function saveInitialState() {
  initial_state = board.value
}

function resetToInitial() {
  replaceBoard(initial_state)
  animationHandler(false)
  state_counter.value = 0
}

function saveAsText(content, file_name) {
  const blob = new Blob([content], { type: 'text/plain' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = file_name;
  a.click();
  URL.revokeObjectURL(url);
}

function boardToString() {
  let content = ''
  for (let i = 0; i < cells_per_side; i++) {
    for (let j = 0; j < cells_per_side; j++) {
      content += board.value[i][j]
    }
    content += '\n'
  }
  return content.trimEnd()
}

function saveBoard() {
  const board_str = boardToString()
  const f_name = 'board.txt'
  saveAsText(board_str, f_name)
}

const loadFileContent = (event) => {
  const file = event.target.files[0]
  const reader = new FileReader()

  return new Promise((resolve, reject) => {
    reader.onload = (e) => {
      // file_content.value = e.target.result
      file_lines.value = e.target.result.split('\n')
      resolve()
    }
    reader.onerror = (error) => {
      reject(error)
    }
    reader.readAsText(file)
  })
}

function parseFileContent() {
  const f_lines_len = file_lines.value.length
  for (let i = 0; i < f_lines_len && i < cells_per_side; i++) {
    // converts '1234' into [1, 2, 3, 4]
    let row = file_lines.value[i].split('').map(Number)
    // only uses enough elements to fill a row, the rest are cut off
    for (let j = row.length; j < cells_per_side; j++) {
      row.push(0)
    }
    replaceBoardRow(i, row.slice(0, cells_per_side))
  }
}

const loadBoard = async (event) => {
  try {
    await loadFileContent(event)
    parseFileContent()
    // alert('File successfully loaded!')
  }
  catch (error) {
    console.log(error)
  }
}

onMounted(initBoard)

</script>

<template>
  <h1 class="center">Conway's Game of Life</h1>
  
  <section class="rules">
    <h2>Rules</h2>
    <ul>
      <li>Live cells with less than 2 live neighbours die</li>
      <li>Live cells with 2 or 3 live neighbours live on</li>
      <li>Live cells with 4+ live neighbours die</li>
      <li>Dead cells with 3 live neighbours become alive</li>
    </ul>
  </section>

  <section class="center full-board">
    <div class="board">
      <div v-for="i in total_cells">
        <Cell 
        :cell_size="cell_size" 
        :dead_cell_colour="dead_cell_colour" 
        :live_cell_colour="live_cell_colour"
        :cell_status="getCellState(i)" 
        @toggle="toggleDeadAlive(i)" 
        ></Cell>
      </div>
    </div>
    <p class="center">State counter: {{ state_counter }}</p>
  </section>

  <section class="center controls">
    <h2>Controls</h2>
    <input type="submit" v-if="interval_id === -99" @click="animationHandler(true)" value="Start">
    <input type="submit" v-else @click="animationHandler(true)" value="Running" disabled>

    <input type="submit" @click="getNextBoardState" value="Step">

    <input type="submit" v-if="interval_id !== -99" @click="animationHandler(false)" value="Stop">
    <input type="submit" v-else @click="animationHandler(false)" value="Stop" disabled>

    <br>
    <label for="time_int">Time Interval (lower = faster)</label><br>
    <input type="range" id="time_int" v-model="time_interval" min="100" max="1000">
    {{ (time_interval / 1000).toFixed(2) }}s

    <br>
    <input type="submit" @click="resetToInitial" value="Reset to Initial State">
    <input type="submit" @click="resetBoard" value="Reset Board">

    <br>
    <input type="submit" @click="saveBoard" value="Save As">
    <input type="file" @change="loadBoard">
  </section>

  <!-- <br>
  <div class="center">
    <div v-for="row in file_lines">
      {{ row }}<br>
    </div>
  </div> -->

</template>

<style scoped>
* {
  box-sizing: border-box;
}

.center {
  margin-left: auto;
  margin-right: auto;
  text-align: center;
}

.board {
  box-sizing: content-box;
  width: 500px;
  height: 500px;
  border-style: solid;
  border-width: 1px;
}

.full-board {
  display: inline-block;
  /* border: 2px dotted; */
  vertical-align: top;
}

.rules {
  display: inline-block;
  width: calc((100% - 525px) / 2);
  border: 2px dotted;
  padding: 10px;
  text-align: center;
  vertical-align: top;
  margin: 5px;
}

ul {
  display: inline-block;
  text-align: left;
}

.controls {
  display: inline-block;
  width: calc((100% - 525px) / 2);
  border: 2px dotted;
  padding: 10px;
  vertical-align: top;
  margin: 5px;
}

input {
  margin: 10px;
  font-size: 16px;
  background-color: #A6CFD5;
}

h2 {
  margin: 0;
  text-align: center;
}

@media only screen and (max-width: 880px) {
  .full-board {
    width: 100%;
  }

  .board {
    margin-left: auto;
    margin-right: auto;
  }

  .rules {
    width: 100%;
  }

  .controls {
    width: 100%;
  }
}

</style>
