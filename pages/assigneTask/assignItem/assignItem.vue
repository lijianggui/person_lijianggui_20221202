<template>
	<view class="pages">
			<view class="" v-if="issTaskItem.length==0" style="text-align:center">
				暂无数据！
			</view>

			<view class="" v-if="issTaskItem.length>0">
					<view class="taskItem" v-for="item in issTaskItem">
						<view class="item" @click="jumpDetail(item.id)">{{item.title}}</view>
						
						<button @click="dealTask(item.id,item.title)"  v-if="showIssue">处理</button>
						<button @click="issueTask(item.title,item.id)">派发</button>
					</view>
			</view>
		

			<view class="diagbox">
				<u-modal :show="show" :title="title" :content='content' showCancelButton=true @confirm="confirm"  @cancel="cancel">
					<view class="slot-content">
						<!-- <view class="">{{titleName}}</view> -->
						<ly-tree :tree-data="radioList"
								show-checkbox
								node-key="id" 
								iconClass="cicleIcon"
								ref="tree"
								checkStrictly=true
								defaultExpandAll=true
						       >
						</ly-tree>
					</view>
				</u-modal>
			</view>
	</view> 
</template>

<script>
	import commonUrl from "../../../commonUrl.js" 
	import LyTree from '@/components/ly-tree/ly-tree.vue';
	export default {
		components: {
				  LyTree
			   },
		data() {
			return {
				// itemShow:false,
				issTaskItem:[],
				 show: false,
				title:'选择派发科室',
				content:"",
				checkboxValue1:"",
				radioList:[],
				titleName:"",
				taskId:"",
				showIssue:false,
				oid:""
			}
		},
		methods: {
			dealTask(id,title){
				console.log(1111)
				uni.navigateTo({
					url:"/pages/dealTask/notDealDetail/notDealDetail?taskId="+id+"&title="+title
				})
			},
			
			// 获取角色信息
			getRole(){
				let _this=this
				uni.request({
					url:commonUrl+ "user/getUserLoginInfo",
					method: "GET",
					header: {'token':uni.getStorageSync("token")},
					success(res) {
						let roleList=res.data.data.roles
					  console.log("获取角色信息的接口",res)	
						 if(roleList.indexOf("B")>-1 && roleList.indexOf("C")>-1 || roleList.indexOf("A")>-1 && roleList.indexOf("B")>-1 && roleList.indexOf("C")>-1){
							_this.showIssue=true
						}
					}
				})
			},
			
			
			confirm(){
				let _this=this
				console.log('获取选择的科室',this.$refs.tree.getCheckedNodes())
				let selectList=this.$refs.tree.getCheckedNodes()
				let  dealSelectList=[]
				selectList.map(item=>{
					dealSelectList.push(item.id)
				})
				uni.request({
					url:commonUrl+"buss/assignTask2Org",
					method:'POST',
					header:{'token':uni.getStorageSync("token")},
					data:{
						 info: "",
						 oids:dealSelectList,
						 tid:_this.taskId
					},
					success: (res) => {
						console.log("分配任务成功",res)
						if(res.data.msg=="成功" && res.statusCode==200){
						    this.show=false
							uni.showToast({
								 title:"派发成功",
								icon:"success"
							})
							this.getList()
						}
						}
				})
				
				
			},
			cancel(){
				this.show=false
			},
			getList(){
				let _this=this
				uni.request({
				  url:commonUrl+"buss/getMyAssignTask?status=1",
				  method: "GET",
				  header: {'token':uni.getStorageSync("token")},//传在请求的header里
				  data:{
					  
				  },
				  success(res) {
				  //请求成功的处理
				  if(res.data.data){
					 _this.issTaskItem=res.data.data
				  }else{
					  _this.issTaskItem=[]
				  }
				
						console.log("------------待分配任务列表",res)	
				  }
				})
			},
			
			// jumpDetail(id,title){
			// 	console.log(1111)
			// 	uni.navigateTo({
			// 		url:"/pages/assigneTask/assignDetail/assignDetail?taskId="+id+"&title="+title
			// 	})
			// },
			jumpDetail(tid){
				console.log(1111,tid)
				uni.navigateTo({
					url:"/pages/assigneTask/complateB/complateB?orgId="+this.oid+"&tid="+tid
				})
			},
			
			//  获取所有的科室
			getkeshi(){
				let _this=this
				uni.request({
					url:commonUrl+"org/findTree",
					method:'POST',
					header:{'token':uni.getStorageSync("token")},
					data:{
						  id: "",
						  name: "",
						  order: 0,
						  pid: "",
						  type: ""
					},
					success: (res) => {
						_this.radioList=res.data.data	
						console.log('获取的所有科室',res)
						
					}
				})
			},
			checkboxChange(){
				
			},
			issueTask(title,id){
				this.content=""
				this.show=true
				this.titleName=title
				this.taskId=id
			}
		},
		onLoad(option) {
			this.getkeshi()
			
			console.log(option)
			this.oid=option.oid
			this.getRole()
		},
		onShow(){
			this.getList()
		}
		
		
	}
</script>

<style>
	.ly-tree-node__label{
		font-size: 16px !important;
	}
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
		left: 20px !important;
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
	font-size: 16px;
}
.item{
	float: left;
	width: 50%;
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
	 margin-left: 5px;
	 font-size: 16px;
	 color:white;
	 line-height:20px;
	float: right;
	
 }
</style>
