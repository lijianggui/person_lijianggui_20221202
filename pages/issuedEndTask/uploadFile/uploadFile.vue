<template>
	<view class="pages">
		<view class="" style="text-align:center"  v-if="list.length==0">
			暂无数据！
		</view>
		
		<view class="" v-if="list.length>0">
			<view class="taskItem" v-for="item in list">
				<view class="item1"  @click="downLoad(item.url)">{{item.name}}</view>
				<view class="item2">{{item.date}}</view>
				<view class="item3">{{item.user}}</view>
			</view>
		</view>
	</view>
</template>

<script>
	import commonUrl from "../../../commonUrl.js" 
	export default {
		data() {
			return {
				list:[]
			}
		},
		methods: {
			downLoad(url){
				let newUrl="https://tas.hemotors.com.cn/img/"+url
				console.log('newUrl',newUrl)
				uni.downloadFile({
							url:newUrl,
							success: (res) => {
								if (res.statusCode === 200) {
									uni.saveFile({
										tempFilePath:res.tempFilePath,
										success(res2) {
											console.log('res2',res2)
												uni.openDocument({
														filePath:res2.savedFilePath,
														showMenu: true,
														success: function(res) {
														  
														}
													  })
										}
									})
								}
							}
						});
			},
			getList(){
				let _this=this
				uni.request({
				  url:commonUrl+"file/findAll",
				  method: "POST",
				  header: {'token':uni.getStorageSync("token")},//传在请求的header里
				  data:{
					  type:1
				  },
				  success(res) {
				  //请求成功的处理
					  console.log("------------",res)	
					  _this.list=res.data.data
					  _this.list.forEach(item=>{
					  	  var date = new Date(item.date)
					  	  const Y = date.getFullYear() + '-'
					  	  const M = date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) + '-' : date.getMonth() + 1 + '-'
					  	  const D = date.getDate() < 10 ? '0' + date.getDate() + ' ' : date.getDate() + ' '
					  	 
					  	  item.date=Y + M + D 
					  })
				  }
				})
			},
		},
		onLoad() {
			
		},
		onShow() {
			this.getList()
		}
	}
</script>

<style>
.pages{
		margin-top: 20px;
	}
.taskItem{
	margin-bottom: 10px;
}
.taskItem{
	display: flex;
	justify-content: space-between;
}
.item1{
	/* float: left; */
	margin-bottom: 10px;
	margin-left: 3%;
	width: 40%;
	overflow:hidden; //超出的文本隐藏
	text-overflow:ellipsis; //溢出用省略号显示
	white-space:nowrap; //溢出不换行
	color:blue;
	font-size: 14px;
}
.item2{
	width: 30%;
	overflow:hidden; //超出的文本隐藏
	text-overflow:ellipsis; //溢出用省略号显示
	white-space:nowrap; //溢出不换行
	font-size: 14px;
	margin-left: 10px;
}
.item3{
	width: 24%;
	overflow:hidden; //超出的文本隐藏
	text-overflow:ellipsis; //溢出用省略号显示
	white-space:nowrap; //溢出不换行
	font-size: 14px;
	margin-right: 3%;
}

</style>
