<template>
  <view class="search-box">
    <!-- 使用 uni-ui 提供的搜索组件 -->
    <uni-search-bar @input="input" :radius="100" cancelButton="none"></uni-search-bar>
  </view>
  <!-- 搜索建议列表 -->
  <view class="sugg-list" v-if="searchResults.length !== 0">
    <view class="sugg-item" v-for="(item, i) in searchResults" :key="i" @click="gotoDetail(item.goods_id)">
      <view class="goods-name">{{ item.goods_name }}</view>
      <uni-icons type="arrowright" size="16"></uni-icons>
    </view>
  </view>
  <!-- 搜索历史 -->
  <view class="history-box" v-else>
    <!-- 标题区域 -->
    <view class="history-title">
      <text>搜索历史</text>
      <uni-icons type="trash" size="17" @click="cleanHistory"></uni-icons>
    </view>
    <!-- 列表区域 -->
    <view class="history-list">
      <uni-tag :text="item" v-for="(item, i) in historys" :key="i" @click="gotoGoodsList(item)"></uni-tag>
    </view>
  </view>
</template>

<script setup>
import { ref, computed } from 'vue';
import { onLoad } from '@dcloudio/uni-app';

// 延时器的 timerId
let timer = null;
// 搜索关键词
const kw = ref('');
// 搜索结果列表
const searchResults = ref([]);
//搜索关键词的历史记录
const historyList = ref([]);

const historys = computed(() => [...historyList.value].reverse());

//根据关键词，搜索商品建议列表
async function getSearchList() {
  if (kw.value === '') {
    searchResults.value = [];
    return;
  }
  //发起请求，获取搜索建议列表
  const { data } = await uni.$http.get('/api/public/v1/goods/qsearch', { query: kw.value });
  if (data.meta.status !== 200) return uni.$showMsg();
  searchResults.value = data.message;
  // 1. 查询到搜索建议之后，调用 saveSearchHistory() 方法保存搜索关键词
  saveSearchHistory();
}

//保存搜索关键字
function saveSearchHistory() {
  const set = new Set(historyList.value);
  set.delete(kw.value);
  set.add(kw.value);
  historyList.value = Array.from(set);
  // 调用 uni.setStorageSync(key, value) 将搜索历史记录持久化存储到本地
  uni.setStorageSync('kw', JSON.stringify(historyList.value));
}

function gotoDetail(goods_id) {
  uni.navigateTo({
    // 指定详情页面的 URL 地址，并传递 goods_id 参数
    url: '/subpkg/goods_detail/goods_detail?goods_id=' + goods_id
  });
}

//点击跳转到商品列表页面
function gotoGoodsList(kw){
  uni.navigateTo({
    url:'/subpkg/goods_list/goods_list?query=' + kw
  })
}
//去除历史记录
function cleanHistory() {
  // 清空 historyList中保存的搜索历史
  historyList.value = [];
  // 清空本地存储中记录的搜索历史
  uni.setStorageSync('kw', '[]');
}

function input(e) {
  //清除 timer 对应的延时器
  clearTimeout(timer);
  // 重新启动一个延时器，并把 timerId 赋值给 timer
  timer = setTimeout(() => {
    kw.value = e;
    //根据关键词，查询搜索建议列表
    getSearchList();
  }, 500);
}

onLoad(() => {
  historyList.value = JSON.parse(uni.getStorageSync('kw') || '[]');
});
</script>

<style lang="scss">
.search-box {
  position: sticky;
  top: 0;
  z-index: 999;
}
.sugg-list {
  padding: 0 5px;

  .sugg-item {
    font-size: 12px;
    padding: 13px 0;
    border-bottom: 1px solid #efefef;
    display: flex;
    align-items: center;
    justify-content: space-between;

    .goods-name {
      // 文字不允许换行（单行文本）
      white-space: nowrap;
      // 溢出部分隐藏
      overflow: hidden;
      // 文本溢出后，使用 ... 代替
      text-overflow: ellipsis;
      margin-right: 3px;
    }
  }
}
.history-box {
  padding: 0 5px;

  .history-title {
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 40px;
    font-size: 13px;
    border-bottom: 1px solid #efefef;
  }

  .history-list {
    display: flex;
    flex-wrap: wrap;

    .uni-tag {
      margin-top: 5px;
      margin-right: 5px;
    }
  }
}
</style>
