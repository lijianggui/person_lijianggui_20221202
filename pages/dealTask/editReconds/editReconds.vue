<template>
	<view>
		<view class="title">
			<text class="">{{title}}</text>
		</view>
		
		<view class="" style="margin-bottom: 50px;">
					<view class="content" v-for="item in detailList">
						<view class="contentTitle" >{{item.label}}</view>
						<view class="contentText" v-for="item2 in item.children" :key="item2.label"> 
							<view  :class="item2.type=='companyType'?'labelView2':'labelView'" > <label class="inputLabel">{{item2.label}}</label></view>
							<text>: </text> 
							
							<input class="input"  type="text" v-if="item2.type!='url'&& item2.type!='companyType'"  v-model="item2.value"  value=""    />
							
							<view  style="width: 60%;"  >
								<select-cy :value="tval" v-if="item2.type=='companyType' && flag" :options="datalist" @change="change"></select-cy>
							</view>
							
							<u-upload
							:previewFullImage="true"
							  v-if="item2.type=='url'"
							   :fileList="item2.value"
								@afterRead="afterRead"
								@delete="deletePic"
								:name="item2.id"
								multiple
								:maxCount="4"
							></u-upload>
										
						</view>
					</view>
		</view>


		
		<button class="taskButton"    @click="editSubmit">提交修改任务</button>
		
	</view>
</template>

<script>
	import commonUrl from "../../../commonUrl.js" 
	
	export default {
		data() {
			return {
				title:"",
				id:"",
				flag: false,
				detailList:[],
				inputList:[{}],
				fileList: [],
				msg:"1111",
				tid:"",
				tval: [],
				datalist:[{
						label: '客运',
						value: '客运'
					},
					{
						label: '普货',
						value: '普货'
					},
					{
						label: '维修',
						value: '维修'
					},
					{
						label: '驾培',
						value: '驾培'
					},
					{
						label: '重点货运源头',
						value: '重点货运源头'
					},
					{
						label: '终检站',
						value: '终检站'
					},
					{
						label: '水运',
						value: '水运'
					},
					{
						label: '小微型客车租赁',
						value: '小微型客车租赁'
					},
					{
						label: '危运',
						value: '危运'
					},
					{
						label: '港口',
						value: '港口'
					},
					{
						label: '交通工程',
						value: '交通工程'
					},
					{
						label: '客运站',
						value: '客运站'
					}
				]
			}
		},
		
		methods: {
			change(item,value) {	
				console.log("this.tval",this.tval)
				this.tval = value
				console.log("this.tval",this.tval)
			},
			editSubmit(){
				let _this=this
				uni.showModal({
					title: '确认修改吗?',
					content: '',
					success: function (res) {
						if (res.confirm) {
							var  taskList={}
							console.log(" _this.detailList", _this.detailList)
							_this.detailList.forEach(item=>{
								item.children.forEach(item2=>{
									if(item2.type=="companyType"){
										item2.value=_this.tval
									}
									if(item2.value){
										taskList[item2.id]=item2.value;
									}
								})
							})
							console.log('taskList',taskList)
							uni.request({
							  url:commonUrl+"buss/submitTask",
							  method: "POST",
							  header: {'token':uni.getStorageSync("token")},//传在请求的header里
							  data:{
								    items:taskList,
								    id: _this.id,
									tid:_this.tid
							  },
							  success(res) {
							  //请求成功的处理
								  console.log("添加任务的结果",res)
								  if(res.statusCode==200){
									  _this.show=false
									  uni.showToast({
									  	 title:"修改成功",
										 icon:"success"
									  })
									  
									  setTimeout(function() {
										uni.navigateBack({
												delta: 1
											});  
									  }, 1000);
									 
								  }
							  }
							})
							
							
						} else if (res.cancel) {
							console.log('用户点击取消');
						}
					}
				});
			},
			// 删除图片
			deletePic(event) {
				this.fileList.splice(event.index, 1)
			},
		// 新增图片
			async afterRead(event) {
					// 当设置 mutiple 为 true 时, file 为数组格式，否则为对象格式
				console.log(event,'event')
				console.log("this.detailList",this.detailList)
				if(event.file[0].size>2097152){
					uni.showToast({
						title:"不能超过2M",
						icon:"error"
					})
					return
				}
				let lists = [].concat(event.file)
				 var template=[]
				this.detailList.forEach(item=>{
					item.children.forEach(item2=>{
						if(item2.id==event.name){
							 template=item2.value
						}
					})
				})
				
				let fileListLen =template.length  //0
				lists.map((item) => {
					template.push({
						...item,
						status: 'uploading',
						message: '上传中'
					})
				})
		
				for (let i = 0; i < lists.length; i++) {
					const result = await this.uploadFilePromise(lists[i].url)
					let item =template[fileListLen]
					template.splice(fileListLen, 1, Object.assign(item, {
						status: 'success',
						message: '',
						url: result
					}))
					console.log('result',result)
					console.log('template',template)	
					fileListLen++
					this.fileList=template
				}	
					
					
			},
			
			uploadFilePromise(url) {
				return new Promise((resolve, reject) => {
					let a = uni.uploadFile({
						header: {'token':uni.getStorageSync("token")},
						url: commonUrl+'file/upload', // 仅为示例，非真实的接口地址
						filePath: url,
						name: 'file',
						formData: {
							user: 'test'
						},
						success: (res) => {
							// console.log("xxxxres",res)
							// let imgList=JSON.parse(res.data)
							// resolve(res.data)
							resolve(JSON.parse(res.data).data)
						},
						fail: (res) => {
							console.log("失败的res",res)
							
						},
					});
				})
			},
			
			
			//  提交任务
			submit(){
				this.show=true
			},
			
			async getList(){
				let _this=this
				uni.request({
				  url:commonUrl+"submitTask/"+_this.id,
				  method: "GET",
				  header: {'token':uni.getStorageSync("token")},//传在请求的header里
				  data:{
					  tid:_this.id
				  },
				  success(res) {
				  //请求成功的处理
					  _this.detailList=res.data.data
					  _this.detailList.forEach(item=>{
						  item.children.forEach(item2=>{
							  if(item2.type=="url"){
								  item2.value=[]
							  }else if (item2.type=="companyType"){
								  console.log("-----------------"+item2.value.length)
								  _this.tval= item2.value
								  _this.flag = true
								  console.log(_this.tval)
							  }
							  
						  })
					  })
					  console.log("------------详情",_this.detailList)	
				  }
				})
			},
		},
		onLoad(option){
			console.log("option",option)
			this.id=option.id
			this.tid=option.tid
			this.getList()
		}
	}
</script>

<style>
.u-upload__wrap__preview__image{
	width: 50px !important;
	height: 50px !important;
}
.u-upload__button{
	width: 40px !important;
	height: 40px !important;
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
.labelView2{
	width: 45%;
	overflow: hidden;  /* 超出一行文字自动隐藏 */
	text-overflow: ellipsis;  /*文字隐藏后添加省略号*/
	white-space: nowrap; 
}

.labelView{
	width: 100%;
	overflow: hidden;  /* 超出一行文字自动隐藏 */
	text-overflow: ellipsis;  /*文字隐藏后添加省略号*/
	white-space: nowrap; 
}

.title{
	font-weight: 600;
	font-size: 12px;
	margin-right: 20px;
	margin-top: 10px;
	margin-bottom: 20px;
	width: 90%;
	margin-left: 5%;
	
}
.content{
	font-size: 12px;
	/* margin-left: 20px; */
	margin-bottom: 20px;
	width: 90%;
	margin-left: 5%;
}
.contentTitle{
	font-weight: 600;
	margin-bottom: 5px;
	font-size: 16px;
}
.contentText{
	display: flex;
	margin-bottom: 6px;
	font-size: 16px;
}
</style>
