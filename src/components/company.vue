<template>
  <div class="main">
    <div class="main-left">
      <leftMenu :current="current"></leftMenu>
    </div>
    <div class="main-right">
  		<div class="main-box">

        <div class="home">
          <div class="user-title">正尚科技后台管理平台</div>
          <div class="main-top">
            <div class="home-title">公司环境</div>
            <user></user>
          </div>
          <div class="home-add"><el-button type="primary" @click="add">添加</el-button></div>
          <div class="home-table">
            <el-table :data="ourServiceList" border style="width: 100%">
              <el-table-column prop="id" label="ID"> </el-table-column>
              <el-table-column prop="url" label="图片">
                <template slot-scope="scope">
                  <img :src="scope.row.url" alt="图片" @click="enlargeFun(scope.row)">
                </template>
              </el-table-column>
              <el-table-column prop="imageGroup" label="图片分组"> </el-table-column>
              <el-table-column prop="isUse" label="是否使用">
                <template slot-scope="scope">
                  <el-switch v-model="scope.row.isUse" :active-value="1" :inactive-value="0" inactive-color="#B9B9B9" @change="changeSwitch(scope.row)"/>
                </template>
              </el-table-column>
              <el-table-column fixed="right" label="操作">
                <template slot-scope="scope">
                  <el-button @click="handleClick(scope.row)" type="text" size="small">修改</el-button>
                  <el-button @click="deleteFun(scope.row)" type="text" size="small">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </div>
          <addOurService ref="addChange" v-if="isAdd"></addOurService>
          <changeOurService ref="changeLayer" v-if="isModify"></changeOurService>
          <Pagination :currentPage="selectItemPage" :pages="totalPages" @selectPage="pageAxios"></Pagination>
         </div>

  		</div>
    </div>
  </div>
</template>
<style scoped>

</style>
<script type='ecmascript-6'>
  import leftMenu from './leftmenu';
  import axios from 'axios'
	import Pagination from './pagination';
  import addOurService from './company/addOurService';
  import changeOurService from './company/changeOurService';
  import user from './user.vue';

	export default {
    props:{
    },
    data () {
      return {
        current:7,
        ourServiceList: [],
        value2: true,
				totalPages:1,  //默认总页数为1
				pageNumber:10,  //默认每页显示10个
				selectItemPage:1  ,//默认选中页面
        changeData:{},
        isModify:false,
        isAdd:false
      }
    },
    created(){

    },
    mounted (){
      var _this=this;
      var pageData={
        pageSize:_this.pageNumber, //每页显示条数（非必填）
        pageNum:_this.selectItemPage  //当前页（非必填）
      }
      _this.$get('/background-companyImage/queryByPage',pageData).then(data=>{
        var seller=JSON.stringify(data);
      	//console.log('结果:'+seller);

        _this.ourServiceList=data.list;
        _this.totalPages=data.pages;
        _this.selectItemPage=data.pageNum;
      }).catch(function (error) {
      		//console.log(error);
      });
    },
    components: {
			Pagination,
      addOurService,
      changeOurService,
      user,
      leftMenu
    },
    computed: {

    },
    methods: {
      deleteFun(row){
        var _this=this;
        _this.$confirm('是否要删除', '是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
        	_this.$get('/background-companyImage/delete',{id:row.id}).then(data=>{
        		var seller=JSON.stringify(data);
        		_this.$message({
        			type: 'success',
        			message: data+'!'
        		});
            setTimeout(function(){
              location.reload();
            }, 500);
        		//_this.ourServiceList=data.list;
        	}).catch(function (error) {
        			//console.log(error);
        	});
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消！'
          });
        	setTimeout(function(){
        		location.reload();
        	}, 500);
        });
      },
      handleClick(row) {
        //console.log(row);
        this.isModify=true;
        this.changeData={
          id:row.id,
          file:row.url,
          imageGroup:row.imageGroup,
          isModify:this.isModify
        }
        //console.log(this.changeData);
        this.$nextTick(() => {
          $("body").css("overflow","hidden");
          this.$refs.changeLayer.setData(this.changeData);  //调用子组件的方法
        });
      },
      changeSwitch (row) {
        //debugger
        //console.log(row,row.isUse);
        //按钮已经取反
        if(row.isUse==1){
          var dataStr = {
            id: row.id,
            isUse: 1,
            imageGroup:row.imageGroup
          };
        }else{
          var dataStr = {
            id: row.id,
            isUse: 0,
            imageGroup:row.imageGroup
          };
        }
        //console.log(dataStr);
        var _this=this;
				var isOpen="";
				if(row.isUse==1){
					isOpen="开启";
				}else{
					isOpen="关闭";
				}
				_this.$confirm('是否设为'+isOpen+', 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
					_this.$get('/background-companyImage/updateUse',dataStr).then(data=>{
						var seller=JSON.stringify(data);
						//console.log('结果:'+seller);
						//console.log(dataStr.id);
						_this.$message({
							type: 'success',
							message: data+'!'
						});
            setTimeout(function(){
              location.reload();
            }, 500);
						//_this.ourServiceList=data.list;
					}).catch(function (error) {
							//console.log(error);
					});
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消！'
          });
					setTimeout(function(){
						location.reload();
					}, 500);
        });
      },
			add(){
        $("body").css("overflow","hidden");
        this.isAdd=true;
        var addTrue=false;
        this.$nextTick(() => {
          this.$refs.addChange.addData(addTrue);  //调用子组件的方法
        })

			},
      pageAxios(page){
        //console.log(page);
        var _this=this;
        _this.selectItemPage=page;
        var dataObject={
          pageNum:_this.selectItemPage,
          pageSize:_this.pageNumber
        }
        //console.log(dataObject);
        _this.$get('/background-companyImage/queryByPage',dataObject).then(data=>{
          var seller=JSON.stringify(data);
          //console.log('结果:'+seller);
          _this.ourServiceList=data.list;
          _this.totalPages=data.pages;
        })
      },
      enlargeFun(item){
        this.$alert('<div class="message-box"><img class="message-img" src="'+item.url+'" /></div>', '图片详情', {
          dangerouslyUseHTMLString: true
        });
      }
    },
    watch: {}
	}
</script>
