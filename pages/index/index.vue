<template>
	<view class="scroll-cnt">
		<scroll-view 
			scroll-y="true"
			:style="{height: listHeight+'px'}" 
			:lower-threshold="200"
			@scrolltolower="lower" 
			:scroll-with-animation="true"
			@scroll="scroll">
			<view class="list">
				<view class="list-item" v-for="(item, i) in list" :key="i" :id="'item'+item.userId">
					<view class="flex-row list-row fake-item" v-if="!item.visible">
						<view class="avatar">{{i}}</view>
						<view class="col">
							<view class="flex-row texts">
							</view>
						</view>
					</view>
					<view class="flex-row list-row" v-else>
						<image :src="item.avatar" class="avatar"></image>
						<view class="col">
							<view class="flex-row texts">
								<view class="nickname">{{item.nickName}}</view>
								<view class="sign-info">
									<text v-if="item.friendBy">  “{{item.friendBy}}”的好友</text>
									<view v-else-if="item.sourceId">来自“<open-data type="groupName" :open-gid="item.sourceId"></open-data>”群聊</view>
									<text v-else>微信好友</text>
								</view>
							</view>
							<view class="mood">{{item.remarks}}</view>
						</view>
						<view class="status" :class="[item.statusIcon]" v-if="type == 1"></view>
						<view class="status remind not-remind" v-else-if="item.friendBy">
							<text>无法提醒</text>
						</view>
						<view class="status remind" :class="{'reminded': item.remind == 0}" v-else>
							<text v-if="item.remind == 0">已提醒</text>
							<button v-else open-type="share" class="status remind" @click.stop="remind(item)">提醒TA</button>
						</view>
						<view class="arrow collapse-btn" :class="{'arrow-up': item.show}"></view>
					</view>
				</view>
			</view>
		</scroll-view>		
	</view>
</template>
<script>
	import Vue from 'vue'
	let listData = require('../data.json')
	
	export default{
		name: 'List',
		data() {
			return {
				listHeight: 400,
				scrollTop: 0,
				index: 0,
				pageSize: 10,
				list: [],
				old: {
					scrollTop: 0
				}
			}
		},
		methods: {
			showNext(){
				let arr = listData.data
				if(this.index * this.pageSize >= arr.length){
					return
				}
				this.index++
				arr = arr.slice(this.index * this.pageSize - this.pageSize, this.index * this.pageSize)
				arr.map( v => {
					this.$set(v, 'visible', true)
				})
				this.list = this.list.concat(arr)
			},
			lower(){
				this.showNext()
			},
			hidePrev(page){
				if(page - 1 > 0){
					for(let i=0; i< (page-1) * this.pageSize; i++){
						this.list[i].visible = false
					}
				}
			},
			hideNext(page){
				if(page + 1 <= this.index){
					let last = ((this.index - page) + page) * this.pageSize
					if(last > this.list.length){
						last = this.list.length
					}
					for(let i=(page + 1) * this.pageSize; i< last; i++){
						this.list[i].visible = false
					}	
				}
			},
			scroll(e){
				let scrollHeight = e.detail.scrollHeight
				let scrollTop = e.detail.scrollTop
				// 通过滚动条滚动到的位置 / 每页占用的高度计算出当前是第几页
				let page = Math.round(scrollTop / (this.pageSize * 74))
				console.log(page)
				
				// 隐藏掉page之前的所有条目
				this.hidePrev(page)
				
				// 只显示当前page对应的条目
				if(page - 1 >= 0){
					let next = (page + 1) * this.pageSize
					if(next > this.list.length){
						next = this.list.length
					}
					for(let i=(page-1) * this.pageSize; i< next; i++){
						this.list[i].visible = true
					}
				}
				// 隐藏掉page之后的所有条目
				this.hideNext(page)
			}
		},
		mounted(){
			this.showNext()
		}
	}
</script>

<style lang="scss" scoped>
	@import '@/scss/common.scss';
	.scroll-cnt{
		padding: 0 28upx;
	}
	.scroll{
		margin-bottom: 80upx;
		background-color: white;
	}
	.list-row{
		box-sizing: border-box;
		padding: 24upx 0;
		height: 148upx;
		overflow: hidden;
		background-color: red;
		border-bottom: 1px solid green;
		.avatar{
			margin-right: 16upx;
			width: 96upx;
			height: 96upx;
			background-color: $colorLight;
			border-radius: 100%;
			overflow: hidden;
		}
		.texts{
			color: #808080;
			font-size: 24upx;
			font-weight: 500;
		}
		.nickname{
			margin-right: 24upx;
		}
		.status{
			margin-right: 16upx;
			width: 72upx;
			height: 72upx;
			background-size: 100%;
		}
		.mood{
			font-size: 32upx;
			line-height: 40upx;
			font-weight: 400;
		}
		.collapse-btn{
			width: 36upx;
			height: 40upx;
		}
		.remind{
			&::after{
				display: none;
			}
			padding: 0;
			width: 132upx;
			height: 64upx;
			text-align: center;
			line-height: 64upx;
			font-size: 24upx;
			color: white;
			background: rgba(51,204,136,1);
			border-radius: 36upx;
		}
		.reminded{
			color: white;
			background: #F2F2F2;
		}
		.not-remind{
			color: #A6A6A6;
			font-size: 24upx;
			background: none;
		}
	}
	.icon-share{
		display: inline-flex;
		align-items: center;
	}
	.icon-share{
		margin-right: 16upx;
		width: 36upx;
		height: 36upx;
		background-image: url(../../static/image/icon-share-white.png);
		background-size: 100%;
	}
	.no-data{
		padding-top: 64upx;
		text-align: center;
		font-size: 32upx;
		color: #A6A6A6;
	}
	.bottom-view{
		padding: 48upx 0 88upx;
	}
	.share-btn{
		margin: 0 auto 0;
		width: 480upx;
		height: 80upx;
		line-height: 80upx;
		font-size: 32upx;
		color: white;
		background: rgba(51,204,136,1);
		box-shadow: 0 4upx 16upx 0 rgba(36,169,110,0.5);
		border-radius: 40upx;
	}
	.health{
		background-image: url(~@/static/image/components/poster/health.png);
	}
	.health-normal{
		background-image: url(~@/static/image/components/poster/health-normal.png);
	}
	.health-bad{
		background-image: url(~@/static/image/components/poster/health-bad.png);
	}
	.health-unsign{
		background-image: url(~@/static/image/components/poster/health-unsign.png);
	}
	.health-min{
		background-image: url(~@/static/image/health.png);
	}
	.health-normal-min{
		background-image: url(~@/static/image/health-normal.png);
	}
	.health-bad-min{
		background-image: url(~@/static/image/health-bad.png);
	}
	
	.fake-item{
		.avatar{
			background-color: #f2f2f2;
		}
		.texts{
			width: 100%;
			height: 40upx;
			background-color: #f2f2f2;
		}
	}
</style>
