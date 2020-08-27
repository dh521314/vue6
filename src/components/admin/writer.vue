<template>
  <div>
    <el-button round size="mini" type="success" @click="show()">添加</el-button>
    <el-table :data="list">
      <el-table-column label="编号" prop="wid"></el-table-column>
      <el-table-column label="作家笔名" prop="wname"></el-table-column>
      <el-table-column label="头像" prop="wphoto"></el-table-column>
      <el-table-column label="作家语录" prop="ana"></el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button round size="mini" type="success" @click="show(scope.row)">修改</el-button>
        </template>
      </el-table-column>
    </el-table>

    <div>
      <!--分页-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[2,10,12]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="parseInt(total)">
      </el-pagination>

    </div>

    <el-dialog :title="title" :visible.sync="dialogFormVisible">
      <el-form :model="writer" label-width="100px">
        <el-form-item label="编号" hidden="hidden">
          <el-input v-model="writer.wid"></el-input>
        </el-form-item>
        <el-form-item label="作家笔名" >
          <el-input v-model="writer.wname"></el-input>
        </el-form-item>
        <el-form-item label="头像" >
          <el-input v-model="writer.wphoto"></el-input>
        </el-form-item>
        <el-form-item label="作家语录" >
          <el-input v-model="writer.ana"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="caozuo(),dialogFormVisible=false">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
    export default {
        name: "writer",
      data:function () {
        return {
          dialogFormVisible:false,
          writer:{},
          title:'',
          list:[],
          // 分页用到的数据
          currentPage: 1,
          pageSize: 10,
          total: 0
        }
      },
      created:function () {
        this.query(1,10);
      },
      methods:{
        query:function(num,size){
          this.$http.post('http://localhost:8088/aiyue/writer/pageFind?num='+num+'&size='+size)
            .then(res=>{
              console.info(res)
              this.list=res.data.list;
              this.total = res.data.total;
              this.currentPage = res.data.pageNum;
            });
        },
        show:function (row) {
          if (row!=null){
            this.title='修改';
            this.dialogFormVisible=true;
            this.writer=Object.assign({},row)
          }else {
            this.title='添加';
            this.dialogFormVisible=true;
            this.writer={}
          }
        },
        caozuo:function () {
          if(this.title == "添加"){
            this.$http.post(`http://localhost:8088/aiyue/writer/addWriter?wname=${this.writer.wname}&wphoto=${this.writer.wphoto}&ana=${this.writer.ana}`)
              .then((resp)=>{
                if(resp.data == 1) {
                  this.query(1,this.pageSize);
                }else if(resp.data == 2){
                  this.$message.info("已存在");
                }else{
                  this.$message.info("添加失败");
                }
             })
          }else {
            this.$http.post(`http://localhost:8088/aiyue/writer/QueryByWriterName?wname=${this.writer.wname}`)
              .then((res) => {
                if (res.data.wname == null){
                  this.$http.post(`http://localhost:8088/aiyue/writer/updateWriters?wid=${this.writer.wid}&wname=${this.writer.wname}&wphoto=${this.writer.wphoto}&ana=${this.writer.ana}`)
                    .then((resp) => {
                      if(resp.data == 1) {
                        this.$message.info("修改成功");
                        this.query(1,this.pageSize);
                      }else {
                        this.$message.info("修改失败");
                      }
                    })
                } else{
                  this.$http.post(`http://localhost:8088/aiyue/writer/updateWriter?wid=${this.writer.wid}&wphoto=${this.writer.wphoto}&ana=${this.writer.ana}`)
                    .then((resps) => {
                      if(resps.data == 1) {
                        this.$message.info("作家已存在或未修改作家名")
                        this.query(1,this.pageSize);
                      }else {
                        this.$message.info("修改失败");
                      }
                    })
                }
              })
           }
        },
        handleSizeChange:function(newSize){
          // pagesize改变触发
          this.pageSize = newSize;
          this.query(this.currentPage,this.pageSize)
        },
        handleCurrentChange(newPage) {
          // 页码改变触发
          this.currentPage = newPage;
          this.query(this.currentPage,this.pageSize)
        }
      }
    }
</script>

<style scoped>

</style>
