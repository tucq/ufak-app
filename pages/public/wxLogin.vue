<template>
    <view>
        <!-- #ifdef MP-WEIXIN -->
        <view v-if="isCanUse">
            <view>
				<!-- <view class="back-btn yticon icon-zuojiantou-up" @click="navBack"></view> -->
                <view class='header'>
                    <image src='/static/basicprofile.jpg'></image>
                </view>
				
                <view class='content'>
                    <view>申请获取以下权限</view>
                    <text>获得你的公开信息(昵称，头像、地区等)</text>
                </view>

                <button class='bottom' type='primary' open-type="getUserInfo" withCredentials="true" lang="zh_CN" @getuserinfo="wxGetUserInfo">
                    授权登录
                </button>
            </view>
        </view>
        <!-- #endif -->
    </view>
</template>

<script>
	import {
	    mapMutations  
	} from 'vuex';
	
    export default {
        data() {
            return {
                SessionKey: '',
                OpenId: '',
                nickName: null,
                avatarUrl: null,
                isCanUse: uni.getStorageSync('isCanUse')||true//默认为true
            };
        },
        methods: {
			...mapMutations(['login']),
            //第一授权获取用户信息===》按钮触发
          async wxGetUserInfo() {
			   let userInfo = {};
			   let that = this;
				uni.login({
				   provider: 'weixin',
				   success: function(loginRes) {
					   uni.getUserInfo({
						   provider: 'weixin',
						   success: function(infoRes) {
							   let code = loginRes.code;
							   uni.request({
								   url: that.$baseUrl + '/sys/getOpenid',
								   data: {
									   code: code,
									   encryptedData: infoRes.encryptedData,
									   iv: infoRes.iv,
								   },
								   method: 'GET',
								   header: {
									   'content-type': 'application/json'
								   },
								   success: (res) => {
									   uni.hideLoading();
									   if(res.data.status === 1){
										   userInfo = {
											   id: res.data.userId,
											   nickname: res.data.nickname,
											   portrait: res.data.avatarUrl
										   }
										   try {
										       uni.setStorageSync('WX_TOKEN', res.data.WX_TOKEN);//同步缓存TOKEN
										   } catch (e) {}
										   
										   if(userInfo.id){
												that.login(userInfo);
												uni.navigateBack();  
										   }else{
												uni.showToast({
													title: '授权登陆失败',
													icon: 'none',
													duration: 3000
												});
										   }
										   
									   }else{
										   uni.showToast({
										   	title: '微信授权失败,请稍后重试',
										   	icon: 'none',
										   	duration: 3000
										   });
									   }
									   
								   }
							   });
						   },
						   fail(res) {
								uni.showToast({
									title: "福安康提醒您，授权通过后方可购物哦！",
									icon: 'none',
									duration: 3000,
								})
						   },
					   });
					   
				   },
				   
				});
				
            },
        },
        onLoad() {//默认加载
            // this.login();
        }
    }
</script>

<style>
    .header {
        margin: 90rpx 0 90rpx 50rpx;
        border-bottom: 1px solid #ccc;
        text-align: center;
        width: 650rpx;
        height: 300rpx;
        line-height: 450rpx;
    }

    .header image {
        width: 200rpx;
        height: 200rpx;
    }

    .content {
        margin-left: 50rpx;
        margin-bottom: 90rpx;
    }

    .content text {
        display: block;
        color: #9d9d9d;
        margin-top: 40rpx;
    }

    .bottom {
        border-radius: 80rpx;
        margin: 70rpx 50rpx;
        font-size: 35rpx;
    }
</style>
