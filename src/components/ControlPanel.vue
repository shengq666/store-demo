<!-- src/components/ControlPanel.vue -->
<template>
  <div class="controls">
    <div class="control-panel">
      <h3 class="panel-title">仓库编辑面板</h3>

      <div class="form-group">
        <label for="warehouse-name">仓库名称</label>
        <input
          type="text"
          id="warehouse-name"
          v-model="warehouseForm.name"
          placeholder="例如: 睡眠中心A区"
        />
      </div>

      <div class="form-group">
        <label for="warehouse-color">仓库颜色</label>
        <input
          type="color"
          id="warehouse-color"
          v-model="warehouseForm.color"
        />
      </div>

      <div class="form-group">
        <label for="warehouse-width">宽度 (px)</label>
        <input
          type="number"
          id="warehouse-width"
          v-model.number="warehouseForm.width"
          min="50"
          max="300"
        />
      </div>

      <div class="form-group">
        <label for="warehouse-height">高度 (px)</label>
        <input
          type="number"
          id="warehouse-height"
          v-model.number="warehouseForm.height"
          min="50"
          max="300"
        />
      </div>

      <button id="add-warehouse" @click="addWarehouse">添加新仓库</button>
      <button
        id="update-warehouse"
        @click="updateWarehouse"
        :disabled="!selectedWarehouse"
      >
        更新仓库属性
      </button>
      <button id="reset-all" class="delete-btn" @click="resetAll">
        重置所有数据
      </button>
    </div>

    <div class="export-panel">
      <h3 class="panel-title">导出功能</h3>
      <button id="export-json" class="export-btn" @click="exportToJSON">
        导出为JSON
      </button>
      <button id="export-image" class="export-btn" @click="exportToImage">
        导出为图片
      </button>
    </div>

    <Instructions />
  </div>
</template>

<script>
import { reactive, watch } from 'vue'
import Instructions from './Instructions.vue'

export default {
  name: 'ControlPanel',
  components: {
    Instructions
  },
  props: {
    selectedWarehouse: {
      type: Object,
      default: null
    }
  },
  emits: [
    'add-warehouse',
    'update-warehouse',
    'reset-all',
    'export-to-json',
    'export-to-image'
  ],
  setup(props, { emit }) {
    const warehouseForm = reactive({
      name: '',
      color: '#3498db',
      width: 150,
      height: 100,
    })

    // 监听选中仓库变化，更新表单
    watch(
      () => props.selectedWarehouse,
      (newVal) => {
        if (newVal) {
          warehouseForm.name = newVal.name
          warehouseForm.color = newVal.color
          warehouseForm.width = newVal.width
          warehouseForm.height = newVal.height
        } else {
          warehouseForm.name = ''
          warehouseForm.color = '#3498db'
          warehouseForm.width = 150
          warehouseForm.height = 100
        }
      }
    )

    const addWarehouse = () => {
      emit('add-warehouse', { ...warehouseForm })
    }

    const updateWarehouse = () => {
      if (!props.selectedWarehouse) return
      emit('update-warehouse', { ...warehouseForm })
    }

    const resetAll = () => {
      emit('reset-all')
    }

    const exportToJSON = () => {
      emit('export-to-json')
    }

    const exportToImage = () => {
      emit('export-to-image')
    }

    return {
      warehouseForm,
      addWarehouse,
      updateWarehouse,
      resetAll,
      exportToJSON,
      exportToImage
    }
  }
}
</script>

<style scoped>
.controls {
  width: 20%;
  padding: 20px;
  background: #f8f9fa;
  overflow-y: auto;
}

.control-panel {
  margin-bottom: 25px;
  padding: 20px;
  background: white;
  border-radius: 10px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.panel-title {
  color: #2c3e50;
  text-align: center;
  margin-bottom: 20px;
  padding-bottom: 10px;
  border-bottom: 2px solid #3498db;
  font-size: 1.5rem;
}

.form-group {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #2c3e50;
}

input,
select,
button {
  width: 100%;
  padding: 12px;
  border: 2px solid #ddd;
  border-radius: 8px;
  font-size: 16px;
  transition: all 0.3s;
}

input:focus,
select:focus {
  border-color: #3498db;
  box-shadow: 0 0 8px rgba(52, 152, 219, 0.5);
  outline: none;
}

button {
  background: linear-gradient(to bottom, #3498db, #2980b9);
  color: white;
  border: none;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s;
  margin-top: 15px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  position: relative;
  overflow: hidden;
}

button:hover {
  background: linear-gradient(to bottom, #2980b9, #2573a7);
  transform: translateY(-3px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
}

button:active {
  transform: translateY(1px);
}

button:disabled {
  background: #bdc3c7;
  cursor: not-allowed;
}

.export-btn {
  background: linear-gradient(to bottom, #27ae60, #219653);
}

.export-btn:hover {
  background: linear-gradient(to bottom, #219653, #1d8449);
}

.delete-btn {
  background: linear-gradient(to bottom, #e74c3c, #c0392b);
  width: auto;
  padding: 8px 15px;
  font-size: 14px;
}

.delete-btn:hover {
  background: linear-gradient(to bottom, #c0392b, #a23526);
}
</style>