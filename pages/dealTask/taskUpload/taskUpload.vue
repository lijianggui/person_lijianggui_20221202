	<template>
	<view class="pages">
		<view class="upload" v-for="item in list">
			<view class="uploadName"  @click="jumpDetail(item.tid,item.userId)">{{item.title}}</view>
		</view>
	</view> 
</template>

<script>
	import commonUrl from "../../../commonUrl.js" 
	export default {
		data() {
			return {
				list:[],
				userId:'',
				flag:""
			}
		},
		methods: {
			jumpDetail(tid,userId){
				console.log(1111)
				uni.navigateTo({
					url:"/pages/dealTask/complateCList/complateCList?tid="+tid+"&userId="+userId+"&flag="+this.flag
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
		onLoad(option){	
			console.log("000000000option",option)
			this.userId=option.userId
			if(option.flag){
				this.flag=option.flag
			}
			
			
			this.getList()
		},
		onShow() {
			
		}
	}
</script>

<style>
.upload{
	/* display: flex;
	justify-content: space-around; */
	margin-top: 10px;
	margin-bottom: 10px;
	color: blue;
	font-size: 16px;
}
.uploadName{
	width: 90%;
	margin-left: 5%;
	overflow: hidden;  /* 超出一行文字自动隐藏 */
	text-overflow: ellipsis;  /*文字隐藏后添加省略号*/
	white-space: nowrap;
	font-size: 16px;
}

</style>

