<template>
	<div class="put-in-storage">
		<div class="table-head clear">
			<ul class="clear">
				<li>	
					<div class="demo-input-suffix">
					  <el-input v-model="searchText" placeholder="请输入库单编号或供应商" style="width: 200px;"></el-input>
					</div>
				</li>
				<li>
			  	    <template>
					  <div class="block">
					    <span class="demonstration">入库时间:</span>
					    <el-date-picker
					      style="width: 150px; margin-left: 5px;"
					      v-model="dateValue1"
					      type="date"
					      placeholder="选择日期">
					    </el-date-picker>
					    <span>至</span>
					    <el-date-picker
					      style="width: 150px; margin-left: 5px;"
					      v-model="dateValue2"
					      type="date"
					      placeholder="选择日期">
					    </el-date-picker>
					  </div>
					</template>
				</li>
				<li>
					<el-button type="primary" @click="checkBut">查询</el-button>
				</li>
				<li>
					<el-button type="info" @click="reset">重置</el-button>
				</li>
				<li style="margin-left: 30px;" v-show="roleName == '库管员'">
					<router-link :to="{path: '/yjw_admin/putInStorageInfo'}">
						<el-button type="warning" @click="reset">新增</el-button>
					</router-link>					
				</li>
			</ul> 
		</div>
		<div class="table-body">
			<template>
			  <el-table ref="multipleTable" :data="tableData" tooltip-effect="dark" style="width: 100%;text-align: left;">			    
			    <el-table-column prop="stockInSn" label="入库单编号">
			    </el-table-column>
			    <el-table-column prop="orderId" label="关联的采购订单">
			    </el-table-column>
			    <el-table-column prop="supplierName" label="供应商">
			    </el-table-column>	
			    <el-table-column prop="repertoryName" label="仓库">
			    </el-table-column>	
			    <el-table-column prop="storeName" label="库房">
			    </el-table-column>
			    <el-table-column prop="inspectorName" label="录货人">
			    </el-table-column>
			    <el-table-column prop="statusString" label="入库单状态">
			    	<template slot-scope="scope">
			    		<el-popover
						  placement="right"
						  width="300"
						  trigger="click"
						  @show="getPutinStorageStatus(scope.row.stockInSn)"
						  >
						  <el-table :data="gridData">
						    <el-table-column width="100" property="operationTime" label="日期"></el-table-column>
						    <el-table-column width="100" property="operationName" label="操作人"></el-table-column>
						    <el-table-column width="100" property="operation" label="操作"></el-table-column>
						  </el-table>
						  <span class="pointer blue-font" slot="reference" v-text="scope.row.statusString"></span>
						</el-popover>	
			    	</template>
			    </el-table-column>
			    <el-table-column prop="currentHumidity" label="查看">
			    	<template slot-scope="scope">
			    		<router-link :to="{path: '/yjw_admin/putInStorageDanju', query:{
			    			stockInSn: scope.row.stockInSn,
			    			status: scope.row.status
			    		}}" v-if="scope.row.status != 2">
			    			<span>预览</span>
			    		</router-link>
			    		<router-link :to="{path: '/yjw_admin/putInStorageDanju', query:{
			    			stockInSn: scope.row.stockInSn,
			    			status: scope.row.status
			    		}}" v-if="scope.row.status == 2">
			    			<span>查看</span>
			    		</router-link>
			    	</template>
			    </el-table-column>
			    <el-table-column prop="currentHumidity" label="操作">
			    	<template slot-scope="scope">
			    		<i class="el-icon-delete" @click="deleteOneLine(scope.row)" title="删除" v-if="scope.row.status == 0"></i>
			    		<router-link :to="{path: '/yjw_admin/putInStorageUpdate',query:{
			    				   	stockInSn: scope.row.stockInSn,
			    				   	status: scope.row.status,
			    				   	pageParameter: '编辑'
			    			}}"  v-if="scope.row.status == 0">
			    			<i class="el-icon-edit" style="margin-left: 10px;" title="编辑"></i>	
			    		</router-link>
			    		<router-link :to="{path: '/yjw_admin/putInStorageUpdate',query:{
			    				   	stockInSn: scope.row.stockInSn,
			    				   	status: scope.row.status,
			    				   	pageParameter: '查看'
			    			}}">
			    			<i class="el-icon-search" style="margin-left: 10px;" v-if="scope.row.status != 0" title="查看"></i>	
			    		</router-link>
			    	</template>
			    </el-table-column>			    
			  </el-table>
			    <el-pagination
			      @size-change="handleSizeChange"
			      @current-change="handleCurrentChange"
			      :current-page.sync="nowPage"
			      :page-size="pageSize"
			      layout="prev, pager, next, jumper"
			      :total="totalCount">
			    </el-pagination>
			</template>				
		</div>			
	</div>
</template>

<script>

export default{
	data(){
		return{
			nowPage: 1,								//当前页
	      	pageSize: this.GLOBAL.pageSize,			//每页显示条数
	      	totalCount: 0,							//总条数
			searchText: '',
			dateValue1: '',
			dateValue2: '',
			tableData: [],
			gridData: [],
			roleName: this.common.SStorage.getItem("saveUserInfo").roleName
		}
	},
	methods:{
		reset(){
			this.searchText = ''
			this.dateValue1 = ''
			this.dateValue2 = ''
		},
		checkBut(){
			this.queryTableData()
		},
		deleteOneLine(row){
			this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
	          confirmButtonText: '确定',
	          cancelButtonText: '取消',
	          type: 'warning'
	        }).then(() => {
	  			let parmUser = {
			  		"stockInSn": row.stockInSn,
					"storeroomId": row.storeroomId,
					"status": row.status,
			  		"signId": this.GLOBAL.userid
			  	}
	  			console.log(parmUser)
					let baseParmUser = this.common.DataToBase64(JSON.stringify(parmUser))
					this.$axios.get('/wms_web/deleteForStockInList?data='+baseParmUser+'&sign='+this.GLOBAL.urlStr(parmUser))
						
					  .then((res) => {				  	
					  	let nowData = JSON.parse(this.common.base64ToData(res.data))
					  	console.log(nowData)
					  	if(nowData.code == 0){	
					  		console.log(nowData.result)
						  	this.$message({
					            type: 'success',
					            message: '删除成功!'
					          })
						  	this.queryTableData()
					  	}else{
					  		console.log(nowData.message)
					  		this.$message({
					          message: nowData.message,
					          type: 'warning'
					        })
					  	}
					  	
					  })
					  .catch((err) => {
					    console.log(err);
					    this.$message({
				          message: err,
				          type: 'warning'
				        })
					  })        	
	          
	        }).catch(() => {
	          this.$message({
	            type: 'info',
	            message: '已取消删除'
	          })        
	        })	
		},
		getPutinStorageStatus(vsn){	//查看入库单状态
			let _this = this
    		let parm = {	
					  "auditSn": vsn,
					"auditType": 3,
			     	   "signId": this.GLOBAL.userid
	    	}  

				let baseParm = _this.common.DataToBase64(JSON.stringify(parm))
				console.log(JSON.stringify(parm))
					
			_this.$axios.get('/wms_web/selectForAuditProcess?data='+baseParm+'&sign='+_this.GLOBAL.urlStr(parm))
			  .then((res) => {
			  	
			  	let nowData = JSON.parse(_this.common.base64ToData(res.data))
				console.log(JSON.stringify(nowData))
			  	if(nowData.code == 0){		
			  		_this.gridData = nowData.result
			  	}else{
			  		console.log(nowData.message)
			  		_this.gridData = []
			  		this.$message({
			          message: nowData.message,
			          type: 'warning'
			        })
			  	}		  			
			  })
			  .catch((err) => {
			    console.log(err);
			    this.$message({
		          message: err,
		          type: 'warning'
		        })
			  })  			
		},
		queryTableData(){
			let _this = this
    		let parm = {	
    			    "selectStr": this.searchText,
    		  "createTimeStart": this.dateValue1? this.common.fomatDate(this.dateValue1, 1) : '',
    		  	"createTimeEnd": this.dateValue2? this.common.fomatDate(this.dateValue2, 1) : '',
					"companyId": this.common.SStorage.getItem("saveUserInfo").companyId,
					   "userId": this.common.SStorage.getItem("saveUserInfo").userId,
					   "roleSn": this.common.SStorage.getItem("roleSn"),
					  "pageNum": this.nowPage,
					 "pageSize": this.pageSize,
			     	   "signId": this.GLOBAL.userid
	    	}  

				let baseParm = _this.common.DataToBase64(JSON.stringify(parm))
				console.log(JSON.stringify(parm))
					
			_this.$axios.get('/wms_web/selectForStockInList?data='+baseParm+'&sign='+_this.GLOBAL.urlStr(parm))
			  .then((res) => {
			  	
			  	let nowData = JSON.parse(_this.common.base64ToData(res.data))
				console.log(JSON.stringify(nowData))
			  	if(nowData.code == 0){		
			  		_this.tableData = nowData.result.list
			  		_this.totalCount = nowData.result.total
			  		this.$message({
			          message: nowData.message,
			          type: 'success'
			        })
			  	}else if(nowData.code == 800){
			  		console.log(nowData.message)
			  		this.$message({
			          message: nowData.message,
			          type: 'warning'
			        })
			  		setTimeout(function(){
			  				_this.common.SStorage.removeItem("userKey")
					  		_this.$router.push('/')
							window.location.reload()
					  	},700)
			  	}else{
			  		console.log(nowData.message)
			  		_this.tableData = []
			  		this.$message({
			          message: nowData.message,
			          type: 'warning'
			        })
			  	}		  			
			  })
			  .catch((err) => {
			    console.log(err);
			    this.$message({
		          message: err,
		          type: 'warning'
		        })
			  })    		
    	},
 		handleSizeChange(val){
    		this.pageSize = val
    		console.log(this.pageSize)
    	},
    	handleCurrentChange(val){
    		this.nowPage = val
    		console.log('当前页是:'+this.nowPage)
    		this.queryTableData()
    	}   	
	},
	mounted (){
		this.queryTableData()   
	}
}	
</script>

<style scope>
.table-head{
	padding: 10px 0; border-bottom: 1px solid #DCDFE6;	
}
.table-head ul li{
	float: left; margin-right: 20px; margin-bottom: 10px;
}		
</style>