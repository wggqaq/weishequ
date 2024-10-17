<template>
	<view class="container">
		<!-- 聊天列表 -->

		<view class="chat-body">
			<block v-for="(item,index) in chatList" :key="index">
				<view class="chat-one" v-if="!item.isMe">
					<image class="chat-face" src="../../static/chat/seiver.jpg"></image>
					<view class="chat-box">
						<view class="chat-sender"></view>
						<view class="chat-content" v-if="item.type==='txt'">{{item.content}}</view>
						<image class="chat-img" v-if="item.type==='img'" :src="item.content" mode="widthFix"></image>
					</view>
				</view>

				<view class="chat-one chat-one-mine" v-else>
					<view class="chat-boxme">
						<view class="chat-contentme" v-if="item.type==='txt'">{{item.content}}</view>
						<image class="chat-img" v-if="item.type==='img'" :src="item.content" mode="widthFix"></image>
					</view>
					<image class="chat-face" src="../../static/chat/pyy.jpg"></image>
				</view>

			</block>  
		</view>
		<!-- 聊天输入 -->
		<view class="chat-footer">
			<image class="img-voice" src="../../static/voice.png" />
			<input class="msg-input" type="text" cursor-spacing="16" v-model="myInput">
			<image class="img-chose" src="../../static/chat/phote.png" @click="changeImage" />
			<view class="send-btn" @click="sendMsg">发送</view>
		</view>
	</view>
</template>

<script>
	
	export default {
		data() {
			return {
				chatList: [{
						isMe: false,
						type: "txt",
						content: '你好，我是微社区客服，请问你想问的吗',
					},
					{
						isMe: false,
						type: "img",
						content: '../../static/chat/seiver.jpg',
					},
					// {
					// 	isMe: true,
					// 	type: "txt",
					// 	content: '你的社区不太行',
					// }, {
					// 	isMe: true,
					// 	type: "img",
					// 	content: '../../static/chat/seiver.jpg',
					// }
				],
				myInput: '',
				xiaobingReply:null,
			}
		},

		onShow() {
			if (!!uni.getStorageSync('chatList')) {
				this.chatList = JSON.parse(uni.getStorageSync('chatList'))

				setTimeout(() => {
					uni.pageScrollTo({
						scrollTop: 999999999,
						duration: 0
					})
				}, 100);


			}
		},

		methods: {
			changeImage() {
				uni.chooseImage({
					count: 1,
					sizeType: ['original', 'compressed'],
					sourceType: ['album', 'camera'],
					success: res => {
						console.log(res.tempFilePaths[0])

						let senMsg = {
							isMe: true,
							type: 'img',
							content: res.tempFilePaths[0]
						}

						this.chatList.push(senMsg)
						let renMsg = {
							isMe: false,
							type: 'img',
							content: res.tempFilePaths[0]
						}
						this.chatList.push(renMsg)
						uni.pageScrollTo({
							scrollTop: 999999999,

						})
						uni.setStorageSync('chatList', JSON.stringify(this.chatList))
					}
				})
			},
			sendMsg() {
				
				
				if (!this.myInput) return
				let senMsg = {
					isMe: true,
					type: "txt",
					content: this.myInput,
				}
				this.chatList.push(senMsg)
				
			
				 //请求青云客api
				var url='http://api.qingyunke.com/api.php?key=free&appid=0&msg='+this.myInput
				
			    uni.request({
			    	url:url,
					method:'GET',
					
					data:{
						json:JSON.stringify(JSON)
					},
					success:res=> {
						this.xiaobingReply=res.data.content;
						
						// this.xiaobingReply=res.data.content
						console.log(this.xiaobingReply);
						console.log(typeof(this.xiaobingReply));
						
						let renMsg = {
							isMe:false,
							type:"txt",
							content:this.xiaobingReply,
							
						}
						this.chatList.push(renMsg)
					}
				   
			    })
				
				//小冰回答话
				
				this.myInput=''
				
				
				setTimeout(() => {
					uni.pageScrollTo({
						scrollTop: 999999999,
						duration: 0
					})
				}, 100);
				uni.setStorageSync('chatList', JSON.stringify(this.chatList))
			}
		}
	}
</script>

<style lang="scss" scoped>
	.container text {
		margin-left: 40%;
	}

	.container {
		background-color: #f1f1f1;
		min-height: 100vh;
	}

	.chat-body {
		padding-bottom: 178upx;

		.chat-time {
			text-align: center;
			color: #888;
			padding: 40upx 0 0;
		}

		.chat-one {
			display: flex;
			flex-direction: row;
			flex-wrap: wrap;
			justify-content: flex-start;
			padding: 20upx 0;
		}

		.chat-one-begin {
			padding: 40upx 0 0;
		}

		.chat-one-mine {
			justify-content: flex-end;
		}

		.chat-face {
			width: 84upx;
			height: 84upx;
			border-radius: 10upx;
			margin-left: 40upx;
		}

		.chat-one-mine .chat-face {
			margin-left: 0;
			margin-right: 40upx;
		}

		.chat-box {
			max-width: calc(100% - 290upx);
			margin-left: 20upx;
			font-size: 30upx;
		}
        
		.chat-boxme {
			max-width: calc(100% - 290upx);
			margin-right: 20upx;
			font-size: 30upx;
		}

		.chat-one-mine .chat-body {
			margin-right: 20upx;
		}

		.chat-sender {
			color: #888;
			font-size: 28upx;
			margin-top: -16upx;
			
		}

		.chat-content {
			background-color: #fff;
			border-radius: 8px;
			padding: 10px;

			.micon {
				margin-right: 20upx;
				color: #666;
			}
		}

         .chat-contentme {
         	background-color: #89D961;
         	border-radius: 8px;
         	padding: 10px;
         
         	.micon {
         		margin-right: 20upx;
         		color: #666;
         	}
         }
		 
		.chat-img {
			float: left;
			max-width: 60%;
			border-radius: 5px;
		}

		.chat-one-mine .chat-img {
			float: right;
		}
	}

	.chat-footer {
		width: 100%;
		padding: 0 40wpx;
		height: 120upx;
		position: fixed;
		bottom: 0;
		left: 0;
		background-color: #f1f1f1;
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
		justify-content: space-between;
		align-items: center;
		align-content: center;
		border-top: 1upx solid #ddd;

		.msg-input {
			background-color: #fff;
			width: calc(100% - 300upx);
			height: 70upx;
			line-height: 70upx;
			font-size: 30upx;
			border-radius: 10upx;
			padding: 0 5upx;
		}

		.img-chose {
			height: 70upx;
			width: 70upx;
		}

        .img-voice {
			height: 70upx;
			width: 70upx;
		}
		.send-btn {
			padding-top: 10upx;
			color: #ddd;
			text-align: center;
			width: 130upx;
			height: 70upx;
			border-radius: 20%;
			;
			background-color: green;
		}
	}
</style>
