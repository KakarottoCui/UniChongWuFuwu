<template>
	<view >

		
		<!--列表-->
		<view class="pet-wrap u-p-20">
			
			<u-cell-group title="服务列表">
				<u-cell-item @click="getInfo(item)" v-for="(item,index) in items" :key="index" icon="setting-fill" :title="item.title+'['+(item.type == 1 ?'商城':'服务')+']'" :label="item.orderTime||deal2(item.stat)" value="详情" :arrow="true"></u-cell-item>
			</u-cell-group>
			
		</view>
		<u-calendar z-index="99999999" max-date="2050-01-01" v-model="showDate" mode="date" @change="showAdd=false" />
		<u-modal v-model="showAdd" title="服务明细" :show-cancel-button="true" confirm-text="确定" >
			<view class="slot-content u-p-30">
				
				<u-field
							v-model="form.title"
							label="服务名"
							placeholder=""
							:disabled="true"
						>
						</u-field>
				
								
				<u-field
							v-model="form.price"
							label="价格"
							placeholder=""
							:disabled="true"
						>
						</u-field>
						
						<u-field
									v-model="form.orderTime"
									label="预约日期"
									placeholder="请选择日期"
									:disabled="true"
									@click="showDate=true;"
								>
								</u-field>
										
						<u-field
									v-model="form.memo"
									label="备注"
									placeholder="请输入相关备注信息"
									type="textarea"
								>
								</u-field>
								
							
								<u-field
											:value="deal(form.stat)"
											label="状态"
											placeholder="状态"
											:disabled="true"
										>
										</u-field>

			</view>
		</u-modal>
		
		<u-modal v-model="showAdd2" title="订单明细" :show-cancel-button="true" @confirm="changeState(form)" :confirm-text="form.stat == 1 ?'收货':'确定'" >
			<view class="slot-content u-p-30">
				
				<u-field
							v-model="form.title"
							label="商品"
							placeholder=""
							:disabled="true"
						>
						</u-field>
				
								
				<u-field
							v-model="form.price"
							label="价格"
							placeholder=""
							:disabled="true"
						>
						</u-field>
						
						<u-field
									v-model="form.createTime"
									label="下单日期"
									placeholder="请选择日期"
									:disabled="true"
									@click="showDate=true;"
								>
								</u-field>
										
								
							
								<u-field
											:value="deal2(form.stat)"
											label="状态"
											placeholder="状态"
											:disabled="true"
										>
										</u-field>
										
								<u-field v-if="form.stat > 0"
											:value="form.expressName"
											label="物流"
											placeholder="状态"
											:disabled="true"
										>
										</u-field>		
		
			</view>
		</u-modal>
		
	</view>
</template>

<script>
	import appRequest from "@/common/appRequestUrl.js";
	export default {
		data() {
			return {
				showDate:false,
				showAdd:false,
				showAdd2:false,
				userInfo: {},
				items:[],
				form:{
					title:"",
					content:"",
					price:""
				}
			};
		},onShow() {
			let userInfo = appRequest.getUserInfo();
			if(!userInfo){
				uni.switchTab({
					url:"/pages/profile/profile"
				})
			}else{
				this.userInfo = userInfo;
			}
			this.getData();
		},methods:{
			deal(text){
				return text==1?'已完成':'未完成';
			},
			deal2(index){
				return ["未发货","已发货","收货完成"][index];
			},
			changeDate(e){
				this.form['orderTime'] = e.result;
			},
			getInfo(item){
			
				this.form = item;
				if(item.type == 0){
					this.showAdd = true;
				}else{
					this.showAdd2 = true;
				}
				
			},
			switchTab(route) {
				uni.navigateTo({
					url:route
				})
				//this.$mRouter.switchTab({ route });
			},
			splitTag(text){
				return text.split(" ");
			},
			getFormObj(){
				return {
					title:"",
					content:"",
					tag:"",
					pic1Str:"",
					pic2Str:"",
					pic3Str:"",
					type:2
				}
			},
			getData(){
				var _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.findNmOrderList,
					data:{
						validFlag:1,
						creater:_this.userInfo.uid
					},
					success: function(res) {
						if (res.data.code == 200) {
							let obj = res.data.data;
							_this.items = obj;
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
			},changeState(form){
				if(form.stat != 1 ){
					return;
				}
				var _this = this;
				this.form.stat = 2;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.editNmOrder,
					data:_this.form,
					success: function(res) {
						if (res.data.code == 200) {
							_this.getData();
							uni.showToast({
								title:"收货成功",
								icon:"none"
							})
						}else{
							uni.showToast({
								title:"失败",
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
			
			},change(name) {
				let _this = this;
				uni.chooseImage({
					count: 1,
					sizeType: ["compressed"],
					success: function(res) {
						let src = res.tempFiles[0].path;
						uni.compressImage({
							src: src,
							width: 450,
							quality: 70,
							success: function(path) {
								let filePath = path.tempFilePath;
								wx.getFileSystemManager().readFile({
									filePath: filePath,
									encoding: 'base64',
									fail: function(errMsg) {
										console.log(errMsg);
									},
									success: function(res) {
										_this.form[name] =
											"data:image/jpeg;base64," + res.data;
									}
								});
							}
						});
					},
					fail: function(errMsg) {
						console.log(errMsg);
					}
				});
			}
		}
	}
</script>

<style lang="scss">
.item {
	display: flex;
	flex-wrap: wrap;
	margin: 30upx;
	.left {
		flex-basis: 33%;
	}
	.right {
		//flex-grow: 1;
		//flex-basis: 67%;
		width: 67%;
		padding: 0 24upx;
		.title {
			font-size: 30upx;
			margin-top: -10upx;
		}
		.base {
			margin-top: 12upx;
			.tag {
				height: 40upx;
				line-height: 40upx;
				background: #F6F7FB;
				color: #666;
				font-size: 24upx;
				border-radius: 10upx;
				padding: 0 20upx;
				margin-right: 14upx;
				display: inline-block;
			}
			.tag:nth-child(3) {
				margin-right: 0;
			}
		}
		.status {
			margin-top: 12upx;
			.tag {
				height: 38upx;
				line-height: 38upx;
				color: #CCC;
				font-size: 24upx;
				border: 2upx #CCC solid;
				border-radius: 10upx;
				padding: 0 20upx;
				margin-right: 14upx;
				display: inline-block;
			}
			.tag:nth-child(3) {
				margin-right: 0;
			}
		}
		.area {
			color: #666;
			font-size: 24upx;
			margin-top: 12upx;
		}
		.reward {
			color: #EB568F;
			font-size: 24upx;
			.money {
				color: #EB568F;
				font-size: 40upx;
			}
		}
		.summary {
			color: #666;
			font-size: 24upx;
			margin-top: 12upx;
		}
	}
	.u-line {
		color: #F1F1F1;
	}
	::v-deep u-line {
		flex-basis: 100%;
		color: #F1F1F1;
	}
	.bottom {
		color: #999;
		display: flex;
		flex-basis: 100%;
		margin-top: 20upx;
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
}

.pet-wrap {
			margin-top: 20upx;
			background: $color-white;
			.sticky {
				display: flex;
				align-items: center;
				.sticky-tabs {
					width: 80%;
				}
				.sticky-more {
					width: 20%;
					height: 80upx;
					line-height: 80upx;
					color: #999;
					font-size: 28upx;
					background: #FFF;
					text-align: center;
				}
			}
		}
</style>
