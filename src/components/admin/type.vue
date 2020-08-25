<template>
  <div>
    <el-table border :data="types" stripe height="700px">
      <el-table-column label="编号" prop="tid"></el-table-column>
      <el-table-column label="名称" prop="tname"></el-table-column>
      <el-table-column label="职位" prop="channel">
        <template slot-scope="scope">
          {{scope.row.channel == 1?"女频":"男频"}}
        </template>
      </el-table-column>
      <el-table-column label="操作" width="130px">
        <!-- scope：返回当前单元格 -->
        <template slot-scope="scope">
          <el-button type="success" round size="mini" icon="el-icon-edit" @click="show(scope.row)"></el-button>
          <el-button type="warning" round size="mini" icon="el-icon-delete" @click="del(scope.row.tid)"></el-button>
        </template>
      </el-table-column>
    </el-table>

    <div>
      <!--分页-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[10,12]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="parseInt(total)">
      </el-pagination>

    </div>

    <el-button type="primary" @click="show()">添加</el-button>

    <el-dialog :title="title" :visible.sync="dialogFormVisible">
      <el-form :model="type" label-width="100px">
        <div id="tid" hidden>
          <el-form-item label="编号">
            <el-input v-model="type.tid" readonly></el-input>
          </el-form-item>
        </div>
        <el-form-item label="名称">
          <el-input v-model="type.tname"></el-input>
        </el-form-item>
        <el-form-item label="类型">
          <el-radio-group v-model="type.channel">
            <el-radio :label=0 checked>男频</el-radio>
            <el-radio :label=1>女频</el-radio>
          </el-radio-group>

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
    name: "Type",
    data() {
      return {
        dialogFormVisible: false,
        title: '',
        type: {},
        types: [],
        // 分页用到的数据
        currentPage: 1,
        pageSize: 10,
        total: 0
      }
    }
    , created(){
      this.selectAll(1,10);
    }
    ,methods:{
      show: function (row) {
        if (row == null) {
          // 添加
          this.title = '添加类型';
          this.dialogFormVisible = true;
          this.type = {};
        } else {
          // 修改
          this.title = '修改类型';

          this.dialogFormVisible = true;
          document.getElementById("tid").removeAttribute("hidden");

          // 复制
          this.type = Object.assign({},row);
        }
      },
      del:function(tid){
        this.$http.post(`http://localhost:8088/aiyue/Type/del?tid=${tid}`).then((resp)=>{
          if(resp.data > 0){
            this.selectAll(1,this.pageSize);
          }else{
            this.$message.info("删除失败");
          }
        });
      },
      sub:function(){
        let tid = this.type.tid;
        let tname = this.type.tname;
        let channel = this.type.channel;
        if(this.title == "添加类型"){
          this.$http.post(`http://localhost:8088/aiyue/Type/add?tname=${tname}&channel=${channel}`).then((resp)=>{
            if(resp.data > 0) {
              this.selectAll(1,this.pageSize);
            }else{
              this.$message.info("添加失败");
            }
          });
        }else{
          this.$http.post(`http://localhost:8088/aiyue/Type/update?tid=${tid}&tname=${tname}&channel=${channel}`).then((resp)=>{
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
        this.$http.post("http://localhost:8088/aiyue/Type/pageFind?num="+num+"&size="+size).then((resp)=>{
          this.types = resp.data.list;
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

<style scoped>

</style>
