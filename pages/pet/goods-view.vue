<template>
	<view class="wrap">
		<mescroll-body ref="mescrollRef" >
			<!-- <u-swiper :list="item.images" height="600" border-radius="0" :current="current" field="content" mode="dot" v-if="item.images.length" @click="previewImage"></u-swiper> -->
			
			<swiper class="swiper-box" :current="current" circular  indicator-dots indicator-color="#FFF" :indicator-active-color="themeColor.color" @change="swiperChange" v-if="item.pic">
				<swiper-item v-for="(image, index) in imgList" :key="index">
					<u-image v-if="image" :src="getPicUrl(image)" width="100%" height="600" mode="aspectFill" @tap="previewImage" /> 
				</swiper-item>
			</swiper>

			<view class="subwrap u-p-b-40">
				<view class="title u-p-t-40">{{item.title}}</view>
				
				<!-- <view class="reward u-m-t-10" v-if="item.type == 2 && item.mode == 2">悬赏￥<text class="money">{{item.money}}</text></view>
 -->
				<view class="bottom u-m-t-30 u-m-b-20">
					<view class="datetime">{{item.createTime}}发布</view>
				</view>
				
				<u-line />
				
				
				
				<view class="sheet u-m-t-30 u-m-b-20">
					<!-- 
					<view class="datetime" v-if="item.type != 1"><u-icon name="clock" /> 发布时间：{{item.cerateTime}}</view> -->
					<!-- <view class="address" @tap="openLocation"><u-icon name="map" /> {{$mData.types[item.type-1].title}}地址：{{item.address}}</view> -->
					<!-- <u-divider half-width="30%" color="#999" :fontSize="24" @tap="openLocation">点击查看地图</u-divider> -->
				</view>
				<view style="display: flex;justify-content: space-between;">
					<view>价格：<text>{{item.price}}元</text></view>
					<view>库存：<text>{{item.num}}</text></view>
				</view>
				
				<view class="subtitle u-m-t-30">
					<view class="line" :style="{ background: themeColor.color }" />
					详情
				</view>

				<view class="content u-m-t-30">{{item.content}}</view>
				
				<!-- <view class="subtitle u-m-t-30" v-if="item.type == 1">
					<view class="line" :style="{ background: themeColor.color }" />
					有偿无偿
				</view> -->
				
				
			
				<!-- <view class="subtitle u-m-t-30" v-if="item.type == 1">
					<view class="line" :style="{ background: themeColor.color }" />
					领养要求
				</view> -->
				
				<view class="requirements u-m-t-30" v-if="item.type == 1">
					<block v-for="(requirement, index) in item.requirements" :key="index">
						<text class="iconfont iconxiala" :style="requirementStyle" /> {{requirement}} <br />
					</block>
					
					<block v-if="item.other_requirement">
						<text class="iconfont iconxiala" :style="requirementStyle" /> {{item.other_requirement}}
					</block>
				</view>
			</view>
			
			
			
			
			
		</mescroll-body>

		<view class="nav-bottom">
			<view class="left">
				<!-- <view>收藏</view> -->
			</view>
			<view class="right">
				<u-button :disabled="!userInfo" @click="addOrder" type="warning" size="medium" shape="circle" ripple :custom-style="{ width: '80%', background: themeColor.color , marginTop: '5rpx' }" >
					{{ userInfo?"购买":"游客不能下单" }}
				</u-button>
			</view>
		</view>
		
		

		
		<u-modal v-model="sendOrder" title="下单" @confirm="subOrder" showCancelButton="true">
			<view class="slot-content u-p-20">
				<view class="u-flex u-row-around">
					<view>{{item.title}}</view>
					<view>{{item.price}}</view>
					<view>x1</view>
				</view>
				<u-form :model="form" ref="uForm">
						<u-form-item label="地址"><u-input type="textarea" v-model="orderForm.address" /></u-form-item>
						<u-form-item label="收货方式">
							<u-radio-group v-model="orderForm.expressName">
								<u-radio v-for="(item, index) in expressNameList" :key="index" :name="item.name" >
									{{ item.name }}
								</u-radio>
							</u-radio-group>
						</u-form-item>
						<u-form-item label="支付方式">
							<u-radio-group v-model="orderForm.pay">
								<u-radio v-for="(item, index) in payList" :key="index" :name="item.name" >
									{{ item.name }}
								</u-radio>
							</u-radio-group>
						</u-form-item>
				</u-form>
			</view>
		</u-modal>
		
		
	</view>
</template>

<script>
import moment from '@/common/moment';
import CardAdopt from '@/common/card/adopt';
import CardPet from '@/common/card/pet';
import CardOwner from '@/common/card/owner';
import { petView, petShare, commentLike } from '@/api/pet';

import appRequest from "@/common/appRequestUrl.js";

//const QQMapWX = require('@/common/qqmap-wx-jssdk.min.js');

export default {
	data() {
		return {
			sendOrder:false,
			id:"",
			title: '宠物详情',
			path: '/pages/pet/pet-view',
			$mData: this.$mData,
			showComment:"",
			hasLogin: false,
			userInfo: {},
			qqmapsdk: null,
			modalShow: false,
			poster: '',
			palette: {},
			painterShow: false,
			downOption: {
				native: true
			},
			upOption: {
				use: false
			},
			current: 0,
			videoContext: {},
			loading: true,
			scrollTop: 0,
			orderForm:{
				type:1,
				stat:0,
				price:0,
				num:1,
				expressName:"",
				address:"",
				sid:"",
				title:"",
				pay:""
			},
			item: {
				type: 1,
				video: '',
				member: {},
				images: [],
				applys: [],
				comments: [],
				updated_at: 0
			},imgList:[],showComment:false,
			commentText:"",
			expressNameList:[{
				name:"快递"
			},{
				name:"自提"
			}],
			payList:[{
				name:"支付宝"
			},{
				name:"微信"
			},{
				name:"云闪付"
			}]
		};
	},
	onLoad(option) {
		this.id = option.id;
		this.getData();
		this.loading = false;
	},
	onShow() {
		let userInfo = appRequest.getUserInfo();
		if(!userInfo){
			
		}else{
			this.userInfo = userInfo;
		}
	},
	onReady() {
		
	},
	onPageScroll(e) {
		this.scrollTop = e.scrollTop;
	},
	mounted() {
		//this.initLogin();
	},
	computed: {
		
		baseCustomStyle() {
			return {
				height: '40rpx',
				lineHeight: '40rpx',
				background: this.themeColor.color,
				color: '#FFF',
				margin: '0 14rpx 14rpx 0'
			}
		},
		statusCustomStyle() {
			return {
				height: '38rpx',
				lineHeight: '38rpx',
				color: this.themeColor.color,
				borderColor: this.themeColor.color,
				margin: '0 14rpx 14rpx 0'
			}
		},
		requirementStyle() {
			return `color: ${this.themeColor.color}; font-size: 36rpx; margin-right: 10rpx;`;
		},
		//小程序无效
		requirementStyle02() {
			return {
				color: this.themeColor.color,
				fontSize: '36rpx',
				marginRight: '10rpx'
			}
		}
	},
	watch: {
		
	},
	filters: {
		datetime(val) {
			return moment(val * 1000).format('YYYY-MM-DD HH:mm');
		},
	},
	methods: {
		subOrder(){
			this.orderForm.title = this.item.title;
			this.orderForm.sid =  this.item.id;
			this.orderForm.price =  this.item.price;
			if(!this.orderForm.expressName||!this.orderForm.pay){
				uni.showToast({
					title:"请填写完整",
					icon:"none"
				})
				return;
			}
			var _this = this;
			appRequest.request({
				method: "POST",
				url: appRequest.urlMap.setNmOrder,
				data:this.orderForm,
				success: function(res) {
					if (res.data.code == 200) {
						uni.showToast({
							title:"下单成功",
							icon:"none"
						})
						_this.sendOrder = false;
					}else{
						uni.showToast({
							title:res.data.msg,
							icon:"none"
						})
					}
				},
				fail: function(res) {
					uni.showToast({
						title:"网络异常",
						icon:"none"
					})
				}
			})
		},
		submitComment(){
			var _this = this;
			appRequest.request({
				method: "POST",
				url: appRequest.urlMap.addNmComment,
				data:{
					aid:_this.item.id,
					content:_this.commentText,
					type:0
				},
				success: function(res) {
					if (res.data.code == 200) {
						uni.showToast({
							title:"发布成功",
							icon:"none"
						})
						_this.showComment = false;
						_this.getComment();
					}else{
						uni.showToast({
							title:"获取失败",
							icon:"none"
						})
					}
				},
				fail: function(res) {
					uni.showToast({
						title:"网络异常",
						icon:"none"
					})
				}
			})
		},
		addOrder(){
			this.sendOrder = true;
			
		},
		getImgList:function(item){
			this.imgList = [item.pic,item.pic2,item.pic3]
		},
		getComment(){
			var _this = this;
			appRequest.request({
				method: "POST",
				url: appRequest.urlMap.findNmCommentList,
				data:{
					validFlag:1,
					aid:_this.id
				},
				success: function(res) {
					if (res.data.code == 200) {
						let obj = res.data.data;
						_this.item['comments'] = obj;
					}else{
						uni.showToast({
							title:"获取留言失败",
							icon:"none"
						})
					}
				},
				fail: function(res) {
					uni.showToast({
						title:"网络异常",
						icon:"none"
					})
				}
			})
		},
		getData(){
			var _this = this;
			appRequest.request({
				method: "POST",
				url: appRequest.urlMap.findNmGoods,
				data:{
					validFlag:1,
					id:_this.id
				},
				success: function(res) {
					if (res.data.code == 200) {
						let obj = res.data.data;
						_this.item = obj;
						_this.item['comments'] = [];
						_this.getImgList(obj);
						//_this.getComment();
					}else{
						uni.showToast({
							title:"获取失败",
							icon:"none"
						})
					}
				},
				fail: function(res) {
					uni.showToast({
						title:"网络异常",
						icon:"none"
					})
				}
			})
		},
		getPicUrl(info) {
			if(info){
				if( info.length > 10 ){
					return info;
				}else {
					return appRequest.urlMap.getPicById + info;
				}
			}else {
				return "";
			}
		
		},
		async initLogin() {
			
			this.userInfo = uni.getStorageSync('userInfo');
		},
		navTo(route) {
			this.$mRouter.push({ route });
		},
		dateToStr(val) {
			return moment(val * 1000).format('YYYY年MM年DD日 HH:mm');
		},
		swiperChange(e) {
			this.current = e.detail.current;
			
			if (this.item.video) {
				this.videoContext = uni.createVideoContext('myVideo');
		
				if (this.current) {
					this.videoContext.pause();
				} else {
					this.videoContext.play();
				}
			}
		},
		makePhoneCall(phoneNumber) {
			uni.makePhoneCall({
			  phoneNumber: phoneNumber
			});
		},
		copyText(text) {
			let _this = this;
		
			uni.setClipboardData({
				data: text,
				success(res) {
					_this.$mHelper.toast('复制成功！');
				}
			});
		},
		previewQrcode(url) {
			uni.previewImage({
				urls: [url]
			});
		},
		openLocation() {
			if (!this.item.coords) {
				this.qqmapsdk.geocoder({
					region: this.item.city,
					address:this.item.address,
					success(res) {
						this.item.coords.lat = res.result.location.lat;
						this.item.coords.lng = res.result.location.lng;
					}
				});
			}

			uni.openLocation({
				latitude: Number.parseFloat(this.item.coords.lat),
				longitude: Number.parseFloat(this.item.coords.lng),
				name: this.item.location,
				address: this.item.address,
				success() {

				}
			});
		},
		getPoster() {
			this.setShare();
			this.painterShow = true;

			if (!this.poster) {
				uni.showLoading({
				  title: '加载中...'
				});
			} else {
				this.modalShow = true;
			}
		},
		comment() {
			if (this.hasLogin) {
				this.navTo(`/pages/pet/comment-post?pet_id=${this.item.id}`);
			} else {
				this.navToLogin();
			}
		},
		apply() {
			if (this.hasLogin) {
				if (this.item.member_id == this.userInfo.id) {
					this.$mHelper.toast('不能申请自己发布的送养宠物！');
				} else {
					this.navTo(`/pages/pet/apply-post?pet_id=${this.item.id}&owner_id=${this.item.member_id}`);
				}
			} else {
				this.navToLogin();
			}
		},
		contact(text) {
			this.$mHelper.toast(text);
		},
		getAstro(month, day) {
		  return '魔羯水瓶双鱼牡羊金牛双子巨蟹狮子处女天秤天蝎射手魔羯'.substr(month * 2-  (day < '102223444433' .charAt(month - 1) - -19) * 2, 2);
		},
		async downCallback() {
			await this.$http.get(`${petView}?expand=member,applys.member,comments,comments.member`, {
				id: this.id
			}).then(async r => {
				if (r.data.state) {
					this.navTo(`/pages/pet/404`);
				}
				
				this.loading = false;
				this.item = r.data;
				
				if (this.item.member.birthday) {
					let birthdays = this.item.member.birthday.split('-');
					let date = new Date();
					let year = date.getFullYear();
					
					this.item.member.age_name = year - birthdays[0];
					this.item.member.astro = this.getAstro(birthdays[1], birthdays[2]) + '座';
				}

				this.item.member.last_time_name = this.$u.timeFrom(this.item.member.last_time, 'yyyy年mm月dd日');
				this.mescroll.endSuccess();
			}).catch(err => {
				this.loading = false;
				this.navTo(`/pages/pet/404`);
			});
		},
		async setShare() {
			await this.$http.get(`${petShare}`, {
				id: this.id
			}).then(async r => {
				this.item.share = r.data;
			});
		},
		async setCommentLike(index, id) {
			if (this.hasLogin) {
				let comment_ids = JSON.parse(uni.getStorageSync('comment_ids') || "[]");
				
				if (comment_ids.includes(id)) {
					this.$mHelper.toast('不能重复点赞！');
				} else {
					await this.$http.get(`${commentLike}`, {
						id: id
					}).then(async r => {
						this.item.comments[index].like = r.data;
						comment_ids.push(id);
						uni.setStorageSync('comment_ids',  JSON.stringify(comment_ids));
					});
				}
			} else {
				this.navToLogin();
			}
		},
		navToLogin() {
			uni.setStorageSync('backToPage', JSON.stringify({ route: `${this.path}?id=${this.item.id}` }));

			let _this = this;
			uni.showModal({
				content: '未登录，是否跳转登录页面？',
				success(res) {
					if (res.confirm) {
						_this.navTo(`/pages/public/logintype`);
					}
				}
			});
		},
		onImgOk(e) {
			this.poster = e.detail.path;
			this.modalShow = true;
			uni.hideLoading();
		},
		onImgErr() {
			this.modalShow = false;
		},
		modalConfirm() {
			let _this = this;
			uni.saveImageToPhotosAlbum({
				filePath: this.poster,
				success() {
					_this.$mHelper.toast('已保存到相册！');
				},
				fail(error) {
					uni.showModal({
						title: '提示',
						content: '须微信授权才能保存到相册！',
						success(res) {
							if (res.confirm) {
								uni.openSetting({
								  success(res) {
								    //console.log(res.authSetting);
								  }
								});
							}
						}
					});
				},
				complete(res) {
					//console.log(res);
				}
			});
		},
		previewImage(index) {
			let urls = this.item.images;

			uni.previewImage({
				urls: urls,
				current: index
			});
		}
	}
};
</script>

<style lang="scss">
page {
	background: $color-white;
	.wrap {
		background: #F5F5F5;
		.swiper-box { 
			height: 600upx;
			video {
				width: 100%;
				height: 600upx;
			}
		}
		.slot-content {
			image {
				display: block;
			}
		}
		.subwrap {
			background: $color-white;
			padding: 0 30upx;
			.title {
				font-size: 30upx;
			}
			.reward {
				color: #EB568F;
				font-size: 24upx;
				.money {
					color: #EB568F;
					font-size: 50upx;
				}
			}
			.summary {
				color: #666;
				font-size: 24upx;
				margin-top: 16upx;
			}
			.bottom {
				color: #999;
				display: flex;
				justify-content: space-between;
				.area {
					width: 33%;
					font-size: 24upx;
					u-icon, .u-icon { 
						color: #FBDC5C
					}
				}
				.datetime {
					font-size: 22upx;
				}
				.view {
					font-size: 22upx;
					margin-left: 40upx;
				}
				.share {
					font-size: 22upx;
					margin-left: 30upx;
				}
			}
			.feature {
				display: flex;
				.left {
					flex-basis: 22%;
				}
				.right {
					//flex-grow: 1;
					flex-basis: 78%;
					.nickname {
						color: #333;
						font-size: 30upx;
					}
					.base {
						.tag {
							height: 40upx;
							line-height: 40upx;
							background: #F6F7FB;
							color: #666;
							font-size: 24upx;
							border-radius: 6upx;
							padding: 0 20upx;
							margin: 0 14upx 14upx 0;
							display: inline-block;
						}
					}
					.status {
						.tag {
							height: 38upx;
							line-height: 38upx;
							color: #CCC;
							font-size: 24upx;
							border: 2upx #CCC solid;
							border-radius: 6upx;
							padding: 0 20upx;
							margin: 0 14upx 14upx 0;
							display: inline-block;
						}
					}
				}
			}
			.sheet {
				color: #666;
				.datetime {
					font-size: 26upx;
				}
				.address {
					font-size: 26upx;
					margin: 10upx 0 20upx 0;
				}
			}
			.subtitle {
				color: #333;
				font-size: 30upx;
				.line {
					width: 6upx;
					height: 26upx;
					background: #FFCE0C;
					border-radius: 6upx;
					display: inline-block;
					margin: 10upx 16upx 0 0;
				}
			}
			.content {
				line-height: 48upx;
				color: #666;
				font-size: 30upx;
			}
			.requirements {
				color: #333;
				font-size: 28upx;
				image {
					width: 28upx;
					height: 28upx;
					margin-right: 20upx;
				}
			}
			.user {
				display: flex;
				.left {
					flex-basis: 22%;
				}
				.right {
					flex-grow: 1;
					.nickname {
						color: #262628;
						font-size: 30upx;
						.level {
							color: #666;
							margin-left: 10upx;
							display: inline-block;
							text {
								font-size: 30upx;
							}
						}
					}
					.profile {
						color: #999;
						font-size: 22upx;
						margin-top: 10upx;
						display: flex;
						text {
							padding: 0 10upx;
						}
					}
				}
			}
			.contact {
				display: flex;
				justify-content: space-around;
				.contact-item {
					text-align: center;
					.icon {
						display: flex;
						justify-content: center;
					}
					.name {
						color: #666;
						font-size: 24upx;
						margin-top: 20upx;
					}
				}
			}
			.applys {
				.u-avatar { margin-right: 20upx;}
			}
			.comments {
				.item {
					display: flex;
					flex-wrap: wrap;
					margin-top: 60upx;
					.left {
						flex-basis: 16%;
					}
					.center {
						flex-basis: 76%;
						.nickname {
							color: #333;
							font-size: 24upx;
							margin-top: 6upx;
						}
						.datetime {
							color: #999;
							font-size: 20upx;
						}
					}
					.right {
						flex-basis: 6%;
						text-align: center;
						image {
							width: 28upx;
							height: 28upx;
							vertical-align: middle;
						}
						.like image {
							height: 26upx;
						}
					}
				}
				.bottom {
					flex-basis: 100%;
					.content {
						color: #333;
						font-size: 24upx;
						padding-top: 10upx;
					}
				}
			}
		}
		.comments-wrap {
			margin-bottom: 150upx;
		}
		.nav-bottom {
			width: 100%;
			background: #FFF;
			border: 2upx #F2F2F2 solid;
			position: fixed;
			bottom: 0;
			padding: 16upx 0 10upx 0;
			z-index: 999;
			display: flex;
			.left {
				flex-basis: 50%;
				display: flex;
				justify-content: space-around;
				align-items: center;
				.item {
					text-align: center;
					image {
						width: 40upx;
						height: 38upx;
					}
					.text {
						color: #333;
						font-size: 20upx;
					}
				}
				button.item {
					height: auto;
					line-height: 40upx;
					background: none;
					font-size: 28upx;
					border: 0;
					padding: 0;
					margin: 0;
				}
				button:after {
					border: 0;
				}
			}
			.right {
				flex-grow: 1;
				text-align: center;
			}
		}
		.slot-content {
			min-height: 400upx;
		}
	}
}
</style>
