<template>
	<view>
		<view class="" v-if="uploadList.length>0">
			<view class="upload" v-for="item in uploadList">
				<view class="uploadName"  @click="jumpUplodDetail(item.id)">{{item.user}}</view>
				<view class="uploadItem"   @click="jumpUplodDetail(item.id)">
					{{item.date}}
				</view>
			</view>
			
		</view>
		
		<view class="noReconds" v-if="uploadList.length==0">
			 暂无数据！
		</view>
		
		<u-modal  @cancel="cancel"  @confirm="confirm" :show="show" :title="title" :content='content' showCancelButton=true></u-modal>
		
		<view class="button" @click="endTask" v-if="showEnd">
			结束
		</view>
	</view>
</template>

<script>
import commonUrl from "../../../commonUrl.js" 
	export default {
		data() {
			return {
				taskId:"",
				uploadList:[],
				show:false,
				title:"确定结束此任务吗？",
				content:"",
				showEnd:true
			}
		},
		methods: {
			cancel(){
				this.show=false
			},
			endTask(){
				this.show=true
			},
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
							uni.redirectTo({
								url:"/pages/issuedEndTask/issuedTask/issuedTask"
							})
						}
						}
				})
			},
			
			jumpUplodDetail(id){
				uni.navigateTo({
					url:"/pages/issuedEndTask/uploadDetail/uploadDetail?uploadId="+id
				})
			},
			getUploadList(){
				let _this=this
				uni.request({
					url:commonUrl+"buss/querySubmitTask",
					method:'POST',
					header:{'token':uni.getStorageSync("token")},
					data:{
						 date: "",
						 id: "",
						 items: {},
						otid: "",
						tid:this.taskId
					},
					success: (res) => {
						console.log("获取上传的记录",res)
						if(res.statusCode==200){
							if(res.data.data.length>0){
								
								_this.uploadList=res.data.data
								_this.uploadList.forEach(item=>{
									  var date = new Date(item.date)
									  const Y = date.getFullYear() + '-'
									  const M = date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) + '-' : date.getMonth() + 1 + '-'
									  const D = date.getDate() < 10 ? '0' + date.getDate() + ' ' : date.getDate() + ' '
									  const h = date.getHours() < 10 ? '0' + date.getHours() + ':' : date.getHours() + ':'
									  const m = date.getMinutes() < 10 ? '0' + date.getMinutes() + ':' : date.getMinutes() + ':'
									  const s = date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds()
									  item.date=Y + M + D +h + m +s
								})
							}else{
								_this.uploadList=[]
							}
					
						}
						}
				})
			}
		},
		onLoad(option) {
			console.log("option",option)
			this.taskId=option.id
			if(option.flag){
				this.showEnd=false
			}
		},
		onShow(){
			this.getUploadList()
		}
	}
</script>

<style>
.button{
	width: 90%;
	margin-left: 5%;
	position: fixed;
	bottom: 10px;
	background-color:rgb(73, 118, 201) !important;
	height: 40px;
	line-height: 40px;
	color: white;
	border-radius: 10px;
	text-align: center;
	z-index: 50;
	font-size: 18px;
}
.upload{
	display: flex;
	justify-content: space-around;
	margin-top: 10px;
	margin-bottom: 10px;
	color: blue;
	font-size: 18px;
}
.uploadName{
	width: 30%;
	margin-left: 3%;
	overflow: hidden;  /* 超出一行文字自动隐藏 */
	text-overflow: ellipsis;  /*文字隐藏后添加省略号*/
	white-space: nowrap;
	
}
.uploadItem{
	width: 64%;
	margin-right: 3%;
	overflow: hidden;  /* 超出一行文字自动隐藏 */
	text-overflow: ellipsis;  /*文字隐藏后添加省略号*/
	white-space: nowrap;
}
.noReconds{
	position: absolute;
	top: 30px;
	left: 50%;
	transform: translateX(-50%);
}
</style>
