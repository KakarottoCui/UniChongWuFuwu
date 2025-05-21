<template>
	<view class="register">
		<!--顶部返回按钮-->
		<text class="back-btn iconfont iconzuo" @tap="navBack"></text>
		<view class="right-top-sign"></view>
		<!-- 设置白色背景防止软键盘把下部绝对定位元素顶上来盖住输入框等 -->
		<view class="wrapper">
			<view class="left-top-sign">REGISTER</view>
			<view class="welcome">
				账号注册！
			</view>
			<view class="input-content">
				<view class="input-item">
					<text class="tit">手机号码</text>
					<input
						type="number"
						v-model="registerParams.phone"
						placeholder="请输入手机号码"
						maxlength="11"
					/>
				</view>
				<view class="input-item">
					<text class="tit">密码</text>
					<input
						type="password"
						v-model="registerParams.password"
						placeholder="请输入密码"
						maxlength="18"
					/>
				</view>
				<view class="input-item">
					<text class="tit">确认密码</text>
					<input
						type="password"
						v-model="registerParams.password_repetition"
						placeholder="请输入确认密码"
						maxlength="18"
					/>
				</view>
				<view class="input-item">
					<text class="tit">昵称</text>
					<input
						type="text"
						v-model="registerParams.nickName"
						placeholder="请输入您的昵称"
						maxlength="12"
					/>
				</view>
				
			</view>
			<button
				class="confirm-btn"
				:class="'bg-' + themeColor.name"
				@tap="toRegister"
			>
				注册
			</button>
		</view>
		
		
	</view>
</template>
<script>
import { mapMutations } from 'vuex';
import { registerByPass, smsCode } from '@/api/login';
import moment from '@/common/moment';
	import appRequest from "@/common/appRequestUrl.js";
	import base64 from "@/common/base64.js"
export default {
	data() {
		return {
			appAgreementDefaultSelect: this.$mSettingConfig.appAgreementDefaultSelect, // 是否允许点击登录按钮
			closeRegisterPromoCode: this.$mSettingConfig.closeRegisterPromoCode, // 是否允许点击登录按钮
			registerParams: {
				phone: '',
				password: '',
				password_repetition: '',
				promoCode: '',
				nickName: '',
				code: ''
			},
			btnLoading: false,
			reqBody: {},
			codeSeconds: 0, // 验证码发送时间间隔
			smsCodeBtnDisabled: true
		};
	},
	onShow() {
		if (uni.getStorageSync('accessToken')) {
			this.$mRouter.reLaunch({ route: '/pages/index/index' });
		}
	},
	onLoad(options) {
		if (this.$mStore.getters.hasLogin) {
			this.$mRouter.reLaunch({ route: '/pages/index/index' });
		}
		const time =
			moment().valueOf() / 1000 - uni.getStorageSync('registerSmsCodeTime');
		if (time < 60) {
			this.codeSeconds =
				this.$mConstDataConfig.sendCodeTime - parseInt(time, 10);
			this.handleSmsCodeTime(this.codeSeconds);
		} else {
			this.codeSeconds = this.$mConstDataConfig.sendCodeTime;
			this.smsCodeBtnDisabled = false;
			uni.removeStorageSync('registerSmsCodeTime');
		}
		const backUrl = uni.getStorageSync('backToPage');
		if (backUrl.indexOf('promo_code') !== -1) {
			this.registerParams.promoCode = JSON.parse(backUrl)['query']['promo_code'];
		} else {
			this.registerParams.promoCode = options.promo_code || uni.getStorageSync('promoCode');
		}
	},
	methods: {
    isAppAgreementDefaultSelect() {
      // 是否选择协议
      this.appAgreementDefaultSelect = !this.appAgreementDefaultSelect;
    },
		...mapMutations(['login']),
		navBack() {
			this.$mRouter.back();
		},
		// 通用跳转
		navTo(route) {
			this.$mRouter.push({ route });
		},
		// 获取手机验证码
		getSmsCode() {
			this.reqBody['mobile'] = this.registerParams['mobile'];
			let checkSendCode = this.$mGraceChecker.check(
				this.reqBody,
				this.$mFormRule.sendCodeRule
			);
			if (!checkSendCode) {
				this.$mHelper.toast(this.$mGraceChecker.error);
				return;
			}
			this.$http
				.post(smsCode, {
					mobile: this.registerParams.mobile,
					usage: 'register'
				})
				.then(r => {
					if (r.data) {
						this.$mHelper.toast(`验证码发送成功, 验证码是${r.data}`);
					} else {
						this.$mHelper.toast('验证码已发送.');
					}
					this.smsCodeBtnDisabled = true;
					uni.setStorageSync('registerSmsCodeTime', moment().valueOf() / 1000);
					this.handleSmsCodeTime(59);
				});
		},
		handleSmsCodeTime(time) {
			let timer = setInterval(() => {
				if (time === 0) {
					clearInterval(timer);
					this.smsCodeBtnDisabled = false;
				} else {
					this.codeSeconds = time;
					this.smsCodeBtnDisabled = true;
					time--;
				}
			}, 1000);
		},
		// 注册账号
		async toRegister() {
			
			this.reqBody['phone'] = this.registerParams['phone'];
			this.reqBody['password'] = this.registerParams['password'];
			this.reqBody['nickName'] = this.registerParams['nickName'];

			if (
				this.registerParams['password'] !==
				this.registerParams['password_repetition']
			) {
				this.$mHelper.toast('两次输入的密码不一致');
				return;
			}
			
			var _this = this;
			let base64en = new base64();
			this.reqBody["password"] = base64en.encode(this.reqBody["password"]);
			appRequest.request({
				method: "POST",
				url: appRequest.urlMap.regist,
				data:this.reqBody,
				success: function(res) {
					if (res.data.code == 200) {
						uni.showToast({
							title:"注册成功",
							icon:"none"
						})
						uni.switchTab({
							url:"/pages/pet/login/login"
						})
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
			
		}
	}
};
</script>
<style lang="scss">
.register {
	padding-top: 60px;
	position: relative;
	width: 100vw;
	overflow: hidden;
	background: #fff;

	.wrapper {
		position: relative;
		width: 100vw;
		z-index: 90;
		background: #fff;
		padding-bottom: 40rpx;

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
				width: calc(100% - 100rpx);
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

	.register-section {
		margin: 30rpx 0 50rpx;
		width: 100%;
		font-size: $font-sm + 2rpx;
		color: $font-color-base;
		text-align: center;

		text {
			color: $font-color-spec;
			margin-left: 10rpx;
		}
	}

}

</style>
