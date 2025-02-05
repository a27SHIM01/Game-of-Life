<script setup>

const props = defineProps([
  'cell_size',
  'dead_cell_colour',
  'live_cell_colour',
  'cell_status'
])

const emit = defineEmits(['toggle'])


function toggleCell() {
  emit('toggle')
}

</script>

<template>
  <div class="cell" :style="{
    background: '#' + live_cell_colour || '#ffffff',
    height: cell_size + 'px' || '10px',
    width: cell_size + 'px' || '10px'
  }"
  @click="toggleCell"
  v-if="cell_status === 1"
  ></div>

  <div class="cell" :style="{
    background: '#' + dead_cell_colour || '#777777',
    height: cell_size + 'px' || '10px',
    width: cell_size + 'px' || '10px'
  }"
  @click="toggleCell"
  v-else
  ></div>


</template>


<style scoped>

.cell {
  box-sizing: border-box;
  float: left;
  border: 1px solid;
  font-size: 50%;
  text-align: center;
}

</style>
