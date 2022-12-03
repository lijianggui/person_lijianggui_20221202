<template>
	<view style="page">
		
		<view class="title">
			<view class="">任务名称：<text>{{title}}</text></view>
		</view>
		
		<view class="" style="margin-bottom: 50px;">
			<view class="content" v-for="item in detailList">
				<view class="contentTitle" >{{item.label}}</view>
				<view class="contentText" v-for="item2 in item.children">{{item2.label}}</view>
			</view>
		</view>
		
		
		<view class="button" @click="addReconds" v-if="showDeal">
			处理
		</view>
		
	</view>
	
		
	
</template>

<script>
	import commonUrl from "../../../commonUrl.js"
	export default {
		data() {
			return {
				taskId:"",
				detailList:[],
				title:"",
				showDeal:true
			}
		},
		methods: {
		
		
			//  点击派发 弹框出现
			addReconds(){
				console.log(66666)
				uni.navigateTo({
					url:"/pages/dealTask/addReconds/addReconds?id="+this.taskId+"&title="+this.title
				})
			},
			  
			getList(){
				let _this=this
				uni.request({
				  url:commonUrl+"buss/getTaskInfo",
				  method: "GET",
				  header: {'token':uni.getStorageSync("token")},//传在请求的header里
				  data:{
					  tid:_this.taskId
				  },
				  success(res) {
				  //请求成功的处理
					  _this.detailList=res.data.data
					  console.log("------------详情",_this.detailList)	
				  }
				})
			},
		},
		onLoad(option) {
			console.log('详情页接收的id',option)
			this.taskId=option.taskId
			this.title=option.title
			if(option.flag){
				this.showDeal=false
			}
			this.getList()
		}
	}
</script>

<style>
.button{
	width: 90%;
	margin-left: 5%;
	position: fixed;
	bottom: 6px;
	background-color:rgb(73, 118, 201) !important;
	height: 40px;
	line-height: 40px;
	color: white;
	border-radius: 10px;
	text-align: center;
	z-index: 50;
}
.content{
	font-size: 16px;
	margin-left: 20px;
	margin-bottom: 20px;
}
.contentTitle{
	font-weight: 600;
	margin-bottom: 5px;
}
.contentText{
	margin-bottom: 3px;
}
.title{
	font-weight: 600;
	font-size: 16px;
	margin-left: 20px;
	margin-right: 20px;
	margin-top: 10px;
	margin-bottom: 20px;
}
</style>
