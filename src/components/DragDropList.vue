<template>
    <div class="drop-area" :data-target="target" v-mutation>
      <div
        v-for="(item, index) in items"
        :key="item.id"
        draggable="true"
        @dragstart="onDragStart($event, item)"
        :data-index="index"
      >
        {{ item.label }}
      </div>
    </div>
  </template>
  
  <script setup>
  const props = defineProps({
    items: Array,
    target: String,
    onDragStart: Function
  })
  
  const onDragStart = (event, item) => {
    props.onDragStart(event, item, props.target)
  }
  </script>
  
  <style scoped>
  .drop-area {
    min-height: 100px;
    border: 1px dashed #ccc;
    padding: 10px;
  }
  </style>