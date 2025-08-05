<!-- src/components/Header.vue -->
<template>
  <header>
    <div class="logo">恒大</div>
    <h1>恒大国际建材家居博览中心 - 仓库管理系统</h1>
    <p class="subtitle">专业版：优化布局设计，支持仓库编辑、缩放和动态连线</p>

    <div class="stats">
      <div class="stat-card">
        <div class="stat-value">{{ warehouseCount }}</div>
        <div class="stat-label">仓库数量</div>
      </div>
      <div class="stat-card">
        <div class="stat-value">{{ productCount }}</div>
        <div class="stat-label">商品数量</div>
      </div>
      <div class="stat-card">
        <div class="stat-value">{{ assignedCount }}</div>
        <div class="stat-label">已分配商品</div>
      </div>
    </div>
  </header>
</template>

<script>
import { computed } from 'vue'

export default {
  name: 'AppHeader',
  props: {
    warehouseCount: {
      type: Number,
      required: true
    },
    productCount: {
      type: Number,
      required: true
    },
    assignedCount: {
      type: Number,
      required: true
    }
  }
}
</script>

<style scoped>
header {
  text-align: center;
  padding: 25px;
  background: linear-gradient(to right, #2c3e50, #4a6491);
  color: white;
  position: relative;
}

h1 {
  font-size: 2.8rem;
  margin-bottom: 10px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
}

.subtitle {
  font-size: 1.3rem;
  max-width: 800px;
  margin: 0 auto;
  line-height: 1.6;
  opacity: 0.9;
}

.logo {
  position: absolute;
  top: 20px;
  left: 30px;
  font-size: 2rem;
  font-weight: bold;
  background: #e74c3c;
  color: white;
  padding: 10px 20px;
  border-radius: 50%;
  transform: rotate(-15deg);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.stats {
  display: flex;
  gap: 20px;
  margin: 20px 0;
  justify-content: center;
}

.stat-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 12px;
  padding: 20px;
  text-align: center;
  flex: 1;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
  border: 1px solid #eee;
  transition: transform 0.3s;
}

.stat-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
}

.stat-value {
  font-size: 2.5rem;
  font-weight: bold;
  color: #3498db;
  margin: 15px 0;
}

.stat-label {
  color: #7f8c8d;
  font-size: 1.1rem;
  font-weight: 600;
}
</style>