<template>
	<view class="pages">
		<view class="taskItem" v-for="item in list">
			<view class="item"  @click="jumpDetail(item.tid,item.orgId)">{{item.title}}</view>
		</view>
	</view> 
</template>

<script>
	import commonUrl from "../../../commonUrl.js" 
	export default {
		data() {
			return {
				list:[],
			}
		},
		methods: {
			jumpDetail(tid,orgId){
				uni.navigateTo({
					url:"/pages/assigneTask/complateB/complateB?tid="+tid+"&orgId="+orgId
				})
			},
			getList(){
				let _this=this
				uni.request({
				  url:commonUrl+"task/getCompleted",
				  method: "POST",
				  header: {'token':uni.getStorageSync("token")},//传在请求的header里
				  data:{
					
				  },
				  success(res) {
				  //请求成功的处理
						console.log("------------",res)		
						_this.list=res.data.data
				  }
				})
			}
		
		},
		onLoad(){	
			console.log("000000000option")
			// this.roleId=option.role
			this.getList()
		}
	}
</script>

<style>
.pages{
		margin-top: 20px;
	}
.taskItem{
	overflow: hidden;
	margin-bottom: 10px;
}
.item{
	margin-left: 5%;
	width: 90%;
	overflow:hidden; //超出的文本隐藏
	text-overflow:ellipsis; //溢出用省略号显示
	white-space:nowrap; //溢出不换行
	color:blue;
	font-size: 16px;

}
</style>

