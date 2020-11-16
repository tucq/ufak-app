<template>
	<view class="container">
		<!-- 空白页 -->
		<view v-if="!hasLogin || empty===true" class="empty">
			<image src="/static/emptyCart.jpg" mode="aspectFit"></image>
			<view v-if="hasLogin" class="empty-tips">
				空空如也
				<navigator class="navigator" v-if="hasLogin" url="../index/index" open-type="switchTab">随便逛逛></navigator>
			</view>
			<view v-else class="empty-tips">
				空空如也
				<view class="navigator" @click="navToLogin">去登陆></view>
			</view>
		</view>
		<view v-else>
			<!-- 列表 -->
			<view class="cart-list">
				<block v-for="(item, index) in cartList" :key="item.id">
					<view
						class="cart-item" 
						:class="{'b-b': index!==cartList.length-1}"
					>
						<view class="image-wrapper">
							<image :src="getAvatarView(item.viewImage)"
								:class="[item.loaded]"
								mode="aspectFill" 
								lazy-load 
								@load="onImageLoad('cartList', index)"
								@error="onImageError('cartList', index)"
							></image>
							<view 
								class="yticon icon-xuanzhong2 checkbox"
								:class="{checked: item.isCheck == '0'}"
								@click="check('item', index)"
							></view>
						</view>
						<view class="item-right">
							<text class="clamp title">{{item.productName}}</text>
							<text class="attr">{{item.specsName}}</text>
							<text class="price">¥{{item.price}}</text>
							<uni-number-box 
								class="step"
								:min="1" 
								:max="item.stock"
								:value="item.buyNum>item.stock?item.stock:item.buyNum"
								:isMax="item.buyNum>=item.stock?true:false"
								:isMin="item.buyNum===1"
								:index="index"
								:disabled="true"
								@eventChange="numberChange"
							></uni-number-box>
						</view>
						<text class="del-btn yticon icon-fork" @click="deleteCartItem(index)"></text>
					</view>
				</block>
			</view>
			<!-- 底部菜单栏 -->
			<view class="action-section">
				<view class="checkbox">
					<image 
						:src="allChecked?'/static/selected.png':'/static/select.png'" 
						mode="aspectFit"
						@click="check('all')"
					></image>
					<!-- <view class="clear-btn" :class="{show: allChecked}" @click="clearCart">
						清空
					</view> -->
				</view>
				<view class="total-box">
					<text class="price">¥{{total}}</text>
					<!-- <text class="coupon">
						已优惠
						<text>74.35</text>
						元
					</text> -->
				</view>
				<button type="primary" class="no-border confirm-btn" @click="createOrder">去结算</button>
			</view>
		</view>
	</view>
</template>

<script>
	import {
		mapState
	} from 'vuex';
	
	import uniNumberBox from '@/components/uni-number-box.vue'
	
	export default {
		components: {
			uniNumberBox
		},
		data() {
			return {
				total: 0, //总价格
				allChecked: false, //全选状态  true|false
				empty: false, //空白页现实  true|false
				cartList: [],
			};
		},
		onShow(){
			if(this.$store.state.hasLogin){
				this.loadData();
			}
		},
		onLoad(){
		},
		watch:{
			//显示空白页
			cartList(e){
				let empty = e.length === 0 ? true: false;
				if(this.empty !== empty){
					this.empty = empty;
				}
			}
		},
		computed:{
			...mapState(['hasLogin'])
		},
		methods: {
			//请求数据
			async loadData(){
				let WX_TOKEN = '';
				let userId = '';
				try {
				    const userInfo = uni.getStorageSync('userInfo');
				    if (userInfo.id) {
						userId = userInfo.id;
				    }
					const token = uni.getStorageSync('WX_TOKEN');
					if (token) {
						WX_TOKEN = token;
					}
				} catch (e) {}
				this.cartList = [];
				uni.request({
					url: this.$baseUrl + '/shopping/car/list',
					data: {
						userId: userId,
						pageNo: 1,
						pageSize: 10000,
					},
					header: {
						'X-Access-Token': WX_TOKEN,
					},
					success: (res) => {
						if (res.data.success) {
							this.cartList = res.data.result.records;
							this.calcTotal();  //计算总价
						}else{
							//token 重新授权登录
							uni.navigateTo({
								url: '/pages/public/wxLogin'
							})
						}
					}
				});
			},
			getAvatarView(imgUrl){
				return this.$baseUrl + '/sys/common/view/' + imgUrl;
			},
			//监听image加载完成
			onImageLoad(key, index) {
				this.$set(this[key][index], 'loaded', 'loaded');
			},
			//监听image加载失败
			onImageError(key, index) {
				// this[key][index].viewImage = '/static/errorImage.jpg';
			},
			navToLogin(){				
				uni.navigateTo({
					url: '/pages/public/wxLogin'
				})
			},
			 //选中状态处理
			check(type, index){
				if(type === 'item'){
					if(this.cartList[index].isCheck == '0'){
						this.cartList[index].isCheck = '1';
						this.updateCheck(this.cartList[index].id,'1');
					}else{
						this.cartList[index].isCheck = '0';
						this.updateCheck(this.cartList[index].id,'0');
					}
				}else{
					let isCheck = !this.allChecked ? '0' : '1';
					const list = this.cartList;
					list.forEach(item=>{
						item.isCheck = isCheck;
					})
					this.allChecked = !this.allChecked;
					this.updateAllCheck(this.cartList[0].userId,isCheck);
				}
				this.calcTotal(type);
			},
			updateCheck(id,isCheck){
				let WX_TOKEN = '';
				try {
					const token = uni.getStorageSync('WX_TOKEN');
					if (token) {
						WX_TOKEN = token;
					}
				} catch (e) {}
				
				uni.request({
					url: this.$baseUrl + '/shopping/car/checked',
					data: {
						id: id,
						isCheck:isCheck,
					},
					method: 'POST',
					header: {
						'content-type': 'application/json',
						'X-Access-Token': WX_TOKEN,
					},
					success: (res) => {
						if (res.data.success) {
							
						}else{
							uni.navigateTo({
								url: '/pages/public/wxLogin'
							})
						}
					}
				});
			},
			updateAllCheck(userId,isCheck){
				let WX_TOKEN = '';
				try {
					const token = uni.getStorageSync('WX_TOKEN');
					if (token) {
						WX_TOKEN = token;
					}
				} catch (e) {}
				
				uni.request({
					url: this.$baseUrl + '/shopping/car/allChecked',
					data: {
						userId: userId,
						isCheck:isCheck,
					},
					method: 'POST',
					header: {
						'content-type': 'application/json',
						'X-Access-Token': WX_TOKEN,
					},
					success: (res) => {
						if (res.data.success) {
							
						}else{
							uni.navigateTo({
								url: '/pages/public/wxLogin'
							})
						}
					}
				});
			},
			updateBuyNum(id,buyNum){
				let WX_TOKEN = '';
				try {
					const token = uni.getStorageSync('WX_TOKEN');
					if (token) {
						WX_TOKEN = token;
					}
				} catch (e) {}
				
				uni.request({
					url: this.$baseUrl + '/shopping/car/updateBuyNum',
					data: {
						id: id,
						buyNum:buyNum,
					},
					method: 'POST',
					header: {
						'content-type': 'application/json',
						'X-Access-Token': WX_TOKEN,
					},
					success: (res) => {
						if (res.data.success) {
							
						}else{
							uni.navigateTo({
								url: '/pages/public/wxLogin'
							})
						}
					}
				});
			},
			//数量
			numberChange(data){
				this.cartList[data.index].buyNum = data.number;
				this.updateBuyNum(this.cartList[data.index].id,data.number);
				this.calcTotal();
			},
			//删除
			deleteCartItem(index){
				let id = this.cartList[index].id;
				let WX_TOKEN = '';
				try {
					const token = uni.getStorageSync('WX_TOKEN');
					if (token) {
						WX_TOKEN = token;
					}
				} catch (e) {}
				
				uni.request({
					url: this.$baseUrl + '/shopping/car/delete',
					data: {
						id: id,
					},
					method: 'POST',
					header: {
						'content-type': 'application/json',
						'X-Access-Token': WX_TOKEN,
					},
					success: (res) => {
						if (res.data.success) {
							this.cartList.splice(index, 1);
							this.calcTotal();
							uni.hideLoading();
						}else{
							uni.navigateTo({
								url: '/pages/public/wxLogin'
							})
						}
					}
				});
				
			},
			//清空
			// clearCart(){
			// 	uni.showModal({
			// 		content: '清空购物车？',
			// 		success: (e)=>{
			// 			if(e.confirm){
			// 				this.cartList = [];
			// 			}
			// 		}
			// 	})
			// },
			//计算总价
			calcTotal(){
				let list = this.cartList;
				if(list.length === 0){
					this.empty = true;
					return;
				}
				let total = 0;
				let checked = true;
				list.forEach(item=>{
					if(item.isCheck === '0'){
						total += item.price * item.buyNum;
					}else if(checked === true){
						checked = false;
					}
				})
				this.allChecked = checked;
				this.total = Number(total.toFixed(2));
			},
			//创建订单
			createOrder(){
				let list = this.cartList;
				let goodsData = [];
				list.forEach(item=>{
					if(item.isCheck === '0'){
						goodsData.push(item);
					}
				});
				if(goodsData.length === 0){
					uni.showToast({
						title: '请选择购买商品',
						icon: 'none',
						duration: 1500
					});
					return;
				}
				
				uni.navigateTo({
					url: `/pages/order/createOrder?orderFlag=1`
				})
			}
		}
	}
</script>

<style lang='scss'>
	.container{
		padding-bottom: 134upx;
		/* 空白页 */
		.empty{
			position:fixed;
			left: 0;
			top:0;
			width: 100%;
			height: 100vh;
			padding-bottom:100upx;
			display:flex;
			justify-content: center;
			flex-direction: column;
			align-items:center;
			background: #fff;
			image{
				width: 240upx;
				height: 160upx;
				margin-bottom:30upx;
			}
			.empty-tips{
				display:flex;
				font-size: $font-sm+2upx;
				color: $font-color-disabled;
				.navigator{
					color: $uni-color-primary;
					margin-left: 16upx;
				}
			}
		}
	}
	/* 购物车列表项 */
	.cart-item{
		display:flex;
		position:relative;
		padding:30upx 40upx;
		.image-wrapper{
			width: 230upx;
			height: 230upx;
			flex-shrink: 0;
			position:relative;
			image{
				border-radius:8upx;
			}
		}
		.checkbox{
			position:absolute;
			left:-16upx;
			top: -16upx;
			z-index: 8;
			font-size: 44upx;
			line-height: 1;
			padding: 4upx;
			color: $font-color-disabled;
			background:#fff;
			border-radius: 50px;
		}
		.item-right{
			display:flex;
			flex-direction: column;
			flex: 1;
			overflow: hidden;
			position:relative;
			padding-left: 30upx;
			.title,.price{
				font-size:$font-base + 2upx;
				color: $font-color-dark;
				height: 40upx;
				line-height: 40upx;
			}
			.attr{
				font-size: $font-sm + 2upx;
				color: $font-color-light;
				height: 50upx;
				line-height: 50upx;
			}
			.price{
				height: 50upx;
				line-height:50upx;
				color: $uni-color-primary;
			}
		}
		.del-btn{
			padding:4upx 10upx;
			font-size:34upx; 
			height: 50upx;
			color: $font-color-light;
		}
	}
	/* 底部栏 */
	.action-section{
		/* #ifdef H5 */
		margin-bottom:100upx;
		/* #endif */
		position:fixed;
		left: 30upx;
		bottom:8upx;
		z-index: 95;
		display: flex;
		align-items: center;
		width: 690upx;
		height: 100upx;
		padding: 0 30upx;
		background: rgba(255,255,255,.9);
		box-shadow: 0 0 20upx 0 rgba(0,0,0,.5);
		border-radius: 16upx;
		.checkbox{
			height:52upx;
			position:relative;
			image{
				width: 52upx;
				height: 100%;
				position:relative;
				z-index: 5;
			}
		}
		.clear-btn{
			position:absolute;
			left: 26upx;
			top: 0;
			z-index: 4;
			width: 0;
			height: 52upx;
			line-height: 52upx;
			padding-left: 38upx;
			font-size: $font-base;
			color: #fff;
			background: $font-color-disabled;
			border-radius:0 50px 50px 0;
			opacity: 0;
			transition: .2s;
			&.show{
				opacity: 1;
				width: 120upx;
			}
		}
		.total-box{
			flex: 1;
			display:flex;
			flex-direction: column;
			text-align:right;
			padding-right: 40upx;
			.price{
				font-size: $font-lg;
				color: $uni-color-primary;
			}
			.coupon{
				font-size: $font-sm;
				color: $font-color-light;
				text{
					color: $font-color-dark;
				}
			}
		}
		.confirm-btn{
			padding: 0 38upx;
			margin: 0;
			border-radius: 100px;
			height: 76upx;
			line-height: 76upx;
			font-size: $font-base + 2upx;
			background: $uni-color-primary;
			box-shadow: 1px 2px 5px rgba(217, 60, 93, 0.72)
		}
	}
	/* 复选框选中状态 */
	.action-section .checkbox.checked,
	.cart-item .checkbox.checked{
		color: $uni-color-primary;
	}
</style>
