<template>
	<view class="pages">
		<view class="" style="text-align:center"  v-if="list.length==0">
			暂无数据！
		</view>
		
		<view class="" v-if="list.length>0">
			<view class="taskItem" v-for="item in list">
				<view class="item"  @click="jumpTaskDetail(item.id)">{{item.title}}</view>
			</view>
		</view>
		
	</view> 
</template>

<script>
	import commonUrl from "../../../commonUrl.js" 
	export default {
		data() {
			return {
				roleId:"",
				list:[],
				flag:""
			}
		},
		methods: {
			jumpTaskDetail(id){
				console.log(9999999999999)
				uni.navigateTo({
					url:'/pages/issuedEndTask/showUpload/showUpload?id='+id+'&flag=end'
				})
			},
			getList(){
				let _this=this
				uni.request({
				  url:commonUrl+"task/getClosed",
				  method: "POST",
				  header: {'token':uni.getStorageSync("token")},//传在请求的header里
				  data:{
					  // status:0,
					  // create:this.roleId,
				  },
				  success(res) {
				  //请求成功的处理
						console.log("------------",res)		
						_this.list=res.data.data
					
				  }
				})
			}
		
		},
		onLoad(option){	
			console.log(option,"000000000option")
			this.roleId=option.role
			this.flag=option.flag
			this.getList()
		}
	}
</script>

<style>
.pages{
		margin-top: 20px;
	}
.taskItem{
	margin-bottom: 10px;
}
.item{
	/* float: left; */
	margin-bottom: 10px;
	margin-left: 5%;
	width: 90%;
	overflow:hidden; //超出的文本隐藏
	text-overflow:ellipsis; //溢出用省略号显示
	white-space:nowrap; //溢出不换行
	color:blue;
	font-size: 14px;

}

</style>
