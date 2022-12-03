<template>
	<view>
		<view class="" v-if="uploadList.length>0">
			<view class="upload" v-for="item in uploadList">
				<view class="uploadName"  @click="jumpUplodDetail(item.id,item.tid,item.userId)">{{item.user}}</view>
				<view class="uploadItem"   @click="jumpUplodDetail(item.id,item.tid,item.userId)">
					{{item.date}}
				</view>
			</view>
			
		</view>
		
		<view class="noReconds" v-if="uploadList.length==0">
			 暂无数据！
		</view>
	</view>
</template>

<script>
import commonUrl from "../../../commonUrl.js" 
	export default {
		data() {
			return {
				uploadList:[],
				orgId:"",
				tid:"",
				roleId:"",
				flag:"C"
			}
		},
		methods: {
			// 获取角色id
			role(){
				let _this=this
				uni.request({
					url:commonUrl+ "user/getUserLoginInfo",
					method: "GET",
					header: {'token':uni.getStorageSync("token")},
					success(res) {
					  console.log("获取角色信息的接口",res)	
						_this.roleId=res.data.data.id
					}
				})
			},
			
			jumpUplodDetail(id,tid,userId){
				if(userId==this.roleId){
					this.flag="C"
				}else{
					this.flag=""
				}
				uni.navigateTo({
					url:"/pages/issuedEndTask/uploadDetail/uploadDetail?uploadId="+id+"&flag="+this.flag+"&tid="+tid	
				})
			},
			
			getUploadList(){
				let _this=this
				uni.request({
					url:commonUrl+"buss/querySubmitTask",
					method:'POST',
					header:{'token':uni.getStorageSync("token")},
					data:{
						oid:this.orgId,
						tid:this.tid
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
			this.orgId=option.orgId
			this.tid=option.tid
			
		},
		onShow(){
			this.getUploadList()
			this.role()
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
}
.upload{
	display: flex;
	justify-content: space-around;
	margin-top: 10px;
	margin-bottom: 10px;
	color: blue;
	
}
.uploadName{
	width: 30%;
	margin-left: 3%;
	overflow: hidden;  /* 超出一行文字自动隐藏 */
	text-overflow: ellipsis;  /*文字隐藏后添加省略号*/
	white-space: nowrap;
	font-size: 16px;
}
.uploadItem{
	width: 64%;
	margin-right: 3%;
	overflow: hidden;  /* 超出一行文字自动隐藏 */
	text-overflow: ellipsis;  /*文字隐藏后添加省略号*/
	white-space: nowrap;
	font-size: 16px;
}
.noReconds{
	position: absolute;
	top: 30px;
	left: 50%;
	transform: translateX(-50%);
}
</style>
