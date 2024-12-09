<template>
    <q-form @submit.prevent="handleSubmit" class="q-gutter-y-sm">
      <q-file
        outlined
        v-model="file"
        :label="label"
        accept=".json"
        @input="handleFileUpload"
      />
      <q-btn type="submit" :label="buttonLabel" color="primary" />
    </q-form>
  </template>
  
  <script setup>
  import { ref } from 'vue'
  
  const props = defineProps({
    label: String,
    buttonLabel: String,
    onFileUpload: Function
  })
  
  const file = ref(null)
  
  const handleFileUpload = (event) => {
    const file = event.target.files[0]
    if (file) {
      const reader = new FileReader()
      reader.onload = (e) => {
        try {
          const content = JSON.parse(e.target.result)
          props.onFileUpload(content)
        } catch (error) {
          console.error('Error parsing JSON:', error)
          alert('Error parsing JSON file. Please ensure the file is in the correct format.')
        }
      }
      reader.readAsText(file)
    } else {
      console.log('No file selected')
    }
  }
  
  const handleSubmit = () => {
    // Handle form submission if needed
  }
  </script>