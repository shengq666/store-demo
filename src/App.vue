<!-- src/App.vue -->
<template>
  <div class="container">
    <AppHeader
      :warehouse-count="warehouseCount"
      :product-count="productCount"
      :assigned-count="assignedCount"
    />

    <div class="main-content">
      <WarehouseMap
        :warehouses="warehouses"
        :selected-warehouse="selectedWarehouse"
        :available-products="availableProducts"
        @select-warehouse="selectWarehouse"
        @edit-warehouse-name="editWarehouseName"
        @delete-warehouse="deleteWarehouse"
        @start-drag="startDrag"
        @start-resize="startResize"
        @handle-product-drag-start="handleProductDragStart"
        @handle-map-drop="handleMapDrop"
        @remove-product-from-warehouse="removeProductFromWarehouse"
      />

      <ControlPanel
        :selected-warehouse="selectedWarehouse"
        @add-warehouse="addWarehouse"
        @update-warehouse="updateWarehouse"
        @reset-all="resetAll"
        @export-to-json="exportToJSON"
        @export-to-image="exportToImage"
      />
    </div>

    <div class="footer">
      &copy; 2023 恒大国际建材家居博览中心 - 仓库管理系统 | 版本 2.0
    </div>
  </div>

  <Preview
    :show-preview="showPreview"
    :json-preview="jsonPreview"
    @close-preview="closePreview"
  />

  <StatusBar :status-message="statusMessage" />
</template>

<script>
import { ref, reactive, computed, onMounted, nextTick, watch } from 'vue'
import html2canvas from 'html2canvas'
import AppHeader from './components/Header.vue'
import WarehouseMap from './components/WarehouseMap.vue'
import ControlPanel from './components/ControlPanel.vue'
import Preview from './components/Preview.vue'
import StatusBar from './components/StatusBar.vue'

export default {
  name: 'WarehouseManager',
  components: {
    AppHeader,
    WarehouseMap,
    ControlPanel,
    Preview,
    StatusBar,
  },
  setup() {
    // 示例数据
    const sampleProducts = [
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
    ]

    // 响应式数据
    const warehouses = ref([])
    const nextId = ref(1)
    const selectedWarehouse = ref(null)
    const availableProducts = ref([...sampleProducts])
    const lines = ref([])
    const statusMessage = ref('系统已就绪，等待操作...')
    const showPreview = ref(false)
    const jsonPreview = ref('')

    // 计算属性
    const warehouseCount = computed(() => warehouses.value.length)

    const productCount = computed(() => sampleProducts.length)

    const assignedCount = computed(() => {
      return warehouses.value.reduce(
        (total, warehouse) => total + warehouse.products.length,
        0
      )
    })

    // 拖拽相关变量
    const dragState = reactive({
      isDragging: false,
      startX: 0,
      startY: 0,
      startLeft: 0,
      startTop: 0,
      warehouse: null,
    })

    const resizeState = reactive({
      isResizing: false,
      startX: 0,
      startY: 0,
      startWidth: 0,
      startHeight: 0,
      startLeft: 0,
      startTop: 0,
      direction: '',
      warehouse: null,
    })

    // 监听仓库位置和大小变化，更新连线
    watch(
      warehouses,
      () => {
        if (selectedWarehouse.value) {
          updateLinesForWarehouse(selectedWarehouse.value.id)
        }
      },
      { deep: true }
    )

    // 方法
    const addWarehouse = (warehouseData) => {
      const newWarehouse = {
        id: `wh-${nextId.value++}`,
        name: warehouseData.name || `仓库${nextId.value}`,
        color: warehouseData.color,
        width: warehouseData.width,
        height: warehouseData.height,
        x: Math.floor(Math.random() * 600) + 50,
        y: Math.floor(Math.random() * 300) + 50,
        products: [],
      }

      warehouses.value.push(newWarehouse)
      selectWarehouse(newWarehouse.id)
      updateStatus(`仓库 "${newWarehouse.name}" 已创建`)
    }

    const selectWarehouse = (warehouseId) => {
      selectedWarehouse.value =
        warehouses.value.find((w) => w.id === warehouseId) || null
      updateStatus(`已选中仓库: ${selectedWarehouse.value?.name || ''}`)
    }

    const updateWarehouse = (warehouseData) => {
      if (!selectedWarehouse.value) return

      selectedWarehouse.value.name = warehouseData.name
      selectedWarehouse.value.color = warehouseData.color
      selectedWarehouse.value.width = warehouseData.width
      selectedWarehouse.value.height = warehouseData.height

      // 更新连线
      updateLinesForWarehouse(selectedWarehouse.value.id)
      updateStatus(`已更新仓库: ${warehouseData.name}`)
    }

    const deleteWarehouse = (warehouseId) => {
      const warehouse = warehouses.value.find((w) => w.id === warehouseId)
      if (!warehouse) return

      if (!confirm(`确定删除仓库 "${warehouse.name}" 及其所有商品吗？`)) return

      // 将商品移回商品池
      warehouse.products.forEach((product) => {
        if (!availableProducts.value.includes(product)) {
          availableProducts.value.push(product)
        }
      })

      // 删除仓库
      warehouses.value = warehouses.value.filter((w) => w.id !== warehouseId)

      // 删除相关连线
      lines.value = lines.value.filter(
        (line) => line.warehouseId !== warehouseId
      )

      // 清除选中状态
      if (
        selectedWarehouse.value &&
        selectedWarehouse.value.id === warehouseId
      ) {
        selectedWarehouse.value = null
      }

      updateStatus(`仓库 "${warehouse.name}" 已删除`)
    }

    const editWarehouseName = (warehouse) => {
      const newName = prompt('输入新名称', warehouse.name)
      if (newName && newName.trim() !== '') {
        warehouse.name = newName
        if (
          selectedWarehouse.value &&
          selectedWarehouse.value.id === warehouse.id
        ) {
          selectedWarehouse.value.name = newName
        }
        updateStatus(`仓库名称更新为: ${newName}`)
      }
    }

    const handleProductDragStart = (e, product) => {
      e.dataTransfer.setData('text/plain', product)
    }

    const handleMapDrop = (e) => {
      e.preventDefault()
      const product = e.dataTransfer.getData('text/plain')

      // 找到目标仓库
      let targetElement = e.target
      while (targetElement && !targetElement.classList.contains('warehouse')) {
        targetElement = targetElement.parentElement
      }

      if (targetElement) {
        const warehouseId = targetElement.dataset.id
        addProductToWarehouse(warehouseId, product)

        // 从商品池移除
        availableProducts.value = availableProducts.value.filter(
          (p) => p !== product
        )
      }
    }

    const addProductToWarehouse = (warehouseId, product) => {
      const warehouse = warehouses.value.find((w) => w.id === warehouseId)
      if (warehouse && !warehouse.products.includes(product)) {
        warehouse.products.push(product)
        drawLine(warehouseId, product)
        updateStatus(`商品 "${product}" 已添加到仓库 "${warehouse.name}"`)
      }
    }

    const removeProductFromWarehouse = (warehouseId, product) => {
      const warehouse = warehouses.value.find((w) => w.id === warehouseId)
      if (!warehouse) return

      warehouse.products = warehouse.products.filter((p) => p !== product)

      // 删除连线
      const lineId = `line-${warehouseId}-${product.replace(/\s+/g, '-')}`
      lines.value = lines.value.filter((line) => line.id !== lineId)

      // 将商品移回商品池
      if (!availableProducts.value.includes(product)) {
        availableProducts.value.push(product)
      }

      updateStatus(`商品 "${product}" 已从仓库移除`)
    }

    const drawLine = (warehouseId, product) => {
      nextTick(() => {
        const mapElement = document.getElementById('map')
        if (!mapElement) return

        const warehouse = warehouses.value.find((w) => w.id === warehouseId)
        if (!warehouse) return

        // 查找商品元素
        const productElements = document.querySelectorAll('.product')
        let productElement = null
        for (let el of productElements) {
          if (el.textContent === product) {
            productElement = el
            break
          }
        }

        if (!productElement) return

        const mapRect = mapElement.getBoundingClientRect()

        // 计算仓库中心点
        const warehouseCenterX = warehouse.x + warehouse.width / 2
        const warehouseCenterY = warehouse.y + warehouse.height / 2

        // 计算商品中心点
        const productRect = productElement.getBoundingClientRect()
        const productCenterX =
          productRect.left - mapRect.left + productRect.width / 2
        const productCenterY =
          productRect.top - mapRect.top + productRect.height / 2

        const lineId = `line-${warehouseId}-${product.replace(/\s+/g, '-')}`

        // 更新或创建线
        const existingLineIndex = lines.value.findIndex(
          (line) => line.id === lineId
        )
        const line = {
          id: lineId,
          warehouseId: warehouseId,
          x1: warehouseCenterX,
          y1: warehouseCenterY,
          x2: productCenterX,
          y2: productCenterY,
        }

        if (existingLineIndex >= 0) {
          lines.value[existingLineIndex] = line
        } else {
          lines.value.push(line)
        }
      })
    }

    const updateLinesForWarehouse = (warehouseId) => {
      const warehouse = warehouses.value.find((w) => w.id === warehouseId)
      if (!warehouse) return

      warehouse.products.forEach((product) => {
        drawLine(warehouseId, product)
      })
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
      dragState.isDragging = true
      dragState.startX = e.clientX
      dragState.startY = e.clientY
      dragState.startLeft = warehouse.x
      dragState.startTop = warehouse.y
      dragState.warehouse = warehouse

      document.addEventListener('mousemove', drag)
      document.addEventListener('mouseup', stopDrag)
      updateStatus('正在移动仓库...')
    }

    const drag = (e) => {
      if (!dragState.isDragging || !dragState.warehouse) return

      const dx = e.clientX - dragState.startX
      const dy = e.clientY - dragState.startY
      console.log('======e', e.clientX, e.clientY)
      // 获取地图元素以计算边界
      const mapElement = document.getElementById('map')
      if (!mapElement) return
      const mapRect = mapElement.getBoundingClientRect()

      // 计算新的位置
      let newLeft = dragState.startLeft + dx
      let newTop = dragState.startTop + dy

      // 边界检测 - 确保仓库不会被拖出地图区域
      // 左边界
      newLeft = Math.max(0, newLeft)
      // 上边界
      newTop = Math.max(0, newTop)
      // 右边界 (仓库右侧不能超出地图右侧)
      newLeft = Math.min(mapRect.width - dragState.warehouse.width - 6, newLeft)
      // 下边界 (仓库下侧不能超出地图下侧)
      newTop = Math.min(mapRect.height - dragState.warehouse.height - 6, newTop)

      // // 确保计算后的值不会小于0（双重保险）
      // newLeft = Math.max(0, newLeft)
      // newTop = Math.max(0, newTop)

      // 应用新位置
      dragState.warehouse.x = newLeft
      dragState.warehouse.y = newTop
    }

    const stopDrag = () => {
      if (dragState.isDragging) {
        dragState.isDragging = false
        document.removeEventListener('mousemove', drag)
        document.removeEventListener('mouseup', stopDrag)
        if (dragState.warehouse) {
          updateStatus(`仓库 "${dragState.warehouse.name}" 已移动到新位置`)
        }
        dragState.warehouse = null
      }
    }

    const startResize = (e, warehouse, direction) => {
      e.preventDefault()
      e.stopPropagation()

      resizeState.isResizing = true
      resizeState.startX = e.clientX
      resizeState.startY = e.clientY
      resizeState.startWidth = warehouse.width
      resizeState.startHeight = warehouse.height
      resizeState.startLeft = warehouse.x
      resizeState.startTop = warehouse.y
      resizeState.direction = direction
      resizeState.warehouse = warehouse

      document.addEventListener('mousemove', resize)
      document.addEventListener('mouseup', stopResize)
      updateStatus('正在调整仓库大小...')
    }

    const resize = (e) => {
      if (!resizeState.isResizing || !resizeState.warehouse) return

      const dx = e.clientX - resizeState.startX
      const dy = e.clientY - resizeState.startY

      // 获取地图元素以计算边界
      const mapElement = document.getElementById('map')
      if (!mapElement) return
      const mapRect = mapElement.getBoundingClientRect()

      // 根据方向调整大小
      let newWidth = resizeState.startWidth
      let newHeight = resizeState.startHeight
      let newLeft = resizeState.startLeft
      let newTop = resizeState.startTop

      // 根据方向调整大小
      switch (resizeState.direction) {
        case 'nw':
          newWidth = resizeState.startWidth - dx
          newHeight = resizeState.startHeight - dy
          newLeft = resizeState.startLeft + dx
          newTop = resizeState.startTop + dy
          break
        case 'n':
          newHeight = resizeState.startHeight - dy
          newTop = resizeState.startTop + dy
          break
        case 'ne':
          newWidth = resizeState.startWidth + dx
          newHeight = resizeState.startHeight - dy
          newTop = resizeState.startTop + dy
          break
        case 'e':
          newWidth = resizeState.startWidth + dx
          break
        case 'se':
          newWidth = resizeState.startWidth + dx
          newHeight = resizeState.startHeight + dy
          break
        case 's':
          newHeight = resizeState.startHeight + dy
          break
        case 'sw':
          newWidth = resizeState.startWidth - dx
          newHeight = resizeState.startHeight + dy
          newLeft = resizeState.startLeft + dx
          break
        case 'w':
          newWidth = resizeState.startWidth - dx
          newLeft = resizeState.startLeft + dx
          break
      }

      // 确保最小尺寸
      newWidth = Math.max(50, newWidth)
      newHeight = Math.max(50, newHeight)

      // 边界检测 - 确保仓库在缩放时不会被拖出地图区域
      // 右边界检测
      if (newLeft + newWidth + 6 > mapRect.width) {
        // 6是左右边框宽度
        newWidth = mapRect.width - newLeft - 6
        // 确保最小宽度
        newWidth = Math.max(50, newWidth)
      }

      // 下边界检测
      if (newTop + newHeight + 6 > mapRect.height) {
        // 6是上下边框宽度
        newHeight = mapRect.height - newTop - 6
        // 确保最小高度
        newHeight = Math.max(50, newHeight)
      }

      // 左边界检测
      if (newLeft < 0) {
        newLeft = 0
      }

      // 上边界检测
      if (newTop < 0) {
        newTop = 0
      }

      // 应用新的尺寸和位置
      resizeState.warehouse.width = newWidth
      resizeState.warehouse.height = newHeight
      resizeState.warehouse.x = newLeft
      resizeState.warehouse.y = newTop
    }

    const stopResize = () => {
      if (resizeState.isResizing) {
        resizeState.isResizing = false
        document.removeEventListener('mousemove', resize)
        document.removeEventListener('mouseup', stopResize)
        updateStatus(`仓库大小调整完成`)
        resizeState.warehouse = null
      }
    }

    const exportToJSON = () => {
      const data = {
        warehouses: warehouses.value.map((warehouse) => ({
          name: warehouse.name,
          color: warehouse.color,
          width: warehouse.width,
          height: warehouse.height,
          x: warehouse.x,
          y: warehouse.y,
          products: warehouse.products,
        })),
      }

      jsonPreview.value = JSON.stringify(data, null, 2)
      showPreview.value = true
      updateStatus('配置数据已导出为JSON')
    }

    const exportToImage = () => {
      const mapElement = document.getElementById('map')
      if (mapElement) {
        const originalStyle = mapElement.style.background
        mapElement.style.background =
          "url('https://images.unsplash.com/photo-1524758631624-e2822e304c36?q=80&w=1000') center/cover"
        html2canvas(mapElement, {
          useCORS: true,
          backgroundColor: null, // 保持透明背景以匹配CSS
        })
          .then((canvas) => {
            // 恢复原始背景样式
            mapElement.style.background = originalStyle
            const link = document.createElement('a')
            link.download = '仓库布局图.png'
            link.href = canvas.toDataURL('image/png')
            link.click()
            updateStatus('仓库布局已导出为图片')
          })
          .catch((error) => {
            // 恢复原始背景样式
            mapElement.style.background = originalStyle
            console.error('导出图片时出错:', error)
            updateStatus('导出图片失败')
          })
      }
    }

    const closePreview = () => {
      showPreview.value = false
    }

    const resetAll = () => {
      if (!confirm('确定要重置所有数据吗？所有仓库和分配的商品将被清除。'))
        return

      warehouses.value = []
      availableProducts.value = [...sampleProducts]
      lines.value = []
      selectedWarehouse.value = null
      updateStatus('所有数据已重置')
    }

    const updateStatus = (message) => {
      statusMessage.value = message
    }

    // 初始化
    const initialize = () => {
      // 生成3000个仓库，确保它们能容纳在2000*1000的画布中
      const newWarehouses = []
      const gridSize = Math.ceil(Math.sqrt(3000)) // 计算网格大小
      const cellWidth = Math.floor(2000 / gridSize) // 每个网格单元的宽度
      const cellHeight = Math.floor(1000 / gridSize) // 每个网格单元的高度

      // 确保仓库尺寸不会超出网格单元
      const maxWarehouseWidth = Math.max(50, cellWidth - 10)
      const maxWarehouseHeight = Math.max(50, cellHeight - 10)

      for (let i = 0; i < 30; i++) {
        // 计算网格坐标
        const gridX = i % gridSize
        const gridY = Math.floor(i / gridSize)

        // 在网格单元内随机生成仓库位置和尺寸
        const width = Math.floor(Math.random() * (maxWarehouseWidth - 50) + 50)
        const height = Math.floor(
          Math.random() * (maxWarehouseHeight - 50) + 50
        )

        // 计算仓库位置，确保不会超出画布边界
        const maxX = Math.max(0, gridX * cellWidth)
        const maxY = Math.max(0, gridY * cellHeight)
        const minX = Math.min(2000 - width, maxX + cellWidth - width)
        const minY = Math.min(1000 - height, maxY + cellHeight - height)

        const x = Math.floor(Math.random() * (minX - maxX + 1) + maxX)
        const y = Math.floor(Math.random() * (minY - maxY + 1) + maxY)

        newWarehouses.push({
          id: `wh-${i + 1}`,
          name: `仓库${i + 1}`,
          color: `hsl(${Math.floor(Math.random() * 360)}, 70%, 60%)`,
          width: width,
          height: height,
          x: x,
          y: y,
          products: [],
        })
      }

      warehouses.value = newWarehouses
      nextId.value = 3001

      // 添加一些初始商品
      setTimeout(() => {
        addProductToWarehouse('wh-1', '1-1 90B3015')
        addProductToWarehouse('wh-1', '1-2 90YSM3001.M26')
        addProductToWarehouse('wh-1', '1-3 PT83226G')

        addProductToWarehouse('wh-2', '3-1 90ZWB022P53')
        addProductToWarehouse('wh-2', '3-2 90M1813')

        addProductToWarehouse('wh-3', '5-1 90ZWB037PQ1')
        addProductToWarehouse('wh-3', '5-2 90M0130S')
        addProductToWarehouse('wh-3', '5-3 PT83222G')

        // 更新商品池
        availableProducts.value = availableProducts.value.filter(
          (product) =>
            ![
              '1-1 90B3015',
              '1-2 90YSM3001.M26',
              '1-3 PT83226G',
              '3-1 90ZWB022P53',
              '3-2 90M1813',
              '5-1 90ZWB037PQ1',
              '5-2 90M0130S',
              '5-3 PT83222G',
            ].includes(product)
        )
      }, 100)
    }

    onMounted(() => {
      initialize()

      // 清理事件监听器
      window.addEventListener('beforeunload', () => {
        document.removeEventListener('mousemove', drag)
        document.removeEventListener('mouseup', stopDrag)
        document.removeEventListener('mousemove', resize)
        document.removeEventListener('mouseup', stopResize)
      })
    })

    return {
      // 数据
      warehouses,
      selectedWarehouse,
      availableProducts,
      lines,
      statusMessage,
      showPreview,
      jsonPreview,

      // 计算属性
      warehouseCount,
      productCount,
      assignedCount,

      // 方法
      addWarehouse,
      selectWarehouse,
      updateWarehouse,
      deleteWarehouse,
      editWarehouseName,
      handleProductDragStart,
      handleMapDrop,
      removeProductFromWarehouse,
      exportToJSON,
      exportToImage,
      closePreview,
      resetAll,
      startDrag,
      startResize,
      updateStatus,
    }
  },
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Microsoft YaHei', 'Segoe UI', Tahoma, Geneva, Verdana,
    sans-serif;
}

body {
  background: linear-gradient(135deg, #1a2a6c, #b21f1f, #fdbb2d);
  color: #333;
  min-height: 100vh;
  padding: 20px;
  overflow-x: hidden;
}

.container {
  max-width: 1800px;
  margin: 0 auto;
  background: rgba(255, 255, 255, 0.92);
  border-radius: 15px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  overflow: hidden;
}

.main-content {
  display: flex;
  height: 1000px;
}

.footer {
  text-align: center;
  padding: 15px;
  background: #2c3e50;
  color: white;
  font-size: 14px;
}
</style>
