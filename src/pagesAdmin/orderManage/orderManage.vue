<script setup lang="ts">
import { ref } from 'vue'
import CommentBox from '../../components/CommentBox.vue'
import { onLoad, onReady } from '@dcloudio/uni-app'
import { useMemberStore } from '@/stores'
import AdminList from '@/pagesAdmin/orderManage/components/adminList.vue'
import { deliverOrder } from '@/services/adminOrder'
import { sendOrderSMS } from '../../services/adminOrder'

const memberStore = useMemberStore()

// 获取页面参数
const query = defineProps<{
  type: string
}>()
const listDom = ref()
const list = ref<any>({})
const { safeAreaInsets } = uni.getSystemInfoSync()
onLoad(() => {
  isShow.value = false
})

const isShow = ref<boolean>()
const showComment = (params) => {
  console.log(params)
  isShow.value = true
}
const onChose = ref<boolean>(false)
const close = () => {
  isShow.value = false
}
// tabs 数据
const orderTabs = ref([
  { orderState: 0, title: '全部', isRender: false },
  { orderState: 1, title: '待付款', isRender: false },
  { orderState: 2, title: '待发货', isRender: false },
  { orderState: 3, title: '待收货', isRender: false },
  { orderState: 4, title: '待评价', isRender: false },
  { orderState: 5, title: '待退款', isRender: false },
])

// 高亮下标
let activeIndex = ref(orderTabs.value.findIndex((v) => v.orderState === Number(query.type)))
// 默认渲染容器
orderTabs.value[activeIndex.value].isRender = true

const onSelected = (d) => {
  list.value = d
}

const deliverOrders = () => {
  deliverOrder(list.value).then((res) => {
    if (res.msg === 'success') {
      uni.showToast({
        title: '发货成功',
        icon: 'success',
      })
    } else {
      uni.showToast({
        title: `发货失败(${res.msg})`,
        icon: 'none',
      })
    }
  })
}
const indexChange = (e) => {
  activeIndex.value = e.detail.current
  orderTabs.value.forEach((item, index) => {
    item.isRender = index === e.detail.current
  })
  onChose.value = false
}

const sendSMS = () => {
  console.log(list.value)
  sendOrderSMS(list.value).then((res) => {
    if (res.msg === 'success') {
      uni.showToast({
        title: '短信发送成功',
        icon: 'success',
      })
    } else {
      uni.showToast({
        title: `短信发送失败(${res.msg})`,
        icon: 'none',
      })
    }
  })
}
</script>

<template>
  <view class="viewport">
    <!-- <comment-box></comment-box> -->
    <CommentBox @close="close" style="position: fixed; z-index: 10" v-if="isShow"></CommentBox>
    <!-- tabs -->
    <view class="tabs">
      <text
        class="item"
        v-for="(item, index) in orderTabs"
        :key="item.title"
        @tap="
          () => {
            activeIndex = index
            item.isRender = true
          }
        "
      >
        {{ item.title }}
      </text>
      <!-- 游标 -->
      <view class="cursor" :style="{ left: activeIndex * 16.66666 + '%' }"></view>
    </view>
    <!-- 滑动容器 -->
    <swiper class="swiper" :current="activeIndex" @change="indexChange">
      <!-- 滑动项 -->
      <swiper-item v-for="item in orderTabs" :key="item.title">
        <!-- 订单列表 -->
        <admin-list
          v-if="item.isRender"
          :order-state="item.orderState"
          :on-chose="onChose"
          @selectedItems="onSelected"
          :ref="(el) => (listDom = el)"
        />
      </swiper-item>
    </swiper>
  </view>
  <view class="toolbar-height" :style="{ paddingBottom: safeAreaInsets?.bottom + 'px' }"></view>
  <view class="toolbar" :style="{ paddingBottom: safeAreaInsets?.bottom + 'px' }">
    <!-- 待付款状态:展示支付按钮 -->
    <view class="button" @tap="deliverOrders" v-if="activeIndex === 2">批量发货 </view>
    <view class="button" @tap="sendSMS" v-if="activeIndex === 3">发短信 </view>
    <!-- <view class="button">批量发短信</view>-->
    <view
      class="button primary"
      @click="onChose = !onChose"
      v-if="activeIndex === 2 || activeIndex === 3"
    >
      多选订单
    </view>
  </view>
</template>

<style lang="scss">
page {
  height: 100%;
  overflow: hidden;
}

.viewport {
  height: 100%;
  display: flex;
  flex-direction: column;
  background-color: #fff;
}

// tabs
.tabs {
  display: flex;

  justify-content: space-around;
  line-height: 60rpx;
  margin: 0 10rpx;
  background-color: #fff;
  box-shadow: 0 4rpx 6rpx rgba(240, 240, 240, 0.6);
  position: relative;
  z-index: 9;

  .item {
    flex: 1;
    text-align: center;
    padding: 25rpx;
    font-size: 25rpx;
    color: #262626;
  }

  .cursor {
    position: absolute;
    left: 0;
    bottom: 0;
    width: 16.666%;
    height: 6rpx;
    padding: 0 50rpx;
    background-color: rgb(255, 234, 189);
    /* 过渡效果 */
    transition: all 0.4s;
  }
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

// swiper
.swiper {
  background-color: #f7f7f8;
}

.toolbar-height {
  height: 100rpx;
  box-sizing: content-box;
}

.toolbar {
  position: fixed;
  left: 0;
  right: 0;
  bottom: calc(var(--window-bottom));
  z-index: 1;

  height: 100rpx;
  padding: 0 20rpx;
  display: flex;
  align-items: center;
  flex-direction: row-reverse;
  border-top: 1rpx solid #ededed;
  border-bottom: 1rpx solid #ededed;
  background-color: #fff;
  box-sizing: content-box;

  .button {
    display: flex;
    justify-content: center;
    align-items: center;

    width: 200rpx;
    height: 72rpx;
    margin-left: 15rpx;
    font-size: 26rpx;
    border-radius: 72rpx;
    border: 1rpx solid #ccc;
    color: #444;
  }

  .delete {
    order: 4;
    color: #cf4444;
  }

  .button {
    order: 3;
  }

  .secondary {
    order: 2;
    color: rgb(255, 174, 0);
    border-color: rgb(255, 174, 0);
  }

  .primary {
    order: 1;
    color: #000;
    background-color: rgb(255, 234, 189);
  }
}
</style>
