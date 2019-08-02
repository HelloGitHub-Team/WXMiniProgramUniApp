<template>
	<view class="uni-tab-bar">
		<scroll-view id="tab-bar" class="uni-swiper-tab" scroll-x :scroll-left="scrollLeft">
			<view v-for="(tab, index) in tabBars" :key="tab.ref" :class="['swiper-tab-list',tabIndex==index ? 'active' : '']"
			 :id="tab.ref" :data-current="index" @click="tapTab(index)">第{{tab.name}}期</view>
		</scroll-view>
		<!-- #ifndef MP-BAIDU -->
		<scroll-view class="list" v-for="(tabItem, idx) in newsList" :key="idx" v-if="tabIndex === idx" scroll-y
		 @scrolltolower="loadMore(idx)">
			<block v-for="(newsItem, newsIndex) in tabItem.data" :key="newsIndex">
				<uni-media-list :data="newsItem" @close="dislike(idx, newsIndex)" @click="goDetail(newsItem)"></uni-media-list>
			</block>
			<view class="uni-tab-bar-loading">
				<view class="loading-more">{{loadingText}}</view>
			</view>
		</scroll-view>
		<!-- #endif -->
		<!-- #ifdef MP-BAIDU -->
		<view class="scroll-wrap" v-for="(tabItem, idx) in newsList" :key="idx">
			<scroll-view class="list" v-if="tabIndex === idx" scroll-y @scrolltolower="loadMore(idx)" :style="scrollViewHeight">
				<block v-for="(newsItem, newsIndex) in tabItem.data" :key="newsIndex">
					<uni-media-list :data="newsItem" @close="dislike(idx, newsIndex)" @click="goDetail(newsItem)"></uni-media-list>
				</block>
				<view class="uni-tab-bar-loading">
					<view class="loading-more">{{loadingText}}</view>
				</view>
			</scroll-view>
		</view>
		<!-- #endif -->
	</view>
</template>
<script>
	import uniMediaList from '@/components/uni-media-list/uni-media-list.vue';
	import uniLoadMore from '@/components/uni-load-more/uni-load-more.vue';

	import {
		friendlyDate
	} from '@/common/util.js';

	export default {
		components: {
			uniMediaList,
			uniLoadMore
		},
		data() {
			return {
				loadingText: {
					contentdown: '上拉加载更多',
					contentrefresh: '正在加载...',
					contentnomore: '没有更多数据了'
				},
				scrollLeft: 0,
				refreshing: false,
				refreshText: '下拉可以刷新',
				newsList: [],
				tabIndex: 0,
				tabBars: []
			}
		},
		computed: {
			scrollViewHeight() {
				return 'height:' + (uni.getSystemInfoSync().windowHeight) + 'px';
			}
		},
		onLoad: function() {
			uni.showLoading({
				title: '加载中'
			});
			// 初始化列表信息
			this.getCategory();
			// this.tabBars.forEach((tabBar) => {
			// 	this.newsList.push({
			// 		data: [],
			// 		requestParams: {
			// 			columnId: tabBar.id,
			// 			minId: 0,
			// 			pageSize: 10,
			// 			column: 'id,post_id,title,author_name,cover,published_at,comments_count'
			// 		},
			// 		loadingText: '加载中...'
			// 	});
			// });
			// this.getList();
		},
		methods: {
			getCategory() {
				uni.request({
					url: 'https://hellogithub.com//api/v1/volume/',
					header:{
						'Authorization':'Basic 32321312312312312312312'
					},
					success: (result) => {
						uni.hideLoading();
						this.tabBars = result.data.payload;
						console.log(this.tabBars)
						this.tabBars.forEach((tabBar) => {
							this.newsList.push({
								data: [],
								refreshing: false,
								refreshFlag: false,
								refreshText: "",
								requestParams: {
									catid: tabBar.id,
									page: 1,
								},
								has_more: true,
								loadingText: '加载中...'
							});
						});
						this.getList()
					}
				});
			},
			getList(action = 1) {
				let activeTab = this.newsList[this.tabIndex];
				activeTab.requestParams.time = new Date().getTime() + '';
				if (action === 1) {
					this.newsList[this.tabIndex].requestParams.page = 1;
				}
				if (!this.newsList[this.tabIndex].has_more){
					this.loadingText = '没有更多数据了';
					return;
				}
				this.loadingText = '加载中...';
				uni.request({
					url: 'https://hellogithub.com//api/v1/volume/projects/?q={"id":'+this.newsList[this.tabIndex].requestParams.catid+', "page":'+this.newsList[this.tabIndex].requestParams.page+'}',
					method:"GET",
					header:{
						'Authorization':'Basic 32321312312312312312312'
					},
					success: (result) => {
						if (result.statusCode == 200) {
							const data = result.data.payload.map((news) => {
								return {
									category: news.category,
									sort_desc: news.sort_desc,
									img_url: news.img_url,
									name: news.name,
									update_time: news.update_time,
									volume: news.volume,
									url: news.url
								};
							});
							if (action === 1) {
								activeTab.data = data;
								this.refreshing = false;
							} else {
								data.forEach((news) => {
									activeTab.data.push(news);
								});
							}
							this.newsList[this.tabIndex].requestParams.page++;
							this.newsList[this.tabIndex].has_more = result.data.has_more;
						}
					}
				});
			},
			goDetail(detail) {
				console.log(detail)
				uni.setClipboardData({
					data: detail.url,
					success: function() {
						uni.showToast({
							title: '项目地址已复制',
							duration: 2000
						});
					}
				});
			},
			// dislike(tabIndex, newsIndex) {
			// 	uni.showModal({
			// 		content: '不感兴趣？',
			// 		success: (res) => {
			// 			if (res.confirm) {
			// 				this.newsList[tabIndex].data.splice(newsIndex, 1);
			// 			}
			// 		}
			// 	})
			// },
			loadMore() {
				console.log('load more');
				this.getList(2);
			},
			async changeTab(event) {
				let index = event.detail.current;
				if (this.isClickChange) {
					this.tabIndex = index;
					this.isClickChange = false;
					return;
				}
				let tabBar = await this.getElSize('tab-bar');
				let tabBarScrollLeft = tabBar.scrollLeft;
				let width = 0;

				for (let i = 0; i < index; i++) {
					let result = await this.getElSize(this.tabBars[i].ref);
					width += result.width;
				}
				let winWidth = uni.getSystemInfoSync().windowWidth,
					nowElement = await this.getElSize(this.tabBars[index].ref),
					nowWidth = nowElement.width;
				if (width + nowWidth - tabBarScrollLeft > winWidth) {
					this.scrollLeft = width + nowWidth - winWidth;
				}
				if (width < tabBarScrollLeft) {
					this.scrollLeft = width;
				}
				this.isClickChange = false;
				this.tabIndex = index;

				// 首次切换后加载数据
				const activeTab = this.newsList[this.tabIndex];
				if (activeTab.data.length === 0) {
					this.getList();
				}
			},
			getNodeSize(node) {
				return new Promise((resolve, reject) => {
					dom.getComponentRect(node, (result) => {
						resolve(result.size);
					});
				});
			},
			onRefresh(event) {
				this.refreshText = '正在刷新...';
				this.refreshing = true;
				this.getList();
			},
			getElSize(id) { //得到元素的size
				return new Promise((res, rej) => {
					uni.createSelectorQuery().select('#' + id).fields({
						size: true,
						scrollOffset: true
					}, (data) => {
						res(data);
					}).exec();
				});
			},
			async tapTab(index) { //点击tab-bar
				if (this.tabIndex === index) {
					return false;
				} else {
					// 					let tabBar = await this.getElSize('tab-bar'),
					// 						tabBarScrollLeft = tabBar.scrollLeft; //点击的时候记录并设置scrollLeft
					// 					this.scrollLeft = tabBarScrollLeft;
					// 					this.isClickChange = true;
					console.log('ssss')
					this.tabIndex = index;
					// 首次切换后加载数据
					const activeTab = this.newsList[this.tabIndex];
					if (activeTab.data.length === 0) {
						this.getList();
					}
				}
			},
		}
	}
</script>
<style>
	page {
		background-color: #FFFFFF;
		height: 100%;
		font-size: 11px;
		line-height: 1.8;
	}

	.uni-tab-bar {
		display: flex;
		flex: 1;
		flex-direction: column;
		overflow: hidden;
		height: 100%;
	}

	.uni-tab-bar .list {
		width: 750upx;
		height: calc(100% - 100upx);
		margin-top: 100upx;
	}

	.uni-swiper-tab {
		width: 100%;
		white-space: nowrap;
		line-height: 100upx;
		height: 100upx;
		border-bottom: 1px solid #c8c7cc;
		position: fixed;
		background: #FFFFFF;
		z-index: 999;
		top: var(--window-top);
		left: 0;
	}

	.swiper-tab-list {
		font-size: 30upx;
		width: 150upx;
		display: inline-block;
		text-align: center;
		color: #555;
	}

	.uni-tab-bar .active {
		color: #007AFF;
	}

	.uni-tab-bar .swiper-box {
		flex: 1;
		width: 100%;
		height: calc(100% - 100upx);
		overflow: scroll;
	}

	.uni-tab-bar-loading {
		text-align: center;
		padding: 20upx 0;
		font-size: 14px;
		color: #CCCCCC;
	}
</style>
