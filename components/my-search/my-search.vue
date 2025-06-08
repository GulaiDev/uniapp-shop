<template>
  <view class="my-search-container" :style="{ 'background-color': bgcolor }" @click="searchBoxHandler">
    <!-- 使用 view 组件模拟 input 输入框的样式 -->
    <view class="my-search-box" :style="{ 'border-radius': radius + 'px' }">
      <uni-icons type="search" size="17"></uni-icons>
      <text class="placeholder">搜索</text>
    </view>
  </view>
</template>

<script setup>
import { ref, defineProps,defineEmits } from 'vue';
import { onLoad } from '@dcloudio/uni-app';
//可用高度
const wh = ref(0);

const emit=defineEmits(['click'])

// 点击了模拟的 input 输入框
function searchBoxHandler() {
  // 触发外界通过 @click 绑定的 click 事件处理函数
  emit('click')
}

const props = defineProps({
  // 背景颜色
  bgcolor: {
    type: String,
    default: '#C00000'
  },
  // 圆角尺寸
  radius: {
    type: Number,
    // 单位是 px
    default: 18
  }
});
onLoad(() => {
  const sysInfo = uni.getSystemInfoSync();
  // 可用高度 = 屏幕高度 - navigationBar高度 - tabBar高度 - 自定义的search组件高度
  wh.value = sysInfo.windowHeight - 50;
});
</script>

<style lang="scss">
.my-search-container {
  height: 50px;
  padding: 0 10px;
  display: flex;
  align-items: center;
  .my-search-box {
    height: 36px;
    width: 100%;
    background-color: #ffffff;
    display: flex;
    align-items: center;
    justify-content: center;
    .placeholder {
      font-size: 15px;
      margin-left: 5px;
    }
  }
}
</style>
