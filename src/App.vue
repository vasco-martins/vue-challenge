<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import Header from './components/Header.vue'
import HomeView from './views/HomeView.vue'
</script>

<template>
  <Header />

  <main>
    <RouterView />
  </main>
</template>
