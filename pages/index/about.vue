<template>
	<view class="content">
		<image class="logo" src="/static/152x152.png"></image>
		<view class="text-area">
			<text class="title flex flex-direction">河海打卡助手</text><br>
			<text class="title flex flex-direction">版本号：{{version}}</text><br>
	
		</view>
		<text class="title flex flex-direction" style="position:absolute;bottom:10rpx">i栋日志（www.idong.site）©版权所有</text>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				version: plus.runtime.version
			}
		},
		onLoad() {

		},
		methods: {

		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		justify-content: center;
		text-align: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
		text-align: center;
	}
</style>
