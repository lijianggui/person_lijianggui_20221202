<template>
	<view class="pages">
		<view class="" v-if="issTaskItem.length==0" style="text-align:center">
			暂无数据！
		</view>
		<view class=""  v-if="issTaskItem.length>0">
			<view class="taskItem" v-for="item in issTaskItem">
				<view class="item" @click="jumpSubmitRecond(item.tid,item.userId)">{{item.title}}</view>
				<button @click="jumpDetail(item.tid,item.title)">处理</button>
			</view>
		</view>
		
	</view> 
</template>

<script>
	import commonUrl from "../../../commonUrl.js" 
	export default {
		data() {
			return {
				issTaskItem:[],
				titleName:"",
				taskId:"",
				flag:"C"
			}
		},
		methods: {
		
			getList(){
				let _this=this
				uni.request({
				  url:commonUrl+"task/getProcessing",
				  method: "POST",
				  header: {'token':uni.getStorageSync("token")},//传在请求的header里
				  data:{
					  
				  },
				  success(res) {
					  console.log('3333333333',res)
				  //请求成功的处理
				  
					 _this.issTaskItem=res.data.data
				  
				  }
				})
			},
			jumpSubmitRecond(tid,userId){
				uni.navigateTo({
					url:"/pages/dealTask/complateCList/complateCList?tid="+tid+"&flag="+this.flag
				})
			},
			jumpDetail(id,title){
				console.log(1111)
				uni.navigateTo({
					url:"/pages/dealTask/notDealDetail/notDealDetail?taskId="+id+"&title="+title
				})
			},
		},
		onLoad() {
			this.getList()
		}
	}
</script>

<style>
	.ly-tree {
	    /* position: relative; */
	    cursor: default;
	    background: #FFF;
	    color: #606266;
	    padding: 30rpx;
	    overflow: auto !important;
	    position: absolute !important;
	    top: 50px !important;
	    height: 110px !important;
		left: 80px !important;
		}
.u-modal__title{
	color:rgb(73, 118, 201) !important;
	font-size: 16px;
	text-align: center;
	left: 120px !important;
	z-index: 50 !important;
	/* border-bottom: 1px solid #d6d7d9 !important; */
/* 	padding-top: 15px !important;
	padding-bottom: 10px !important; */
}
.u-modal{
	height: 250px !important;
}

.u-modal__content{
	height: 122px !important;
}


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
	font-size: 16px;

}
 button{
	 width: 60px;
	 height:20px;
	 border-radius: 10px;
	 background-color: cornflowerblue;
	 margin-left: 20px;
	 font-size: 16px;
	 color:white;
	 line-height:20px;
	float: right;
	
 }
</style>
