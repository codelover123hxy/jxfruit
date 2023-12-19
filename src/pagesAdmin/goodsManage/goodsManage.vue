<script setup lang="ts">
import { getCategoryTop } from '@/services/category'
import type { CategoryTopItem } from '@/types/category'
import type { GoodsItem, GoodsResult, SkuItem, GoodsImageItem } from '@/types/goods'
import { getGoodsById, getSkuInfo } from '@/services/goods'
import { onLoad, onReachBottom } from '@dcloudio/uni-app'
import { computed, ref, watch, onMounted } from 'vue'
// import PageSkeleton from './components/PageSkeleton.vue'
// import ScalePanel from '../../components/ScalePanel.vue'
import { useMemberStore } from '@/stores'
const scrolltolower = () => {
  console.log("下拉")
  // activeIndex.value = activeIndex.value + 1
}

// 获取会员Store
const memberStore = useMemberStore()

enum SkuMode {
  Both = 1,
  Cart = 2,
  Buy = 3,
}

const navList = [
  {
    id: 1,
    name: '鲜果专区'
  },
  {
    id: 2,
    name: '今日特价'
  },
  {
    id: 3,
    name: '山姆代购'
  },
  {
    id: 4,
    name: '活动商品'
  }
]

// 获取商品详情信息
const goods = ref<GoodsResult>()
const getGoodsByIdData = async (id: number) => {
  const res = await getGoodsById(id)
  goods.value = res.data
  console.log("goodsValue", goods.value)
}

const add = async (item) => {
  if (memberStore.profile) {
    await getGoodsByIdData(item.id)
    openPopup('scale')
  } else {
    uni.showToast({
      title: '请先登录',
      icon: 'error'
    })
    setTimeout(() => {
      uni.navigateTo({
        url: '/pages/login/login'
      })
    }, 1000)
  }
}

// uni-ui 弹出层组件 ref
const popup = ref<{
  open: (type?: UniHelper.UniPopupType) => void
  close: () => void
}>()

var mode = ref<SkuMode>(SkuMode.Both)
// 弹出层条件渲染
const popupName = ref<'address' | 'service' | 'scale' | 'share'>()
const openPopup = (name: typeof popupName.value) => {
  // 修改弹出层名称
  popupName.value = name
  popup.value?.open()
}

// 获取分类列表数据
const categoryList = ref<CategoryTopItem[]>([])
const activeIndex = ref(0)
const getCategoryTopData = async () => {
  const res = await getCategoryTop()
  categoryList.value = res.data
}

onReachBottom(() => {
  console.log("下拉到底了")
})

const navName = ref<string>()

// 是否数据加载完毕
const isFinish = ref(false)
// 页面加载
onLoad(async () => {
  await Promise.all([ getCategoryTopData()])
  const type = uni.getStorageSync("type")
  console.log("type", type)
  if (type != undefined && type != "") {
    navName.value = navList[Number(type)].name
  }
  else {
    navName.value = navList[0].name
    console.log("navName", navName.value)
  }
  activeIndex.value = categoryList.value.findIndex(item => item.category === navName.value)
  isFinish.value = true
  uni.removeStorageSync("type")
})

const nav = (item) => {
  uni.navigateTo({
    url: `/pagesAdmin/goodsDetails/goodsDetails?id=${item.id}`,
  })
}
// 获取页面参数
const query = defineProps<{
  type?: number
}>()

// 提取当前二级分类数据
const subCategoryList = computed(() => {
  return categoryList.value[activeIndex.value]?.list || []
})
</script>

<template>
  <view class="viewport" v-if="isFinish">
    <!-- 分类 -->
    <view class="categories">
      <!-- 左侧：一级分类 -->
      <scroll-view class="primary" scroll-y>
        <view
          v-for="(item, index) in categoryList"
          :key="item.id"
          class="item"
          :class="{ active: index === activeIndex }"
          @tap="activeIndex = index"
        >
          <text class="name">
            {{ item.category }}
          </text>
        </view>
      </scroll-view>
      <!-- 右侧：二级分类 -->
      <scroll-view enable-back-to-top class="secondary"
                   scroll-y
                   @scrolltolower="scrolltolower"
      >
        <!-- 内容区域 -->
        <view class="panel">
          <view class="title">
            <text class="name">{{ categoryList[activeIndex].category }}</text>
            <!-- <navigator class="more" hover-class="none">全部</navigator> -->
          </view>
          <view class="section" >
            <view v-for="item in subCategoryList" :key="item.id"
              class="goods"
              @click.stop="nav(item)"
            >
              <image class="image" mode="aspectFill" :src="item.thumbNail" lazy-load></image>
              <view class="right">
                <view class="name ellipsis">{{ item.goodsName }}</view>
                <!-- <view class="name">{{ item.description }}</view> -->
                <view class="price">
                  <text class="symbol">¥</text>
                  <text class="number">{{ item.price }}</text>
                </view>
              </view>
            </view>
          </view>
        </view>
      </scroll-view>
    </view>
    <!-- uni-ui 弹出层 -->
  </view>
<!--  <PageSkeleton v-else />-->
<!--  <uni-popup ref="popup" type="bottom" background-color="#fff">-->
<!--    <ScalePanel ref="scale" :mode="mode" :goodsInfo="goods" v-if="popupName === 'scale'" @close="popup?.close()" />-->
<!--  </uni-popup>-->
</template>

<style lang="scss" scoped>
page {
  height: 100%;
  overflow: hidden;
}
.viewport {
  height: 100%;
  display: flex;
  flex-direction: column;
}
.search {
  padding: 20rpx 30rpx;
  background-color: #fff;
  .input {
    display: flex;
    align-items: center;
    justify-content: space-between;
    height: 64rpx;
    padding-left: 26rpx;
    color: #8b8b8b;
    font-size: 28rpx;
    border-radius: 32rpx;
    background-color: #f3f4f4;
  }
}
.icon-search {
  &::before {
    margin-right: 10rpx;
  }
}
/* 分类 */
.categories {
  flex: 1;
  min-height: 400rpx;
  display: flex;
}
/* 一级分类 */
.primary {
  overflow: hidden;
  width: 180rpx;
  flex: none;
  background-color: #f6f6f6;
  .item {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 96rpx;
    font-size: 26rpx;
    color: #595c63;
    position: relative;
    &::after {
      content: '';
      position: absolute;
      left: 42rpx;
      bottom: 0;
      width: 96rpx;
      border-top: 1rpx solid #e3e4e7;
    }
  }
  .active {
    background-color: #fff;
    &::before {
      content: '';
      position: absolute;
      left: 0;
      top: 0;
      width: 8rpx;
      height: 100%;
      background-color: #27ba9b;
    }
  }
}
.primary .item:last-child::after,
.primary .active::after {
  display: none;
}
/* 二级分类 */
.secondary {
  background-color: #fff;
  .carousel {
    height: 200rpx;
    margin: 0 30rpx 20rpx;
    border-radius: 4rpx;
    overflow: hidden;
  }
  .panel {
    margin: 0 30rpx 0rpx;
  }
  .title {
    height: 60rpx;
    line-height: 60rpx;
    color: #333;
    font-size: 28rpx;
    border-bottom: 1rpx solid #f7f7f8;
    .more {
      float: right;
      padding-left: 20rpx;
      font-size: 24rpx;
      color: #999;
    }
  }
  .more {
    &::after {
      font-family: 'erabbit' !important;
      content: '\e6c2';
    }
  }
  .section {
    width: 100%;
    display: flex;
    flex-wrap: wrap;
    padding: 20rpx 0;
    .goods {
      width: 150rpx;
      margin: 0rpx 20rpx 20rpx 0;
      &:nth-child(3n) {
        margin-right: 0;
      }
      .image {
        width: 150rpx;
        height: 150rpx;
      }
      .name {
        padding: 5rpx;
        font-size: 22rpx;
        color: #333;
      }
      .price {
        padding: 5rpx;
        font-size: 18rpx;
        color: #cf4444;
      }
      .number {
        font-size: 24rpx;
        margin-left: 2rpx;
      }
    }
  }
}

</style>
