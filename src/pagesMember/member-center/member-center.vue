<script setup lang="ts">
import type { MemberItem } from '@/types/member'
import { computed, onMounted, ref, watch } from 'vue'
import { useMemberStore } from '@/stores'
import { getLotteryCoupon, getMemberInfo } from '@/services/member'
const memberInfo = getMemberInfo()
const level = ref()
const memberInformation = ref()
const diff = ref()
const birthdayGift = ref()
const couponData = {
  isFlipped: false,
}
const levelMap = new Map([
  ['白银会员', '暂时还没有哦'],
  ['黄金会员', '满20减5优惠券1张'],
  ['铂金会员', '满20减5优惠券2张\n满50减10优惠券1张'],
  ['钻石会员', '满20减5优惠券2张\n满50减10优惠券1张'],
  ['星耀会员', '满20减5优惠券2张\n满50减10优惠券1张\n10元无门槛优惠券3张'],
])
const showToast = () => {
  uni.showModal({
    title: '会员须知',
    content:
      '当前该等级会员: ' +
      memberInformation.value.requirement +
      '\r\n' +
      memberInformation.value.treatment,
  })
}
const coupon = ref()
const variable = ref() // 初始化变量
memberInfo
  .then((result) => {
    memberInformation.value = result.data.membership
    console.log(memberInformation.value)
    diff.value = memberInformation.value.diffToNextLevel
    level.value = memberInformation.value.name
    variable.value = memberInformation.value.vipLevel
    birthdayGift.value = memberInformation.value.birthdayGift
    coupon.value = levelMap.get(level.value)
  })
  .catch((error) => {
    // 处理错误
    console.error(error)
  })
// 计算属性，根据变量的值返回相应的背景颜色
const cardColor = computed(() => {
  switch (variable.value) {
    case 0:
      return '#c2b2a4'
    case 1:
      return '#f8c9a0'
    case 2:
      return '#eca263'
    case 3:
      return '#d97f32'
    case 4:
      return '#b95b09'
  }
})
console.log(coupon)
const memberTypes = [
  {
    type: 'a1',
    text: '会员免运费',
    imgUrl: 'https://image.familystudy.cn/image/jxfruit/会员免运费1.webp',
  },
  {
    type: 'a2',
    text: '每月一分领',
    imgUrl: 'https://image.familystudy.cn/image/jxfruit/每月一分领.webp',
  },
  {
    type: 'a3',
    text: '专享免减券',
    imgUrl: 'https://image.familystudy.cn/image/jxfruit/专享免减券.webp',
  },
  {
    type: 'a4',
    text: '抽优惠券',
    imgUrl: 'https://image.familystudy.cn/image/jxfruit/抽优惠券.webp',
  },
]

const handleLottery = async () => {
  const res = await getLotteryCoupon()
  console.log(res)
  if (res.msg === 'success') {
    await uni.showModal({
      title: '抽奖结果',
      content: `恭喜你抽中满${res.data.effectivePrice}减${res.data.price}的优惠券！`,
    })
  } else {
    await uni.showModal({
      title: '抽奖结果',
      content: `谢谢惠顾`,
    })
  }
}
</script>
<template>
  <view class="box">
    <div class="membercontainer">
      <swiper-item>
        <view class="card" :style="{ backgroundColor: cardColor }" @click="showToast">
          <text class="member">{{ level }}</text>
          <view style="width: 300rpx; margin-left: 90rpx; margin-top: 40rpx">
            <text style="font-size: 20rpx; margin-top: 10rpx"
              >0目前距离升级还需要消费{{ diff }}
            </text>
            <progress :percent="0" :active="true" :border-radius="3" :stroke-width="3"></progress>
            <img src="../../static/images/jx_logo.png" class="image" />
          </view>
        </view>
      </swiper-item>
    </div>
    <view class="section">
      <view
        v-for="item in memberTypes"
        :key="item.type"
        class="navigator"
        style="padding-bottom: 15rpx"
      >
        <image :src="item.imgUrl" mode="aspectFit" style="margin-bottom: 10rpx"></image>
        <view>{{ item.text }}</view>
      </view>
    </view>
    <view class="container">
      <view class="children1">
        <text class="title1">会员免运费</text>
        <div style='display: flex;justify-content: center;align-items: center;height: 70%'>
          <span style='font-size: 1.5rem'>敬请期待</span>
        </div>
      </view>
      <view class="children1">
        <text class="title1">专享免减券</text>
        <div style='font-size: 0.9rem;display: flex;height: 70%;flex-flow: column;justify-content: center;margin-left: 4%;margin-top: 6px'>
          <div>生日礼:&ensp;{{ birthdayGift }}</div>
          <view><text>会员礼:&ensp;<text style='display: inline-flex;justify-content: start'>{{ coupon }}</text></text></view>
        </div>
      </view>
      <view class="children1">
        <text class="title1">抽优惠券</text>
        <div style="padding: 20px">
          <button style="background: #61d272; width: 80%; border-radius: 4px" @tap="handleLottery">
            点击抽取优惠券
          </button>
        </div>
      </view>
    </view>
  </view>
</template>

<style lang="scss">
.swiper_buy {
  display: flex;
  justify-content: space-between;
}
.item {
  width: 200rpx;
  height: 200rpx;
  background-color: #fff;
  margin: 10rpx;
  margin-top: -2rpx;
  border-radius: 15rpx;
}
.swiper_coupon {
  display: flex;
  justify-content: space-between;
}
.itemList {
  height: 250rpx;
  padding: 10px;
}
.couponList {
  padding: 10px; /* 调整内边距 */
  height: 150rpx;
}
.coupon {
  width: 200rpx;
  height: 120rpx;
  background-color: rgb(255, 240, 228);
  margin: 10rpx;
  margin-top: 20rpx;
  border-radius: 15rpx;
}
.card {
  width: 650rpx;
  height: 280rpx;
  border-radius: 50rpx;
  box-shadow: 0rpx 0rpx 5rpx #888888;
  margin: 20rpx auto 0;

  .member {
    position: relative;
    top: 55rpx;
    font-size: 80rpx;
    margin-left: 80rpx;
    background: rgb(255, 240, 228);
    -webkit-background-clip: text;
    color: transparent;
    display: inline-block;
  }
}

.section {
  width: 100%;
  margin-top: 10rpx;
  display: flex;
  justify-content: space-between;
  padding: 40rpx 50rpx 40rpx;

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

.container {
  display: flex;
  flex-direction: column;
  background-color: rgb(225, 225, 165);
  border-radius: 90rpx;
  margin: 20rpx auto 0;
  padding: 27rpx;

  .children1 {
    width: 100%;
    height: 250rpx;
    background-color: burlywood;
    border-radius: 60rpx;
    padding: 20rpx;
    margin: auto;
    margin-bottom: 30rpx;

    .title1 {
      margin-left: 15rpx;
    }
  }

  .children2 {
    width: 100%;
    height: 320rpx;
    background-color: burlywood;
    border-radius: 60rpx;
    padding: 20rpx;
    margin: auto;
    margin-bottom: 30rpx;

    .title1 {
      margin-left: 15rpx;
    }
  }
}

.membercontainer {
  width: 100%;
  height: 300rpx;
}

.box {
  background-color: rgb(225, 225, 165);
}

.image {
  width: 380rpx;
  height: 380rpx;
  margin-left: 240rpx;
  margin-top: -260rpx;
}
</style>
