<template>
	<view>
		<view class="logo-box">
			<image class="logo" src="/static/image/logo.png"></image>
			<view><text>分享 GitHub 上\n有趣、入门级的开源项目</text></view>
		</view>
		<view class="uni-padding-wrap tip-box">
			<view class="uni-title uni-common-mt tip-title">
				请选择感兴趣的标签
				<text>\n我们将根据你的选择提供个性化推荐</text>
			</view>
		</view>
		<view class="liketab-box">
			<checkbox-group @change="checkboxChange">
				<label class="liketab-cell" v-for="(item,index) in items" :key="index"  :style="{backgroundColor: item.checkcss}" >
					<checkbox :value="item.id" :checked="item.checked" style="display: none;"/>{{item.name}}
				</label>
			</checkbox-group>
		</view>
		<view class="uni-padding-wrap">
			<button :type="buttonType" :disabled="buttonDisabled" style="float: left;" size="mini" @tap="getLikeTab">{{buttonText}}</button>
			<button type="default" style="float: right;" size="mini" @tap="goIndex">跳过</button>
		</view>
	</view>
</template>
<script>
	export default {
		data() {
			return {
				items: [],
				buttonText: '至少选择1个标签', //已选择n个标签
				buttonDisabled: true,
				buttonType: "default",
				likeTabs: [],
				isActive: -1,
				myTabs:[]
			}
		},
		onLoad() {
			uni.showLoading({
				title: '加载中'
			});
			if(uni.getStorageSync('likeTabsStatus')){
				uni.reLaunch({
					url: '/pages/category/category'
				})
			}
			const value = uni.getStorageSync('likeTabs');
			if (value) {
				this.likeTabs = this.myTabs = value;
				this.buttonText = "已选择 " + this.myTabs.length + " 个标签";
				this.buttonDisabled = false;
				this.buttonType = "primary";
			}
			this.getCategory();
		},
		methods: {
			checkboxChange: function(e) {
				var items = this.items,
                    values = e.detail.value;
                for (var i = 0, lenI = items.length; i < lenI; ++i) {
                    const item = items[i];
                    if(values.includes(item.id.toString())){
                        this.$set(item,'checked',true)
                        this.$set(item,'checkcss',"cornflowerblue")
                    }else{
                        this.$set(item,'checked',false)
                        this.$set(item,'checkcss',"gainsboro")
                    }
                }
				
				if (e.detail.value.length > 0) {
					this.buttonText = "已选择 " + e.detail.value.length + "/"+this.items.length+" 个标签";
					this.buttonDisabled = false;
					this.buttonType = "primary";
					this.likeTabs = values;
				} else {
					this.buttonText = "至少选择1个标签";
					this.buttonDisabled = true;
					this.buttonType = "default";
				}
			},
			getCategory() {
				uni.request({
					url: 'https://hellogithub.com//api/v1/category/',
					header: {
						'Authorization': 'Basic 32321312312312312312312'
					},
					success: (result) => {
						uni.hideLoading();
						this.items = result.data.payload;
						this.items.forEach((item) => {
							item['check'] = false;
							item['checkcss'] = "gainsboro";
							if(this.myTabs.includes(item.id.toString())){
							    this.$set(item,'checked',true)
							    this.$set(item,'checkcss',"cornflowerblue")
							}else{
							    this.$set(item,'checked',false)
							    this.$set(item,'checkcss',"gainsboro")
							}
						});
					}
				});
			},
			getLikeTab() {
				uni.setStorageSync('likeTabs', this.likeTabs);
				uni.setStorageSync('likeTabsStatus', true);
				uni.reLaunch({
					url: '/pages/category/category'
				})
			},
			goIndex(){
				uni.removeStorageSync('likeTabs');
				uni.setStorageSync('likeTabsStatus', false);
				uni.reLaunch({
					url: '/pages/category/category'
				})
			}
		}
	}
</script>

<style>
	.logo-box {
		text-align: center;
	}

	.logo {
		width: 100px;
		height: 100px;
	}

	.liketab-box {
		margin: 0 10px;
	}

	.liketab-cell {
		margin-right: 10px;
		margin-bottom: 10px;
		display: inline-block;
		background-color: gainsboro;

		color: #FFFFFF;
		padding: 5px;
		border-radius: 5px;
	}

	.check-cell:checked {
		background-color: cornflowerblue;
	}
</style>
