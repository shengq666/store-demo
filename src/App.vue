<template>
  <div class="container">
    <header>
      <h1>恒大国际建材家居博览中心 - 仓库管理系统</h1>
      <p class="subtitle">优化版：基于Vue3 + Konva实现高性能仓库管理</p>
    </header>

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

    <div class="main-content">
      <div class="map-container">
        <h2 class="map-title">仓库布局图</h2>
        <div
          class="map"
          ref="mapContainer"
          @dragover.prevent
          @drop="handleMapDrop"
        >
          <v-stage ref="stage" :config="stageConfig" @click="handleStageClick">
            <v-layer ref="backgroundLayer">
              <v-image :config="backgroundConfig" />
            </v-layer>
            <!-- <v-layer ref="linesLayer">
              <v-arrow
                v-for="(line, index) in connectionLines"
                :key="'line-' + index"
                :config="line"
              />
            </v-layer> -->
            <v-layer ref="warehouseLayer">
              <v-group
                v-for="warehouse in warehouses"
                :key="'group-' + warehouse.id"
                :config="getGroupConfig(warehouse)"
                @click="selectWarehouse(warehouse.id)"
                @dblclick="editWarehouseName(warehouse)"
                ref="warehouseGroups"
              >
                <v-rect :config="getWarehouseConfig(warehouse)" />
                <v-text :config="getLabelConfig(warehouse)" />
              </v-group>
            </v-layer>
            <v-layer>
              <v-transformer
                ref="transformer"
                :config="transformerConfig"
                @transformend="handleTransformEnd"
              />
            </v-layer>
          </v-stage>
        </div>

        <div class="products-container">
          <div class="product-pool">
            <h3 class="section-title">商品池</h3>
            <div class="products-grid">
              <div
                v-for="(product, index) in unassignedProducts"
                :key="'product-' + index"
                class="product"
                draggable="true"
                @dragstart="startProductDrag($event, product)"
              >
                {{ product }}
              </div>
            </div>
          </div>

          <div class="product-list">
            <h3 class="section-title">商品分配列表</h3>
            <div id="assigned-products">
              <div
                v-for="warehouse in warehousesWithProducts"
                :key="'wh-' + warehouse.id"
                class="warehouse-section"
              >
                <div class="warehouse-title">{{ warehouse.name }}</div>
                <ul>
                  <li
                    v-for="(product, pIndex) in warehouse.products"
                    :key="'p-' + pIndex"
                  >
                    {{ product }}
                    <button
                      class="delete-btn"
                      @click="removeProduct(warehouse, product)"
                    >
                      移除
                    </button>
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="controls">
        <div class="control-panel">
          <h3 class="section-title">仓库编辑面板</h3>

          <div class="form-group">
            <label for="warehouse-name">仓库名称</label>
            <input
              type="text"
              id="warehouse-name"
              v-model="currentWarehouse.name"
              placeholder="例如: 睡眠中心A区"
            />
          </div>

          <div class="form-group">
            <label for="warehouse-color">仓库颜色</label>
            <input
              type="color"
              id="warehouse-color"
              v-model="currentWarehouse.color"
            />
          </div>

          <div class="form-group">
            <label for="warehouse-width">宽度 (px)</label>
            <input
              type="number"
              id="warehouse-width"
              v-model="currentWarehouse.width"
              min="50"
              max="300"
            />
          </div>

          <div class="form-group">
            <label for="warehouse-height">高度 (px)</label>
            <input
              type="number"
              id="warehouse-height"
              v-model="currentWarehouse.height"
              min="50"
              max="300"
            />
          </div>

          <button @click="addWarehouse">添加新仓库</button>
          <button @click="updateWarehouse">更新仓库属性</button>
          <button class="delete-btn" @click="resetAll">重置所有数据</button>
        </div>

        <div class="export-panel">
          <h3 class="section-title">导出功能</h3>
          <button class="export-btn" @click="exportJSON">导出为JSON</button>
          <button class="export-btn" @click="exportImage">导出为图片</button>
        </div>

        <div class="instructions">
          <h3>使用说明</h3>
          <ol>
            <li><strong>添加仓库</strong>：点击"添加新仓库"创建新仓库</li>
            <li><strong>选择仓库</strong>：点击仓库可选中（蓝色边框）</li>
            <li>
              <strong>编辑仓库</strong>：选中仓库后修改属性，点击"更新仓库属性"
            </li>
            <li><strong>移动仓库</strong>：拖拽仓库主体移动位置</li>
            <li><strong>缩放仓库</strong>：拖拽仓库边缘的8个控制点调整大小</li>
            <li><strong>分配商品</strong>：从商品池拖拽商品到仓库</li>
            <li><strong>导出数据</strong>：使用导出功能保存配置结果</li>
          </ol>
        </div>
      </div>
    </div>
  </div>

  <div class="preview-container" v-if="showPreview">
    <div class="preview-content">
      <h3>配置数据预览</h3>
      <div class="json-preview">{{ jsonPreview }}</div>
      <button class="close-btn" @click="showPreview = false">关闭预览</button>
    </div>
  </div>

  <div class="status-bar">{{ statusMessage }}</div>
</template>

<script>
import { ref, reactive, computed, onMounted, nextTick, watch } from 'vue'
import Konva from 'konva'

export default {
  setup() {
    // 仓库背景图
    const backgroundImage = new Image()
    backgroundImage.src =
      'https://images.unsplash.com/photo-1524758631624-e2822e304c36?q=80&w=1000'
    backgroundImage.onload = () => {
      backgroundConfig.image = backgroundImage
    }

    // 仓库数据
    const warehouses = ref([])
    const currentWarehouse = reactive({
      id: null,
      name: '',
      color: '#3498db',
      width: 150,
      height: 100,
      x: 50,
      y: 50,
    })
    const selectedWarehouseId = ref(null)

    // 商品数据
    const sampleProducts = ref([
      '3-1 90ZWB022P53',
      '3-2 90M1813',
      '3-3 PT3126-B',
      '4-1 90B951',
      '4-2 90M0188A',
      '4-3 PT3039G-A',
      '5-1 90ZWB037PQ1',
      '5-2 90M0130S',
      '5-3 PT83222G',
      '6-1 90B958PQ1',
      '6-2 90M0128B',
      '6-3 PT89099G',
      '8-1 90B1855PQ1',
      '8-2 90M1076P0',
      '8-3 PT83227G',
      '7-1 90ZWB057P72',
      '7-2 90M1026D',
      '7-3 PTW3001G-A',
      '2-1 90B1856PQ1',
      '2-2 90JD.M1002.H24',
      '2-3 PT3022G',
      '1-1 90B3015',
      '1-2 90YSM3001.M26',
      '1-3 PT83226G',
      '15-1 90B1085PQ3',
      '15-2 90M1288',
      '15-3 PTW7009G',
      '14-1 90B1066',
      '14-2 90M0577',
      '14-3 PT83210G',
      '13-1 90B8007',
      '13-2 90M0578',
      '13-3 PT83221G',
      '11-1 90ZWB063PQ4',
      '11-2 90M1051A',
      '12-1 90B1859PQ1',
      '12-2 90BY.M2818',
      '12-3 PT83211G',
    ])

    // Konva配置
    const stage = ref(null)
    const stageConfig = reactive({
      width: 970,
      height: 500,
      draggable: false,
    })
    const backgroundConfig = reactive({
      width: 970,
      height: 500,
    })

    // Transformer配置
    const transformer = ref(null)
    const transformerConfig = reactive({
      rotateEnabled: true,
      boundBoxFunc: (oldBox, newBox) => {
        // 限制最小尺寸
        if (newBox.width < 50 || newBox.height < 50) {
          return oldBox
        }
        return newBox
      },
    })

    // UI状态
    const statusMessage = ref('就绪')
    const showPreview = ref(false)
    const jsonPreview = ref('')
    const connectionLines = ref([])
    const mapContainer = ref(null)

    // 仓库组引用
    const warehouseGroups = ref([])

    // 计算属性
    const warehouseCount = computed(() => warehouses.value.length)
    const productCount = computed(() => sampleProducts.value.length)
    const assignedCount = computed(() => {
      return warehouses.value.reduce(
        (total, wh) => total + wh.products.length,
        0
      )
    })
    const unassignedProducts = computed(() => {
      const assigned = new Set()
      warehouses.value.forEach((wh) => {
        wh.products.forEach((p) => assigned.add(p))
      })
      return sampleProducts.value.filter((p) => !assigned.has(p))
    })
    const warehousesWithProducts = computed(() => {
      return warehouses.value.filter((wh) => wh.products.length > 0)
    })

    // 仓库操作
    const addWarehouse = () => {
      const id = `wh-${Date.now()}`
      const x = Math.floor(
        Math.random() * (stageConfig.width - currentWarehouse.width)
      )
      const y = Math.floor(
        Math.random() * (stageConfig.height - currentWarehouse.height)
      )

      const newWarehouse = {
        id,
        name: currentWarehouse.name || `仓库${warehouses.value.length + 1}`,
        color: currentWarehouse.color,
        width: currentWarehouse.width,
        height: currentWarehouse.height,
        x,
        y,
        rotation: 0,
        scaleX: 1,
        scaleY: 1,
        products: [],
        draggable: true,
      }

      warehouses.value.push(newWarehouse)
      selectWarehouse(id)
      updateStatus(`仓库 "${newWarehouse.name}" 已创建`)
    }

    const selectWarehouse = (id) => {
      selectedWarehouseId.value = id
      const warehouse = warehouses.value.find((w) => w.id === id)
      if (warehouse) {
        Object.assign(currentWarehouse, {
          id: warehouse.id,
          name: warehouse.name,
          color: warehouse.color,
          width: warehouse.width,
          height: warehouse.height,
          x: warehouse.x,
          y: warehouse.y,
        })
        updateStatus(`已选中仓库: ${warehouse.name}`)
      }

      // 更新变换器
      updateTransformer()
    }

    const updateWarehouse = () => {
      if (!selectedWarehouseId.value) return

      const warehouse = warehouses.value.find(
        (w) => w.id === selectedWarehouseId.value
      )
      if (warehouse) {
        warehouse.name = currentWarehouse.name
        warehouse.color = currentWarehouse.color
        warehouse.width = currentWarehouse.width
        warehouse.height = currentWarehouse.height

        // 更新Konva节点
        const group = findKonvaGroup(warehouse.id)
        if (group) {
          group.width(warehouse.width)
          group.height(warehouse.height)
          group.getChildren().forEach((child) => {
            if (child.className === 'Rect') {
              child.width(warehouse.width)
              child.height(warehouse.height)
            } else if (child.className === 'Text') {
              child.x(warehouse.width / 2)
              child.y(warehouse.height / 2)
              child.text(`${warehouse.name} (${warehouse.products.length})`)
            }
          })
          group.getLayer().batchDraw()
        }

        // 更新变换器
        updateTransformer()

        updateStatus(`已更新仓库: ${warehouse.name}`)
      }
    }

    // 变换结束处理
    const handleTransformEnd = (e) => {
      const node = transformer.value.getNode()
      const activeShape = node.nodes()[0]

      if (!activeShape) return

      const warehouse = warehouses.value.find((w) => w.id === activeShape.id())
      if (!warehouse) return

      // 更新仓库属性
      warehouse.x = activeShape.x()
      warehouse.y = activeShape.y()
      warehouse.rotation = activeShape.rotation()
      warehouse.scaleX = activeShape.scaleX()
      warehouse.scaleY = activeShape.scaleY()
      warehouse.width = activeShape.width() * activeShape.scaleX()
      warehouse.height = activeShape.height() * activeShape.scaleY()

      // 重置缩放因子
      activeShape.scaleX(1)
      activeShape.scaleY(1)

      // 更新控制面板
      if (selectedWarehouseId.value === warehouse.id) {
        currentWarehouse.width = warehouse.width
        currentWarehouse.height = warehouse.height
        currentWarehouse.x = warehouse.x
        currentWarehouse.y = warehouse.y
      }

      // 更新文本位置
      const textNode = activeShape.findOne('Text')
      if (textNode) {
        textNode.x(warehouse.width / 2)
        textNode.y(warehouse.height / 2)
      }

      updateLinesForWarehouse(warehouse.id)
      updateStatus(`仓库 "${warehouse.name}" 变换完成`)
    }

    // 更新变换器节点
    const updateTransformer = () => {
      if (!transformer.value) return

      const transformerNode = transformer.value.getNode()
      transformerNode.nodes([])

      if (selectedWarehouseId.value) {
        const selectedNode = findKonvaGroup(selectedWarehouseId.value)
        if (selectedNode) {
          transformerNode.nodes([selectedNode])
        }
      }
    }

    // 商品操作
    const startProductDrag = (e, product) => {
      e.dataTransfer.setData('text/plain', product)
      updateStatus(`开始拖拽商品: ${product}`)
    }

    const handleMapDrop = (e) => {
      e.preventDefault()
      const product = e.dataTransfer.getData('text/plain')

      // 获取鼠标在画布上的位置
      const stageNode = stage.value.getNode()
      const pointerPos = stageNode.getPointerPosition()

      if (!pointerPos) return

      // 查找该位置下的仓库
      const warehouseShape = stageNode.getIntersection(pointerPos, (node) => {
        return node.name() === 'warehouse'
      })

      if (warehouseShape) {
        addProductToWarehouse(warehouseShape.id(), product)

        // 从商品池中移除该商品
        const productEls = document.querySelectorAll('.product')
        for (const el of productEls) {
          if (el.textContent === product) {
            el.remove()
            break
          }
        }

        updateStatus(`商品 "${product}" 已添加到仓库`)
      } else {
        updateStatus(`未找到目标仓库，请拖放到仓库上`)
      }
    }

    const addProductToWarehouse = (warehouseId, product) => {
      const warehouse = warehouses.value.find((w) => w.id === warehouseId)
      if (warehouse && !warehouse.products.includes(product)) {
        warehouse.products.push(product)

        // 更新仓库文本显示商品数量
        const group = findKonvaGroup(warehouseId)
        if (group) {
          const textNode = group.findOne('Text')
          if (textNode) {
            textNode.text(`${warehouse.name} (${warehouse.products.length})`)
            group.getLayer().batchDraw()
          }
        }

        // 更新连线
        updateConnectionLines()
        updateStatus(`商品 "${product}" 已添加到仓库 "${warehouse.name}"`)

        // 更新右侧分配列表
        nextTick(() => {
          updateConnectionLines()
        })
      }
    }

    const removeProduct = (warehouse, product) => {
      warehouse.products = warehouse.products.filter((p) => p !== product)

      // 更新仓库文本显示商品数量
      const group = findKonvaGroup(warehouse.id)
      if (group) {
        const textNode = group.findOne('Text')
        if (textNode) {
          textNode.text(`${warehouse.name} (${warehouse.products.length})`)
          group.getLayer().batchDraw()
        }
      }

      // 将商品添加回商品池
      const productEl = document.createElement('div')
      productEl.className = 'product'
      productEl.textContent = product
      productEl.draggable = true
      productEl.addEventListener('dragstart', (e) => {
        e.dataTransfer.setData('text/plain', product)
      })

      const productPool = document.querySelector('.products-grid')
      if (productPool) {
        productPool.appendChild(productEl)
      }

      // 更新连线
      updateConnectionLines()
      updateStatus(`商品 "${product}" 已从仓库移除`)
    }

    // 连线操作
    const updateConnectionLines = () => {
      connectionLines.value = []

      warehouses.value.forEach((warehouse) => {
        warehouse.products.forEach((product) => {
          const warehouseCenterX = warehouse.x + warehouse.width / 2
          const warehouseCenterY = warehouse.y + warehouse.height / 2

          // 在实际应用中，这里应该获取商品在分配列表中的位置
          const productX = warehouse.x + warehouse.width + 50
          const productY = warehouse.y + warehouse.height / 2

          connectionLines.value.push({
            points: [warehouseCenterX, warehouseCenterY, productX, productY],
            stroke: '#e74c3c',
            strokeWidth: 2,
            pointerLength: 10,
            pointerWidth: 10,
            dash: [5, 5],
            lineCap: 'round',
            lineJoin: 'round',
          })
        })
      })
    }

    const updateLinesForWarehouse = (warehouseId) => {
      updateConnectionLines()
    }

    // 工具函数
    const findKonvaGroup = (id) => {
      if (!stage.value) return null
      return stage.value.getNode().findOne(`#${id}`)
    }

    const updateStatus = (message) => {
      statusMessage.value = message
    }

    const resetAll = () => {
      if (confirm('确定要重置所有数据吗？所有仓库和分配的商品将被清除。')) {
        warehouses.value = []
        selectedWarehouseId.value = null
        Object.assign(currentWarehouse, {
          id: null,
          name: '',
          color: '#3498db',
          width: 150,
          height: 100,
          x: 50,
          y: 50,
        })
        connectionLines.value = []

        // 重置变换器
        if (transformer.value) {
          transformer.value.getNode().nodes([])
        }

        // 重置商品池
        const productPool = document.querySelector('.products-grid')
        if (productPool) {
          productPool.innerHTML = ''
          sampleProducts.value.forEach((product) => {
            const productEl = document.createElement('div')
            productEl.className = 'product'
            productEl.textContent = product
            productEl.draggable = true
            productEl.addEventListener('dragstart', (e) => {
              e.dataTransfer.setData('text/plain', product)
            })
            productPool.appendChild(productEl)
          })
        }

        updateStatus('所有数据已重置')
      }
    }

    const exportJSON = () => {
      const data = {
        warehouses: warehouses.value.map((wh) => ({
          id: wh.id,
          name: wh.name,
          color: wh.color,
          width: wh.width,
          height: wh.height,
          x: wh.x,
          y: wh.y,
          rotation: wh.rotation,
          scaleX: wh.scaleX,
          scaleY: wh.scaleY,
          products: [...wh.products],
        })),
      }

      jsonPreview.value = JSON.stringify(data, null, 2)
      showPreview.value = true
      updateStatus('配置数据已导出为JSON')
    }

    const exportImage = () => {
      const uri = stage.value.getNode().toDataURL()
      const link = document.createElement('a')
      link.download = '仓库布局图.png'
      link.href = uri
      link.click()
      updateStatus('仓库布局已导出为图片')
    }

    const editWarehouseName = (warehouse) => {
      const newName = prompt('输入新名称', warehouse.name)
      if (newName && newName.trim() !== '') {
        warehouse.name = newName

        // 更新Konva文本
        const group = findKonvaGroup(warehouse.id)
        if (group) {
          const textNode = group.findOne('Text')
          if (textNode) {
            textNode.text(`${newName} (${warehouse.products.length})`)
            group.getLayer().batchDraw()
          }
        }

        // 更新控制面板
        if (selectedWarehouseId.value === warehouse.id) {
          currentWarehouse.name = newName
        }

        updateStatus(`仓库名称更新为: ${newName}`)
      }
    }

    // 处理画布点击
    const handleStageClick = (e) => {
      console.log('Stage clicked:', e, e.target.getStage())
      // 如果点击的是空白区域，取消选择
      if (e.target === e.target.getStage()) {
        selectedWarehouseId.value = null
        updateTransformer()
        updateStatus('已取消选择仓库')
      }
    }

    // Konva配置生成器
    const getGroupConfig = (warehouse) => {
      return {
        id: warehouse.id,
        x: warehouse.x,
        y: warehouse.y,
        width: warehouse.width,
        height: warehouse.height,
        draggable: true,
        rotation: warehouse.rotation,
        name: 'warehouse',
        scaleX: 1,
        scaleY: 1,
      }
    }

    const getWarehouseConfig = (warehouse) => {
      return {
        width: warehouse.width,
        height: warehouse.height,
        fill: warehouse.color,
        stroke:
          warehouse.id === selectedWarehouseId.value ? '#3498db' : '#2c3e50',
        strokeWidth: warehouse.id === selectedWarehouseId.value ? 3 : 2,
        cornerRadius: 5,
        shadowColor: 'rgba(0,0,0,0.3)',
        shadowBlur: 5,
        shadowOffset: { x: 2, y: 2 },
        shadowOpacity: 0.5,
      }
    }

    const getLabelConfig = (warehouse) => {
      return {
        x: warehouse.width / 2,
        y: warehouse.height / 2,
        text: `${warehouse.name} (${warehouse.products.length})`,
        fontSize: 14,
        fontFamily: 'Arial',
        fill: 'white',
        align: 'center',
        verticalAlign: 'middle',
        width: warehouse.width,
        padding: 5,
        listening: false,
      }
    }

    // 监听选中的仓库变化
    watch(selectedWarehouseId, () => {
      updateTransformer()
    })

    // 生命周期钩子
    onMounted(() => {
      // 添加一些初始仓库
      warehouses.value.push({
        id: 'wh-1',
        name: '睡眠中心A区',
        color: '#3498db',
        width: 160,
        height: 120,
        x: 100,
        y: 80,
        rotation: 0,
        scaleX: 1,
        scaleY: 1,
        products: ['1-1 90B3015', '1-2 90YSM3001.M26', '1-3 PT83226G'],
      })

      warehouses.value.push({
        id: 'wh-2',
        name: '家居展示区',
        color: '#2ecc71',
        width: 140,
        height: 100,
        x: 300,
        y: 150,
        rotation: 0,
        scaleX: 1,
        scaleY: 1,
        products: ['3-1 90ZWB022P53', '3-2 90M1813'],
      })

      warehouses.value.push({
        id: 'wh-3',
        name: '建材存储区',
        color: '#e74c3c',
        width: 180,
        height: 110,
        x: 200,
        y: 300,
        rotation: 0,
        scaleX: 1,
        scaleY: 1,
        products: ['5-1 90ZWB037PQ1', '5-2 90M0130S', '5-3 PT83222G'],
      })

      updateConnectionLines()

      // 初始化变换器
      nextTick(() => {
        updateTransformer()
      })
    })

    return {
      // 数据
      warehouses,
      currentWarehouse,
      sampleProducts,
      stage,
      stageConfig,
      backgroundConfig,
      statusMessage,
      showPreview,
      jsonPreview,
      connectionLines,
      mapContainer,
      warehouseGroups,
      transformer,
      transformerConfig,

      // 计算属性
      warehouseCount,
      productCount,
      assignedCount,
      unassignedProducts,
      warehousesWithProducts,

      // 方法
      addWarehouse,
      selectWarehouse,
      updateWarehouse,
      handleTransformEnd,
      startProductDrag,
      handleMapDrop,
      removeProduct,
      resetAll,
      exportJSON,
      exportImage,
      editWarehouseName,
      handleStageClick,
      getGroupConfig,
      getWarehouseConfig,
      getLabelConfig,
    }
  },
}
</script>

<style scoped>
/* 样式保持不变，与之前相同 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

body {
  background: linear-gradient(135deg, #1a2a6c, #b21f1f, #fdbb2d);
  color: #333;
  min-height: 100vh;
  padding: 20px;
}

.container {
  max-width: 1400px;
  margin: 0 auto;
}

header {
  text-align: center;
  padding: 20px;
  margin-bottom: 30px;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 10px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

h1 {
  color: #2c3e50;
  margin-bottom: 10px;
  font-size: 2.5rem;
}

.subtitle {
  color: #7f8c8d;
  font-size: 1.2rem;
  max-width: 800px;
  margin: 0 auto;
  line-height: 1.6;
}

.main-content {
  display: flex;
  gap: 25px;
  margin-bottom: 30px;
}

.map-container {
  flex: 3;
  background: rgba(255, 255, 255, 0.85);
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
  position: relative;
  min-height: 600px;
  overflow: hidden;
}

.controls {
  flex: 1;
  background: rgba(255, 255, 255, 0.85);
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.map-title {
  text-align: center;
  margin-bottom: 20px;
  color: #2c3e50;
  font-size: 1.8rem;
}

.map {
  width: 100%;
  height: 500px;
  border: 2px solid #7f8c8d;
  border-radius: 8px;
  position: relative;
  overflow: hidden;
  box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.3);
}

.products-container {
  display: flex;
  gap: 20px;
  margin-top: 25px;
}

.product-pool,
.product-list {
  background: rgba(255, 255, 255, 0.9);
  border-radius: 10px;
  padding: 15px;
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.1);
  flex: 1;
  max-height: 400px;
  overflow-y: auto;
}

.section-title {
  color: #2c3e50;
  margin-bottom: 15px;
  padding-bottom: 10px;
  border-bottom: 2px solid #3498db;
  font-size: 1.3rem;
}

.products-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
  gap: 12px;
  margin-top: 15px;
}

.product {
  background: #3498db;
  color: white;
  padding: 10px;
  border-radius: 5px;
  text-align: center;
  cursor: grab;
  transition: all 0.2s;
  font-size: 14px;
  box-shadow: 0 3px 5px rgba(0, 0, 0, 0.2);
}

.product:hover {
  background: #2980b9;
  transform: translateY(-3px);
  box-shadow: 0 5px 8px rgba(0, 0, 0, 0.3);
}

.product-list ul {
  list-style-type: none;
  padding: 0;
}

.product-list li {
  background: #f8f9fa;
  padding: 10px;
  margin-bottom: 8px;
  border-radius: 5px;
  border-left: 4px solid #3498db;
  display: flex;
  justify-content: space-between;
}

.product-list .warehouse-title {
  font-weight: bold;
  color: #2c3e50;
  margin-top: 15px;
  padding-bottom: 5px;
  border-bottom: 1px dashed #ccc;
}

.control-panel {
  margin-bottom: 25px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
}

input,
select,
button {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 16px;
}

button {
  background: #3498db;
  color: white;
  border: none;
  cursor: pointer;
  font-weight: bold;
  transition: background 0.3s;
  margin-top: 10px;
}

button:hover {
  background: #2980b9;
}

.export-btn {
  background: #27ae60;
}

.export-btn:hover {
  background: #219653;
}

.delete-btn {
  background: #e74c3c;
  width: auto;
  padding: 5px 10px;
  font-size: 12px;
}

.delete-btn:hover {
  background: #c0392b;
}

.instructions {
  background: rgba(255, 255, 255, 0.9);
  border-radius: 10px;
  padding: 20px;
  margin-top: 25px;
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.1);
}

.instructions h3 {
  color: #2c3e50;
  margin-bottom: 15px;
}

.instructions ol {
  padding-left: 20px;
  line-height: 1.8;
}

.instructions li {
  margin-bottom: 10px;
}

.preview-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  z-index: 100;
  display: flex;
  justify-content: center;
  align-items: center;
}

.preview-content {
  background: white;
  padding: 20px;
  border-radius: 10px;
  max-width: 90%;
  max-height: 90%;
  overflow: auto;
}

.json-preview {
  background: #2c3e50;
  color: #ecf0f1;
  padding: 15px;
  border-radius: 5px;
  font-family: monospace;
  white-space: pre-wrap;
  max-height: 400px;
  overflow-y: auto;
}

.close-btn {
  background: #e74c3c;
  margin-top: 15px;
}

.highlight {
  animation: highlight 1.5s ease;
}

@keyframes highlight {
  0% {
    background-color: #ffeb3b;
  }
  100% {
    background-color: transparent;
  }
}

.stats {
  display: flex;
  gap: 15px;
  margin-top: 20px;
  justify-content: center;
}

.stat-card {
  background: rgba(255, 255, 255, 0.9);
  border-radius: 8px;
  padding: 15px;
  text-align: center;
  flex: 1;
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.1);
}

.stat-value {
  font-size: 2rem;
  font-weight: bold;
  color: #3498db;
  margin: 10px 0;
}

.stat-label {
  color: #7f8c8d;
  font-size: 1rem;
}

.status-bar {
  position: fixed;
  bottom: 10px;
  left: 10px;
  background: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 8px 15px;
  border-radius: 20px;
  font-size: 14px;
  z-index: 200;
}

@media (max-width: 1100px) {
  .main-content {
    flex-direction: column;
  }

  .products-container {
    flex-direction: column;
  }
}
</style>
