<script setup lang="ts">
import { getCategoryTop } from '@/services/category'
import type { CategoryTopItem } from '@/types/category'
import type { GoodsItem, GoodsResult, SkuItem, GoodsImageItem } from '@/types/goods'
import { getGoodsById, getSkuInfo } from '@/services/goods'
import { onLoad, onReachBottom } from '@dcloudio/uni-app'
import { computed, ref } from 'vue'
import PageSkeleton from './components/PageSkeleton.vue'
import ScalePanel from '../../components/ScalePanel.vue'
const scrolltolower = () => {
  console.log("下拉")
  // activeIndex.value = activeIndex.value + 1

}
enum SkuMode {
  Both = 1,
  Cart = 2,
  Buy = 3,
}

onLoad(() => {
    goodsList.value = uni.getStorageSync("goodsInfo")

})


// 获取商品详情信息
const goods = ref<GoodsResult>()
const getGoodsByIdData = async (id) => {
  const res = await getGoodsById(id)
  goods.value = res.data
  // SKU组件所需格式
  // localdata.value = {
  //   _id: res.data.id,
  //   name: res.data.goodsName,
  //   price: res.data.price,
  //   imgUrl: res.data.imgUrl

  // }
}

const goodsList = ref<GoodsItem[]>()


const add = (item) => {
  getGoodsByIdData(item.id)
  openPopup('scale')
}

// uni-ui 弹出层组件 ref
const popup = ref<{
  open: (type?: UniHelper.UniPopupType) => void
  close: () => void
}>()


var mode = ref<SkuMode>(SkuMode.Cart)

// 弹出层条件渲染
const popupName = ref<'address' | 'service' | 'scale' | 'share'>()
const openPopup = (name: typeof popupName.value) => {
  // 修改弹出层名称
  popupName.value = name
  popup.value?.open()
}

onReachBottom(() => {
  console.log("下拉到底了")
})
// 是否数据加载完毕
const isFinish = ref(false)
// 页面加载
onLoad(async () => {
  isFinish.value = true
})

const nav = (item) => {
  uni.navigateTo({
    url: `/pages/goods/goods?id=${item.id}`
  })
}
</script>

<template>
  <view class="viewport" v-if="isFinish">
    <!-- 搜索框 -->
    <!-- <view class="search"> -->
      <!-- <view class="input">
        <text class="icon-search">女靴</text>
      </view>
    </view> -->
    <!-- 分类 -->
    <view class="categories">
      <scroll-view enable-back-to-top class="secondary"
      scroll-y
      @scrolltolower="scrolltolower"
      >
        <!-- 内容区域 -->
        <view class="panel">
          <view class="section" >
            <view v-for="item in goodsList" :key="item.id"
              class="goods"
              @click.stop="nav(item)"
            >
              <image class="image" mode="aspectFill" :src="item.thumbNail"></image>
              <view class="right">
                <view class="name ellipsis">{{ item.goodsName }}</view>
                <!-- <view class="name">{{ item.description }}</view> -->
                <view class="price">
                  <text class="symbol">¥</text>
                  <text class="number">{{ item.price }}</text>
                </view>
              </view>
              <view class="add" @click.stop="add(item)">
                +
              </view>
            </view>
          </view>
        </view>
      </scroll-view>
    </view>
    <!-- uni-ui 弹出层 -->
  </view>
  <PageSkeleton v-else />
  <uni-popup ref="popup" type="bottom" background-color="#fff">
      <ScalePanel ref="scale" :mode="mode" :goodsInfo="goods" v-if="popupName === 'scale'" @close="popup?.close()" />
  </uni-popup>
</template>

<style lang="scss">
@import './styles/category.scss';
</style>
