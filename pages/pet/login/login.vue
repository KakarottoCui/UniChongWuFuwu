<template>
	<view>
		<view class="container">
			<!--顶部返回按钮-->
			<text class="back-btn iconfont iconzuo" @tap="navBack"></text>
			<!--插画-->
			<view class="right-top-sign"></view>
			<!-- 设置白色背景防止软键盘把下部绝对定位元素顶上来盖住输入框等 -->
			<view class="wrapper">
				<view class="left-top-sign">LOGIN</view>
				<view class="welcome">
					欢迎回来！
				</view>
				<view class="input-content">
					<!--<view class="btn-group">-->
					<!--<button type="primary" @tap="loginTest('15083980039', '123456')">小明-父</button>-->
					<!--</view>-->
					<view class="input-item">
						<text class="tit">手机号</text>
						<input
							type="number"
							name="mobile"
							v-model="loginParams.phone"
							placeholder="请输入手机号"
							maxlength="11"
							
						/>
					</view>
					<view class="input-item" >
						<text class="tit">密码</text>
						<input
							name="password"
							type="password"
							v-model="loginParams.password"
							placeholder="请输入密码"
							maxlength="20"
						/>
					</view>
		
					
					<button
						class="confirm-btn"
						:class="'bg-' + themeColor.name"
						:disabled="btnLoading"
						:loading="btnLoading"
						@tap="login"
					>
						登录
					</button>
				</view>
			</view>
			<view class="register-section">
				还没有账号?
				<text @tap="navTo('/pages/pet/register/register')">马上注册</text>
				
			</view>
		</view>
	</view>
</template>

<script>
	import appRequest from "@/common/appRequestUrl.js";
	import base64 from "@/common/base64.js"
	export default {
		data() {
			return {
				loginParams: {
					phone: '',
					password: ''
				},
				registerParams: {
					mobile: '',
					password: '',
					password_repetition: '',
					promoCode: '',
					nickname: '',
					code: ''
				},
				btnLoading: false,
				reqBody: {},
				codeSeconds: 0, // 验证码发送时间间隔
				loginByPass: true,
				smsCodeBtnDisabled: true,
				userInfo: null,
				loginBg: this.$mAssetsPath.loginBg,
				loginPic: this.$mAssetsPath.loginPic,
				appName: this.$mSettingConfig.appName,
				styleLoginType: this.$mSettingConfig.styleLoginType,
				closeRegisterPromoCode: this.$mSettingConfig.closeRegisterPromoCode,
				tabCurrentIndex: 0,
				typeList: [
					{
						text: '登录'
					},
					{
						text: '注册'
					}
				]
			}
		},
		onLoad:function(){
			uni.clearStorage();
		},
		methods: {
			navTo(route) {
				this.$mRouter.push({ route });
			},
			login:function(){
				var _this = this;
				let base64en = new base64();
				let pass = base64en.encode(this.loginParams.password);
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.login,
					data:{
						phone:this.loginParams.phone,
						password:pass
					},
					success: function(res) {
						if (res.data.code == 200) {
							uni.showToast({
								title:"登录成功",
								icon:"none"
							})
							uni.setStorageSync('userInfo', res.data.data);
							uni.switchTab({
								url:"/pages/index/index"
							})
						}else{
							uni.showToast({
								title:"登录失败",
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
			}
		}
	}
</script>

<style lang="scss">
page {
	background: $color-white;
}
.container {
	padding-top: 115px;
	position: relative;
	width: 100vw;
	overflow: hidden;
	background: #fff;
	.wrapper {
		position: relative;
		z-index: 90;
		background: #fff;
		padding-bottom: 40rpx;
	}
	.back-btn {
		position: absolute;
		left: 40rpx;
		z-index: 9999;
		padding-top: var(--status-bar-height);
		top: 40rpx;
		font-size: 40rpx;
		color: $font-color-dark;
	}
	.left-top-sign {
		font-size: 120rpx;
		color: $page-color-base;
		position: relative;
		left: -16rpx;
	}
	.right-top-sign {
		position: absolute;
		top: 80rpx;
		right: -30rpx;
		z-index: 95;

		&:before,
		&:after {
			display: block;
			content: '';
			width: 400rpx;
			height: 80rpx;
			background: #b4f3e2;
		}

		&:before {
			transform: rotate(50deg);
			border-radius: 0 50px 0 0;
		}

		&:after {
			position: absolute;
			right: -198rpx;
			top: 0;
			transform: rotate(-50deg);
			border-radius: 50px 0 0 0;
			/* background: pink; */
		}
	}
	.left-bottom-sign {
		position: absolute;
		left: -270rpx;
		bottom: -320rpx;
		border: 100rpx solid #d0d1fd;
		border-radius: 50%;
		padding: 180rpx;
	}
	.welcome {
		position: relative;
		left: 50rpx;
		top: -90rpx;
		font-size: 46rpx;
		color: #555;
		text-shadow: 1px 0px 1px rgba(0, 0, 0, 0.3);
	}
	.input-content {
		padding: 0 60rpx;
	}
	.input-item {
		display: flex;
		flex-direction: column;
		align-items: flex-start;
		justify-content: center;
		padding: 0 30rpx;
		background: $page-color-light;
		height: 120rpx;
		border-radius: 4px;
		margin-bottom: 50rpx;

		&:last-child {
			margin-bottom: 0;
		}

		.tit {
			height: 50rpx;
			line-height: 56rpx;
			font-size: $font-sm + 2rpx;
			color: $font-color-base;
		}

		input {
			height: 60rpx;
			font-size: $font-base + 2rpx;
			color: $font-color-dark;
			width: 100%;
		}
	}
	.input-item-sms-code {
		position: relative;
		width: 100%;
		.sms-code-btn {
			position: absolute;
			color: #111;
			right: 20rpx;
			bottom: 20rpx;
			font-size: 28rpx;
		}

		.sms-code-resend {
			color: #999;
		}

		.sms-code-btn:after {
			border: none;
			background-color: transparent;
		}
	}
	.forget-section {
		font-size: $font-sm + 2rpx;
		color: $font-color-spec;
		text-align: center;
		margin-top: 40rpx;
	}
	.register-section {
		margin: 30rpx 0 50rpx 0;
		width: 100%;
		font-size: $font-sm + 2rpx;
		color: $font-color-base;
		text-align: center;

		text {
			color: $font-color-spec;
			margin-left: 10rpx;
		}

		text:first-child {
			margin-right: 10rpx;
		}
	}
	.btn-group {
		display: flex;
		margin-bottom: 20rpx;
	}
}
.login-type-2 {
	width: 100%;
	position: relative;
	.back-btn {
		position: absolute;
		left: 40rpx;
		z-index: 9999;
		padding-top: var(--status-bar-height);
		top: 40rpx;
		font-size: 48rpx;
		color: $color-white;
	}
	.login-top {
		height: 460rpx;
		position: relative;
		.desc {
			position: absolute;
			top: 200rpx;
			left: 40rpx;
			font-size: 48rpx;
			.title {
				font-size: 48rpx;
			}
		}
		.login-pic {
			position: absolute;
			width: 220rpx;
			height: 270rpx;
			right: 30rpx;
			top: 100rpx;
		}
	}
	.login-type-content {
		position: relative;
		top: -72rpx;
		.login-bg {
			width: 94vw;
			height: 94vw;
			margin: 0 3vw;
		}
		.main {
			width: 94vw;
			position: absolute;
			top: 0;
			left: 3vw;
			.nav-bar {
				display: flex;
				height: 100rpx;
				justify-content: center;
				align-items: center;
				position: relative;
				z-index: 10;
				.nav-bar-item {
					flex: 1;
					display: flex;
					height: 100%;
					line-height: 96rpx;
					font-size: $font-lg;
					display: flex;
					margin: 0 120rpx;
					justify-content: center;
				}
				.nav-bar-item-active {
					border-bottom: 5rpx solid;
				}
			}
			.login-type-form {
				width: 80%;
				margin: 50rpx auto;
				.input-item {
					position: relative;
					height: 90rpx;
					line-height: 90rpx;
					margin-bottom: $spacing-lg;
					.iconfont {
						font-size: 50rpx;
						position: absolute;
						left: 0;
					}
					.login-type-input {
						height: 90rpx;
						padding-left: 80rpx;
						border-bottom: 1rpx solid rgba(0, 0, 0, .1);
					}
					.sms-code-btn, sms-code-resend {
						width: 240rpx;
						font-size: $font-base - 2rpx;
					}
				}
			}
			.login-type-tips {
				padding: 0 50rpx;
				display: flex;
				justify-content: space-between;
			}
			.confirm-btn {
				height: 80rpx;
				line-height: 80rpx;
			}
		}
	}
	.login-type-bottom {
		width: 100%;
		padding-bottom: 30rpx;
		text-align: center;
		font-size: $font-lg;
	}
}
</style>
