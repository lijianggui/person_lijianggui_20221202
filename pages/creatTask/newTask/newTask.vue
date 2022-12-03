<template>
	<view>
		<u-search  shape="square" searchIconSize=0  v-model="keyword" placeholder="请输入任务名称" label="任务名称 :" :showAction="showAction"></u-search>
		
		<view class="content" style="margin-bottom: 50px;">
				<ly-tree :tree-data="treeData" 
						show-checkbox
						node-key="id" 
						iconClass="cicleIcon"
						ref="tree"
						checkOnlyLeaf=true
						>
						</ly-tree>
				<button class="creatTask" type="primary" @click="generateTask">生成任务</button>

				
				<u-modal  @cancel="cancel"  @confirm="confirm" :show="modalControl" :title="title" :content='content' showCancelButton=true></u-modal>
		</view>
	
	</view>
	
</template>

<script>
	import LyTree from '@/components/ly-tree/ly-tree.vue';
	import zModal from '@/components/z-modal/z-modal.vue';
	import commonUrl from "../../../commonUrl.js"
	export default {
	   components: {
	             LyTree,
				 zModal
	          },
		data() {
			return {
				showAction:false,
				keyword:"",
				treeData: [],
				modalControl:false,
				checkTreeData:[],
				searchName:'',
				taskId:"",
				title:'确认生成任务吗？',
				content:''
			}
		},
		methods: {
			// 获取列表
			getTaskList(){
				let _this=this
				var header;
				header = {
				   'token':uni.getStorageSync("token")//读取cookie
				};
				uni.request({
					url:commonUrl+"item/findTree",
					method:'POST',
					header: header,
					data:{
						category: "",
						id: "",
						name:this.keyword,
						order: 0
					},
					success: (res) => {
						console.log(res)
						this.treeData=res.data.data
					}
				})
			},
			// 生成任务
			generateTask(){
				var str=""
				this.modalControl=true
				console.log('this.$refs.tree.getCheckedNodes()',this.$refs.tree.getCheckedNodes())
			},
			cancel(){
				this.modalControl=false
				console.log(111111111111)
			},
			//  点击确认生成触发的事件
			confirm(){
				let _this=this
				let selectList=this.$refs.tree.getCheckedNodes()
				let  dealSelectList=[]
				selectList.map(item=>{
					dealSelectList.push(item.id)
				})
				if(this.keyword==""){
					uni.showToast({
						title:"任务名称不能为空",
						icon:'error'
					})
				}else if(dealSelectList.length==0){
					uni.showToast({
						title:"请选择任务",
						icon:'error'
					})
				}else if(this.keyword!="" && dealSelectList.length>0){
						uni.request({
							url: commonUrl+'task/add',
							method:'POST',
							header:{'token':uni.getStorageSync("token")},
							data: {
								  info: "",
								  items:dealSelectList,
								  title:_this.keyword
							},
							success: (res) => {
								console.log(99999999,res)
								uni.setStorage({
									key: 'id',
									data:res.data.data.id ,
									success: function () {
										console.log('缓存成功');
									}
								});
								_this.modalControl=false
								uni.navigateTo({
									url:`/pages/creatTask/issueTask/issueTask`
								})
							},	
						});
				}
			
				
			},
			//  搜索事件
			// searchTreeEvent(){
			// 	this.getTaskList()
			// }

		},
		onLoad() {
			this.getTaskList()
		},
	}
</script>

<style>
	.u-search__content__input{
		font-size: 16px !important;
	}
	.u-search__content__label{
		font-size: 16px !important;
	}
	.ly-tree-node__label{
		font-size: 16px !important;
	}
	.u-search{
		width: 90% !important;
		margin-left: 5% !important;
		position: fixed !important;
		top: 0 !important;
		z-index: 50 !important;
	}

	.modal-title{
		color: black !important;
		padding-top: 20px !important;
	}
	.content{
		margin-left: 5%;
		margin-top: 30px;
	}
	.creatTask{
		width: 90%;
	/* 	margin-left: 5%; */
		position: fixed;
		bottom: 6px;
		background-color:rgb(73, 118, 201) !important;
		height: 40px;
		line-height: 40px;
		color: white;
		border-radius: 10px;
		text-align: center;
		z-index: 50;
		font-size: 16px;
	}
	.searchButton{
		width: 60px;
		background-color:rgb(73, 118, 201) ;
		height: 30px;
		z-index: 999;
		position: fixed;
		top: 8px;
		right: 10px;
		color: white;
		border-radius: 20px;
		line-height: 30px;
		text-align: center;
		font-size: 12px;
	}
</style>
