<template>
	<view class="content">

		<view class="welcom">欢迎您！</view>
		<view class="fanwei">此小程序为交通局的员工使用,如要使用 请先登录</view>
		
		<!-- <button open-type="getPhoneNumber" @getphonenumber="getPhoneNumber">授权手机号</button> -->
		
		<button open-type="getPhoneNumber" @getphonenumber="getPhoneNumber">点击登录</button>
	</view>
</template>

<script>
	import commonUrl from "../../commonUrl.js"
	import  WXBizDataCrypt from '../../WXBizDataCrypt.js'
	export default {
		 components: {
		        // lauwenSelect
		},
		data() {
			return {
				isShow:true,
				openId:"",
				unionid:"",
				session_key:""
				
			}
		},
		onLoad() {
			// this.isEmpower()
			
			this.getuserNew()
			
			
			
		},
		methods: {
			//  调式
			// getPhoneNumber(){
			// 	uni.redirectTo({
			// 		url:"/pages/homePage/homePage/homePage"
			// 	})
			// },
			
				
			getuserNew(){
				let _this=this
					uni.login({
					  provider: 'weixin',
					  success:res=>{
							console.log('res.code',res.code);  
							 uni.request({  
								url: commonUrl+'login/loginByCode?code='+res.code,  
									method:'POST',  
									data: {  
										                         //你的小程序的APPID  
									},  
									success: (res) => {  
										 console.log("11111111111",res)
										 let msg=res.data.data
										 _this.openId=msg.openId
										 _this.session_key=msg.session_key
									}  
								}); 	  
					  }
					});
			},
		getPhoneNumber(e){
			   console.log('允许登录的code',e)
			   uni.request({
					url: commonUrl+'login/getPhoneNumber?code='+e.detail.code,  
						method:'GET',  
						data: {  
						},  
						success: (res) => {  
							 console.log("2222",res)
							 if(res.statusCode==200 && res.data.data){
									uni.request({
											url:commonUrl+'login/loginByPhone', //仅为示例，并非真实接口地址。
											method:"GET",
											data: {
												phone:res.data.data
											},
											success: (res) => {
												console.log("333333",res)
												if(res.data.data){
													uni.setStorageSync("token", res.data.data.token)
												}
												
												if(res.statusCode==200 && res.data.msg=="成功"){
													uni.redirectTo({
														url:"/pages/homePage/homePage/homePage"
													})
												}else{
													uni.showToast({
														title:"账号未注册",
														icon:"error"
													})
												}
											},
									})
							 }
						},
						
					}); 

			},
	
		
			
		}
	}
</script>

<style>
		.givePower{
			width: 100px;
			height: 30px;
			margin-top: 100px;
			line-height: 30px;
		}
		
		.content{
			text-align: center;
		}
		.welcom{
			font-size: 18px;
			position: absolute;
			top:25%;
			left: 50%;
			transform: translateX(-50%);
			
		}
		.fanwei{
			font-size: 14px;
			position: absolute;
			top:10%;
			left: 50%;
			transform: translateX(-50%);
		}
		button{
			width: 60%;
			position: absolute;
			left: 20%;
			top:35%;
		}
	
	
</style>
