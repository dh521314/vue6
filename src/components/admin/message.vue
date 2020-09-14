<template>
  <div>
    <el-form
      :inline="true"
      :model="formSearch"
      ref="formSearch"
      class="demo-form-inline"
      label-width="68px"
    >
      <el-form-item class="form_input" label="昵称" prop="name">
        <el-input v-model="formSearch.name" placeholder="小说名称"></el-input>
      </el-form-item>
      <el-form-item class="form_select" label="类别" prop="type">
        <el-select v-model="formSearch.type" placeholder="类别">
          <el-option label="请选择" value=""></el-option>
          <el-option v-for="type in types"
                     :key="type.tid"
                     :label="type.tname"
                     :value="type.tid"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSearch">查询</el-button>
        <el-button type="warning" @click="onReset" plain>重置</el-button>
      </el-form-item>
    </el-form>



    <el-table border :data="mes" stripe height="700px" v-loading="loading">
      <el-table-column label="编号" prop="meid"></el-table-column>
      <el-table-column label="名称" prop="mename"></el-table-column>
      <el-table-column label="小说类型" prop="type.tname"></el-table-column>
      <el-table-column label="封面图">
        <template slot-scope="scope">
          　　　　<img :src="'http://localhost:8088/aiyue'+scope.row.surface" width="60" height="80" class="head_pic"/>
          　　</template>
      </el-table-column>
      <el-table-column label="小说简介" prop="synopsis" :show-overflow-tooltip="true"></el-table-column>
      <el-table-column label="作家" prop="writer.wname"></el-table-column>
      <!--<el-table-column label="操作" width="160px">
        &lt;!&ndash; scope：返回当前单元格 &ndash;&gt;
        <template slot-scope="scope">
          <el-button type="success" size="mini" @click="show(scope.row)">编辑</el-button>
          <el-button type="warning" size="mini" @click="del(scope.row.meid)">删除</el-button>
        </template>
      </el-table-column>-->
    </el-table>

    <div>
      <!--分页-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[10,20,50]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="parseInt(total)">
      </el-pagination>

    </div>

    <!--<el-button type="primary" @click="show()">添加</el-button>-->

    <el-dialog :title="title" :model="me" :visible.sync="dialogFormVisible" @close="close">
      <el-form label-width="100px">
          <el-form-item label="编号" id="tid" :style="display">
            <el-input v-model="me.meid" readonly></el-input>
          </el-form-item>
        <el-form-item label="名称">
          <el-input v-model="me.mename"></el-input>
        </el-form-item>
        <el-form-item label="小说类型">
          <el-select v-model="me.typeid" >
            <el-option v-for="type in types"
                       :key="type.tid"
                       :label="type.tname"
                       :value="type.tid"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="封面图">
            <input type="file" id="file1" ref="file1"></input>
        </el-form-item>
        <el-form-item label="简介">
          <el-input type="textarea" v-model="me.synopsis" :autosize="{ minRows: 3, maxRows: 10}"></el-input>
        </el-form-item>
        <el-form-item label="作家">
<!--          <el-input v-model="me.writerid"></el-input>-->
          <el-select v-model="me.writerid" >
            <el-option v-for="item in writes"
                       :key="item.wid"
                       :label="item.wname"
                       :value="item.wid"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="close">取 消</el-button>
        <el-button type="primary" @click="sub()">确 定</el-button>
      </div>
    </el-dialog>

  </div>
</template>
.
<script>
  export default {
    name: "message",
    data() {
      return {
        loading:true,
        dialogFormVisible: false,
        title: '',
        me: {},
        mes: [],
        types:[],
        writes:[],
        formSearch: {
          name: "",
          type: "",
          gender: null,
          startdate: null, //开始时间
          enddate: null ,//结束时间
        },
        display:"display:none;",
        // 分页用到的数据
        currentPage: 1,
        pageSize: 10,
        total: 0
      }
    }
    , created(){
      this.selectAll(1,10);
      this.$http.post("http://localhost:8088/aiyue/Type/findAll").then((resp)=> {
        this.types = resp.data;
      });
      this.$http.post('http://localhost:8088/aiyue/writer/writerQuery')
        .then(res=>{
          this.writes = res.data;
          this.loading = false;
        });
    }
    ,methods:{
      show: function (row) {
        console.log("row"+row)
        if (row == null) {
          // 添加
          this.title = '新增';
          this.dialogFormVisible = true;
          this.me = {};
        } else {
          // 复制
          this.me = Object.assign({},row);
          // 修改
          this.title = '修改';

          this.dialogFormVisible = true;
          //document.getElementById("tid").style.display = "block";
          this.display = "display:block;";
        }
      },
      close:function(){
        document.getElementById("file1").value = null;
        this.dialogFormVisible = false;
        // document.getElementById("tid").style.display = "none";
        this.display = "display:none;";
      },
      onSearch(){
        console.log(this.formSearch);
        let formData = new FormData();
        formData.append("mename",this.formSearch.name);
        formData.append("typeid",this.formSearch.type);
        this.$http.post("http://localhost:8088/aiyue/Message/findAll",formData).then(resp=>{
          this.mes = resp.data.list;
          console.log(resp.data);
          this.total = resp.data.total;
          this.currentPage = resp.data.pageNum;
        });
      },
      onReset() {
        //重置
        this.$refs["formSearch"].resetFields();
      },
      del:function(tid){
        this.$http.post(`http://localhost:8088/aiyue/Message/delMessage?meid=`+tid).then((resp)=>{
          if(resp.data > 0){
            this.selectAll(1,this.pageSize);
          }else{
            this.$message.info("删除失败");
          }
        });
      },
      sub:function(){
        let config = {headers: {'Content-Type':'multipart/form-data'}};
        var formData = new FormData();

        formData.append("typeid",this.me.typeid);
        formData.append("mename",this.me.mename);
        formData.append("surface",this.$refs.file1.files[0]);
        formData.append("synopsis",this.me.synopsis);
        formData.append("writerid",this.me.writerid);

        if(this.title == "新增"){
          this.$http.post(`http://localhost:8088/aiyue/Message/addMessage`,formData,config).then((resp)=>{
            if(resp.data > 0) {
              this.selectAll(1,this.pageSize);
            }else{
              this.$message.info("添加失败");
            }
          });
        }else{
          formData.append("meid",this.me.meid);
          this.$http.post(`http://localhost:8088/aiyue/Message/updMessage`,formData,config).then((resp)=>{
            if(resp.data > 0) {
              this.selectAll(1,this.pageSize);
            }else{
              this.$message.info("修改失败");
            }
          });
        }
        //清空文件筐
        document.getElementById("file1").value = null;
        this.dialogFormVisible = false;
        document.getElementById("tid").setAttribute("hidden","hidden");
      },
      selectAll:function(num,size){
        this.$http.post("http://localhost:8088/aiyue/Message/findAll",this.$qs.stringify({num:num,size:size})).then((resp)=>{
          this.mes = resp.data.list;
          console.log(resp.data);
          this.total = resp.data.total;
          this.currentPage = resp.data.pageNum;
        });
      },
      handleSizeChange:function(newSize){
        // pagesize改变触发
        this.pageSize = newSize;
        this.selectAll(this.currentPage,this.pageSize)
      },
      handleCurrentChange(newPage) {
        // 页码改变触发
        this.currentPage = newPage;
        this.selectAll(this.currentPage,this.pageSize)
      }
    }
  }
</script>

<style ang="scss">
  .el-tooltip__popper{max-width: 400px}
</style>
