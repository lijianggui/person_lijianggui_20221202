<template>
	<view>
		<view class="content" style="margin-bottom: 50px;">
				<ly-tree :tree-data="treeData" 
						show-checkbox
						node-key="id" 
						iconClass="cicleIcon"
						ref="tree"	
						checkStrictly=true
						>
						</ly-tree>
		</view>
		<button type="primary" @click="issueTask"> 下发任务</button>
		<z-modal :show=modalControl :btnGroup="btnGroup"    titleColor="blue"   :contentType="1"  :contentText="contentText" :titleText="titleText" @cancle="cancle" @sure="sure"  class="modalbox" >
		</z-modal>
	</view>
	
</template>

<script>
	import LyTree from '@/components/ly-tree/ly-tree.vue';
	import commonUrl from "../../../commonUrl.js";
	import zModal from '@/components/z-modal/z-modal.vue';
	export default {
	   components: {
	             LyTree,
	          },
		data() {
			return {
				treeData: [],
				modalControl:false,
				btnGroup: [{
							text: '取消',
							color: '#FFFFFF',
							bgColor: '#999999',
							width: '100px',
							height: '30px',
							shape: 'fillet',
							eventName: 'cancle'
						}, 
						{
							text: '确认',
							color: '#FFFFFF',
							bgColor: '#007AFF',
							width: '100px',
							height: '30px',
							shape: 'fillet',
							eventName: 'sure'
						}],
						titleText:'确认下发任务吗？',
						contentText:''
			}
		},
		methods: {
			issueTask(){
				this.modalControl=true
			},
			cancle(){
				this.modalControl=false
			},
			sure(){
				console.log('this.$refs.tree.getCheckedNodes()',this.$refs.tree.getCheckedNodes())
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
						 oids:dealSelectList ,
						 tid:uni.getStorageSync("id")
					},
					success: (res) => {
						console.log(res)
						if(res.data.msg=="成功" && res.statusCode==200){
							uni.reLaunch({
								url:"/pages/homePage/homePage/homePage?status=1"
							})
						}
						// this.treeData=res.data.data
					}
				})
			},
			// 获取列表
			getTaskList(){
				let _this=this
				var header;
				header = {
				   'token':uni.getStorageSync("token")//读取cookie
				};
				uni.request({
					url:commonUrl+"org/findTree",
					method:'POST',
					header: header,
					data:{
						  id: "",
						  name: "",
						  order: 0,
						  pid: "",
						  type: ""
					},
					success: (res) => {
						console.log(res)
						this.treeData=res.data.data
					}
				})
			},
		},
		onLoad() {
			this.getTaskList();
			
		   console.log("111111111111111111",uni.getStorageSync("id")) 
		},
	}
</script>

<style>
.modal-btn{
	font-size: 16px !important;
}
.modal-title{
	font-size: 16px !important;
}
.ly-tree-node__label{
	font-size: 16px !important;
}
.modal-title{
	color: black !important;
	padding-top: 20px !important;
}
button{
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
</style>
