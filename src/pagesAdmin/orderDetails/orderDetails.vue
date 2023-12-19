<script setup lang="ts">
import { OrderState, orderStateList } from '@/services/constants'
import {
  deleteMemberOrder,
  getMemberOrderById,
  getMemberOrderCancelById,
  putMemberOrderReceiptById
} from '@/services/order'
import CommentBox from '../../components/CommentBox.vue'
import { onLoad, onReady } from '@dcloudio/uni-app'
import { ref } from 'vue'
import { getPayWxPayMiniPay, wxPay} from '@/services/pay'

// 获取屏幕边界到安全区域距离
const { safeAreaInsets } = uni.getSystemInfoSync()

// 弹出层组件
const popup = ref<UniHelper.UniPopupInstance>()
// 订单取消原因
const reason = ref('')
// 复制内容
const onCopy = (orderNo: string) => {
  // 设置系统剪贴板的内容
  uni.setClipboardData({ data: String(orderNo) })
}
// 获取页面参数
const query = defineProps<{
  id: string
}>()

// 获取页面栈
const pages = getCurrentPages()

// 基于小程序的 Page 类型扩展 uni-app 的 Page
type PageInstance = Page.PageInstance & WechatMiniprogram.Page.InstanceMethods<any>

// #ifdef MP-WEIXIN
// 获取当前页面实例，数组最后一项
const pageInstance = pages.at(-1) as PageInstance

// 页面渲染完毕，绑定动画效果
onReady(() => {
  // 动画效果,导航栏背景色
  pageInstance.animate(
    '.navbar',
    [{ backgroundColor: 'transparent' }, { backgroundColor: '#f8f8f8' }],
    1000,
    {
      scrollSource: '#scroller',
      timeRange: 1000,
      startScrollOffset: 0,
      endScrollOffset: 50,
    },
  )
  // 动画效果,导航栏标题
  pageInstance.animate('.navbar .title', [{ color: 'transparent' }, { color: '#000' }], 1000, {
    scrollSource: '#scroller',
    timeRange: 1000,
    startScrollOffset: 0,
    endScrollOffset: 50,
  })
  // 动画效果,导航栏返回按钮
  pageInstance.animate('.navbar .back', [{ color: '#fff' }, { color: '#000' }], 1000, {
    scrollSource: '#scroller',
    timeRange: 1000,
    startScrollOffset: 0,
    endScrollOffset: 50,
  })
})
// #endif

// 获取订单详情
const order = ref<OrderResult>()
const getMemberOrderByIdData = async () => {
  const res = await getMemberOrderById(Number(query.id))
  order.value = res.data
  // if (
  //   [OrderState.DaiShouHuo, OrderState.DaiPingJia, OrderState.YiWanCheng].includes(
  //     order.value.orderState,
  //   )
  // )
  // {
  // getMemberOrderLogisticsByIdData()
  // }
}

// // 获取物流信息
// const logisticList = ref<LogisticItem[]>([])
// const getMemberOrderLogisticsByIdData = async () => {
//   const res = await getMemberOrderLogisticsByIdAPI(query.id)
//   logisticList.value = res.result.list
// }

onLoad(() => {
  getMemberOrderByIdData()
  isShow.value = false
})

const orderStatueText = ['待付款', '待发货', '待收货', '待评价', '已完成', '已取消']

const emit = defineEmits(['showComment'])


// 模拟发货
const onRefund = async (id: number) => {
  uni.navigateTo({
    url: `../refund/refund?orderId=${query.id}&id=${id}`
  })
  // if (isDev) {
  //   await getMemberOrderConsignmentByIdAPI(query.id)
  //   uni.showToast({ icon: 'success', title: '模拟发货完成' })
  //   // 主动更新订单状态
  //   order.value!.orderState = OrderState.DaiShouHuo
  // }
}
// 确认收货
const onOrderConfirm = () => {
  // 二次确认弹窗
  uni.showModal({
    content: '为保障您的权益，请收到货并确认无误后，再确认收货',
    confirmColor: '#27BA9B',
    success: async (success) => {
      if (success.confirm) {
        const res = await putMemberOrderReceiptById(query.id)
        // 更新订单状态
        order.value = res.data
      }
    },
  })
}
// 删除订单
const onOrderDelete = () => {
  // 二次确认
  uni.showModal({
    content: '是否删除订单',
    confirmColor: '#27BA9B',
    success: async (success) => {
      if (success.confirm) {
        await deleteMemberOrder(query.id)
        uni.redirectTo({ url: '/pagesOrder/list/list' })
      }
    },
  })
}

const isShow = ref<boolean>()

const close = () => {
  isShow.value = false
}

const onComment = (id: number) => {
  console.log("评价", id)
  isShow.value = true
  uni.setStorageSync("goodsId", id)
}

// 取消订单
const onOrderCancel = async () => {
  // 发送请求
  const res = await getMemberOrderCancelById(query.id, { cancelReason: reason.value })
  // 更新订单信息
  order.value = res.data
  // 关闭弹窗
  popup.value?.close!()
  // 轻提示
  uni.showToast({ icon: 'none', title: '订单取消成功' })
  uni.navigateBack()
}
</script>

<template>
  <CommentBox @close="close" style="position: fixed;z-index: 10" v-if="isShow"></CommentBox>
  <scroll-view
    enable-back-to-top
    scroll-y
    class="viewport"
    id="scroller"
    @scrolltolower="onScrolltolower"
  >
    <template v-if="order">
      <!-- 订单状态 -->
      <view class="overview" :style="{ paddingTop: safeAreaInsets?.top  + 'px' }">
<!--          <view class="status icon-clock">{{orderStatueText[order.orderState]}}</view>-->
<!--          &lt;!&ndash; 订单状态文字 &ndash;&gt;-->
          <view class="status"> {{ orderStateList[order.orderState].text }} </view>
      </view>
      <!-- 配送状态 -->
      <view class="shipment">
        <!-- 订单物流信息 -->
        <view v-for="item in logisticList" :key="item.id" class="item">
          <view class="message">
            {{ item.text }}
          </view>
          <view class="date"> {{ item.time }} </view>
        </view>
        <!-- 用户收货地址 -->
        <view class="locate">
          <view class="user"> {{ order.address.receiverName }} {{ order.address.receiverPhone }} </view>
          <view class="address"> {{ order.address.campus }} {{ order.address.roomAddress }} </view>
        </view>
      </view>

      <!-- 商品信息 -->
      <view class="goods">
        <view class="item">
          <navigator
            class="navigator"
            v-for="item in order.skus"
            :key="item.id"
            :url="`/pages/goods/goods?id=${item.goodsId}`"
            hover-class="none"
          >
            <image class="cover" :src="item.thumbNail"></image>
            <view class="meta">
              <view class="name ellipsis">{{ item.goodsName }}</view>
              <view class="type">{{ item.scale }}</view>
              <view class="price">
                <view class="actual">
                  <text class="symbol">¥</text>
                  <text>{{ item.price }}</text>
                </view>
              </view>
            </view>
          </navigator>
        </view>
        <!-- 合计 -->
        <view class="total">
          <view class="row">
            <view class="text">商品总价: </view>
            <view class="symbol">{{ order.totalMoney }}</view>
          </view>
          <view class="row">
            <view class="text">运费: </view>
            <view class="symbol">{{ order.postFee }}</view>
          </view>
          <view class="row">
            <view class="text">应付金额: </view>
            <view class="symbol primary">{{ order.payMoney }}</view>
          </view>
        </view>
      </view>

      <!-- 订单信息 -->
      <view class="detail">
        <view class="title">订单信息</view>
        <view class="row">
          <view class="item">
            订单编号: {{ order.orderTradeNo }} <text class="copy" @tap="onCopy(order.orderTradeNo)">复制</text>
          </view>
          <view class="item">
            订单号: {{ order.orderId }} <text class="copy" @tap="onCopy(order.orderId)">复制</text>
          </view>
          <view class="item">下单时间: {{ order.submitTime }}</view>
        </view>
      </view>

      <!-- 底部操作栏 -->
      <view class="toolbar-height" :style="{ paddingBottom: safeAreaInsets?.bottom + 'px' }"></view>
      <view class="toolbar" :style="{ paddingBottom: safeAreaInsets?.bottom + 'px' }">
        <!-- 待付款状态:展示支付按钮 -->
        <view class="button primary" v-if="order.orderState!==6" > 退款 </view>
        <view class="button delete"> 取消订单 </view>
          <view class="button">
            复制订单
          </view>
      </view>
    </template>
    <template v-else>
      <!-- 骨架屏组件 -->
      <PageSkeleton />
    </template>
  </scroll-view>
  <!-- 取消订单弹窗 -->
  <uni-popup ref="popup" type="bottom" background-color="#fff">
    <view class="popup-root">
      <view class="title">订单取消</view>
      <view class="description">
        <view class="tips">请选择取消订单的原因：</view>
        <view class="cell" v-for="item in reasonList" :key="item" @tap="reason = item">
          <text class="text">{{ item }}</text>
          <text class="icon" :class="{ checked: item === reason }"></text>
        </view>
      </view>
      <view class="footer">
        <view class="button" @tap="popup?.close?.()">取消</view>
        <view class="button primary" @tap="onOrderCancel">确认</view>
      </view>
    </view>
  </uni-popup>
</template>

<style lang="scss">
page {
  display: flex;
  flex-direction: column;
  height: 100%;
  overflow: hidden;
}

.navbar {
  width: 750rpx;
  color: #000;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 9;
  /* background-color: #f8f8f8; */
  background-color: transparent;

  .wrap {
    position: relative;

    .title {
      height: 44px;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 32rpx;
      /* color: #000; */
      color: transparent;
    }

    .back {
      position: absolute;
      left: 0;
      height: 44px;
      width: 44px;
      font-size: 44rpx;
      display: flex;
      align-items: center;
      justify-content: center;
      /* color: #000; */
      color: #fff;
    }
  }
}

.viewport {
  background-color: #f7f7f8;
}

.refund {
  margin-top: 30rpx;
}

.overview {
  display: flex;
  flex-direction: column;
  align-items: center;

  line-height: 1;
  padding-bottom: 30rpx;
  color: #000;
  background-color: rgb(255,234,189);
  background-size: cover;

  .status {
    font-size: 36rpx;
  }

  .status::before {
    margin-right: 6rpx;
    font-weight: 500;
  }

  .tips {
    margin: 30rpx 0;
    display: flex;
    font-size: 14px;
    align-items: center;

    .money {
      margin-right: 30rpx;
    }
  }

  .button-group {
    margin-top: 30rpx;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .button {
    width: 260rpx;
    height: 64rpx;
    margin: 0 10rpx;
    text-align: center;
    line-height: 64rpx;
    font-size: 28rpx;
    color: #000;
    border-radius: 68rpx;
    background-color: #fff;
  }

}

.shipment {
  line-height: 1.4;
  padding: 0 20rpx;
  margin: 20rpx 20rpx 0;
  border-radius: 10rpx;
  background-color: #fff;

  .locate,
  .item {
    min-height: 120rpx;
    padding: 30rpx 30rpx 25rpx 75rpx;
    background-size: 50rpx;
    background-repeat: no-repeat;
    background-position: 6rpx center;
  }

  .locate {
    background-image: url(https://pcapi-xiaotuxian-front-devtest.itheima.net/miniapp/images/locate.png);

    .user {
      font-size: 26rpx;
      color: #444;
    }

    .address {
      font-size: 24rpx;
      color: #666;
    }
  }

  .item {
    background-image: url(https://pcapi-xiaotuxian-front-devtest.itheima.net/miniapp/images/car.png);
    border-bottom: 1rpx solid #eee;
    position: relative;

    .message {
      font-size: 26rpx;
      color: #444;
    }

    .date {
      font-size: 24rpx;
      color: #666;
    }
  }
}

.goods {
  margin: 20rpx 20rpx 0;
  padding: 0 20rpx;
  border-radius: 10rpx;
  background-color: #fff;

  .item {
    padding: 30rpx 0;
    border-bottom: 1rpx solid #eee;

    .navigator {
      display: flex;
      margin: 20rpx 0;
    }

    .cover {
      width: 170rpx;
      height: 170rpx;
      border-radius: 10rpx;
      margin-right: 20rpx;
    }

    .meta {
      flex: 1;
      display: flex;
      flex-direction: column;
      justify-content: center;
      position: relative;
    }

    .name {
      height: 80rpx;
      font-size: 26rpx;
      color: #444;
    }

    .type {
      line-height: 1.8;
      padding: 0 15rpx;
      margin-top: 6rpx;
      font-size: 24rpx;
      align-self: flex-start;
      border-radius: 4rpx;
      color: #888;
      background-color: #f7f7f8;
    }

    .price {
      display: flex;
      margin-top: 6rpx;
      font-size: 24rpx;
    }

    .symbol {
      font-size: 20rpx;
    }

    .original {
      color: #999;
      text-decoration: line-through;
    }

    .actual {
      margin-left: 10rpx;
      color: #444;
    }

    .text {
      font-size: 22rpx;
    }

    .quantity {
      position: absolute;
      bottom: 0;
      right: 0;
      font-size: 24rpx;
      color: #444;
    }

    .button {
      width: 200rpx;
      height: 60rpx;
      text-align: center;
      justify-content: center;
      line-height: 60rpx;
      margin-left: 20rpx;
      border-radius: 60rpx;
      border: 1rpx solid #ccc;
      font-size: 26rpx;
      color: #444;
    }

    .action {
      // display: flex;
      // flex-direction: row-reverse;
      // justify-content: flex-start;


      // .button {
      //   margin: 0 auto;
      //   width: 200rpx;
      //   height: 60rpx;
      //   text-align: center;
      //   justify-content: center;
      //   line-height: 60rpx;
      //   margin-left: 20rpx;
      //   border-radius: 60rpx;
      //   border: 1rpx solid #ccc;
      //   font-size: 26rpx;
      //   color: #444;
      // }

      .primary {
        color: rgb(255, 174, 0);
        border-color: rgb(255, 174, 0);
      }
    }
  }

  .total {
    line-height: 1;
    font-size: 26rpx;
    padding: 20rpx 0;
    color: #666;

    .row {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 10rpx 0;
    }

    .symbol::before {
      content: '¥';
      font-size: 80%;
      margin-right: 3rpx;
    }

    .primary {
      color: #cf4444;
      font-size: 36rpx;
    }
  }
}

.detail {
  line-height: 1;
  padding: 30rpx 20rpx 0;
  margin: 20rpx 20rpx 0;
  font-size: 26rpx;
  color: #666;
  border-radius: 10rpx;
  background-color: #fff;

  .title {
    font-size: 30rpx;
    color: #444;
  }

  .row {
    padding: 20rpx 0;

    .item {
      padding: 10rpx 0;
      display: flex;
      align-items: center;
    }

    .copy {
      border-radius: 20rpx;
      font-size: 20rpx;
      border: 1px solid #ccc;
      padding: 5rpx 10rpx;
      margin-left: 10rpx;
    }
  }
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
    background-color: rgb(255,234,189);
  }
}

.popup-root {
  padding: 30rpx 30rpx 0;
  border-radius: 10rpx 10rpx 0 0;
  overflow: hidden;

  .title {
    font-size: 30rpx;
    text-align: center;
    margin-bottom: 30rpx;
  }

  .description {
    font-size: 28rpx;
    padding: 0 20rpx;

    .tips {
      color: #444;
      margin-bottom: 12rpx;
    }

    .cell {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 15rpx 0;
      color: #666;
    }

    .icon::before {
      content: '\e6cd';
      font-family: 'erabbit' !important;
      font-size: 38rpx;
      color: #999;
    }

    .icon.checked::before {
      content: '\e6cc';
      font-size: 38rpx;
      color: rgb(255,234,189);
    }
  }

  .footer {
    display: flex;
    justify-content: space-between;
    padding: 30rpx 0 40rpx;
    font-size: 28rpx;
    color: #444;

    .button {
      flex: 1;
      height: 72rpx;
      text-align: center;
      line-height: 72rpx;
      margin: 0 20rpx;
      color: #444;
      border-radius: 72rpx;
      border: 1rpx solid #ccc;
    }

    .primary {
      color: #fff;
      background-color: rgb(255,234,189);
      border: none;
    }
  }
}
</style>
