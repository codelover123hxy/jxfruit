<template>
  <view class="viewport">
    <view class="logo">
      <image mode="aspectFit"
        src="https://image.familystudy.cn/image/jxfruit/%E6%AD%A3%E5%BC%8Flogo.webp"
      ></image>
    </view>
    <view class="login">
      <!-- 网页端表单登录 -->
      <!-- #ifdef H5 -->
      <input v-model="form.account" class="input" type="text" placeholder="请输入用户名/手机号码" />
      <input v-model="form.password" class="input" type="text" password placeholder="请输入密码" />
      <button @tap="onSubmit" class="button phone">登录</button>
      <!-- #endif -->

      <!-- 小程序端授权登录 -->
      <!-- #ifdef MP-WEIXIN -->
      <button class="button phone" v-if="!admin"
      @tap="onGetphonenumber">
        <text class="icon"></text>
        一键登录/注册
      </button>
      <view v-if="admin">
        <input v-model="form.account" class="input" type="text" placeholder="请输入用户名/手机号码" />
        <input v-model="form.password" class="input" type="text" password placeholder="请输入密码" />
        <button @tap="onSubmit" class="button phone">登录</button>
      </view>
      <!-- #endif -->

      <!-- <view class="extra">
        <view class="caption">
          <text>其他登录方式</text>
        </view>
        <view class="options"> -->
          <!-- 通用模拟登录 -->
          <!-- <button @tap="onGetphonenumberSimple">
            <text class="icon icon-phone">模拟快捷登录</text>
          </button>
        </view>
      </view> -->
      <view class="tips">
        <checkbox-group class="check">
          <checkbox @tap="check"></checkbox>
        </checkbox-group>
        您已知晓并同意 <text class="content-link" @tap="jumpPrivacy">《小程序隐私保护指引》</text></view>
    </view>
    <view class="admin">
      <text class="content-link" @tap="showAdmin">管理员登录</text>
    </view>
    <!-- #ifdef MP-WEIXIN -->
    <ws-wx-privacy id="privacy-popup"></ws-wx-privacy>
    <!-- #endif -->
  </view>
</template>
<script lang="ts">
  import type { LoginResult, LoginParams, ProfileDetail } from '@/types/user'
  import { loginByPwd, loginByWechat, registerByWechat } from '@/services/login'
  import { useMemberStore } from '@/stores'
  export default{
    data(){
      return{
        approve: false,
        admin: false,
        form: {
          account: '13123456789',
          password: ''
        }
      }
    },
    onLoad() {
      // //#ifdef MP-WEIXIN
      // wx.requirePrivacyAuthorize({
      //   success: () => {
      //     console.log('点击同意');
      //   },
      //     fail: () => {
      //       console.log('点击拒绝');
      //   },
      //     complete: () => {
      //       console.log('用户已点击');
      //   }
      // })
    },
    methods:{
      check () {
        console.log("change")
        this.approve= !this.approve
      },
      async onSubmit(){
        // const res = await postLoginAPI(form.value)
        // loginSuccess(res.result)
      },
      onSuccess(result: LoginResult) {
       // 保存会员信息
        var { userInfo } = result
        var { nickName, gender, grade, username, campus, avatarUrl, id } = userInfo
        const profile = { nickName, gender, grade, username, campus, avatarUrl, token: result.token, userId: id }
        console.log(profile)
        const memberStore = useMemberStore()
        memberStore.setProfile(profile)
        // 成功提示
        uni.showToast({ icon: 'success', title: '登录成功' })




          setTimeout(() => {
            // 页面跳转
            uni.navigateBack()
          }, 500)
      },

      jumpPrivacy() {
        wx.openPrivacyContract({
          success: () => {}, // 打开成功
          fail: () => {}, // 打开失败
        })
      },

      showAdmin() {
        this.admin = !this.admin
      },

      // //#endif
      // async onGetphonenumberSimple() {
      //   // const res = await postLoginWxMinSimpleAPI('13123456789')
      //   const query = {
      //     username:"hxy123",
      //     password:"123456"
      //   }
      //   const res = await loginByPwd(query)
      //   this.onSuccess(res.data)
      // },

      onGetphonenumber() {
        console.log(this.approve)
        if (!this.approve) {
          return uni.showToast({
            title: '请同意隐私保护指引方可登录',
            icon: 'error'
          })
        }
        let this_ = this
        uni.login({
          success: async function (res) {
            const code = res.code
            const response = await loginByWechat({ code })
            if (response.code === 200) {
              this_.onSuccess(response.data)
            }
            else {
              uni.login({
                success: async function (res) {
                  const registerRes = await registerByWechat({ code: res.code })
                  if (registerRes.code === 200) {
                    console.log(registerRes.data.openId)
                    localStorage.setItem("openId", openId)
                    this_.onSuccess(registerRes.data)
                  }
                }
              })
            }
          }
        })
      }
    }
  }
</script>

<style lang="scss">
page {
  height: 100%;
}

checkbox .wx-checkbox-input {
  border-radius: 50%;/*圆角*/ width: 35rpx;/*背景的宽*/ height: 35rpx;/*背景的高*/
}

checkbox .wx-checkbox-input.wx-checkbox-input-checked::before{
    color: black; /* 对勾颜色 白色 */
    background: transparent;
    transform:translate(-70%, -60%) scale(0.7);
    // -webkit-transform:translate(-50%, -50%) scale(1);
    width: 30rpx;
    height: 30rpx;
}

.content-link {
  color: #0D91EE;
}

.viewport {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 20rpx 40rpx;
}

.logo {
  flex: 1;
  text-align: center;
  image {
    width: 350rpx;
    height: 350rpx;
    margin-top: 15vh;
  }
}

.login {
  display: flex;

  flex-direction: column;
  height: 60vh;
  padding: 40rpx 20rpx 20rpx;

  .input {
    width: 100%;
    margin-top: 20px;
    height: 80rpx;
    font-size: 28rpx;
    border-radius: 72rpx;
    border: 1px solid #ddd;
    padding-left: 30rpx;
    margin-bottom: 20rpx;
  }

  .button {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 80rpx;
    font-size: 28rpx;
    border-radius: 72rpx;
    color: #fff;
    .icon {
      font-size: 40rpx;
      margin-right: 6rpx;
    }
  }

  .phone {
    background-color: rgb(255,234,189);
    color: black;
  }

  .wechat {
    background-color: rgb(255,234,189);
  }

  .extra {
    flex: 1;
    padding: 70rpx 70rpx 0;
    .caption {
      width: 440rpx;
      line-height: 1;
      border-top: 1rpx solid #ddd;
      font-size: 26rpx;
      color: #999;
      position: relative;
      text {
        transform: translate(-40%);
        background-color: #fff;
        position: absolute;
        top: -12rpx;
        left: 50%;
      }
    }

    .options {
      display: flex;
      justify-content: center;
      align-items: center;
      margin-top: 70rpx;
      button {
        padding: 0;
        background-color: transparent;
        &::after {
          border: none;
        }
      }
    }

    .icon {
      font-size: 24rpx;
      color: #444;
      display: flex;
      flex-direction: column;
      align-items: center;

      &::before {
        display: flex;
        align-items: center;
        justify-content: center;
        width: 80rpx;
        height: 80rpx;
        margin-bottom: 6rpx;
        font-size: 40rpx;
        border: 1rpx solid #444;
        border-radius: 50%;
      }
    }
    .icon-weixin::before {
      border-color: rgb(255,234,189);
      color: rgb(255,234,189);
    }
  }
}

.tips {
  display: flex;
  position: absolute;
  bottom: 80rpx;
  left: 50%;
  transform: translate(-50%);
  right: 20rpx;
  font-size: 22rpx;
  color: #999;
  text-align: center;
  width: 500rpx;
  align-items: center;
}
.admin {
  position: absolute;
  bottom: 20rpx;
  left: 50%;
  transform: translate(-50%);
  right: 20rpx;
  font-size: 22rpx;
  color: #999;
  text-align: center;
  width: 500rpx;
  align-items: center;
}
</style>
