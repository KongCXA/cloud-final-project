<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import HeaderContent from './components/HeaderContent.vue'
</script>

<template>
    <div class="absolute p-8 flex flex-col gap-[10px] z-20">
      <HeaderContent />

      <div class="flex gap-[10px]">
        <RouterLink to="/"><p class="text-black">Home</p></RouterLink>
        <RouterLink to="/dashboard"><p class="text-black">Dashboard</p></RouterLink>
      </div class="flex gap-[10px]">
    </div>

  <RouterView />
</template>

<style scoped></style>
