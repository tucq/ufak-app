<template>
	<view class="container">
		<view class="carousel">
			<swiper indicator-dots circular=true duration="400">
				<swiper-item class="swiper-item" v-for="(item,index) in imgList" :key="index">
					<view class="image-wrapper">
						<image :src="getAvatarView(item)" class="loaded" mode="aspectFill"></image>
					</view>
				</swiper-item>
			</swiper>
		</view>

		<view class="introduce-section">
			<view class="uni-flex uni-row">
				<view><text class="title">{{productInfo.name}}</text></view>
				<view><text class="description">{{productInfo.description}}</text></view>
			</view>
			<view class="price-box">
				<text class="price-tip">¥</text>
				<text class="price">{{priceSelected.price}}</text>
				<text class="m-price">{{priceSelected.virtualPrice}}</text>
				<!-- <text class="coupon-tip">7折</text> -->
			</view>
			<view class="bot-row">
				<text>销量: {{productInfo.salesVolume}}</text>
				<text>库存: {{priceSelected.stock}}</text>
				<!-- <text>浏览量: 768</text> -->
			</view>
		</view>

		<view class="c-list">
			<view class="c-row b-b" @click="toggleSpec">
				<text class="tit">购买类型</text>
				<view class="con">
					<text class="selected-text">
						{{priceSelected.specsName}} {{productInfo.quantity}}件
					</text>
					<!-- <text class="selected-text" v-for="(sItem, sIndex) in specSelected" :key="sIndex">
						{{sItem.name}}
					</text> -->
				</view>
				<text class="yticon icon-you"></text>
			</view>

			<view class="c-row b-b">
				<text class="tit">服务</text>
				<view class="bz-list con">
					<text v-for="(item,index) in serviceList" :key="index">{{item}}·</text>
				</view>
			</view>
		</view>

		<view class="detail-desc">
			<view class="d-header">
				<text>图文详情</text>
			</view>
			<div v-for="(item,index) in detailImgList" :key="index">
				<img style="width: 100%;" :src="getAvatarView(item)" mode="widthFix" />
			</div>
		</view>
		

		<!-- 底部操作菜单 -->
		<view class="page-bottom">
			<navigator url="/pages/index/index" open-type="switchTab" class="p-b-btn">
				<text class="yticon icon-xiatubiao--copy"></text>
				<text>首页</text>
			</navigator>
			<navigator url="/pages/cart/cart" open-type="switchTab" class="p-b-btn">
				<text class="yticon icon-gouwuche"></text>
				<text>购物车</text>
			</navigator>
			<!-- <view class="p-b-btn" :class="{active: favorite}" @click="toFavorite">
				<text class="yticon icon-shoucang"></text>
				<text>收藏</text>
			</view> -->

			<view class="action-btn-group">
				<button type="primary" class=" action-btn no-border add-cart-btn" @click="addCar">加入购物车</button>
				<button type="primary" class=" action-btn no-border buy-now-btn" @click="buy">立即购买</button>
			</view>
		</view>


		<!-- 规格-模态层弹窗 -->
		<view class="popup spec" :class="specClass" @touchmove.stop.prevent="stopPrevent" @click="toggleSpec">
			<!-- 遮罩层 -->
			<view class="mask"></view>
			<view class="layer attr-content" @click.stop="stopPrevent">
				<view class="a-t">
					<image :src="getAvatarView(priceSelected.specsImage)"></image>
					<view class="right">
						<view class="price-box">
							<text class="price-tip">¥</text>
							<text class="price">{{priceSelected.price}}</text>
							<text class="m-price">{{priceSelected.virtualPrice}}</text>
							<!-- <text class="coupon-tip">7折</text> -->
						</view>
						<text class="stock">库存：{{priceSelected.stock}}</text>
						<view class="selected">
							已选：{{priceSelected.specsName}}
							<!-- <text class="selected-text" v-for="(sItem, sIndex) in specSelected" :key="sIndex">
								{{sItem.name}}
							</text> -->
						</view>
					</view>
				</view>

				<view class="attr-list" v-if="specsTitleList.length > 0">
					<text>{{specsTitleList[0].specsTitle}}</text>
					<view class="item-list">
						<text v-for="(item, index) in specsOneList" :key="item.id" class="tit" :class="{selected: item.selected}" @click="selectSpec(index, item.id,1)">
							{{item.specsTitle}}
						</text>
					</view>
				</view>
				<view class="attr-list" v-if="specsTitleList.length > 1">
					<text>{{specsTitleList[1].specsTitle}}</text>
					<view class="item-list">
						<text v-for="(item, index) in specsTwoList" :key="item.id" class="tit" :class="{selected: item.selected}" @click="selectSpec(index, item.id,2)">
							{{item.specsTitle}}
						</text>
					</view>
				</view>

				<view class="attr-list">
					<text>数量</text>
					<uni-number-box class="step" :min="1" :max="priceSelected.stock" :value="1" @eventChange="numberChange">
					</uni-number-box>
				</view>
				<button class="btn" @click="toggleSpec">完成</button>
			</view>
		</view>

	</view>
</template>

<script>
	import uniNumberBox from '@/components/uni-number-box.vue'

	export default {
		components: {
			uniNumberBox
		},
		data() {
			return {
				productInfo: {},
				imgList: [],
				detailImgList: [],
				serviceList: [],
				specsTitleList: [],
				specsOneList: [],
				specsTwoList: [],
				priceList: [],
				priceSelected: {},
				specClass: 'none',
				favorite: false,
			};
		},
		async onLoad(options) {
			let productId = options.productId;
			uni.request({
				url: this.$baseUrl + '/api/product/' + productId,
				data: {},
				header: {},
				success: (res) => {
					if (res.data.success) {
						this.productInfo = res.data.result;
						this.productInfo.quantity = 1;
						this.imgList = this.productInfo.image.substring(0, this.productInfo.image.length - 1).split(',');
						this.detailImgList = this.productInfo.detailImages.substring(0, this.productInfo.detailImages.length - 1).split(
							',');
						this.serviceList = this.productInfo.service.split(',');
						this.specsTitleList = this.productInfo.specsTitleList;
						this.specsOneList = this.productInfo.specsOneList;
						this.specsTwoList = this.productInfo.specsTwoList;
						this.priceList = this.productInfo.priceList;

						this.initDefaultPrice();
					}
				},
			});
		},
		methods: {
			//规格弹窗开关
			toggleSpec() {
				if (this.specClass === 'show') {
					this.specClass = 'hide';
					setTimeout(() => {
						this.specClass = 'none';
					}, 250);
				} else if (this.specClass === 'none') {
					this.specClass = 'show';
				}
			},
			initDefaultPrice() {
				for (let i = 0; i < this.priceList.length; i++) {
					let pitem = this.priceList[i];
					if (pitem.defaultFlag === '0') {
						let specs1Id = pitem.specs1Id;
						for (let j = 0; j < this.specsOneList.length; j++) {
							if (specs1Id === this.specsOneList[j].id) {
								this.selectSpec(j, specs1Id, 1);
								break;
							}
						}
						if (this.specsTitleList.length > 1) {
							let specs2Id = pitem.specs2Id;
							for (let k = 0; k < this.specsTwoList.length; k++) {
								if (specs2Id === this.specsTwoList[k].id) {
									this.selectSpec(k, specs2Id, 2);
									break;
								}
							}
						}

						break;
					}
				}
			},
			//选择规格
			selectSpec(index, id, type) {
				let specList = [];
				if (type === 1) {
					specList = this.specsOneList;
					this.priceSelected.specs1Id = this.specsOneList[index].id;
				} else {
					specList = this.specsTwoList;
					this.priceSelected.specs2Id = this.specsTwoList[index].id;
				}

				specList.forEach(item => {
					if (item.id != id) {
						this.$set(item, 'selected', false);
					}
				})

				this.$set(specList[index], 'selected', true);

				for (let i = 0; i < this.priceList.length; i++) {
					let item = this.priceList[i];
					if (item.specs1Id == this.priceSelected.specs1Id && item.specs2Id == this.priceSelected.specs2Id) {
						this.priceSelected.specs1Id = item.specs1Id;
						this.priceSelected.specs2Id = item.specs2Id;
						this.priceSelected.price = item.price;
						this.priceSelected.virtualPrice = item.virtualPrice;
						this.priceSelected.stock = item.stock;
						this.priceSelected.specsName = item.specsName;
						this.priceSelected.specsImage = item.specsImage;
						break;
					}
				}

			},
			numberChange(data) {
				this.productInfo.quantity = data.number;
			},
			addCar() {
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

				uni.request({
					url: this.$baseUrl + '/shopping/car/add',
					data: {
						userId: userId,
						productId: this.productInfo.id,
						specs1Id: this.priceSelected.specs1Id,
						specs2Id: this.priceSelected.specs2Id,
						buyNum: this.productInfo.quantity,
					},
					method: 'POST',
					header: {
						'content-type': 'application/json',
						'X-Access-Token': WX_TOKEN,
					},
					success: (res) => {
						if (res.data.success) {
							uni.showToast({
								title: '已加入购物车',
								mask: true,
								duration: 1500
							});
						}else{
							uni.navigateTo({
								url: '/pages/public/wxLogin'
							})
						}
					},
					
				});
			},
			getAvatarView(imgUrl) {
				return this.$baseUrl + '/sys/common/view/' + imgUrl;
			},
			//收藏
			// toFavorite(){
			// 	this.favorite = !this.favorite;
			// },
			buy() {
				uni.navigateTo({
					url: `/pages/order/createOrder?orderFlag=2&productId=`+this.productInfo.id+`&specs1Id=`+this.priceSelected.specs1Id
					+`&specs2Id=`+this.priceSelected.specs2Id+`&buyNum=`+this.productInfo.quantity
				})
			},
			stopPrevent() {}
		},

	}
</script>

<style lang='scss'>
	page {
		background: $page-color-base;
		padding-bottom: 160upx;
	}

	.icon-you {
		font-size: $font-base + 2upx;
		color: #888;
	}

	.carousel {
		height: 722upx;
		position: relative;

		swiper {
			height: 100%;
		}

		.image-wrapper {
			width: 100%;
			height: 100%;
		}

		.swiper-item {
			display: flex;
			justify-content: center;
			align-content: center;
			height: 750upx;
			overflow: hidden;

			image {
				width: 100%;
				height: 100%;
			}
		}

	}

	/* 标题简介 */
	.introduce-section {
		background: #fff;
		padding: 20upx 30upx;

		.title {
			font-size: 32upx;
			color: $font-color-dark;
			height: 50upx;
			line-height: 50upx;
		}

		.price-box {
			display: flex;
			align-items: baseline;
			height: 64upx;
			padding: 10upx 0;
			font-size: 26upx;
			color: $uni-color-primary;
		}

		.price {
			font-size: $font-lg + 2upx;
		}

		.m-price {
			margin: 0 12upx;
			color: $font-color-light;
			text-decoration: line-through;
		}

		.coupon-tip {
			align-items: center;
			padding: 4upx 10upx;
			background: $uni-color-primary;
			font-size: $font-sm;
			color: #fff;
			border-radius: 6upx;
			line-height: 1;
			transform: translateY(-4upx);
		}

		.bot-row {
			display: flex;
			align-items: center;
			height: 50upx;
			font-size: $font-sm;
			color: $font-color-light;

			text {
				flex: 1;
			}
		}
	}

	/* 分享 */
	.share-section {
		display: flex;
		align-items: center;
		color: $font-color-base;
		background: linear-gradient(left, #fdf5f6, #fbebf6);
		padding: 12upx 30upx;

		.share-icon {
			display: flex;
			align-items: center;
			width: 70upx;
			height: 30upx;
			line-height: 1;
			border: 1px solid $uni-color-primary;
			border-radius: 4upx;
			position: relative;
			overflow: hidden;
			font-size: 22upx;
			color: $uni-color-primary;

			&:after {
				content: '';
				width: 50upx;
				height: 50upx;
				border-radius: 50%;
				left: -20upx;
				top: -12upx;
				position: absolute;
				background: $uni-color-primary;
			}
		}

		.icon-xingxing {
			position: relative;
			z-index: 1;
			font-size: 24upx;
			margin-left: 2upx;
			margin-right: 10upx;
			color: #fff;
			line-height: 1;
		}

		.tit {
			font-size: $font-base;
			margin-left: 10upx;
		}

		.icon-bangzhu1 {
			padding: 10upx;
			font-size: 30upx;
			line-height: 1;
		}

		.share-btn {
			flex: 1;
			text-align: right;
			font-size: $font-sm;
			color: $uni-color-primary;
		}

		.icon-you {
			font-size: $font-sm;
			margin-left: 4upx;
			color: $uni-color-primary;
		}
	}

	.c-list {
		font-size: $font-sm + 2upx;
		color: $font-color-base;
		background: #fff;

		.c-row {
			display: flex;
			align-items: center;
			padding: 20upx 30upx;
			position: relative;
		}

		.tit {
			width: 140upx;
		}

		.con {
			flex: 1;
			color: $font-color-dark;

			.selected-text {
				margin-right: 10upx;
			}
		}

		.bz-list {
			height: 40upx;
			font-size: $font-sm+2upx;
			color: $font-color-dark;

			text {
				display: inline-block;
				margin-right: 30upx;
			}
		}

		.con-list {
			flex: 1;
			display: flex;
			flex-direction: column;
			color: $font-color-dark;
			line-height: 40upx;
		}

		.red {
			color: $uni-color-primary;
		}
	}

	/* 评价 */
	.eva-section {
		display: flex;
		flex-direction: column;
		padding: 20upx 30upx;
		background: #fff;
		margin-top: 16upx;

		.e-header {
			display: flex;
			align-items: center;
			height: 70upx;
			font-size: $font-sm + 2upx;
			color: $font-color-light;

			.tit {
				font-size: $font-base + 2upx;
				color: $font-color-dark;
				margin-right: 4upx;
			}

			.tip {
				flex: 1;
				text-align: right;
			}

			.icon-you {
				margin-left: 10upx;
			}
		}
	}

	.eva-box {
		display: flex;
		padding: 20upx 0;

		.portrait {
			flex-shrink: 0;
			width: 80upx;
			height: 80upx;
			border-radius: 100px;
		}

		.right {
			flex: 1;
			display: flex;
			flex-direction: column;
			font-size: $font-base;
			color: $font-color-base;
			padding-left: 26upx;

			.con {
				font-size: $font-base;
				color: $font-color-dark;
				padding: 20upx 0;
			}

			.bot {
				display: flex;
				justify-content: space-between;
				font-size: $font-sm;
				color: $font-color-light;
			}
		}
	}

	/*  详情 */
	.detail-desc {
		background: #fff;
		margin-top: 16upx;

		.d-header {
			display: flex;
			justify-content: center;
			align-items: center;
			height: 80upx;
			font-size: $font-base + 2upx;
			color: $font-color-dark;
			position: relative;

			text {
				padding: 0 20upx;
				background: #fff;
				position: relative;
				z-index: 1;
			}

			&:after {
				position: absolute;
				left: 50%;
				top: 50%;
				transform: translateX(-50%);
				width: 300upx;
				height: 0;
				content: '';
				border-bottom: 1px solid #ccc;
			}
		}
		
	}
	


	/* 规格选择弹窗 */
	.attr-content {
		padding: 10upx 30upx;

		.a-t {
			display: flex;

			image {
				width: 170upx;
				height: 170upx;
				flex-shrink: 0;
				margin-top: -40upx;
				border-radius: 8upx;
				;
			}

			.right {
				display: flex;
				flex-direction: column;
				padding-left: 24upx;
				font-size: $font-sm + 2upx;
				color: $font-color-base;
				line-height: 42upx;

				.price {
					font-size: $font-lg;
					color: $uni-color-primary;
					margin-bottom: 10upx;
				}

				.m-price {
					margin: 0 12upx;
					color: $font-color-light;
					text-decoration: line-through;
				}

				.selected-text {
					margin-right: 10upx;
				}
			}
		}

		.attr-list {
			display: flex;
			flex-direction: column;
			font-size: $font-base + 2upx;
			color: $font-color-base;
			padding-top: 30upx;
			padding-left: 10upx;
		}

		.item-list {
			padding: 20upx 0 0;
			display: flex;
			flex-wrap: wrap;

			text {
				display: flex;
				align-items: center;
				justify-content: center;
				background: #eee;
				margin-right: 20upx;
				margin-bottom: 20upx;
				border-radius: 100upx;
				min-width: 60upx;
				height: 60upx;
				padding: 0 20upx;
				font-size: $font-base;
				color: $font-color-dark;
			}

			.selected {
				background: #fbebee;
				color: $uni-color-primary;
			}
		}
	}

	/*  弹出层 */
	.popup {
		position: fixed;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
		z-index: 99;

		&.show {
			display: block;

			.mask {
				animation: showPopup 0.2s linear both;
			}

			.layer {
				animation: showLayer 0.2s linear both;
			}
		}

		&.hide {
			.mask {
				animation: hidePopup 0.2s linear both;
			}

			.layer {
				animation: hideLayer 0.2s linear both;
			}
		}

		&.none {
			display: none;
		}

		.mask {
			position: fixed;
			top: 0;
			width: 100%;
			height: 100%;
			z-index: 1;
			background-color: rgba(0, 0, 0, 0.4);
		}

		.layer {
			position: fixed;
			z-index: 99;
			bottom: 0;
			width: 100%;
			min-height: 40vh;
			border-radius: 10upx 10upx 0 0;
			background-color: #fff;

			.btn {
				height: 66upx;
				line-height: 66upx;
				border-radius: 100upx;
				background: $uni-color-primary;
				font-size: $font-base + 2upx;
				color: #fff;
				margin: 30upx auto 20upx;
			}
		}

		@keyframes showPopup {
			0% {
				opacity: 0;
			}

			100% {
				opacity: 1;
			}
		}

		@keyframes hidePopup {
			0% {
				opacity: 1;
			}

			100% {
				opacity: 0;
			}
		}

		@keyframes showLayer {
			0% {
				transform: translateY(120%);
			}

			100% {
				transform: translateY(0%);
			}
		}

		@keyframes hideLayer {
			0% {
				transform: translateY(0);
			}

			100% {
				transform: translateY(120%);
			}
		}
	}

	/* 底部操作菜单 */
	.page-bottom {
		position: fixed;
		left: 30upx;
		bottom: 30upx;
		z-index: 95;
		display: flex;
		justify-content: center;
		align-items: center;
		width: 690upx;
		height: 100upx;
		background: rgba(255, 255, 255, .9);
		box-shadow: 0 0 20upx 0 rgba(0, 0, 0, .5);
		border-radius: 16upx;

		.p-b-btn {
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;
			font-size: $font-sm;
			color: $font-color-base;
			width: 96upx;
			height: 80upx;

			.yticon {
				font-size: 40upx;
				line-height: 48upx;
				color: $font-color-light;
			}

			&.active,
			&.active .yticon {
				color: $uni-color-primary;
			}

			.icon-fenxiang2 {
				font-size: 42upx;
				transform: translateY(-2upx);
			}

			.icon-shoucang {
				font-size: 46upx;
			}
		}

		.action-btn-group {
			display: flex;
			height: 76upx;
			border-radius: 100px;
			overflow: hidden;
			box-shadow: 0 20upx 40upx -16upx #fa436a;
			box-shadow: 1px 2px 5px rgba(219, 63, 96, 0.4);
			background: linear-gradient(to right, #ffac30, #fa436a, #F56C6C);
			margin-left: 20upx;
			position: relative;

			&:after {
				content: '';
				position: absolute;
				top: 50%;
				right: 50%;
				transform: translateY(-50%);
				height: 28upx;
				width: 0;
				border-right: 1px solid rgba(255, 255, 255, .5);
			}

			.action-btn {
				display: flex;
				align-items: center;
				justify-content: center;
				width: 180upx;
				height: 100%;
				font-size: $font-base;
				padding: 0;
				border-radius: 0;
				background: transparent;
			}
		}
	}

	.uni-numbox {
		position: inherit !important;
		margin-top: 20upx;
	}

	.description {
		font-size: $font-sm+2upx;
		color: $font-color-base;
		line-height: 50upx;
	}
</style>
