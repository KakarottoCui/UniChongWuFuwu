<template>
	<view class="rf-notice-detail">
		<view class="banner" v-if="announceDetail.cover">
			<rf-image class="banner-img" :src="announceDetail.cover"></rf-image>
			<view class="banner-title in2line">{{ announceDetail.title }}</view>
		</view>
		<view class="banner-title" v-if="!announceDetail.cover">{{
			announceDetail.title
		}}</view>
		<view class="article-meta" v-if="announceDetail.created_at">
			<text class="article-author">{{ announceDetail.author_name }}</text>
			<text class="article-text">发布于</text>
			<text class="article-time">{{ announceDetail.created_at | time }}</text>
		</view>
		<view class="article-content">
			<rf-parser lazy-load :html="announceDetail.content"></rf-parser>
		</view>
		<rf-empty
			info="暂无商城公告详情"
			v-if="!announceDetail && !loading"
		></rf-empty>
		<!--加载动画-->
		<rfLoading isFullScreen :active="loading"></rfLoading>
	</view>
</template>

<script>
import { notifyAnnounceView } from '@/api/basic';
import moment from '@/common/moment';

export default {
	data() {
		return {
			announceDetail: {},
			id: undefined,
			loading: true
		};
	},
	filters: {
		// 时间格式化
		time(val) {
			return moment(val * 1000).format('YYYY-MM-DD HH:mm');
		}
	},
	onLoad(event) {
		this.id = event.id;
		this.getNotifyAnnounceView(event.id);
	},
	onShareAppMessage() {
		return {
			title: this.banner.title,
			path: `/pages/index/notice/detail?id=${this.id}`
		};
	},
	methods: {
		// 获取通知列表
		async getNotifyAnnounceView(id) {
			await this.$http
				.get(`${notifyAnnounceView}`, { id })
				.then(r => {
					this.loading = false;
					this.announceDetail = r.data;
					uni.setNavigationBarTitle({
						title: r.data.title
					});
				})
				.catch(() => {
					this.loading = false;
				});
		}
	}
};
</script>

<style lang="scss">
page {
	background: $color-white;
}
.rf-notice-detail {
	.banner {
		overflow: hidden;
		position: relative;
		height: 320rpx;
		width: 100%;
		.banner-img {
			display: inline-block;
			height: 100%;
			width: 100%;
		}
		.banner-title {
			max-height: 84rpx;
			overflow: hidden;
			position: absolute;
			bottom: 0;
			width: 100%;
			font-size: 32rpx;
			font-weight: 400;
			color: white;
			z-index: 11;
			background-color: rgba(0, 0, 0, 0.25);
			padding: 4rpx 20rpx;
		}
	}
	.banner-title {
		padding: $spacing-lg $spacing-lg 0;
		font-size: $font-lg;
	}
	.article-meta {
		padding: 20rpx 8rpx;
		display: flex;
		flex-direction: row;
		justify-content: flex-start;
		color: gray;

		.article-text {
			font-size: 26rpx;
			line-height: 50rpx;
			margin: 0 20rpx;
		}

		.article-author,
		.article-time {
			font-size: 30rpx;
		}
	}

	.article-content {
		padding: 0 30rpx;
		overflow: hidden;
		font-size: 30rpx;
		margin-bottom: 30rpx;
	}
}
</style>
