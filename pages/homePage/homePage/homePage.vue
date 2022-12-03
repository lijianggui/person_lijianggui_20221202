<template>
	<view class="content">
		<view class="baseInfo" >
			
			<p v-if="issueSuccess"  class="issueSuccess">任务下发成功</p>
			
		
			<view class="personInfo"  v-if="!issueSuccess">
					<h2 class="title">{{rolenName}}</h2>
			</view>
			
			
			<view class="taskContent" style="font-size: 18px;">
				<view class="showA show" v-if="showA">
					<p   @click="issuedTask">{{issueNum}}</p>
					<p  @click="endTask">{{endNum}}</p>
				</view>
				
				<view class="showB show" v-if="showB">
					<p  @click="jumpIssueTask">{{processingNum}}</p>
					<p  @click="completedB">{{completed}}</p>
				</view>
				
				<view class="showC show" v-if="showC">
					<p @click="notDeal"  >{{waitdeal}}</p>
					<p @click="completedC"  >{{completed}}</p>
				</view>
				
				<view class="showAll show" v-if="showAll">
					<p @click="issuedTask" >{{issueNum}}</p>
					<p @click="jumpIssueTask"  >{{processingNum}}</p>
					<p @click="endTask"  >{{endNum}}</p>
				</view>
				
				<view class="uploadFile" >
					<view v-if="showA || showAll"  class="" @click="openDoc"  style="margin-bottom: 10px;">上传学习文件</view>
					<view   class="" @click="getUploadFile">已上传文件</view>
				</view>
					
				
			</view>
				
			<button class="taskButton"  v-if="showA|| showAll"   @click="issueTask">新建下发任务</button>
		</view>
		
	</view>
</template>

<script>
	import commonUrl from "../../../commonUrl.js" 
	export default {
		 components: {
		        // lauwenSelect
		},
		data() {
			return {		
				nickName:"",
				imgUrl:"",
				showA:false,
				showB:false,
				showC:false,
				issueSuccess:false,
				roleId:"",
				pageSize:1000,
				pageNum:1,
				issueNum:"",
				endNum:'',
				assignNum:'',
				processingNum:"",
				completed:"",
				processingNumber:"",
				waitdeal:"",
				rolenName:"",
				uploadFilename:"",
				submitC:"",
				type:"",
				oid:"",
				showAll:false
				
			}
		},
		 onLoad(option) {
			console.log('option',option)
			if(option.status==1){
				this.issueSuccess=true
			}
			this.role()
			
		},
		onShow(){
			this.getNum()
			// this.getCsubmit()
			
			const type = uni.getSystemInfoSync().uniPlatform
			console.log("type",type)
			this.type=type
		},
		methods: {	
			getUploadFile(){
				uni.navigateTo({
					url:"/pages/issuedEndTask/uploadFile/uploadFile"
				})
			},
			openDoc(){
				let _this=this
					uni.chooseMessageFile({
						count: 100, //最多可以选择的文件个数，可以 0～100
						type: 'file', //所选的文件的类型，具体看官方文档
						extension:['.doc','.xlsx','.docx'],
						success (res) {
							console.log(9999999999,res)
							const tempFilePaths = res.tempFiles[0].path
							let filename = res.tempFiles[0].name; 
							let fileSize=res.tempFiles[0].size
							if(fileSize>10485760){
								uni.showToast({
									title:"文件过大",
									icon:"error"
								})
							}else if(filename.indexOf(".doc")==-1 && filename.indexOf(".docx")==-1 && filename.indexOf(".xlsx")==-1 && filename.indexOf(".pdf")==-1){
								uni.showToast({
									title:"文件格式不正确",
									icon:"error"
								})
							}else{
									uni.uploadFile({
										header: {'token':uni.getStorageSync("token")},
										url:commonUrl+'file/upload2?name='+encodeURIComponent(filename), //上传的路径
										filePath: tempFilePaths, //刚刚在data保存的文件路径
										name: 'file',   //后台获取的凭据
										formData:{ //如果是需要带参数，请在formData里面添加，不需要就去掉这个就可以的
											// "userId": 1,
											// "name": filename
										},
										success(res) { 
											if(res.statusCode==200){
												uni.showToast({
													title:"上传成功",
													icon:"success"
												})
											}
											console.log(res)
										}
									})
							}
						
							
						}
					})
			},
			//  c角色的已完成
			completedC(){
				uni.navigateTo({
					url:"/pages/dealTask/taskUpload/taskUpload?userId="+this.roleId+"&flag="+"C"
				})
			},
			//  b角色的已完成
			completedB(){
				 console.log(2222222)
				uni.navigateTo({
					url:"/pages/assigneTask/complatedTask/complatedTask"
				})
			},
			// 得到任务数量
			   getNum(){
					let _this=this
					uni.request({
					  url:commonUrl+"buss/getTaskNumInfo",
					  method: "GET",
					  header: {'token':uni.getStorageSync("token")},//传在请求的header里
					  data:{
						  
					  },
					  success(res) {
					  //请求成功的处理
							console.log("------------数量",res)
							_this.issueNum="已下发任务"+res.data.data.issued+"个"
							_this.endNum="已结束任务"+res.data.data.closed+"个"
							_this.assignNum="待分配任务"+res.data.data.closed+"个"
							_this.processingNum="进行中任务"+res.data.data.processing+"个"
							_this.processingNumber=res.data.data.processing
							_this.completed="已完成任务"+res.data.data.completed+"个"
							_this.waitdeal="待处理任务"+res.data.data.processing+"个"
							
							
					  }
					})
			},
			
			// 已下发任务  跳转到已下发任务页面
			issuedTask(){
				uni.navigateTo({
					url:"/pages/issuedEndTask/issuedTask/issuedTask?role="+this.roleId
				})
			},
			//  A已结束任务
			endTask(){
				uni.navigateTo({
					url:"/pages/issuedEndTask/endTask/endTask?role="+this.roleId+"&flag=end"
				})
			},
			
			
			role(){
				let _this=this
				uni.request({
					url:commonUrl+ "user/getUserLoginInfo",
					method: "GET",
					header: {'token':uni.getStorageSync("token")},
					success(res) {
					  console.log("获取角色信息的接口",res)	
						let roleList=res.data.data.roles
						_this.roleId=res.data.data.id
						_this.rolenName=res.data.data.name+" "+"欢迎您！"
						_this.oid=res.data.data.oid
						
						if(roleList.indexOf("A")>-1){
							if(roleList.indexOf("B")>-1 && roleList.indexOf("C")>-1){
								_this.showAll=true
							}else if(roleList.indexOf("A")>-1 && !(roleList.indexOf("B")>-1 && roleList.indexOf("C")>-1)){
								_this.showA=true
							}
						}else if(roleList.indexOf("B")>-1 ){
							if(roleList.indexOf("B")>-1 &&  roleList.indexOf("C")>-1 ){
								_this.showB=true
							}else if(roleList.indexOf("B")>-1 &&  !roleList.indexOf("C")>-1){
								_this.showB=true
							}
						}else if(roleList.indexOf("C")>-1){
							_this.showC=true
						}
					  _this.getNum()
					  // _this.getCsubmit()
					}
				})
			},
			

			notDeal(){
				uni.navigateTo({
					url:'/pages/dealTask/notDeal/notDeal'
				})
			},
			issueTask(){
				uni.navigateTo({
					url:'/pages/creatTask/newTask/newTask'
				});
			},
			jumpIssueTask(){
				uni.navigateTo({
					url:'/pages/assigneTask/assignItem/assignItem?oid='+this.oid
				});
			},
			
		}
	}
</script>

<style>
	.uploadFile{
		position: absolute;
		left: 50%;
		bottom: 30%;
		transform: translateX(-50%);
		color: blue;
	}
	.personInfo{
		/* margin-top: 40px; */
		position: absolute;
		left: 50%;
		bottom: 70%;
		transform: translateX(-50%);
		width: 90%;
		font-size: 18px;
	}

	.show{
		position: absolute;
		left: 50%;
		bottom: 50%;
		transform: translateX(-50%);
		text-decoration: underline grey;
	}

	.issueSuccess{
		font-size: 18px;
		color: blue;
		position: absolute;
		top:60px;
		left: 50%;
		transform: translateX(-50%);
	}
	.baseInfo{
		text-align: center;
	}
	.lauwen-select-input{
		border: none !important;
		width: 30% !important;
		margin-left: 35%;
		margin-bottom: 10px;
	}
	
		p{
			margin-bottom: 10px;
			
		}
		.content{
			text-align: center;
		}
	
		.taskButton{
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
