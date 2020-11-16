<template>
	<view class="content">
		<text class="success-icon yticon icon-xuanzhong2"></text>
		<text class="tit">付款失败</text>
		<text class="tit2">请在{{minutes}}:{{seconds}}内完成付款</text>
		<text class="tit2">否则订单会被系统取消</text>
		
		<view class="btn-group">
			<button type="default" @click="payfor" class="mix-btn">去付款</button>
			<button type="default" @click="seeOrder" class="mix-btn">查看订单</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				expiresTime:0,//失效时间
				minutes:'',//倒计时 分
				seconds:'',//倒计时 秒
				orderId:null,
			}
		},
		onLoad(option){
			this.expiresTime = Math.floor(option.createTime) + 30 * 60 * 1000;//创建时间加上30分钟即为失败时间
			this.orderId = option.orderId;
		    this.countdown();
		},
		methods: {
			countdown(){
				let milliseconds = this.expiresTime - new Date().getTime();
				let d = new Date(milliseconds);
				let second = this.zeroize(d.getSeconds());
				let minute = this.zeroize(d.getMinutes());
				if(Math.floor(milliseconds) > 0){
					const self = this;
					setTimeout(function(){
						self.minutes = minute;
						self.seconds = second;
						if(Math.floor(milliseconds) > 0){ 
							self.countdown();//继续调用自己
						}else{
							console.log('倒计时结束');
						}
					},1000);
				}
				
			},
			zeroize(value){
			  value = String(value);
			  for (var i = 0, zeros = ''; i < (2 - value.length); i++){
			    zeros += '0';
			  }
			  return zeros + value;
			},
			payfor() {
				let WX_TOKEN = '';
				try {
					const token = uni.getStorageSync('WX_TOKEN');
					if (token) {
						WX_TOKEN = token;
					}
				} catch (e) {}
				let orderId = this.orderId;
				uni.request({
					url: this.$baseUrl + `/orderUnpaid/${orderId}`,
					header: {
						'X-Access-Token': WX_TOKEN,
					},
					success: (res) => {
						if (res.data.success) {
							let orderUnpaid = res.data.result;
							uni.requestPayment({
								timeStamp: orderUnpaid.timeStamp,
								nonceStr: orderUnpaid.nonceStr,
								package: orderUnpaid.packageStr,
								signType: orderUnpaid.signType,
								paySign: orderUnpaid.paySign,
								success: (res) => {
									uni.redirectTo({
										url: `/pages/order/paySuccess?orderId=`+this.orderId
									})
								},
							})
						}
					},
				})
				
			},
			seeOrder() {
				uni.redirectTo({
					url: `/pages/order/orderDetail?orderId=`+this.orderId
				})
			},
		}
	}
</script>

<style lang='scss'>
	.content{
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}
	.success-icon{
		font-size: 160upx;
		color: #fa436a;
		margin-top: 100upx;
	}
	.tit{
		font-size: 38upx;
		color: #303133;
	}
	.tit2{
		font-size: 28upx;
		color: #303133;
	}
	.btn-group{
		padding-top: 100upx;
	}
	.mix-btn {
		margin-top: 30upx;
		display: flex;
		align-items: center;
		justify-content: center;
		width: 600upx;
		height: 80upx;
		font-size: $font-lg;
		color: #fff;
		background-color: $base-color;
		border-radius: 10upx;
		&.hollow{
			background: #fff;
			color: #303133;
			border: 1px solid #ccc;
		}
	}
</style>
