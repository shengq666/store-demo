<!-- src/components/ProductPool.vue -->
<template>
  <div class="product-pool">
    <h3 class="section-title">商品池</h3>
    <div class="products-grid" id="product-pool">
      <div
        v-for="product in availableProducts"
        :key="product"
        class="product"
        draggable="true"
        @dragstart="($event) => handleProductDragStart($event, product)"
      >
        {{ product }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ProductPool',
  props: {
    availableProducts: {
      type: Array,
      required: true,
    },
  },
  emits: ['handle-product-drag-start'],
  setup(props, { emit }) {
    const handleProductDragStart = (e, product) => {
      emit('handle-product-drag-start', e, product)
    }

    return {
      handleProductDragStart,
    }
  },
}
</script>

<style scoped>
.product-pool {
  flex: 1;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 10px;
  padding: 15px;
  box-shadow: 0 3px 15px rgba(0, 0, 0, 0.15);
  overflow-y: auto;
  border: 1px solid #ddd;
}

.section-title {
  color: #2c3e50;
  margin-bottom: 15px;
  padding-bottom: 10px;
  border-bottom: 2px solid #3498db;
  font-size: 1.4rem;
  text-align: center;
}

.products-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
  gap: 15px;
}

.product {
  background: linear-gradient(to bottom, #3498db, #2980b9);
  color: white;
  padding: 12px;
  border-radius: 6px;
  text-align: center;
  cursor: grab;
  transition: all 0.3s;
  font-size: 15px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  position: relative;
  overflow: hidden;
}

.product:hover {
  background: linear-gradient(to bottom, #2980b9, #2573a7);
  transform: translateY(-5px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
}

.product:before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.4),
    transparent
  );
  transition: 0.5s;
}

.product:hover:before {
  left: 100%;
}
</style>
