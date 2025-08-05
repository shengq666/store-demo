<!-- src/components/WarehouseMap.vue -->
<template>
  <div class="map-container">
    <h2 class="map-title">仓库布局图</h2>
    <div class="map" id="map" @dragover.prevent @drop="handleMapDrop">
      <!-- 仓库 -->
      <div
        v-for="warehouse in warehouses"
        :key="warehouse.id"
        class="warehouse"
        :class="{ selected: selectedWarehouse?.id === warehouse.id }"
        :data-id="warehouse.id"
        :style="{
          transform: `translate(${warehouse.x}px, ${warehouse.y}px)`,
          width: warehouse.width + 'px',
          height: warehouse.height + 'px',
          backgroundColor: warehouse.color,
        }"
        @click="() => selectWarehouse(warehouse.id)"
        @dblclick="() => editWarehouseName(warehouse)"
        @mousedown.left="($event) => startDrag($event, warehouse)"
        v-memo="[
          warehouse.width,
          warehouse.height,
          warehouse.x,
          warehouse.y,
          warehouse.color,
          selectedWarehouse?.id === warehouse.id,
        ]"
      >
        <div class="label">{{ warehouse.name }}</div>
        <button
          class="delete-btn"
          @click.stop="() => deleteWarehouse(warehouse.id)"
        >
          ×
        </button>

        <!-- 控制点 -->
        <div
          v-if="selectedWarehouse?.id === warehouse.id"
          v-for="handle in resizeHandles"
          :key="handle"
          class="resize-handle"
          :class="`resize-${handle}`"
          @mousedown.left.stop="
            ($event) => startResize($event, warehouse, handle)
          "
        ></div>
      </div>
    </div>

    <div class="products-section">
      <ProductPool
        :available-products="availableProducts"
        @handle-product-drag-start="handleProductDragStart"
      />

      <ProductList
        :warehouses="warehouses"
        @remove-product-from-warehouse="removeProductFromWarehouse"
      />
    </div>
  </div>
</template>

<script>
import { computed } from 'vue'
import ProductPool from './ProductPool.vue'
import ProductList from './ProductList.vue'

export default {
  name: 'WarehouseMap',
  components: {
    ProductPool,
    ProductList,
  },
  props: {
    warehouses: {
      type: Array,
      required: true,
    },
    selectedWarehouse: {
      type: Object,
      default: null,
    },
    availableProducts: {
      type: Array,
      required: true,
    },
  },
  emits: [
    'select-warehouse',
    'edit-warehouse-name',
    'delete-warehouse',
    'start-drag',
    'start-resize',
    'handle-product-drag-start',
    'handle-map-drop',
    'remove-product-from-warehouse',
  ],
  setup(props, { emit }) {
    const resizeHandles = ['nw', 'n', 'ne', 'e', 'se', 's', 'sw', 'w']

    const selectWarehouse = (warehouseId) => {
      emit('select-warehouse', warehouseId)
    }

    const editWarehouseName = (warehouse) => {
      emit('edit-warehouse-name', warehouse)
    }

    const deleteWarehouse = (warehouseId) => {
      emit('delete-warehouse', warehouseId)
    }

    const startDrag = (e, warehouse) => {
      // 避免与resize handle冲突
      if (
        e.target.classList.contains('resize-handle') ||
        e.target.classList.contains('delete-btn')
      ) {
        return
      }
      e.preventDefault()
      emit('start-drag', e, warehouse)
    }

    const startResize = (e, warehouse, direction) => {
      emit('start-resize', e, warehouse, direction)
    }

    const handleProductDragStart = (e, product) => {
      emit('handle-product-drag-start', e, product)
    }

    const handleMapDrop = (e) => {
      emit('handle-map-drop', e)
    }

    const removeProductFromWarehouse = (warehouseId, product) => {
      emit('remove-product-from-warehouse', warehouseId, product)
    }

    return {
      resizeHandles,
      selectWarehouse,
      editWarehouseName,
      deleteWarehouse,
      startDrag,
      startResize,
      handleProductDragStart,
      handleMapDrop,
      removeProductFromWarehouse,
    }
  },
}
</script>

<style scoped>
.map-container {
  width: 80%;
  padding: 20px;
  display: flex;
  flex-direction: column;
  border-right: 2px dashed #3498db;
}

.map-title {
  text-align: center;
  margin-bottom: 15px;
  color: #2c3e50;
  font-size: 1.8rem;
  padding-bottom: 10px;
  border-bottom: 2px solid #3498db;
}

.map {
  flex: 1;
  background: url('https://images.unsplash.com/photo-1524758631624-e2822e304c36?q=80&w=1000')
    center/cover;
  border: 2px solid #7f8c8d;
  border-radius: 8px;
  position: relative;
  overflow: hidden;
  /* box-shadow: inset 0 0 15px rgba(0, 0, 0, 0.4); */
  margin-bottom: 20px;
}

.warehouse {
  position: absolute;
  border: 3px solid #2c3e50;
  border-radius: 8px;
  cursor: move;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  color: white;
  text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.8);
  z-index: 20;
  transition: transform 0.2s, box-shadow 0.2s;
  overflow: hidden;
  user-select: none;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.warehouse.selected {
  box-shadow: 0 0 0 4px #3498db, 0 0 20px rgba(52, 152, 219, 0.7);
  z-index: 30;
}

.warehouse .label {
  background: rgba(0, 0, 0, 0.6);
  padding: 6px 12px;
  border-radius: 5px;
  font-size: 16px;
  pointer-events: none;
  text-align: center;
  max-width: 90%;
  overflow: hidden;
  text-overflow: ellipsis;
}

.warehouse .delete-btn {
  position: absolute;
  top: 8px;
  right: 8px;
  background: #e74c3c;
  color: white;
  border: none;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  font-size: 14px;
  cursor: pointer;
  display: none;
  z-index: 25;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
}

.warehouse:hover .delete-btn {
  display: block;
}

/* 缩放控制点 */
.resize-handle {
  position: absolute;
  width: 12px;
  height: 12px;
  background: #3498db;
  border: 2px solid white;
  border-radius: 50%;
  z-index: 40;
  cursor: pointer;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
}

.resize-nw {
  top: -6px;
  left: -6px;
  cursor: nw-resize;
}
.resize-ne {
  top: -6px;
  right: -6px;
  cursor: ne-resize;
}
.resize-sw {
  bottom: -6px;
  left: -6px;
  cursor: sw-resize;
}
.resize-se {
  bottom: -6px;
  right: -6px;
  cursor: se-resize;
}
.resize-n {
  top: -6px;
  left: 50%;
  margin-left: -6px;
  cursor: n-resize;
}
.resize-s {
  bottom: -6px;
  left: 50%;
  margin-left: -6px;
  cursor: s-resize;
}
.resize-w {
  top: 50%;
  left: -6px;
  margin-top: -6px;
  cursor: w-resize;
}
.resize-e {
  top: 50%;
  right: -6px;
  margin-top: -6px;
  cursor: e-resize;
}

.products-section {
  display: flex;
  gap: 20px;
  height: 250px;
}
</style>
