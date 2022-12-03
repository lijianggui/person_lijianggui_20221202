<template>
	<view class="pages">
		<view class="taskItem" v-for="item in list">
			<view class="item"  @click="jumpTaskDetail(item.id)">{{item.id}}</view>
		</view>
	</view> 
</template>

<script>
	import commonUrl from "../../../commonUrl.js" 
	export default {
		data() {
			return {
				list:[],
				taskId:""
			}
		},
		methods: {
			jumpTaskDetail(id){
				uni.navigateTo({
					url:''
				})
			},
			getList(){
				let _this=this
				uni.request({
				  url:commonUrl+"buss/querySubmitTask",
				  method: "POST",
				  header: {'token':uni.getStorageSync("token")},//传在请求的header里
				  data:{
					 tid:_this.taskId
				  },
				  success(res) {
				  //请求成功的处理
						console.log("------------",res)		
						_this.list=res.data.data.data
				  }
				})
			}
		
		},
		onLoad(option){	
			console.log(option,"000000000option")
			this.taskId=option.taskId
			this.getList()
		}
	}
</script>

<style>
.pages{
		margin-top: 20px;
	}
.taskItem{
	margin-left: 7%;
	overflow: hidden;
	width:85%;
	margin-bottom: 10px;
}
.item{
	float: left;
	width: 70%;
	overflow:hidden; //超出的文本隐藏
	text-overflow:ellipsis; //溢出用省略号显示
	white-space:nowrap; //溢出不换行
	color:blue;
	font-size: 14px;

}
 button{
	 width: 60px;
	 height:20px;
	 border-radius: 10px;
	 background-color: cornflowerblue;
	 margin-left: 20px;
	 font-size: 12px;
	 color:white;
	 line-height:20px;
	float: right;
	
 }
</style>

