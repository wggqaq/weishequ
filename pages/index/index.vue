<template>
	<view id="index">
		<hello ref="hello" v-model="feedsList">
			<swiper class="box" :vertical="true" :circular="true">
				<swiper-item class="spitem" v-for="(item,index) in feedsList" key="index">
					<image class="bg-img" :src="item.cover">
						<view class="right_menu">
							<image class="user_hp" src="../../static/nopic.png" />
							<view class="imgBox" @tap="clickLove(feedsList[index])">
								<image v-if="item.has_like" src="@/static/lover.png" class="spitemImagelove" />
								<image v-else src="@/static/love.png" class="spitemImagelove" />
								<view><text class="rtext" v-if=" item.like_count>0 ">{{ item.like_count }}</text>
								</view>
							</view>
							<view class="imgBox">
								<navigator open-type="navigate" :url=" '/subpages/feedinfo/feedinfo?id=' + item.id">
									<image class="spitemImageremark" src="@/static/remark.png" />
									<view><text class="rtext"
											v-if=" item.feed_comment_count>0 ">{{ item.feed_comment_count }}</text>
									</view>
								</navigator>
							</view>
							<button class="imgBox" open-type="share" style="background:none;">
								<image class="spitemImageshare" src="@/static/share.png" />

							</button>
						</view>
						<view class="bottom_info">
							<view><text class="user_name">@{{item.name}}</text></view>
							<view><text class="video_introduce">{{item.feed_content}}</text></view>
						</view>
					</image>

				</swiper-item>


			</swiper>
		</hello>

		<!-- 登陆组件 -->
		<login ref="login" />
	</view>
</template>

<script>
	import {
		mapState,
		mapActions
	} from 'vuex'
	// 引入 时间日期格式化显示函数
	import timeFrom from '@/tools/timeFrom.js'
	// 引入 个性化 瀑布流组件
	import hello from '@/components/waterfall-sns/index.vue'
	import feedMixin from '@/givelike/todoFeed.js'
	export default {
		//导入js点赞模块js
		mixins: [feedMixin, hello],
		data() {
			return {
				// navBar 显示状态控制
				navBarShowTag: false,

				// 动态列表数据
				feedsList: [],

				// 资讯列表数据

				// // 当前 推荐 资讯 滑动位置
				// currentswiperIndex: 0,
				// 滑动页面轮播器的高度
				swiperSliderHeight: '500px',
				swiperSliderFeedsHeight: 0,
				swiperSliderNewsHeight: 0,
				// 记录滚动所在的位置
				oldFeedsScrollTop: 0,
				oldNewsScrollTop: 0
			}
		},

		computed: {
			...mapState(['loginState', 'userInfo'])
		},
		async onLoad() {

			uni.$on('indexUserLogin', () => {

				this.feedsList = []
				this.getFeedsList()
			})
			uni.$on('indexUserLogout', () => {

				this.feedsList = []
				this.getFeedsList()
			})

			// 发布新的动态后，触发数据更新
			uni.$on("indexFeedsUpdate", () => {

				this.feedsList = []
				this.getFeedsList()
			})

			// 个人中心删除一条动态后，触发更新首页数据
			uni.$on("indexFeedRemove", fid => {
				this.feedsList = []
				this.getFeedsList()

			})

			// 用户点赞一条动态后触发数据更新
			uni.$on('indexFeedLoveChange', item => {

				this.modify(item.id, "like_count", item.like_count);
				this.modify(item.id, "has_like", item.has_like)
			})

			uni.$on('myFeedCommentChange', item => {
				this.modify(item.id, "feed_comment_count", item.feed_comment_count)
			})
			// 我们要在这里初始化请求相关数据
			this.getFeedsList()
		},


		onReachBottom() {
			console.log('下拉到底啦')
			// 请求新的数据
			this.getFeedsList()
		},
		onPullDownRefresh() {
			console.log('顶部下拉刷新')
			this.feedsList = []
			// this.$refs.swiper.clear()
			this.getFeedsList()

		},
		filters: {
			timeFormate(timeDate) {
				let Time = new Date(timeDate);
				let timestemp = Time.getTime();
				let t = timeFrom(timestemp, "yyyy年mm月dd日");
				return t;
			}
		},
		methods: {

			async getFeedsList() {
				let feeds = await this.$u.api.getFeeds();
				console.log(feeds);
				let feedList = feeds.data.feeds.map(item => {
					return {
						...item,
						cover: this.$BaseFileURL + item.images[0].file,
						avatar: !!item.user.avatar ? item.user.avatar.url : '/static/nopic.png',
						name: item.user.name,
						has_like: item.has_like,
					}
				})
				this.feedsList = [...this.feedsList, ...feedList]
				// 在这里注册一个 uniAPP 的顶层事件，用来作为数据通信
				uni.$once("swiperHeightChange", height => {
					console.log('计算出来的高度为:' + height)
					this.swiperSliderFeedsHeight = height
					this.swiperSliderHeight = height
				})
				console.log(this.feedsList);
			},
			

			// 修改某条数据的某个属性  id, key, value
			modify(id, key, value) {
				//如果findIndex找不到合适的条件，就会返回-1
				let index = -1;
				index = this.feedsList.findIndex(val => val[this.idKey] == id);
				if (index != -1) {
					// 如果index不等于-1，说明已经找到了要找的id，修改对应key的值
					this.feedsList[index][key] = value;
				}
				// 修改父组件的数据中的对应id的条目
				index = this.value.findIndex(val => val[this.idKey] == id);
				if (index != -1) {
					// 首先复制一份value的数据
					let data = this.cloneData(this.value);
					// 修改对应索引的key属性的值为value
					data[index][key] = value;
					// 修改父组件通过v-model绑定的变量的值
					this.$emit('input', data);
				}
			},


		}
	}
</script>

<style lang="scss" scoped>
	#sns {
		background-color: #f1f1f1;
	}

	.box {
		height: 1224upx;

		.spitem {
			background-color: #333;
			height: 1224upx;
			text-align: center;
		}
	}

	.bg-img {
		height: 1224upx;
	}

	.right_menu {
		position: absolute;
		padding-top: 10px;
		bottom: 50rpx;
		right: 30rpx;
		z-index: 990;

		.user_hp {
			margin-bottom: 30rpx;
			width: 50px;
			height: 50px;
			border: 2px solid #ffffff;
			background-color: #fff;
			border-radius: 50%;
		}

		.imgBox {


			padding-top: 10px;
			align-items: center;

			.spitemImagelove {
				width: 90rpx;
				height: 90rpx;
			}

			.spitemImageremark {
				width: 70rpx;
				height: 80rpx;
			}

			.spitemImageshare {
				width: 80rpx;
				height: 90rpx;
			}

			.rtext {
				text-align: center;
				color: #ffffff;
				font-size: 24rpx;
				margin-top: 10rpx;
				margin-bottom: 30rpx;
			}
		}
	}

	.bottom_info {
		position: absolute;
		left: 30rpx;
		bottom: 40rpx;
		z-index: 990;

		.user_name {
			font-size: 40rpx;
			font-weight: bold;
			color: #ffffff;
			margin-bottom: 20rpx;
		}

		.video_introduce {
			font-size: 28rpx;
			top: 5rpx;
			color: #ffffff;
		}
	}
</style>