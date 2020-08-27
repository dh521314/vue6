<template>
  <div>
    <el-table border :data="mes" stripe height="700px" v-loading="loading">
      <el-table-column label="编号" prop="meid"></el-table-column>
      <el-table-column label="名称" prop="mename"></el-table-column>
      <el-table-column label="小说类型" prop="typeid"></el-table-column>
      <el-table-column label="封面图" prop="surface"></el-table-column>
      <el-table-column label="小说简介" prop="synopsis" :show-overflow-tooltip="true"></el-table-column>
      <el-table-column label="作家" prop="writerid"></el-table-column>
      <el-table-column label="操作" width="160px">
        <!-- scope：返回当前单元格 -->
        <template slot-scope="scope">
          <el-button type="success" size="mini" @click="show(scope.row)">编辑</el-button>
          <el-button type="warning" size="mini" @click="del(scope.row.tid)">删除</el-button>
        </template>
      </el-table-column>
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

    <el-button type="primary" @click="show()">添加</el-button>

    <el-dialog :title="title" :visible.sync="dialogFormVisible">
      <el-form :model="me" label-width="100px">
        <div id="tid" hidden>
          <el-form-item label="编号">
            <el-input v-model="me.meid" readonly></el-input>
          </el-form-item>
        </div>
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
          <el-input v-model="me.surface"></el-input>
        </el-form-item>
        <el-form-item label="简介">
          <el-input type="textarea" v-model="me.synopsis" :autosize="{ minRows: 2, maxRows: 10}"></el-input>
        </el-form-item>
        <el-form-item label="作家">
          <el-input v-model="me.writerid"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="close">取 消</el-button>
        <el-button type="primary" @click="sub()">确 定</el-button>
      </div>
    </el-dialog>

  </div>
</template>

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
      setTimeout(function(){},3000);
      this.loading = false;
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
          document.getElementById("tid").removeAttribute("hidden");

        }
      },
      del:function(tid){
        this.$http.post(`http://localhost:8088/aiyue/Message/delMessage?meid=${this.me.meid}`).then((resp)=>{
          if(resp.data > 0){
            this.selectAll(1,this.pageSize);
          }else{
            this.$message.info("删除失败");
          }
        });
      },
      sub:function(){
        if(this.title == "新增"){
          this.$http.post(`http://localhost:8088/aiyue/Message/addMessage?typeid=${this.me.typeid}&mename=${this.me.mename}&surface=${this.me.surface}&synopsis=${this.me.synopsis}&writerid=${this.me.writerid}`).then((resp)=>{
            if(resp.data > 0) {
              this.selectAll(1,this.pageSize);
            }else{
              this.$message.info("添加失败");
            }
          });
        }else{
          this.$http.post(`http://localhost:8088/aiyue/Message/updMessage?meid=${this.me.meid}&typeid=${this.me.typeid}&mename=${this.me.mename}&surface=${this.me.surface}&synopsis=${this.me.synopsis}&writerid=${this.me.writerid}`).then((resp)=>{
            if(resp.data > 0) {
              this.selectAll(1,this.pageSize);
            }else{
              this.$message.info("修改失败");
            }
          });
        }
        this.dialogFormVisible = false;
        document.getElementById("tid").setAttribute("hidden","hidden");
      },
      close:function(){
        this.dialogFormVisible = false;
        document.getElementById("tid").setAttribute("hidden","hidden");
      },
      selectAll:function(num,size){
        this.$http.post("http://localhost:8088/aiyue/Message/findAll?num="+num+"&size="+size).then((resp)=>{
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
