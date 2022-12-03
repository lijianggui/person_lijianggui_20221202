<template>
	<view class="pages">
		<view class=""  v-if="!showList"  style="text-align:center; font-size: 18px;" >
			暂无数据！
		</view>
		
		<view class="" v-if="showList">
				<view class="taskItem" v-for="item in list"  >
					<view class="item" @click="issueDetail(item.id)">{{item.title}}</view>
					<button @click="issueTask(item.id)">结束</button>
				</view>
		</view>
	
		
		<u-modal  @cancel="cancel"  @confirm="confirm" :show="show" :title="title" :content='content' showCancelButton=true></u-modal>
	</view> 
</template>

<script>
	import commonUrl from "../../../commonUrl.js" 
	export default {
		data() {
			return {
				show:false,
				roleId:"",
				pageNum:1,
				pageSize:17,
				list:[],
				modalControl:false,
				title:'确定结束此任务吗?',
				contentText:"",
				taskId:"",
				showList:false
			}
		},
		onReachBottom(){//页面上拉触底事件的处理函数
				this.pageNum=this.pageNum+1
				this.getList()
			},
		methods: {
			
			confirm(){
				uni.request({
					url:commonUrl+"/buss/completeTask/"+this.taskId,
					method:'POST',
					header:{'token':uni.getStorageSync("token")},
					data:{
					},
					success: (res) => {
						console.log("结束任务的接口",res)
						if(res.statusCode==200){
							this.show=false
							this.getList2()
							uni.showToast({
								title:"操作成功",
								icon:"success"
							})
						}
						}
				})
			},
			
			cancel(){
				this.show=false
			},
			issueDetail(id){
				uni.navigateTo({
					url:"/pages/issuedEndTask/showUpload/showUpload?id="+id
				})
			},
			issueTask(id){
				this.show=true
				this.taskId=id
			},
			cancle(){
				this.modalControl=false
			},
			sure(){
				
			},
			getList2(){
				let _this=this
				uni.request({
				  url:`${commonUrl}task/page?pageSize=${this.pageSize}&pageNum=${this.pageNum}`,
				  method: "POST",
				  header: {'token':uni.getStorageSync("token")},//传在请求的header里
				  data:{
					  status:1,
					  create:this.roleId,
				  },
				  success(res) {
				  //请求成功的处理
						console.log("------------",res)	
						_this.list=res.data.data.data;
						if(_this.list.length>0){
							_this.showList=true
						}else{
							_this.showList=false
						}
				  }
				})
			},
			getList(){
				let _this=this
				uni.request({
				  url:`${commonUrl}task/page?pageSize=${this.pageSize}&pageNum=${this.pageNum}`,
				  method: "POST",
				  header: {'token':uni.getStorageSync("token")},//传在请求的header里
				  data:{
					  status:1,
					  create:this.roleId,
				  },
				  success(res) {
				  //请求成功的处理
						console.log("------------",res)		
						_this.list=[..._this.list,...res.data.data.data];
						if(_this.list.length>0){
							_this.showList=true
						}else{
							_this.showList=false
						}
				  }
				})
			}
		
		},
		onLoad(option){	
			console.log(option,"000000000option")
			this.roleId=option.role
			
		},
		onShow() {
			this.getList2()
		}
	}
</script>

<style>
.u-modal__title{
	color: rgb(73, 118, 201) !important;
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
