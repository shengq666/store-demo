<!-- src/components/Preview.vue -->
<template>
  <div class="preview-container" id="preview-container" v-show="showPreview">
    <div class="preview-content">
      <h3>配置数据预览</h3>
      <div class="json-preview">{{ jsonPreview }}</div>
      <button class="close-btn" @click="closePreview">关闭预览</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Preview',
  props: {
    showPreview: {
      type: Boolean,
      required: true
    },
    jsonPreview: {
      type: String,
      required: true
    }
  },
  emits: ['close-preview'],
  setup(props, { emit }) {
    const closePreview = () => {
      emit('close-preview')
    }

    return {
      closePreview
    }
  }
}
</script>

<style scoped>
.preview-container {
  display: flex;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.85);
  z-index: 1000;
  justify-content: center;
  align-items: center;
}

.preview-container[v-show='true'] {
  display: flex;
}

.preview-content {
  background: white;
  padding: 30px;
  border-radius: 15px;
  width: 80%;
  max-height: 85%;
  overflow: auto;
  box-shadow: 0 15px 50px rgba(0, 0, 0, 0.5);
}

.json-preview {
  background: #2c3e50;
  color: #ecf0f1;
  padding: 20px;
  border-radius: 8px;
  font-family: monospace;
  white-space: pre-wrap;
  max-height: 500px;
  overflow-y: auto;
  font-size: 14px;
  line-height: 1.5;
}

.close-btn {
  background: linear-gradient(to bottom, #e74c3c, #c0392b);
  margin-top: 20px;
  padding: 12px 20px;
  font-size: 16px;
  border-radius: 8px;
}
</style>