<script setup>
import { ref } from 'vue'
import { useQuasar } from 'quasar'

const $q = useQuasar()
const darkMode = ref($q.dark.isActive)

const toggleDarkMode = () => {
  $q.dark.set(!darkMode.value)
  darkMode.value = !darkMode.value
}
</script>

<template>
  <q-layout view="lHh Lpr lFf">
    <q-header :class="$q.dark.isActive ? 'bg-secondary' : 'bg-primary'">
      <q-toolbar>
        <q-toolbar-title> Data Mapping Export </q-toolbar-title>
        <q-btn flat round icon="brightness_6" @click="toggleDarkMode" />
      </q-toolbar>
    </q-header>
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>
