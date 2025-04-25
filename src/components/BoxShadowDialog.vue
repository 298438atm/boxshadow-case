<template>
  <el-dialog v-model="visible" width="80%">
    <template #header>
      修改盒子阴影<el-popover placement="bottom" trigger="hover" width="400">
        <template #reference>
          <el-icon style="padding-left: 4px; position: relative; top: 2px">
            <InfoFilled />
          </el-icon>
        </template>
        <p>1. x和y轴偏移量的值都在-100到100之间</p>
        <p>2. 模糊半径和影影面积在0-100之间</p>
        <p>3. 透明度取值在0-1之间</p>
      </el-popover>
    </template>
    <div class="container">
      <div class="container_left" :style="{ boxShadow: currentShadowCss }"></div>
      <div class="container_right">
        <el-table :data="tableData" border style="width: 100%">
          <el-table-column type="index" min-width="50" align="center">
          </el-table-column>
          <el-table-column prop="inset" label="是否展示" align="center" min-width="100">
            <template #default="{ row }">
              <el-switch v-model="row.show" active-color="#13ce66" inactive-color="#333" />
            </template>
          </el-table-column>
          <el-table-column prop="inset" label="是否内嵌" align="center" min-width="100">
            <template #default="{ row }">
              <el-switch v-model="row.inset" active-color="#13ce66" inactive-color="#333" />
            </template>
          </el-table-column>
          <el-table-column prop="x" label="x轴偏移量" min-width="180" align="center" size="mini">
            <template #default="{ row }">
              <el-input-number v-model="row.x" :min="-100" :max="100"></el-input-number>
            </template>
          </el-table-column>
          <el-table-column prop="y" label="y轴偏移量" min-width="180" align="center">
            <template #default="{ row }">
              <el-input-number v-model="row.y" :min="-100" :max="100"></el-input-number>
            </template>
          </el-table-column>
          <el-table-column prop="blur" label="模糊半径" min-width="180" align="center">
            <template #default="{ row }">
              <el-input-number v-model="row.blur" :min="-100" :max="100"></el-input-number>
            </template>
          </el-table-column>
          <el-table-column prop="spread" label="阴影面积" min-width="180" align="center">
            <template #default="{ row }">
              <el-input-number v-model="row.spread" :min="-100" :max="100"></el-input-number>
            </template>
          </el-table-column>
          <el-table-column prop="color" label="颜色" align="center" min-width="180">
            <template #default="{ row }">
              <el-color-picker v-model="row.color"></el-color-picker>
            </template>
          </el-table-column>
          <el-table-column prop="opacity" label="透明度" align="center" min-width="180">
            <template #default="{ row }">
              <el-input-number v-model="row.opacity" :min="0" :max="1" :step="0.1"></el-input-number>
            </template>
          </el-table-column>
          <el-table-column label="操作" align="center" min-width="150" fixed="right">
            <template #default="{ row, $index }">
              <el-button type="danger" icon="Delete" circle @click="handleDel($index)"></el-button>
              <el-button v-if="$index === tableData.length - 1" type="primary" icon="Plus" circle
                @click="handleAdd"></el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-input v-model="currentShadowCss" readonly style="margin-top: 10px">
          <template #append>
            <el-button :icon="CopyDocument" @click="handleCopy"></el-button>
          </template>
        </el-input>
      </div>
    </div>
  </el-dialog>
</template>

<script setup>
import { CopyDocument } from '@element-plus/icons-vue'

import { ref, computed, watch } from 'vue'
const props = defineProps({
  styleStr: {
    type: String,
    default: 'rgba(149, 157, 165, 0.2) 0 8px 24px',
  },
})
const visible = defineModel()
const tableData = ref([])
const currentShadowCss = computed(() => {
  const shadowList = tableData.value
    .filter((item) => item.show)
    .map(({ inset, x, y, blur, spread, color, opacity }) => {
      return `${inset ? 'inset' : ''
        } ${x}px ${y}px ${blur}px ${spread}px ${hexToRgba(color, opacity)}`
    })
  return shadowList.join(',')
})

const analysisShadow = (shadowCss) => {
  tableData.value = []
  const colorReg =
    /rgba?\((\d{1,3}),\s*(\d{1,3}),\s*(\d{1,3}),\s*(\d?(\.\d+)?)\)/g
  const splitReg = /,(?![^(]*\))/ // 匹配逗号，但不包含在括号中的逗号
  const cssList = shadowCss.split(splitReg)
  cssList.forEach((item) => {
    let cssData = {
      show: true,
    }
    if (item.includes('inset')) {
      cssData.inset = true
      item = item.replace('inset', '')
    }
    const color = item.match(colorReg)
    if (color) {
      const colorRGB = color[0].match(
        /rgba?\((\d{1,3}),\s*(\d{1,3}),\s*(\d{1,3}),\s*(\d?(\.\d+)?)\)/
      )
      cssData.color = rgbToHex(colorRGB[1], colorRGB[2], colorRGB[3])
      cssData.opacity = colorRGB[4]
      item = item.replace(color[0], '')
    }
    // 去前后空格
    item = item.trim()
    const shiftingList = item.split(/\s+/).map((itm) => {
      return itm.replace('px', '')
    })
      ;[1, 1, 1, 1].forEach((item, index) => {
        if (index === 0) {
          cssData.x = shiftingList[index] || 0
        } else if (index === 1) {
          cssData.y = shiftingList[index] || 0
        } else if (index === 2) {
          cssData.blur = shiftingList[index] || 0
        } else {
          cssData.spread = shiftingList[index] || 0
        }
      })
    tableData.value.push(cssData)
  })
}
const rgbToHex = (r, g, b) => {
  const toHex = (number) => {
    const hex = number.toString(16) // 转换为十六进制字符串
    return hex.length === 1 ? '0' + hex : hex // 如果是一位数，则在前面补 0
  }
  // 确保 RGB 值在有效范围内
  const red = Math.max(0, Math.min(255, r))
  const green = Math.max(0, Math.min(255, g))
  const blue = Math.max(0, Math.min(255, b))
  const hexR = toHex(red)
  const hexG = toHex(green)
  const hexB = toHex(blue)
  const hexColor = `#${hexR}${hexG}${hexB}`
  return hexColor
}
const hexToRgba = (hex, opacity) => {
  // 去除可能存在的 # 号
  hex = hex.replace('#', '')
  // 将十六进制字符串拆分为R、G和B分量
  const r = parseInt(hex.substring(0, 2), 16)
  const g = parseInt(hex.substring(2, 4), 16)
  const b = parseInt(hex.substring(4, 6), 16)
  return `rgba(${r}, ${g}, ${b}, ${opacity})`
}

const handleDel = (index) => {
  tableData.value.splice(index, 1)
}
const handleAdd = () => {
  tableData.value.push({
    color: '#000000',
    opacity: '0.2',
    x: 1,
    y: 1,
    inset: false,
    spread: 0,
    blur: 0,
  })
}

const handleCopy = () => {
  let str = `box-shadow: ${currentShadowCss.value};`
  navigator.clipboard.writeText(str).then(() => {
    ElMessage.success('复制成功')
  })
}

watch(
  () => props.styleStr,
  (newVal) => {
    analysisShadow(newVal)
  },
  { immediate: true }
)
</script>

<style lang="scss" scoped>
.container {
  padding: 60px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  overflow: hidden;

  &_left {
    flex-shrink: 0;
    width: 200px;
    height: 200px;
  }

  &_right {
    margin-left: 60px;
    flex: 1;
    overflow: hidden;
  }
}
</style>
