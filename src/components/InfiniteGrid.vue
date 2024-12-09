<template>
    <div class="infinite-grid">
      <div
        v-for="(cell, index) in gridCells"
        :key="index"
        class="grid-cell"
        :data-index="index"
        @dragover="allowDrop"
        @drop="onGridDrop"
      >
        <div v-if="cell">
          {{ cell.label }}
          <button @click="resetCell(index)">Reset</button>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  
  const props = defineProps({
    gridCells: Array,
    onGridDrop: Function,
    onResetCell: Function
  })
  
  const allowDrop = (event) => {
    event.preventDefault()
  }
  
  const onGridDrop = (event) => {
    props.onGridDrop(event)
  }
  
  const resetCell = (index) => {
    props.onResetCell(index)
  }
  </script>
  
  <style scoped>
  .infinite-grid {
    display: grid;
    grid-template-columns: repeat(100, 100px);
    grid-template-rows: repeat(100, 100px);
    gap: 10px;
    overflow: auto;
    height: 400px;
    width: 100%;
  }
  
  .grid-cell {
    display: flex;
    align-items: center;
    justify-content: center;
    border: 1px solid #ccc;
    background-color: #f9f9f9;
    height: 100px;
    width: 100px;
    position: relative;
  }
  
  .grid-cell button {
    position: absolute;
    top: 5px;
    right: 5px;
  }
  </style>