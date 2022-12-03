<template>
	<view style="page">
		
		<view class="title">
			<view class="">任务名称：<text>{{title}}</text></view>
		</view>
		
		<view class="content" v-for="item in detailList">
			<view class="contentTitle" >{{item.label}}</view>
			<view class="contentText" v-for="item2 in item.children">{{item2.label}}</view>
		</view>
		
		<view class="button" @click="addReconds">
			派发
		</view>
		
		<view class="diagbox">
			<u-modal :show="show" :title="modalTitle" :content='content' showCancelButton=true @confirm="confirm"  @cancel="cancel">
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
	import LyTree from '@/components/ly-tree/ly-tree.vue';
	import commonUrl from "../../../commonUrl.js"
	export default {
		components: {
				  LyTree
			   },
		data() {
			return {
				taskId:"",
				detailList:[],
				modalTitle:"选择派发科室",
				content:"",
				show:false,
				radioList:[],
				title:""
			}
		},
		methods: {
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
							uni.showToast({
								 title:"派发成功",
								icon:"success"
							})
						_this.show=false
						}
						}
				})
			},
			cancel(){
				this.show=false
			},
			//  点击派发 弹框出现
			addReconds(){
				this.show=true
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
			this.getList()
			this.getkeshi()
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




.button{
	width: 90%;
	margin-left: 5%;
	position: absolute;
	bottom: 30px;
	background-color:rgb(73, 118, 201) !important;
	height: 40px;
	line-height: 40px;
	color: white;
	border-radius: 10px;
	text-align: center;
}
.content{
	font-size: 12px;
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
	font-size: 12px;
	margin-left: 20px;
	margin-right: 20px;
	margin-top: 10px;
	margin-bottom: 20px;
}
</style>
