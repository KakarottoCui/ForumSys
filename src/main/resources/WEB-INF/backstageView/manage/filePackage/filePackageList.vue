<!-- 压缩文件列表 -->
<template id="filePackageList-template">
	<div>
		<div class="main">
			<div class="navbar">
				<el-button type="primary" plain size="small" @click="$router.push({path: '/admin/control/filePackage/manage/package'});">打包文件</el-button>
			</div>
			<div class="data-table" >
				<el-table ref="multipleTable" :data="tableData" tooltip-effect="dark" style="width: 100%" @selection-change="handleSelectionChange" stripe empty-text="没有内容">
					<el-table-column prop="fileName" label="文件名称" align="center" ></el-table-column>
					<el-table-column prop="size" label="文件大小" align="center" width="200"></el-table-column>
					<el-table-column prop="createTime" label="创建时间" align="center" width="200"></el-table-column>
					<el-table-column label="操作" align="center" width="200">
						<template #default="scope">
							<div class="button-group-wrapper">
								<el-button-group>
									<el-button type="primary" size="mini" @click="download($event,scope.row)">下载</el-button>
									<el-button type="primary" size="mini" @click="deleteFilePackage($event,scope.row)">删除</el-button>
								</el-button-group>
							</div>
				    	</template>
					
					</el-table-column>
				</el-table>
			</div>
		</div>
	</div>
</template>

<script>
//压缩文件列表
export default({
	name: 'filePackageList',//组件名称，keep-alive缓存需要本参数
	template : '#filePackageList-template',
	inject:['reload'], 
	data : function data() {
		return {
			tableData: [],//表格内容
			multipleSelection: [],
		  
			downloadForm:false,//是否显示文件下载表单
			submitForm_disabled:false,//提交按钮是否禁用
			
			downloadProgressPercent: 0, // 下载进度条默认为0
		};
	},
	
	created : function created() {
		//设置缓存
		this.$store.commit('setCacheComponents',  [this.$route.name]);
		
		
		//初始化
		this.queryFilePackageList();
	},
	methods : {
		//查询压缩文件列表
		queryFilePackageList : function() {
			let _self = this;
			_self.$ajax.get('control/filePackage/list', {
			    params: {
			    	page :  _self.currentpage
			    }
			})
			.then(function (response) {
				if(response == null){
					return;
				}
			    let result = response.data;
			    if(result){
			    	let returnValue = JSON.parse(result);
			    	if(returnValue.code === 200){//成功
			    		_self.tableData = returnValue.data;
			    		
			    	}else if(returnValue.code === 500){//错误
			    		
			    		
			    	}
			    }
			})
			.catch(function (error) {
				console.log(error);
			});
		},
		//分页
		page: function(page) {
		
			//删除缓存
			this.$store.commit('setCacheNumber');
			this.$router.push({
				path: '/admin/control/filePackage/list', 
				query:{ page : page}
			});
	    },
		//删除压缩文件
	    deleteFilePackage : function(event,row) {
			//强制失去焦点
			let target = event.target;
			// 根据button组件内容 里面包括一个span标签，如果设置icon，则还包括一个i标签，其他情况请自行观察。
    		// 所以，在我们点击到button组件上的文字也就是span标签上时，直接执行e.target.blur()不会生效，所以要加一层判断。
        	if(target.nodeName == 'SPAN' || target.nodeName == 'I'){
            	target = event.target.parentNode;
       		}
        	target.blur();
			
			
	    	let _self = this;
	    	this.$confirm('此操作将删除该项, 是否继续?', '提示', {
	            confirmButtonText: '确定',
	            cancelButtonText: '取消',
	            type: 'warning'
	        }).then(() => {
	        	let formData = new FormData();
		    	formData.append('fileName', row.fileName);
		    	
		    	
				this.$ajax({
			        method: 'post',
			        url: 'control/filePackage/manage?method=delete',
			        data: formData
				})
				.then(function (response) {
					if(response == null){
						return;
					}
				    let result = response.data;
				    if(result){
				    	let returnValue = JSON.parse(result);
				    	if(returnValue.code === 200){//成功
				    		_self.$message.success("删除成功");
				    		_self.queryFilePackageList();
				    	}else if(returnValue.code === 500){//错误
				    		
				    		let errorMap = returnValue.data;
				    		let htmlContent = "";
				    		let count = 0;
				    		for (let key in errorMap) {   
				    			count++;
				    			htmlContent += "<p>"+count + ". " + errorMap[key]+"</p>";
				    			
				    	    }
				    		_self.$alert(htmlContent, '错误', {
				    			showConfirmButton :false,
				    			dangerouslyUseHTMLString: true
				    		})
				    		.catch(function (error) {
								console.log(error);
							});
				    		
				    		
				    	}
				    }
				})
				.catch(function (error) {
					console.log(error);
				});
	        	
	        }).catch((error) => {
	        	console.log(error);
	        });
	    },
	  	//下载
  		download : function(event,row) {
  			//强制失去焦点
  			let target = event.target;
  			// 根据button组件内容 里面包括一个span标签，如果设置icon，则还包括一个i标签，其他情况请自行观察。
      		// 所以，在我们点击到button组件上的文字也就是span标签上时，直接执行e.target.blur()不会生效，所以要加一层判断。
          	if(target.nodeName == 'SPAN' || target.nodeName == 'I'){
              	target = event.target.parentNode;
         		}
          	target.blur();
          	
          	/**
          	let access_token = "";
          	let oauth2Token = window.sessionStorage.getItem('oauth2Token');
  			if(oauth2Token != null){
  				let oauth2Object = JSON.parse(oauth2Token);
  				access_token = oauth2Object.access_token;
  			}
  			const link = document.createElement('a'); // 生成一个a标签。
  			link.href = "control/filePackage/manage?method=download&fileName="+row.fileName+"&access_token="+access_token; // href属性指定下载链接
  			link.download = row.fileName; // dowload属性指定文件名
  			link.setAttribute("rel","opener");
  			link.click(); // click()事件触发下载
  			**/
  			 
          	
  			let _self = this;
  			_self.downloadForm = true;
  			_self.downloadProgressPercent =0;


  			_self.$ajax.get('control/filePackage/manage', {
  			    params: {
  			    	method:'download',
  			    	fileName:row.fileName
  			    },
  			    headers: {
  				  	'showLoading': false,//是否显示图标
  				},
  			    responseType: 'blob',// 表明返回服务器返回的数据类型
  			    onDownloadProgress: (evt) => {
  					// 对原生进度事件的处理
  					_self.downloadProgressPercent = parseInt(evt.loaded / evt.total * 100);
  				}
  			})
  			.then(function (response) {
  				if(response == null){
  					return;
  				}
  				var blob = response.data;
  				if(blob.type ==  "text/json"){
  					_self.$message({
  						duration :0,
  			            showClose: true,
  			            message: (response.headers.exception != undefined ? decodeURIComponent(response.headers.exception) :""),
  			            type: 'error'
  			        });
  			        _self.downloadForm = false;
  					return;
  				}
  				
  				
  			    var blob = response.data;
  			    var a = document.createElement('a');
  			    // blob.type = "application/octet-stream";
  			    var url = window.URL.createObjectURL(blob);

  				var fileName = row.fileName;

  			    if(window.navigator.msSaveBlob){
  			    	try {
  			        	window.navigator.msSaveBlob(blob,fileName)
  			    	} catch (e) {
  			        	console.log(e);
  			    	}
  			    }else{
  			    	a.href = url;
  			    	a.download = fileName ;
  			    	document.body.appendChild(a); // 火狐浏览器 必须把元素插入body中
  			    	a.click();
  			    	document.body.removeChild(a);
  			   		//释放之前创建的URL对象
  			    	window.URL.revokeObjectURL(url);
  			    }
  			     _self.downloadForm = false;
  			})
  			.catch(function (error) {
  				console.log(error);
  			});
  		},
	}
});
</script>
