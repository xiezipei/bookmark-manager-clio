<template>
  <transition name="fade">
    <button
      v-show="isVisible"
      @click="scrollToTop"
      class="fixed bottom-4 right-4 bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 shadow-lg transition-colors duration-300"
    >
      ↑
    </button>
  </transition>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const isVisible = ref(false)

const toggleVisibility = () => {
  if (window.pageYOffset > 300) {
    isVisible.value = true
  } else {
    isVisible.value = false
  }
}

const scrollToTop = () => {
  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  })
}

onMounted(() => {
  window.addEventListener('scroll', toggleVisibility)
})

onUnmounted(() => {
  window.removeEventListener('scroll', toggleVisibility)
})
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
