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
				<view 
					class="list-item"
					v-for="(item, i) in list" :key="i" 
					:id="'item'+item.userId">
					<view class="flex-row list-row fake-item" v-if="!item.visible">
						<view class="avatar">{{i}}</view>
						<view class="col">
							<view class="flex-row texts">
							</view>
						</view>
					</view>
					<view 
						v-else
						class="flex-row list-row">
						<image :src="item.avatar" class="avatar"></image>
						<view class="col">
							<view class="flex-row texts">
								<view class="nickname">{{item.nickName}}</view>
								<view class="sign-info">微信好友</view>
							</view>
							<view class="mood">{{item.remarks}}</view>
						</view>
						<view class="status remind"><text>{{i}}</text></view>
					</view>
				</view>
			</view>
		</scroll-view>		
	</view>
</template>
<script>
	import Vue from 'vue'
	// 共291条假数据
	let listData = require('../data.json')
	export default{
		name: 'List',
		data() {
			return {
				listHeight: 500,
				itemHeight: 74,
				scrollTop: 0,
				pageOffset: 0,
				pageSize: 10,
				pageTotal: Math.ceil(listData.data.length / 10),
				list: [],
			}
		},
		methods: {
			// 模拟加载显示下一页
			showNext(){
				let arr = listData.data
				
				if(this.pageOffset * this.pageSize >= arr.length){
					// 分页加载完了
					return
				}
				
				this.pageOffset++
				arr = arr.slice(this.pageOffset * this.pageSize - this.pageSize, this.pageOffset * this.pageSize)
				arr.map( v => {
					this.$set(v, 'visible', true)
				})
				this.list = this.list.concat(arr)
			},
			lower(){
				this.showNext()
			},
			// 当前页的前页的前页隐藏
			hidePrev(page){
				if(page - 2 > 0){
					for(let i=0; i< (page - 2) * this.pageSize; i++){
						this.list[i].visible = false
					}
				}
			},
			// 当前页的下页的下页隐藏
			hideNext(page){
				if(page + 1 <= this.pageOffset){
					let last = ((this.pageOffset - page) + page) * this.pageSize
					if(last > this.list.length){
						last = this.list.length
					}
					for(let i= (page + 1) * this.pageSize; i< last; i++){
						this.list[i].visible = false
					}	
				}
			},
			scroll(e){
				let scrollHeight = e.detail.scrollHeight
				let scrollTop = e.detail.scrollTop
				// 通过滚动条滚动到的位置 / 每页占用的高度计算出当前处于第几页
				let page = Math.ceil(scrollTop / (this.pageSize * this.itemHeight))
				
				// 滚动列表前一部分
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
	.page-divider{
		background-color: green;
	}
</style>
