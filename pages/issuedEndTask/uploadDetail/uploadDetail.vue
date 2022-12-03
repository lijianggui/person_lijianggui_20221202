<template>
	<view >
		<view class="" style="margin-bottom: 60px;">
			<view class="content" v-for="item in detailList"  style="font-size: 14px;">
				<view class="contentTitle" >{{item.label}}</view>
				<view class="contentText" v-for="item2 in item.children"> 
					<view  class="labelView" style="width: 40%;"> <label class="inputLabel">{{item2.label}}</label></view>
					<text style="margin-left: 10px;">:</text> 
					<view class="input"  v-if="item2.type!='url'" >{{item2.value}}</view>
					<view class="input" v-if="item2.type=='url'">
						<image @click="clickImg(item3.url.url)" v-for="item3 in item2.value" style="width: 50px;height: 50px;margin-right: 10px;"  :src="'https://tas.hemotors.com.cn/img/'+item3.url.url" mode=""></image>
					</view>
				</view>
			</view>
		</view>
		
		
		<view class="mask" v-show="showMask" @click="closeMask">
			<image  class="maskImg" :src="url2" mode=""></image>
		</view>

	<view class="buttonGroup" v-if="flag">
		<button class="button1"  @click="reback">撤销</button> 
		<button class="button2"  @click="xiugai">修改</button> 
	</view>
		
	</view>
</template>

<script>
	import commonUrl from "../../../commonUrl.js" 
	
	export default {
		data() {
			return {
				showMask:false,
				title:"",
				detailList:[],
				inputList:[{}],
				uploadId:"",   //从提交记录列表中传递过来的id
				url2:"",
				flag:"",
				tid:""
			}
		},
		methods: {
		xiugai(){
			uni.navigateTo({
				url:"/pages/dealTask/editReconds/editReconds?id="+this.uploadId+"&tid="+this.tid
			})
		
		},
		reback(){
			let _this=this
			uni.showModal({
				title: '确认撤销吗?',
				content: '',
				success: function (res) {
					if (res.confirm) {
						uni.request({
						  url:commonUrl+`submitTask/del/${_this.uploadId}`,
						  method: "POST",
						  header: {'token':uni.getStorageSync("token")},//传在请求的header里
						  data:{
						  },
						  success(res) {
						  //请求成功的处理
							 uni.showToast({
							 	title:"撤销成功",
								icon:"success"
							 })
							 
							 setTimeout(function() {
								uni.navigateBack({
										delta: 1
									}); 
							 }, 1000);
							
						  }
						})
					} else if (res.cancel) {
						
					}
				}
			});
		},
		closeMask(){
			this.showMask=false
		},
		clickImg(url){
			this.showMask=true
			this.url2="https://tas.hemotors.com.cn/img/"+url
		},
			
			getList(){
				let _this=this
				uni.request({
				  url:commonUrl+"submitTask/"+_this.uploadId,
				  method: "GET",
				  header: {'token':uni.getStorageSync("token")},//传在请求的header里
				  data:{
				  },
				  success(res) {
				  //请求成功的处理
					  _this.detailList=res.data.data
					  console.log("------------提交任务详情",res)	
				  }
				})
			},
		},
		onLoad(option){
			console.log("提交详情5555option",option)
			this.uploadId=option.uploadId
			this.tid=option.tid
			if(option.flag=="C"){
				this.flag=true
			}
		},
		onShow() {
			this.getList()
		}
		
	}
</script>

<style>
.buttonGroup{
	position: fixed;
	bottom:10px;
	left: 0;
	font-size: 14px;
	display: flex;
	justify-content: flex-start;
	width: 100%;
}
.button1{
	font-size: 14px;
	 background-color:rgb(73, 118, 201) ;
	 color: white;
	 width: 100px;
	 text-align: center;
	
}
.button2{
	font-size: 14px;
	background-color:rgb(73, 118, 201) ;
	 color: white;
	 width: 100px;
	 text-align: center;
	
}
.maskImg{
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translateX(-50%) translateY(-50%);
	
}
/* uni-page-body,html,body{  
        height: 100%;  
    } */
.mask{
	position: absolute;
	width: 100%;
	height: 100%;
	background-color: rgba(0, 0, 0, 0.6);
	top:0;
	left: 0;
	
}

.inputLabel{
	vertical-align:midde;
	width: 100%;
	  
}
.input{
	/* margin-bottom: 6px; */
	width: 60%;
	display: inline-block;
	margin-left: 10px;
	/* margin-bottom: -2px; */
}
.labelView{
	width: 40%;
	overflow: hidden;  /* 超出一行文字自动隐藏 */
	text-overflow: ellipsis;  /*文字隐藏后添加省略号*/
	white-space: nowrap; 
	font-size: 16px;
}

.title{
	font-weight: 600;
	font-size: 16px;
	margin-right: 20px;
	margin-top: 10px;
	margin-bottom: 20px;
	width: 90%;
	margin-left: 5%;
	
}
.content{
	font-size: 16px;
	/* margin-left: 20px; */
	/* margin-bottom: 45px; */
	width: 90%;
	margin-left: 5%;
	margin-top: 10px;
}
.contentTitle{
	font-weight: 600;
	margin-bottom: 10px;
	font-size: 16px;
}
.contentText{
	display: flex;
	margin-bottom: 6px;
	font-size: 16px;
}
</style>
