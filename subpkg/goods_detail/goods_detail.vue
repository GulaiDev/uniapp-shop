<template>
  <view v-if="goods_info.goods_name">
    <!-- 轮播图区域 -->
    <swiper :indicator-dots="true" :autoplay="true" :interval="3000" :duration="1000" :circular="true">
      <swiper-item v-for="(item, i) in goods_info.pics" :key="i">
        <!-- 把当前点击的图片的索引，传递到 preview() 处理函数中 -->
        <image :src="item.pics_big" @click="preview(i)"></image>
      </swiper-item>
    </swiper>
    <!-- 商品信息区域 -->
    <view class="goods-info-box">
      <!-- 商品价格 -->
      <view class="price">￥{{ goods_info.goods_price }}</view>
      <!-- 信息主体区域 -->
      <view class="goods-info-body">
        <!-- 商品名称 -->
        <view class="goods-name">{{ goods_info.goods_name }}</view>
        <!-- 收藏 -->
        <view class="favi">
          <uni-icons type="star" size="18" color="gray"></uni-icons>
          <text>收藏</text>
        </view>
      </view>
      <!-- 运费 -->
      <view class="yf">快递：免运费</view>
    </view>
    <!-- 商品详情信息 -->
    <rich-text :nodes="goods_info.goods_introduce"></rich-text>
    <!-- 商品导航组件 -->
    <view class="goods_nav">
      <!-- fill 控制右侧按钮的样式 -->
      <!-- options 左侧按钮的配置项 -->
      <!-- buttonGroup 右侧按钮的配置项 -->
      <!-- click 左侧按钮的点击事件处理函数 -->
      <!-- buttonClick 右侧按钮的点击事件处理函数 -->
      <uni-goods-nav :fill="true" :options="options" :buttonGroup="buttonGroup" @click="onClick" @buttonClick="buttonClick" />
    </view>
  </view>
</template>

<script setup>
import { ref } from 'vue';
import { onLoad } from '@dcloudio/uni-app';

//商品详情对象
const goods_info = ref({});
//左侧按钮组的配置对象
const options = [
  {
    icon: 'shop',
    text: '店铺'
  },
  {
    icon: 'cart',
    text: '购物车',
    info: 2
  }
];
//右侧按钮组配置对象
const buttonGroup = [
  {
    text: '加入购物车',
    backgroundColor: '#ff0000',
    color: '#fff'
  },
  {
    text: '立即购买',
    backgroundColor: '#ffa200',
    color: '#fff'
  }
];

// 定义请求商品详情数据的方法
async function getGoodsDetail(goods_id) {
  const { data } = await uni.$http.get('/api/public/v1/goods/detail', { goods_id });
  if (data.meta.status !== 200) return uni.$showMsg();

  // 使用字符串的 replace() 方法，为 img 标签添加行内的 style 样式，从而解决图片底部空白间隙的问题
  // 使用字符串的 replace() 方法，将 webp 的后缀名替换为 jpg 的后缀名
  data.message.goods_introduce = data.message.goods_introduce.replace(/<img /g, '<img style="display:block;" ').replace(/webp/g, 'jpg');
  //为goods_info赋值
  goods_info.value = data.message;
}

//实现轮播图预览效果
function preview(i) {
  //调用uni.previewImage()方法预览图片
  uni.previewImage({
    //预览时，默认显示图片索引
    current: i,
    //所有图片的url地址的数组
    urls: goods_info.value.pics.map((x) => x.pics_big)
  });
}
//左侧按钮的点击事件处理函数
function onClick(e) {
  if (e.content.text === '购物车') {
    // 切换到购物车页面
    uni.switchTab({
      url: '/pages/cart/cart'
    });
  }
}

onLoad((options) => {
  //获取商品Id
  const goods_id = options.goods_id;
  //调用商品详情数据的方法
  getGoodsDetail(goods_id);
});
</script>

<style lang="scss">
swiper {
  height: 750rpx;

  image {
    width: 100%;
    height: 100%;
  }
}
// 商品信息区域的样式
.goods-info-box {
  padding: 10px;
  padding-right: 0;

  .price {
    color: #c00000;
    font-size: 18px;
    margin: 10px 0;
  }

  .goods-info-body {
    display: flex;
    justify-content: space-between;

    .goods-name {
      font-size: 13px;
      padding-right: 10px;
    }
    // 收藏区域
    .favi {
      width: 120px;
      font-size: 12px;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      border-left: 1px solid #efefef;
      color: gray;
    }
  }

  // 运费
  .yf {
    margin: 10px 0;
    font-size: 12px;
    color: gray;
  }
}

.goods-detail-container {
  // 给页面外层的容器，添加 50px 的内padding，
  // 防止页面内容被底部的商品导航组件遮盖
  padding-bottom: 50px;
}

.goods_nav {
  // 为商品导航组件添加固定定位
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
}
</style>
