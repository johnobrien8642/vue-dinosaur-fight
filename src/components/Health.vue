<template>
  <div
    class="d-flex flex-column justify-content-end health-container w-100 h-100 mt-auto pb-3"
    :style="cssProps"
  >
    <label class="ms-auto">Health {{health}}</label>
    <div class="health bar"><div class="meter me-auto" /></div>
    <label class="ms-auto">Confidence {{confidence}}</label>
    <div class="confidence bar"><div class="meter me-auto" /></div>
  </div>
</template>

<script>

export default {
  name: "Health",
  props: {
    health: Number,
    confidence: Number
  },
  computed: {
    cssProps() {
      return {
        '--health': (this.health * 2) + '%',
        '--confidence': (this.confidence * 2) + '%'
      }
    }
  }
}
</script>