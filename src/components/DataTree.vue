<template>
    <q-tree
      :nodes="nodes"
      node-key="id"
      selection="single"
      v-model:selected="selectedNode"
      @update:selected="updateSelectedNode"
      @dragstart="onDragStart"
      draggable="true"
    />
  </template>
  
  <script setup>
  import { ref } from 'vue'
  
  const props = defineProps({
    nodes: Array,
    onNodeSelect: Function,
    onDragStart: Function
  })
  
  const selectedNode = ref(null)
  
  const updateSelectedNode = (selected) => {
    props.onNodeSelect(selected)
  }
  </script>