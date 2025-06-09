<template>
  <view>
    <view class="goods-list">
      <view v-for="(item, i) in goodsList" :key="i" @click="gotoDetail(item)">
        <my-goods :goods="item"></my-goods>
      </view>
    </view>
  </view>
</template>

<script setup>
import { reactive, ref } from 'vue';
import { onLoad, onReachBottom, onPullDownRefresh } from '@dcloudio/uni-app';
//请求参数对象
const queryObj = reactive({
  // 查询关键词
  query: '',
  // 商品分类Id
  cid: '',
  // 页码值
  pagenum: 1,
  // 每页显示多少条数据
  pagesize: 10
});
//商品列表数据
const goodsList = ref([]);
//总数量
const total = ref(0);
// 是否正在请求数据
const isloading = ref(false);

//跳转到商品详情
function gotoDetail(item) {
  uni.navigateTo({
    url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
  });
}

//获取商品列表数据
async function getGoodsList(cb) {
  //打开节流阀
  isloading.value = true;
  //发起请求
  const { data } = await uni.$http.get('/api/public/v1/goods/search', queryObj);
  //关闭节流阀
  isloading.value = false;
  //只要数据请求完毕，就立即按需调用 cb 回调函数
  cb && cb();

  if (data.meta.status !== 200) return uni.$showMsg();
  //为数据赋值
  goodsList.value = [...goodsList.value, ...data.message.goods];
  total.value = data.message.total;
}
//加载时获取数据
onLoad((options) => {
  queryObj.query = options.query || '';
  queryObj.cid = options.cid || '';
  // 调用获取商品列表数据的方法
  getGoodsList();
});

//触底事件
onReachBottom(() => {
  // 判断是否还有下一页数据
  if (queryObj.pagenum * queryObj.pagesize >= total.value) return uni.$showMsg('数据加载完毕！');
  // 判断是否正在请求其它数据，如果是，则不发起额外的请求
  if (isloading.value) return;
  //让页码值自增加1
  queryObj.pagenum += 1;
  //重新获取数据
  getGoodsList();
});

//下拉刷新事件
onPullDownRefresh(() => {
  //重置数据
  queryObj.pagenum = 1;
  total.value = 0;
  isloading = false;
  goodsList = [];

  //重新发起请求
  getGoodsList(() => uni.stopPullDownRefresh());
});
</script>

<style lang="scss"></style>
