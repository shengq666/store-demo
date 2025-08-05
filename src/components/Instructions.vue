<!-- src/components/Instructions.vue -->
<template>
  <div class="instructions">
    <h3>使用说明</h3>
    <ol>
      <li><strong>添加仓库</strong>：点击"添加新仓库"创建新仓库</li>
      <li><strong>选择仓库</strong>：点击仓库可选中（蓝色边框）</li>
      <li>
        <strong>编辑仓库</strong>：选中仓库后修改属性，点击"更新仓库属性"
      </li>
      <li><strong>移动仓库</strong>：拖拽仓库主体移动位置</li>
      <li><strong>缩放仓库</strong>：使用仓库边缘的8个控制点调整大小</li>
      <li><strong>分配商品</strong>：从商品池拖拽商品到仓库</li>
      <li><strong>导出数据</strong>：使用导出功能保存配置结果</li>
    </ol>
  </div>
</template>

<script>
export default {
  name: 'Instructions'
}
</script>

<style scoped>
.instructions {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 10px;
  padding: 20px;
  margin-top: 25px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
  border: 1px solid #eee;
}

.instructions h3 {
  color: #2c3e50;
  margin-bottom: 15px;
  padding-bottom: 10px;
  border-bottom: 2px solid #3498db;
  text-align: center;
}

.instructions ol {
  padding-left: 25px;
  line-height: 1.8;
}

.instructions li {
  margin-bottom: 12px;
  padding-left: 10px;
}
</style>