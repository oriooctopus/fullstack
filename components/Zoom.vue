<script setup>
import { ref } from 'vue'

const props = defineProps({
  scale: { type: Number, default: 1.8 },
})

const zoomed = ref(false)
const origin = ref('center center')

function handleClick(e) {
  if (!zoomed.value) {
    const rect = e.currentTarget.getBoundingClientRect()
    const x = ((e.clientX - rect.left) / rect.width) * 100
    const y = ((e.clientY - rect.top) / rect.height) * 100
    origin.value = `${x}% ${y}%`
  }
  zoomed.value = !zoomed.value
}
</script>

<template>
  <div
    @click="handleClick"
    :style="{
      display: 'flex',
      justifyContent: 'center',
      alignItems: 'center',
      transform: zoomed ? `scale(${props.scale})` : 'scale(1)',
      transformOrigin: origin,
      transition: 'transform 0.3s ease',
      cursor: zoomed ? 'zoom-out' : 'zoom-in',
    }"
  >
    <slot />
  </div>
</template>
