<!-- src/components/ProductList.vue -->
<template>
  <div class="product-list">
    <h3 class="section-title">商品分配列表</h3>
    <div id="assigned-products">
      <div v-for="warehouse in warehousesWithProducts" :key="warehouse.id">
        <div class="warehouse-title">{{ warehouse.name }}</div>
        <ul>
          <li v-for="product in warehouse.products" :key="product">
            {{ product }}
            <button
              class="delete-btn"
              @click="() => removeProductFromWarehouse(warehouse.id, product)"
            >
              移除
            </button>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import { computed } from 'vue'

export default {
  name: 'ProductList',
  props: {
    warehouses: {
      type: Array,
      required: true,
    },
  },
  emits: ['remove-product-from-warehouse'],
  setup(props, { emit }) {
    const warehousesWithProducts = computed(() => {
      return props.warehouses.filter(
        (warehouse) => warehouse.products.length > 0
      )
    })

    const removeProductFromWarehouse = (warehouseId, product) => {
      emit('remove-product-from-warehouse', warehouseId, product)
    }

    return {
      warehousesWithProducts,
      removeProductFromWarehouse,
    }
  },
}
</script>

<style scoped>
.product-list {
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

.product-list ul {
  list-style-type: none;
  padding: 0;
}

.product-list li {
  background: #f8f9fa;
  padding: 12px;
  margin-bottom: 10px;
  border-radius: 6px;
  border-left: 5px solid #3498db;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: black;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
}

.product-list li:hover {
  transform: translateX(5px);
  background: #edf7ff;
}

.product-list .warehouse-title {
  font-weight: bold;
  color: #2c3e50;
  margin-top: 15px;
  padding-bottom: 8px;
  border-bottom: 2px dashed #3498db;
  font-size: 1.2rem;
}

.delete-btn {
  background: linear-gradient(to bottom, #e74c3c, #c0392b);
  width: auto;
  padding: 4px 10px;
  font-size: 12px;
  border: none;
  border-radius: 4px;
  color: white;
  font-weight: bold;
}
</style>
