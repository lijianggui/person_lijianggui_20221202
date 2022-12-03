<template>
	<view>
		<view class="title">
			<text class="">{{title}}</text>
		</view>
		
		<view class="" style="margin-bottom: 50px;">
					<view class="content" v-for="item in detailList">
						<view class="contentTitle" >{{item.label}}</view>
						<view class="contentText" v-for="item2 in item.children"> 
							<view  :class="item2.type=='companyType'?'labelView2':'labelView'"  > <label class="inputLabel">{{item2.label}}</label></view>
							<text>: </text> 
							<input class="input"  v-if="item2.type!='url'&& item2.type!='companyType'"  v-model="item2.value"   ref="inputValue"  value=""  placeholder="请输入"  />
							<view  style="width: 60%;"  >       
								<select-cy :value="tval"  v-if="item2.type=='companyType'"     :options="datalist" @change="change"></select-cy>
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
								
							<!-- <input class="input"  v-if="item2.type=='date'"     v-model="item2.value"   value=""  placeholder="请输入日期"  />
							<input class="input"  v-if="item2.type=='num'"     v-model="item2.value"   value=""  placeholder="请输入数字"  /> -->
						</view>
					</view>
		</view>


		<view class="button" @click="submit" >
			提交任务
		</view>
		
		<u-modal  @cancel="cancel"  @confirm="confirm" :show="show" :title="modalTitle" :content='content' showCancelButton=true></u-modal>		
	</view>
</template>

<script>
import commonUrl from "../../../commonUrl.js" 
	
	export default {
		data() {
			return {
				value1: Number(new Date()),
				showDatePick:false,
				title:"",
				id:"",
				detailList:[],
				show:false,
				modalTitle:"确认提交任务吗？",
				content:"",
				inputList:[{}],
				fileList: [],
				 tval:[],
				datalist:[
					{
						label:'客运' ,
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
					},
					
				]
			
			}
		},
		methods: {
				
		   change(item,value) {
			   
			   console.log("item",item,value,"value")
		                 this.tval = value;
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
			

			cancel(){
				this.show=false
			},
			confirm(){
				console.log('this.tval',this.tval)
				
				
				let _this=this
				var  taskList={}
				console.log(" _this.detailList", this.detailList)
				this.detailList.forEach(item=>{
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
					    tid: _this.id
				  },
				  success(res) {
				  //请求成功的处理
					  console.log("添加任务的结果",res)
					  if(res.statusCode==200){
						  _this.show=false
						  uni.showToast({
						  	 title:"提交成功",
							 icon:"success"
						  })
						setTimeout(function() {
							uni.redirectTo ({
								url:"/pages/homePage/homePage/homePage"
							})
						}, 2000);
					  }
				  }
				})
			},
			
			//  提交任务
			submit(){
				this.show=true
			},
			
			getList(){
				let _this=this
				uni.request({
				  url:commonUrl+"buss/getTaskInfo",
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
							  }else{
								  item2.value=""
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
			this.title="任务名称: "+option.title
			this.id=option.id
			this.getList()
			
			
		},
		onReady() {
			
		}
		
	}
</script>

<style>
/* .uni-select-cy-select :nth-child(2){
	margin-right: 70px !important;
} */
.uni-select-cy .uni-select-cy-select{
	    /* display: block !important; */
	    line-height: 36px !important;
		font-size: 12px !important;
}
.u-upload__wrap__preview__image{
	width: 50px !important;
	height: 50px !important;
}
.u-upload__button{
	width: 40px !important;
	height: 40px !important;
}
.button{
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
	margin-bottom: 20px;
	width: 90%;
	margin-left: 5%;
}
.contentTitle{
	font-weight: 600;
	margin-bottom: 5px;
}
.contentText{
	display: flex;
	margin-bottom: 6px;
}
</style>
