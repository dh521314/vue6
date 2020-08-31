<template>
  <div>
    <el-button type="primary" @click="show()">添加</el-button>
    <el-table border :data="sections" stripe height="700px">
      <el-table-column label="编号" prop="sid"></el-table-column>
      <el-table-column label="章节名" prop="sname"></el-table-column>
      <el-table-column label="小说名" prop="message.mename"></el-table-column>
      <el-table-column label="内容" prop="content"></el-table-column>
      <el-table-column label="字数" prop="number"></el-table-column>
      <el-table-column label="更新时间" prop="updatetiem"></el-table-column>
      <el-table-column label="操作" width="130px">
        <!-- scope：返回当前单元格 -->
        <template slot-scope="scope">
          <el-button type="success" round size="mini" icon="el-icon-edit" @click="show(scope.row)"></el-button>
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
      <el-form :model="section" label-width="100px">
        <el-form-item label="编号" hidden="hidden">
          <el-input v-model="section.sid"></el-input>
        </el-form-item>
        <el-form-item label="小说名称" :style="display">
          <el-input v-model="section.mename"></el-input>
          <el-autocomplete
            v-model="section.mename"
            :fetch-suggestions="querySearchAsync"
            placeholder="请输入小说名称"
            @select="handleSelect"
          ></el-autocomplete>
        </el-form-item>
        <el-form-item label="章节名称" >
          <el-input v-model="section.sname"></el-input>
        </el-form-item>
        <el-form-item label="内容" >
          <el-input
            type="textarea"
            :autosize="{ minRows: 2, maxRows: 4}"
            placeholder="请输入内容"
            v-model="section.content">
          </el-input>
        </el-form-item>
        <el-form-item label="字数" >
          <el-input v-model="section.number"></el-input>
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
      name: "Section",
      data() {
        return {
          dialogFormVisible: false,
          title: '',
          section: {},
          sections: [],
          // 分页用到的数据
          currentPage: 1,
          pageSize: 10,
          total: 0,
          restaurants: [],
          state:'',
          timeout:  null
        }
      },
      created() {
        this.sectionList(1,10);
      },
      methods:{
        show: function (row) {
          if (row == null) {
            // 添加
            this.title = '添加章节';
            this.dialogFormVisible = true;
            this.type = {};
          } else {
            // 修改
            this.title = '修改章节';

            this.dialogFormVisible = true;
            // 复制
            this.section = Object.assign({},row);
            //隐藏
            this.display = "display:none;";
          }
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
        },
        sectionList:function (num,size) {
          this.$http.post("http://localhost:8088/aiyue/Section/pageFind?num="+num+"&size="+size).then((resp)=>{
            this.sections = resp.data.list;
            console.log(resp.data);
            this.total = resp.data.total;
            this.currentPage = resp.data.pageNum;
          });
        },
        querySearchAsync:function(queryString, callback) {
          if (queryString == null || queryString == undefined) {
            this.$http.post(`http://localhost:8088/aiyue/Section/findMessage`).then((resp)=> {
              this.restaurants = resp.data;
              console.log("findMessage");
              for(let i of this.restaurants){
                i.value = i.mename;  //将想要展示的数据作为value
              }
              let list = resp.data;
              clearTimeout(this.timeout);
              this.timeout = setTimeout(() => {
                callback(list);
              }, 300 * Math.random());
            });
          }else{
            this.$http.post(`http://localhost:8088/aiyue/Section/findMenameByMename?mename=${queryString}`).then((res)=> {
              this.restaurants = res.data;
              let list1 = res.data;
              clearTimeout(this.timeout);
              this.timeout = setTimeout(() => {
                callback(list1);
              }, 300 * Math.random());
              console.log("findMenameByMename");
              for(let j of this.restaurants){
                j.value1 = mename;  //将想要展示的数据作为value
              }
            });
          }
        },
        createStateFilter:function(queryString) {
          return (state) => {
            return (state.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0);
          };
        },
        handleSelect(section) {
          console.log(section);
        },
        caozuo:function(){
          this.$http.post(`http://localhost:8088/aiyue/Section/findMeidByMename?mename=${this.section.mename}`).then((res)=>{
            console.log(res.data);
            messageid = res.data;
            if(this.title == "添加章节"){
              this.$http.post(`http://localhost:8088/aiyue/Section/addSection?sname=${this.section.sname}&messageid=${messageid}&content=${this.section.content}&number=${this.section.number}`).then((resp)=>{
                if(resp.data > 0) {
                  this.sectionList(1,this.pageSize);
                }else{
                  this.$message.info("添加失败");
                }
              });
            }else{
              this.$http.post(`http://localhost:8088/aiyue/Section/updSection?sid=${this.section.sid}&sname=${this.section.sname}&content=${this.section.content}&number=${this.section.number}`).then((resp)=>{
                if(resp.data > 0) {
                  this.sectionList(1,this.pageSize);
                }else{
                  this.$message.info("修改失败");
                }
              });
            }
          });
          this.dialogFormVisible = false;
          document.getElementById("sid").setAttribute("hidden","hidden");
        }
      }
    }
</script>

<style scoped>

</style>
