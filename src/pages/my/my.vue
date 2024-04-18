
<script lang="ts" setup>
//import profile from '@/components/profile.vue'

import type { MemberItem } from '@/types/member'
import { getMemberInfo } from '@/services/member'
import { onMounted, ref } from 'vue'
import { useMemberStore } from '@/stores'
import { useRoleStore } from '@/stores/index'
const memberStore = useMemberStore()
const roleStore = useRoleStore()


const member = ref<MemberItem>()

const getMemberData = async () => {
  const res = await getMemberInfo()
  member.value = res.data
}
const priviledges = ref()

onMounted(async () => {
  console.log("role", roleStore.role)
  if (memberStore.profile) {
    await getMemberData()
  } else {
    member.value = {
      couponNum: 0,
      couponCodeNum: 0,
      vipLevel: 0,
      points: 0
    }
  }
  console.log(member.value)
  priviledges.value = [
    {
      id: 1,
      name: '优惠券',
      title: `${member.value?.couponNum}张`,
      type: 0
    },
    {
      id: 2,
      name: '核销券',
      title:`${member.value?.couponCodeNum}张`,
      type: 1
    },
    {
      id: 3,
      name: '会员中心',
      title: `VIP${member.value?.vipLevel}`,
      type: 1
    },
  ]
})
// 获取屏幕边界到安全区域距离
const { safeAreaInsets } = uni.getSystemInfoSync()
const orderTypes = [
  { type: '1', text: '待付款', icon: 'icon-currency' },
  { type: '2', text: '待发货', icon: 'icon-gift' },
  { type: '3', text: '待收货', icon: 'icon-check' },
  { type: '4', text: '待评价', icon: 'icon-comment' },
]
const memberTypes = [
  {
    type: '1', text: '会员免运费',
    imgUrl: 'https://image.familystudy.cn/image/jxfruit/会员免运费1.webp'
  },
  {
    type: '2',
    text: '每月一分领',
    imgUrl: 'https://image.familystudy.cn/image/jxfruit/每月一分领.webp'
  },
  {
    type: '3',
    text: '专享免减券',
    imgUrl: 'https://image.familystudy.cn/image/jxfruit/专享免减券.webp'
  },
  {
    type: '4',
    text: '抽优惠券',
    imgUrl: 'https://image.familystudy.cn/image/jxfruit/抽优惠券.webp'
  }
]
</script>

<template>
  <view class="viewport">
    <view class="profile" :style="{ paddingTop: safeAreaInsets!.top + 'px' }">
      <!-- 情况1：已登录 -->
      <view class="overview" v-if="memberStore.profile">
        <navigator url="/pagesMember/profile/profile" hover-class="none">
          <image class="avatar" :src="memberStore.profile.avatarUrl" mode="aspectFill"></image>
        </navigator>
        <view class="meta">
          <view class="nickname">
            {{ memberStore.profile.nickName }}
          </view>
          <navigator class="extra" url="/pagesMember/profile/profile" hover-class="none">
            <text class="update">更新头像昵称</text>
          </navigator>
        </view>
      </view>
      <!-- 情况2：未登录 -->
      <view class="overview" v-else>
        <navigator url="/pages/login/login" hover-class="none">
          <image class="avatar gray" mode="aspectFit"
            src="https://image.familystudy.cn/image/jxfruit/%E6%AD%A3%E5%BC%8Flogo.webp"></image>
        </navigator>
        <view class="meta">
          <navigator url="/pages/login/login" hover-class="none" class="nickname">
            未登录
          </navigator>
          <view class="extra">
            <text class="tips">点击登录账号</text>
          </view>
        </view>
      </view>
      <navigator class="settings" url="/pagesMember/settings/settings" hover-class="none">
        设置
      </navigator>
    </view>

    <view class="orders member">
      <view class="title" style="padding-left: 20rpx;">
         会员中心
        <navigator class="navigator"
        v-if="roleStore.role === 1"
        url="/pagesMember/member-center/member-center" hover-class="none">
          全部<text class="icon-right"></text>
        </navigator>
        <navigator class="navigator"
        v-if="roleStore.role === 2"
        url="/pagesAdmin/goodsManage/goodsManage" hover-class="none">
          商品管理<text class="icon-right"></text>
        </navigator>
      </view>
      <view class="section">
        <!-- 订单 -->
        <navigator v-for="item in memberTypes" :key="item.type"  class="navigator"
          hover-class="none" style=" padding-bottom: 15rpx;">
          <image :src="item.imgUrl" mode="aspectFit" style="margin-bottom: 10rpx;"></image>
          <view style="">{{ item.text }}</view>
        </navigator>
        <!-- 客服 -->
        <!-- #ifdef MP-WEIXIN -->
        <!-- <button class="contact icon-handset" open-type="contact">售后</button> -->
        <!-- #endif -->
      </view>
    </view>

    <!-- 我的订单 -->
    <view class="orders">
      <view class="title">
        我的订单

        <navigator class="navigator"
          :url="memberStore.profile ? '/pagesOrder/list/list?type=0' : '/pages/login/login'"
          hover-class="none"
          v-if="roleStore.role === 1"
        >
          全部<text class="icon-right"></text>
        </navigator>
        <navigator class="navigator"
          v-if="roleStore.role === 2"
          :url="memberStore.profile ? '/pagesAdmin/orderManage/orderManage' : '/pages/login/login'" hover-class="none">
          订单管理<text class="icon-right"></text>
        </navigator>
      </view>
      <view class="section">
        <!-- 订单 -->
        <navigator v-for="item in orderTypes" :key="item.type" :class="item.icon" :url="memberStore.profile ?
          `/pagesOrder/list/list?type=${item.type}`
          : '/pages/login/login'" class="navigator" hover-class="none">
          {{ item.text }}
        </navigator>

      </view>
    </view>
    <view class="orders">
      <view class="title">
        我的权益
        <navigator v-if="roleStore.role === 1"
        class="navigator" :url="memberStore.profile ?
          '/pagesMember/priviledge/priviledge' : '/pages/login/login'" hover-class="none">
          全部<text class="icon-right"></text>
        </navigator>
        <navigator class="navigator"
          v-if="roleStore.role === 2"
          :url="memberStore.profile ?
          '/pagesAdmin/couponManage/couponManage' : '/pages/login/login'" hover-class="none">
          优惠券管理<text class="icon-right"></text>
        </navigator>
        <navigator class="navigator"
          v-if="roleStore.role === 2"
          :url="memberStore.profile ?
          '/pagesAdmin/qrCodeVerify/qrCodeVerify' : '/pages/login/login'" hover-class="none">
          购物券管理<text class="icon-right"></text>
        </navigator>
      </view>
      <view class="section">
        <!-- 权益 -->
        <navigator v-for="item in priviledges" :key="item.id" :url="memberStore.profile ?
          `/pagesMember/priviledge/priviledge?type=${item.type}` :
          '/pages/login/login'" class="navigator" hover-class="none">
          <view class="priviledge-title">
            {{ item.title }}
          </view>
          <view class="priviledge-name">
            {{ item.name }}
          </view>
        </navigator>
        <button class="contact icon-handset" open-type="contact">专属客服</button>
      </view>
    </view>
  </view>
</template>

<style lang="scss">
page {
  height: 100%;
  padding: 0;
  overflow: hidden;
  background-image: linear-gradient(rgb(255, 255, 246), rgb(255, 255, 236));
}

.priviledge-title {
  font-size: 35rpx;
  height: 50rpx;
}

.priviledge-name {
  font-size: 24rpx;
  margin-top: 10rpx;
}

.viewport {
  height: 100%;
  width: 100%;
  margin-top: 0;
}

/* 用户信息 */
.profile {
  background-color: rgb(255, 234, 189);
  height: 280rpx;
  color: black;
  position: relative;
  margin: 0 auto;
  padding-top: 100rpx;
  border-bottom-left-radius: 80rpx;
  border-bottom-right-radius: 80rpx;

  .overview {
    display: flex;
    height: 120rpx;
    padding: 0rpx 36rpx;
    margin-top: 50rpx;
    color: #fff;
    color: black;
  }

  .avatar {
    width: 120rpx;
    height: 120rpx;
    border-radius: 50%;
    background-color: #eee;
  }

  .gray {
    filter: grayscale(100%);
  }

  .meta {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: flex-start;
    line-height: 30rpx;
    padding: 16rpx 0;
    margin-left: 20rpx;

    .nickname {
      max-width: 180rpx;
      margin-bottom: 16rpx;
      font-size: 30rpx;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }

    .extra {
      display: flex;
      font-size: 20rpx;
    }
  }



  .tips {
    font-size: 22rpx;
  }

  .update {
    padding: 3rpx 10rpx 1rpx;
    color: rgba(255, 255, 255, 0.8);
    color: black;
    border: 1rpx solid rgba(0, 0, 0, 0.8);
    margin-right: 10rpx;
    border-radius: 30rpx;
  }

  .settings {
    position: absolute;
    bottom: 20rpx;
    right: 40rpx;
    font-size: 30rpx;
    color: #fff;
    color: black;
  }
}



/* 我的订单 */
.orders {
  position: relative;
  z-index: 99;
  padding: 30rpx;
  margin: 50rpx 20rpx 0;
  background-color: #fff;
  border-radius: 10rpx;
  box-shadow: 0 4rpx 6rpx rgba(240, 240, 240, 0.6);

  .member {
    background-color: rgb(255, 240, 228);
  }

  .title {
    height: 40rpx;
    line-height: 40rpx;
    font-size: 28rpx;
    color: #1e1e1e;


    .navigator {
      font-size: 24rpx;
      color: #939393;
      float: right;

    }
  }

  .section {
    width: 100%;
    display: flex;
    justify-content: space-between;
    padding: 40rpx 20rpx 40rpx;

    .navigator,
    .contact {
      text-align: center;
      font-size: 24rpx;
      color: #333;

      &::before {
        display: block;
        font-size: 60rpx;
        color: #ff9545;
      }

      &::after {
        border: none;
      }
    }

    .contact {
      padding: 0;
      margin: 0;
      border: 0;
      background-color: transparent;
      line-height: inherit;
    }
  }
}

.member {
  background-color: burlywood;
  padding: 10rpx 10rpx 10rpx;

  border-radius: 20rpx;

  .title {
    color: white;
    font-weight: bold;
  }

  .section {
    background: white;
    border-radius: 15rpx;
  }
}
</style>
