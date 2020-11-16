<template>
	<view>
		<page-head :title="title"></page-head>
		<view class="uni-padding-wrap">
			<view style="background:#FFF; padding:50rpx 0;">
				<view class="uni-hello-text uni-center">支付金额</text></view>
				<view class="uni-h1 uni-center uni-common-mt">
					<text class="rmbLogo">￥</text>
					<input class="price" type="digit" :value="price" @input="priceChange" />
				</view>
			</view>
			<view class="uni-btn-v uni-common-mt">
				<!-- #ifdef MP-WEIXIN || MP-QQ -->
				<button type="primary" @click="weixinPay" :loading="loading">微信支付</button>
				<!-- #endif -->
				<!-- <template v-if="providerList.length > 0">
					<button v-for="(item,index) in providerList" :key="index" @click="requestPayment(item,index)" :loading="item.loading">{{item.name}}支付</button>
				</template> -->
			</view>
		</view>
	</view>
	</view>
</template>
<script>
	export default {
		data() {
			return {
				title: 'request-payment',
				loading: false,
				price: 0.01,
				providerList: []
			}
		},
		onLoad: function() {
			uni.getProvider({
				service: "payment",
				success: (e) => {
					let providerList = [];
					e.provider.map((value) => {
						switch (value) {
							case 'alipay':
								providerList.push({
									name: '支付宝',
									id: value,
									loading: false
								});
								break;
							case 'wxpay':
								providerList.push({
									name: '微信',
									id: value,
									loading: false
								});
								break;
							default:
								break;
						}
					})
					this.providerList = providerList;
				},
				fail: (e) => {
					console.log("获取支付通道失败：", e);
				}
			});
		},
		methods: {
			weixinPay() {
				this.loading = true;
				let that = this;
				let WX_TOKEN = '';
				try {
					const token = uni.getStorageSync('WX_TOKEN');
					if (token) {
						WX_TOKEN = token;
					}
				} catch (e) {}
				uni.login({
					success: (e) => {
						uni.request({
							url: that.$baseUrl + '/pay/wxPay',
							data: {
								totalPrice: that.price,
								orderNum: "202011050006",
							},
							header: {
								'X-Access-Token': WX_TOKEN,
							},
							success: (res) => {
								this.loading = false;
								if (res.statusCode !== 200) {
									uni.showModal({
										content: "支付失败，请重试！",
										showCancel: false
									});
									return;
								}
								if (res.data.success) {
									let paymentData = res.data.result;
									uni.requestPayment({
										timeStamp: paymentData.timeStamp,
										nonceStr: paymentData.nonceStr,
										package: paymentData.package,
										signType: paymentData.signType,
										paySign: paymentData.paySign,
										success: (res) => {
											uni.showToast({
												title: "感谢您的测试!"
											})
										},
										fail: (res) => {
											uni.showModal({
												content: "支付失败,原因为: " + res.errMsg,
												showCancel: false
											})
										},
										complete: () => {
											this.loading = false;
										}
									})
								} else {
									uni.showModal({
										content: res.data.message,
										showCancel: false
									})
								}
							},
							fail: (e) => {
								console.log("fail-1", e);
								this.loading = false;
								uni.showModal({
									content: "支付失败,原因为: " + e.errMsg,
									showCancel: false
								})
							}
						})
					},
					fail: (e) => {
						console.log("fail-2", e);
						this.loading = false;
						uni.showModal({
							content: "支付失败,原因为: " + e.errMsg,
							showCancel: false
						})
					}
				})
			},
			// async requestPayment(e, index) {
			// 	console.log(JSON.stringify(e),index);
			// 	this.providerList[index].loading = true;
			// 	let orderInfo = await this.getOrderInfo(e.id);
			// 	console.log("得到订单信息", orderInfo);
			// 	if (orderInfo.statusCode !== 200) {
			// 		console.log("获得订单信息失败", orderInfo);
			// 		uni.showModal({
			// 			content: "获得订单信息失败",
			// 			showCancel: false
			// 		})
			// 		return;
			// 	}
			// 	uni.requestPayment({
			// 		provider: e.id,
			// 		orderInfo: orderInfo.data,
			// 		success: (e) => {
			// 			console.log("success", e);
			// 			uni.showToast({
			// 				title: "感谢您的赞助!"
			// 			})
			// 		},
			// 		fail: (e) => {
			// 			console.log("fail", e);
			// 			uni.showModal({
			// 				content: "支付失败,原因为: " + e.errMsg,
			// 				showCancel: false
			// 			})
			// 		},
			// 		complete: () => {
			// 			this.providerList[index].loading = false;
			// 		}
			// 	})
			// },
			// getOrderInfo(e) {
			// 	let appid = "";
			// 	// #ifdef APP-PLUS
			// 	appid = plus.runtime.appid;
			// 	// #endif
			// 	console.log("appid="+appid);
			// 	let url = 'https://demo.dcloud.net.cn/payment/?payid=' + e + '&appid=' + appid + '&total=' + this.price;
			// 	return new Promise((res) => {
			// 		uni.request({
			// 			url: url,
			// 			success: (result) => {
			// 				res(result);
			// 			},
			// 			fail: (e) => {
			// 				res(e);
			// 			}
			// 		})
			// 	})
			// },
			priceChange(e) {
				console.log(e.detail.value)
				this.price = e.detail.value;
			}
		}
	}
</script>

<style>
	.rmbLogo {
		font-size: 40rpx;
	}

	button {
		background-color: #00aa00;
		color: #ffffff;
	}

	.uni-h1.uni-center {
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: flex-end;
	}

	.price {
		border-bottom: 1px solid #eee;
		width: 150rpx;
		/* height: 50rpx; */
		padding-left: 15rpx;
		/* padding-bottom: 4rpx; */
	}

	.ipaPayBtn {
		margin-top: 30rpx;
	}
</style>
