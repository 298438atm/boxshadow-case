<template>
  <view class="page">
    <h3 class="title">box-show</h3>
    <div class="box_list">
      <div class="box_list_item" v-for="(item, index) in boxShadowList" :key="index" :style="{ boxShadow: item }">
        <el-button class="box_list_item_btn" type="primary" @click="handleOpen(item)">样式调整</el-button>
      </div>
    </div>
  </view>
  <BoxShadowDialog v-model="visible" :styleStr="styleStr"></BoxShadowDialog>
</template>

<script setup>
import { ref } from 'vue'
import boxShadowList from './data.js'
const visible = ref(false)
const styleStr = ref('')
const tableData = ref([])
const handleOpen = (item) => {
  styleStr.value = item
  visible.value = true
}

</script>

<style lang="scss" scoped>
.title {
  font-size: 50px;
  text-align: center;
  color: rgb(74, 74, 74);
  text-shadow: 1px -1px #fff, -1px 1px #999, -10px 10px 5px #80808080;
}

.box_list {
  display: flex;
  flex-wrap: wrap;
  width: 100%;
  justify-content: space-between;

  &_item {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 250px;
    height: 250px;
    margin: 0 50px 50px 50px;

    &_btn {
      display: none;
    }

    &:hover &_btn {
      display: block;
    }
  }

  &_item:last-child {
    margin-right: auto;
  }
}
</style>
