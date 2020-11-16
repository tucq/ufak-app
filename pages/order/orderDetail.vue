<template>
	<view >
		<text>收货人：{{order.username}}</text>
		<text>收货电话号码：{{order.telephone}}</text>
		<text>地址：{{order.address}}{{order.detailAddress}}</text>
		<text>订单状态：{{order.orderStatus}}</text>
		<view v-for="(detail, detailIndex) in orderDetails" :key="detailIndex">
			<image :src="getAvatarView(detail.viewImage)" mode="aspectFill"></image>
			<view >
				<text>{{detail.productName}}</text>
				<text>{{detail.specsName}}  x {{detail.buyNum}}</text>
				<text >{{detail.price}}</text>
			</view>
		</view>
		<text>订单总金额：{{order.totalAmount}}</text>
		<text>运费：{{order.freightAmount}}</text>
		<text>订单编号：{{order.orderNo}}</text>
		<text>下单时间：{{order.createTime}}</text>
		<text>微信交易号：{{order.transactionId}}</text>
		<text>活动优惠：{{order.eventAmount + order.couponAmount}}</text>
		<text>合计支付：{{order.totalFee / 100 }}</text>
		<button v-if="order.orderStatus == 0" type="default" @click="cancel" class="mix-btn">取消订单</button>
		<button v-if="order.orderStatus == 0" type="default" @click="payfor" class="mix-btn">去付款</button>
		<button v-if="order.orderStatus != 0" type="default" @click="buyAgain" class="mix-btn">再次购买</button>
	</view>
</template>

<script>
	
	export default {
		data() {
			return {
				orderId:'',
				order: null,
				orderDetails: [],
			}
		},
		onLoad(option){
			this.orderId = option.orderId;
			this.loadData();
		},
		methods: {
			loadData(){
				let WX_TOKEN = '';
				try {
					const token = uni.getStorageSync('WX_TOKEN');
					if (token) {
						WX_TOKEN = token;
					}
				} catch (e) {}
				uni.request({
					url: this.$baseUrl + '/order/'+this.orderId,
					header: {
						'X-Access-Token': WX_TOKEN,
					},
					success: (res) => {
						if (res.data.success) {
							this.order = res.data.result;
							this.orderDetails = this.order.orderDetails;
							console.log("this.order",this.order);
							console.log("this.orderDetails",this.order.orderDetails);
						}
					},
				});
			},
			getAvatarView(imgUrl){
			    return this.$baseUrl + '/sys/common/view/' + imgUrl;
			},
			cancel(){
				let WX_TOKEN = '';
				try {
					const token = uni.getStorageSync('WX_TOKEN');
					if (token) {
						WX_TOKEN = token;
					}
				} catch (e) {}
				uni.request({
					url: this.$baseUrl + '/order/edit',
					data:{
						id: this.orderId,
						orderStatus: '3',
					},
					method:'PUT',
					header: {
						'X-Access-Token': WX_TOKEN,
					},
					success: (res) => {
						if (res.data.success) {
							uni.showToast({
								title: '订单已取消',
								mask: true,
								duration: 2000,
								success: (res) => {
									setTimeout(function () {
									    uni.redirectTo({
									    	url: `/pages/order/order?orderStatus=4`
									    })
									}, 2000);
								},
							});
						}
					},
				});
			},
			payfor(){
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
			
		},
		
	}
</script>

<style>
</style>
