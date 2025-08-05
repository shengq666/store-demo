<!-- src/components/StatusBar.vue -->
<template>
  <div class="status-bar" id="status-bar">{{ statusMessage }}</div>
</template>

<script>
export default {
  name: 'StatusBar',
  props: {
    statusMessage: {
      type: String,
      required: true
    }
  }
}
</script>

<style scoped>
.status-bar {
  position: fixed;
  bottom: 20px;
  left: 20px;
  background: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 12px 25px;
  border-radius: 30px;
  font-size: 16px;
  z-index: 200;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}
</style>