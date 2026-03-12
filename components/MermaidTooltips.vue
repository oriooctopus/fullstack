<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps({
  tooltips: { type: Object, required: true },
})

const container = ref(null)

function applyTooltips() {
  if (!container.value) return

  // Slidev renders mermaid inside shadow DOM, so we need to reach into it
  const mermaidDiv = container.value.querySelector('.mermaid')
  const shadow = mermaidDiv?.shadowRoot
  const svg = shadow?.querySelector('svg') || container.value.querySelector('svg')
  if (!svg) return

  for (const [nodeId, text] of Object.entries(props.tooltips)) {
    // Mermaid gives nodes IDs like "flowchart-NodeId-0"
    const node = svg.querySelector(`[id*="-${nodeId}-"]`)
    if (!node) continue

    if (node.dataset.tooltipApplied) continue
    node.dataset.tooltipApplied = 'true'

    // SVG <title> only works on SVG elements, but mermaid uses
    // foreignObject with HTML divs for label text. Set the HTML
    // title attribute on those so the tooltip covers the whole block.
    for (const fo of node.querySelectorAll('foreignObject')) {
      for (const el of fo.querySelectorAll('div, span')) {
        el.title = text
        el.style.cursor = 'help'
      }
    }

    // Also cover the SVG rect/shape itself for the non-text areas.
    // pointer-events: all ensures the fill area triggers hover, not just the stroke.
    for (const shape of node.querySelectorAll('rect, polygon, circle, ellipse')) {
      const svgTitle = document.createElementNS('http://www.w3.org/2000/svg', 'title')
      svgTitle.textContent = text
      shape.prepend(svgTitle)
      shape.style.pointerEvents = 'all'
    }

    node.style.cursor = 'help'
  }
}

onMounted(() => {
  // Mermaid renders async inside shadow DOM, so retry a few times
  const attempts = [500, 1500, 3000, 5000]
  attempts.forEach(delay => {
    setTimeout(applyTooltips, delay)
  })
})
</script>

<template>
  <div ref="container">
    <slot />
  </div>
</template>
