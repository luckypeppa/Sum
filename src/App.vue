<script setup lang="ts">
import { RouterView } from "vue-router";
import AppNavVue from "./components/AppNav.vue";
</script>

<template>
  <div class="container px-4 wrapper mx-auto mb-24">
    <AppNavVue />
    <AppNotis />
    <RouterView />
  </div>
  <ProgressBar />
</template>

<style scoped></style>
