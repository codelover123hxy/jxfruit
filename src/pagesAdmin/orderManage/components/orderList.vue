<script setup lang="ts">
import { OrderState } from '@/services/constants'
import { orderStateList } from '@/services/constants'
import { putMemberOrderReceiptById } from '@/services/order'
import { deleteMemberOrder } from '@/services/order'
import { getMemberOrder } from '@/services/order'

import { getPayWxPayMiniPay, wxPay } from '@/services/pay'
import { useMemberStore } from '@/stores/index'
import type { OrderItem } from '@/types/order'
import type { OrderListParams } from '@/types/order'
import {onMounted, ref, watch} from 'vue'

const memberStore = useMemberStore()
// 获取屏幕边界到安全区域距离
const { safeAreaInsets } = uni.getSystemInfoSync()

// 定义 porps
const props = defineProps<{
  orderState: number
  onChose: Boolean
}>()

// 请求参数
const queryParams: Required<OrderListParams> = {
  pageNum: 1,
  pageSize: 5,
  orderState: props.orderState,
}

const emit = defineEmits(['showComment'])

// 获取订单列表
const orderList = ref<OrderItem[]>([])
// 是否加载中标记，用于防止滚动触底触发多次请求
const isLoading = ref(false)

const getMemberOrderData = async () => {
  // 如果数据出于加载中，退出函数
  if (isLoading.value) return
  // 退出分页判断
  if (isFinish.value === true) {
    return uni.showToast({ icon: 'none', title: '没有更多数据~' })
  }
  // 发送请求前，标记为加载中
  isLoading.value = true
  // 发送请求
  const res = await getMemberOrder(queryParams)
  // 发送请求后，重置标记
  isLoading.value = false
  // 数组追加
  orderList.value.push(...res.data.records)
  // // 分页条件
  if (queryParams.pageNum < res.data.pages) {
    //   // 页码累加
    queryParams.pageNum++
  } else {
    //   // 分页已结束
    isFinish.value = true
  }

  console.log("res", res.data.records)
}

onMounted(() => {
  getMemberOrderData()
})



// 订单支付
const onOrderPay = async (myOrder) => {
  const res = await wxPay({
    amount: Math.round(myOrder.payMoney * 100),
    orderId: myOrder.orderId
  })
  wx.requestPayment({
    timeStamp: res.data.timeStamp, // 时间戳，从1970年1月1日00:00:00至今的秒数，即当前的时间
    nonceStr: res.data.nonceStr,   // 随机字符串，长度为32个字符以下。
    package: res.data.package,     // 统一下单接口返回的 prepay_id 参数值，格式如“prepay_id=*”
    signType: res.data.signType,   // 签名算法类型，默认为 MD5，支持RSA等其他加密算法
    paySign: res.data.paySign,     // 签名，详见签名生成算法
    success: function (response) {
      // 支付成功后的回调函数， res.errMsg = 'requestPayment:ok'
      console.log(response)
      // 更新订单状态
      const order = orderList.value.find((v) => v.orderId === id)
      order!.orderState = OrderState.DaiFaHuo
      // uni.redirectTo({ url: `/pagesOrder/payment/payment?id=${query.id}` })
    },
    fail: function (response) {
      console.log(response)
      // 支付失败或取消支付后的回调函数， res.errMsg = 'requestPayment:fail cancel' 取消支付；res.errMsg = 'requestPayment:fail (detail error message)'
    }
  })
}

// 确认收货
const onOrderConfirm = (id: number) => {
  uni.showModal({
    content: '为保障您的权益，请收到货并确认无误后，再确认收货',
    confirmColor: '#27BA9B',
    success: async (res) => {
      if (res.confirm) {
        console.log("id", id)
        await putMemberOrderReceiptById(id)
        uni.showToast({ icon: 'success', title: '确认收货成功' })
        orderList.value = orderList.value.filter((v) => v.orderId !== id)
      }
    }
  })
}

const onComment = (item: any) => {
  if (item.isCommented === 0) {
    emit('showComment', true)
    uni.setStorageSync("goodsId", item.goodsId)
    uni.setStorageSync("skuId", item.id)
    uni.setStorageSync("orderId", item.orderId)
  }
}

// 删除订单
const onOrderDelete = (id: number) => {
  uni.showModal({
    content: '你确定要删除该订单？',
    confirmColor: '#27BA9B',
    success: async (res) => {
      if (res.confirm) {
        console.log("id", id)
        await deleteMemberOrder(id)
        // 删除成功，界面中删除订单
        const index = orderList.value.findIndex((v) => v.id === id)
        orderList.value.splice(index, 1)
      }
    },
  })
}

// 是否分页结束
const isFinish = ref(false)
// 是否触发下拉刷新
const isTriggered = ref(false)
// 自定义下拉刷新被触发
const onRefresherrefresh = async () => {
  // 开始动画
  isTriggered.value = true
  // 重置数据
  queryParams.pageNum = 1
  orderList.value = []
  isFinish.value = false
  // 加载数据
  await getMemberOrderData()
  // 关闭动画
  isTriggered.value = false
}

const toEditOrder = async (order) => {
  await uni.navigateTo({
    url: `/pagesAdmin/orderDetails/orderDetails?id=${order.orderId}`
  })
}

</script>

<template>
  <scroll-view
    enable-back-to-top
    scroll-y
    class="orders"
    refresher-enabled
    :refresher-triggered="isTriggered"
    @refresherrefresh="onRefresherrefresh"
    @scrolltolower="getMemberOrderData"
  >
    <view class="card" v-for="order in orderList" :key="order.orderId">
      <!-- 订单信息 -->
      <view class="status">
        <text class="date">{{ order.submitTime }}</text>
        <!-- 订单状态文字 -->
        <text>{{ orderStateList[order.orderState].text }}</text>
        <!-- 待评价/已完成/已取消 状态: 展示删除订单 -->
        <text
          v-if="order.orderState >= OrderState.DaiPingJia"
          class="icon-delete"
          @tap="onOrderDelete(order.orderId)"
        >
        </text>
      </view>
      <!-- 商品信息，点击商品跳转到订单详情，不是商品详情 -->
      <navigator
        v-for="item in order.skus"
        :key="item.id"
        class="goods"
        :url="`/pagesAdmin/orderDetails/orderDetails?id=${order.orderId}`"
        hover-class="none"
      >
        <view class="cover">
          <image class="image" mode="aspectFit" :src="item.thumbNail"></image>
        </view>
        <view class="meta">
          <view class="name">订单号: {{ item.orderId }}</view>
          <view class="sku-title">
            <view class="name ellipsis">{{ item.goodsName }}</view>

            <view class="num">x{{ item.num }}</view>
          </view>

          <view class="type">{{ item.scale }}</view>
          <view style="display: flex;align-items: center;justify-content: space-between;">
            <view class="single-price"> ￥ {{ item.price }} </view>

            <view
              v-if="order.orderState === OrderState.DaiPingJia"
              class="button primary"
              @tap.stop="onComment(item)"
            >

            </view>
          </view>
        </view>
      </navigator>
      <!-- 支付信息 -->
      <view class="payment">
        <text class="quantity">共{{ order.totalNum }}件商品</text>
        <text>实付</text>
        <text class="amount"> <text class="symbol">¥</text>{{ order.payMoney }}</text>
      </view>
      <!-- 订单操作按钮 -->
      <view class="action">
         <checkbox v-if="props.onChose.valueOf()"></checkbox>
          <view class="button primary" @tap="toEditOrder(order)">编辑订单</view>
      </view>
    </view>
    <!-- 底部提示文字 -->
    <view class="loading-text" :style="{ paddingBottom: safeAreaInsets?.bottom + 'px' }">
      {{ isFinish ? '没有更多数据~' : '正在加载...' }}
    </view>
  </scroll-view>
</template>

<style lang="scss">
// 订单列表
.orders {
  .card {
    min-height: 100rpx;
    padding: 20rpx;
    margin: 20rpx 20rpx 0;
    border-radius: 10rpx;
    background-color: #fff;

    &:last-child {
      padding-bottom: 40rpx;
    }
  }

  .status {
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 28rpx;
    color: #999;
    margin-bottom: 15rpx;

    .date {
      color: #666;
      flex: 1;
    }

    .primary {
      color: #ff9240;
    }

    .icon-delete {
      line-height: 1;
      margin-left: 10rpx;
      padding-left: 10rpx;
      border-left: 1rpx solid #e3e3e3;
    }
  }

  .goods {
    display: flex;
    margin-bottom: 20rpx;

    .cover {
      width: 170rpx;
      height: 170rpx;
      margin-right: 20rpx;
      border-radius: 10rpx;
      overflow: hidden;
      position: relative;
      .image {
        width: 170rpx;
        height: 170rpx;
      }
    }

    .quantity {
      position: absolute;
      bottom: 0;
      right: 0;
      line-height: 1;
      padding: 6rpx 4rpx 6rpx 8rpx;
      font-size: 24rpx;
      color: #fff;
      border-radius: 10rpx 0 0 0;
      background-color: rgba(0, 0, 0, 0.6);
    }

    .meta {
      flex: 1;
      display: flex;
      flex-direction: column;
      justify-content: center;
    }

    .name {
      height: 80rpx;
      font-size: 26rpx;
      color: #444;
    }

    .type {
      line-height: 1.8;
      padding: 0 15rpx;
      margin-top: 10rpx;
      font-size: 24rpx;
      align-self: flex-start;
      border-radius: 4rpx;
      color: #888;
      background-color: #f7f7f8;
    }

    .single-price {
      margin-top: 30rpx;
      width: fit-content;
      font-size: 35rpx;
      color: #ff9240;
    }

    .sku-title{
      display: flex;
      justify-content: space-between;
    }

    .num {
      font-size: 30rpx;
      width:45rpx;
      height: 45rpx;
      border-radius: 5rpx;
      border: 1rpx solid gainsboro;
      text-align: center;
      line-height: 45rpx;
    }

    .more {
      flex: 1;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 22rpx;
      color: #333;
    }

    .button {
      width: 180rpx;
      height: 60rpx;
      display: flex;
      justify-content: center;
      align-items: center;
      margin-left: 20rpx;
      border-radius: 60rpx;
      border: 1rpx solid #ccc;
      font-size: 26rpx;
      color: #444;
    }
  }

  .payment {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    line-height: 1;
    padding: 20rpx 0;
    text-align: right;
    color: #999;
    font-size: 28rpx;
    border-bottom: 1rpx solid #eee;

    .quantity {
      font-size: 24rpx;
      margin-right: 16rpx;
    }

    .amount {
      color: #444;
      margin-left: 6rpx;
    }

    .symbol {
      font-size: 20rpx;
    }
  }

  .action {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    padding-top: 20rpx;

    .button {
      width: 180rpx;
      height: 60rpx;
      display: flex;
      justify-content: center;
      align-items: center;
      margin-left: 20rpx;
      border-radius: 60rpx;
      border: 1rpx solid #ccc;
      font-size: 26rpx;
      color: #444;
    }

    .secondary {
      color: black;
      border-color: rgb(255,234,189);
    }

    .primary {
      color: black;
      background-color: rgb(255,234,189);
      border-color: rgb(255,234,189);
    }
  }

  .loading-text {
    text-align: center;
    font-size: 28rpx;
    color: #666;
    padding: 20rpx 0;
  }
}
</style>
