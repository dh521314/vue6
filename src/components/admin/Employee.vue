<template>
  <div>
    <!--查询-->
    <el-button type="primary" @click="show()" style="float:left">添加</el-button>
      <el-input v-model="list.realname" placeholder="姓名" style="width:500px; heght:30px;"></el-input>
      <el-button type="success" icon="el-icon-search" @click="getName"></el-button>
    <el-table :data="list.slice((currentPage-1)*pagesize,currentPage*pagesize)">
      <el-table-column prop="eid" label="编号" ></el-table-column>
      <el-table-column prop="ename" label="账号" ></el-table-column>
      <el-table-column prop="epwd" label="密码" ></el-table-column>
      <el-table-column prop="realname" label="姓名" ></el-table-column>
      <el-table-column prop="idcard"  label="身份证号" ></el-table-column>
      <el-table-column prop="ephoto" label="头像" ></el-table-column>
      <el-table-column prop="ephone" label="手机号" ></el-table-column>
      <el-table-column prop="email" label="邮箱" ></el-table-column>
      <el-table-column prop="pname" label="职位" >

      </el-table-column>
      <el-table-column prop="state" label="状态">
        <template slot-scope="scope">
          <span v-if="scope.row.state==0">在职</span>
          <span v-if="scope.row.state==1">离职</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="210px">
        <template slot-scope="scope">
          <el-button round size="mini"
                     icon="el-icon-s-tools"
                     title="密码重置"
                     @click="updateEmpPwd(scope.row.eid)"></el-button>
          <el-button type="success"
                     round size="mini"
                     icon="el-icon-edit"
                     title="在职"
                     @click="zaizhi(scope.row.eid)">
          </el-button>
          <el-button type="danger"
                     round size="mini"
                     icon="el-icon-edit"
                     title="离职"
                     @click="lizhi(scope.row.eid)">
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <!--分页-->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[5, 10, 15, 20]"
      :page-size="pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="list.length">
    </el-pagination>

    <!--添加-->
    <el-dialog :title="title" :visible.sync="dialogFormVisible">
      <el-form :model="employees" label-width="100px" ref="form" >
        <el-form-item label="账号">
          <el-input v-model="employees.ename"></el-input>
        </el-form-item>
        <el-form-item label="密码">
          <el-input v-model="employees.epwd" show-password></el-input>
        </el-form-item>
        <el-form-item label="职位">
          <el-select v-model="employees.postid" placeholder="请选择">
            <el-option
              v-for="item in posts"
              :key="item.postid"
              :value="item.pid"
              :label="item.pname">
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addorupdate()">确 认</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  import Qs from 'qs';
  export default {
    name: 'Employee',
    data(){
      return{
        employees: {},
        dialogFormVisible:false,
        title:"",
        list:[],
        posts:{},
        currentPage:1,
        pagesize:10,
      }
    },created () {
        this.query(),
        this.$http.post(`http://localhost:8088/aiyue/Post/findPost?pid=${this.posts.pid}&pname=${this.posts.pname}`)
          .then(res =>{
            this.posts = res.data;
          })
    },
    methods:{
      show:function(row){
        if (row == null){
          this.title = "添加";
          this.dialogFormVisible = true;
          this.employees = {ename:"",epwd: "66668888"};
        }
      },
      handleSizeChange: function (size) {
        this.pagesize = size;
        console.log(this.pagesize)
      },
      handleCurrentChange: function(currentPage){
        this.currentPage = currentPage;
        console.log(this.currentPage)
      },
      query:function(){
        this.$http.post("http://localhost:8088/aiyue/Emp/findAll")
          .then(res => {
            this.list = res.data;
          })
      },
      addorupdate:function () {
        if (this.title==="添加"){
          let ename = this.employees.ename;
          let epwd = this.employees.epwd;
          let postid = this.employees.postid;
          this.$http.post(`http://localhost:8088/aiyue/Emp/addEmp?ename=${ename}&epwd=${epwd}&postid=${postid}&state=0`)
            .then(res =>{
              if(res.data === 1) {
                alert("添加成功！")
                this.dialogFormVisible=false;
                this.query();
              }else if(res.data === 2){
                this.$message.info("已存在");
              }else{
                this.$message.info("添加失败");
              }
            })
        }
      },
      updateEmpPwd:function(eid){
        this.$http.post(`http://localhost:8088/aiyue/Emp/updateEmpPwd?eid=${eid}&epwd=66668888`,)
          .then(res=>{
            this.query();
          })
      },
      zaizhi:function (eid) {
        this.$http.post(`http://localhost:8088/aiyue/Emp/updateEmp?eid=${eid}&state=0`)
          .then(res=>{
            this.query();
          })
      },
      lizhi:function (eid) {
        this.$http.post(`http://localhost:8088/aiyue/Emp/updateEmp?eid=${eid}&state=1`)
          .then(res=>{
            this.query();
          })
      },
      getName(){
        this.$http.post(`http://localhost:8088/aiyue/Emp/findName?realname=${this.list.realname}`)
          .then(res =>{
            this.list = res.data;
          })
      }
    }
  }
</script>

<style scoped>

</style>
