<template>
	<view class="center">
		<view class="logo">
			<view class="logo-img">
				<open-data type="userAvatarUrl"></open-data>
			</view>
			<view class="logo-title">
				<view class="uer-name">Hi，<view class="uer-name">
						<open-data type="userNickName" lang="zh_CN" ></open-data>
					</view>
				</view>
			</view>
		</view>
		<view class="center-list">
			<view class="center-list-item border-bottom" open-type="getUserInfo" @tap="goToLikeTab">
				<image class="list-icon" src="../../static/image/category_HL.png"></image>
				<text class="list-text">个性标签</text>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				login: false
			}
		},
		onLoad() {
			uni.showLoading({
				title: '加载中'
			});
			this.goLogin();
		},
		methods: {
			goLogin() {
				uni.hideLoading();
				if (!this.login) {
					uni.login({
						provider: 'weixin',
						success: function(loginRes) {
							console.log(loginRes);
							// 获取用户信息
							uni.getUserInfo({
								provider: 'weixin',
								success: function(infoRes) {
									console.log(infoRes);
									this.login = true;
								}
							});
						}
					});
				}
			},
			goToLikeTab() {
				uni.setStorageSync('likeTabsStatus', false);
				uni.reLaunch({
					url: '/pages/liketab/liketab'
				})
			}
		}
	}
</script>

<style>

	page,
	view {
		display: flex;
	}

	page {
		background-color: #f8f8f8;
	}

	.center {
		flex-direction: column;
	}

	.logo {
		width: 750upx;
		height: 240upx;
		padding: 20upx;
		box-sizing: border-box;
		background-color: #2F85FC;
		flex-direction: row;
		align-items: center;
	}

	.logo-hover {
		opacity: 0.8;
	}

	.logo-img {
		width: 150upx;
		height: 150upx;
		border-radius: 150upx;
	}

	.logo-title {
		height: 150upx;
		flex: 1;
		align-items: center;
		justify-content: space-between;
		flex-direction: row;
		margin-left: 20upx;
	}

	.uer-name {
		height: 60upx;
		line-height: 60upx;
		font-size: 38upx;
		color: #FFFFFF;
	}

	.go-login.navigat-arrow {
		font-size: 38upx;
		color: #FFFFFF;
	}

	.login-title {
		height: 150upx;
		align-items: self-start;
		justify-content: center;
		flex-direction: column;
		margin-left: 20upx;
	}

	.center-list {
		background-color: #FFFFFF;
		margin-top: 20upx;
		width: 750upx;
		flex-direction: column;
	}

	.center-list-item {
		height: 90upx;
		width: 750upx;
		box-sizing: border-box;
		flex-direction: row;
		padding: 0upx 20upx;
	}

	.border-bottom {
		border-bottom-width: 1upx;
		border-color: #c8c7cc;
		border-bottom-style: solid;
	}

	.list-icon {
		width: 45upx;
		height: 45upx;
		line-height: 90upx;
		font-size: 34upx;
		text-align: center;
		margin-right: 20upx;
		margin-top: 23upx;
		
	}

	.list-text {
		height: 90upx;
		line-height: 90upx;
		font-size: 34upx;
		color: #555;
		flex: 1;
		text-align: left;
	}

</style>
