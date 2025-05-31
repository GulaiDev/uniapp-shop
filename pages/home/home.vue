<template>
  <view>
    <!-- 轮播图区域 -->
    <swiper :indicator-dots="true" :autoplay="true" :interval="3000" :duration="1000" :circular="true">
      <!-- 循环渲染轮播图的 item 项 -->
      <swiper-item v-for="(item, i) in swiperList" :key="i">
        <navigator class="swiper-item" :url="`/subpkg/goods_detail/goods_detail?goods_id=${item.goods_id}`">
          <!-- 动态绑定图片的 src 属性 -->
          <image :src="item.image_src"></image>
        </navigator>
      </swiper-item>
    </swiper>
    <!-- 分类导航区域 -->
    <view class="nav-list">
      <view class="nav-item" v-for="(item, i) in navList" :key="i" @click="navClickHandler(item)">
        <image class="nav-img" :src="item.image_src"></image>
      </view>
    </view>
  </view>
  <!-- 楼层区域 -->
  <view class="floor-list">
    <!-- 楼层 item 项 -->
    <view class="floor-item" v-for="(item, i) in floorList" :key="i">
      <!-- 楼层标题 -->
      <image :src="item.floor_title.image_src" class="floor-title"></image>
      <!-- 楼层图片区域 -->
      <view class="floor-img-box">
        <!-- 左侧大图片的盒子 -->
        <navigator class="left-img-box" :url="item.product_list[0].url">
          <image :src="item.product_list[0].image_src" :style="{width: item.product_list[0].image_width + 'rpx'}" mode="widthFix"></image>
        </navigator>
        <!-- 右侧 4 个小图片的盒子 -->
        <view class="right-img-box">
          <navigator class="right-img-item" v-for="(item2, i2) in filteredProductList(item.product_list)" :key="i2" :url="item2.url">
            <image :src="item2.image_src" mode="widthFix" :style="{width: item2.image_width + 'rpx'}"></image>
          </navigator>
        </view>
      </view>
    </view>
  </view>
</template>

<script setup>
import { $http } from '@escook/request-miniprogram';
import { onMounted, ref } from 'vue';
import { onLoad } from '@dcloudio/uni-app';
//轮播图数据列表
const swiperList = ref([]);
//分类导航数据列表
const navList = ref([]);
//楼顶的数据列表
const floorList = ref([]);


//去掉index===0的数据
function filteredProductList(list){
  return list.filter((_,index)=>index!==0);
}
//获取轮播图列表数据
async function getSwiperList() {
  //发起请求
  const { data } = await uni.$http.get('/api/public/v1/home/swiperdata');
  //请求失败
  if (data.meta.status !== 200) {
    return uni.$showMsg();
  }
  //请求成功
  swiperList.value = data.message;
}

//获取分类导航列表数据
async function getNavList() {
  const { data } = await uni.$http.get('/api/public/v1/home/catitems');
  if (data.meta.status != 200) {
    return uni.$showMsg();
  }
  navList.value = data.message;
}

//获取楼顶列表数据
async function getFloorList() {
  const { data } = await uni.$http.get('/api/public/v1/home/floordata');
  if (data.meta.status != 200) {
    return uni.$showMsg();
  }
  floorList.value = data.message;
  // 通过双层 forEach 循环，处理 URL 地址
    data.message.forEach(floor => {
      floor.product_list.forEach(prod => {
        prod.url = '/subpkg/goods_list/goods_list?' + prod.navigator_url.split('?')[1]
      })
    })
}

//nav-item项被点击事件处理
function navClickHandler(item) {
  if (item.name === '分类') {
    uni.switchTab({
      url: '/pages/cate/cate'
    });
  }
}

//在onLoad调用函数获取数据
onLoad(() => {
  getSwiperList();
  getNavList();
  getFloorList();
});
</script>

<style lang="scss">
.swiper-item {
  height: 330rpx;
  image {
    width: 100%;
    height: 100%;
  }
}
.nav-list {
  margin: 15px 0;
  display: flex;
  align-items: center;
  justify-content: space-around;
  .nav-img {
    width: 128rpx;
    height: 140rpx;
  }
}
.floor-title {
  height: 60rpx;
  width: 100%;
  display: flex;
}
.right-img-box {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
}

.floor-img-box {
  display: flex;
  padding-left: 10rpx;
}
</style>
