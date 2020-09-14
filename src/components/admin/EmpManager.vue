<template>
    <div>
      <el-form label-width="100px" :model="employees" label-suffix="：" >
        <el-form-item label="头像" class="p">
           <input type="file" id="file1" ref="file1"></input>
           <!--<template slot-scope="scope">
     　　　　<img :src="'http://localhost:8088/aiyue'+scope.row.surface" width="60" height="80" class="head_pic"/>
       　　</template>-->
        </el-form-item>
        <el-form-item label="用户名">
          <el-input v-model="employees.ename" :value="employees.ename" :disabled="true" style="width:500px; heght:30px;"></el-input>
        </el-form-item>
        <el-form-item label="真实姓名">
          <el-input v-model="employees.realname" :value="employees.realname" :disabled="true" style="width:500px; heght:30px;"></el-input>
        </el-form-item>
        <el-form-item label="身份证号">
          <el-input v-model="employees.idcard" :value="employees.idcard" :disabled="true" style="width:500px; heght:30px;"></el-input>
        </el-form-item>
        <el-form-item label="手机号">
          <el-input v-model="employees.ephone" :value="employees.ephone" :disabled="true" style="width:500px; heght:30px;"></el-input>
        </el-form-item>
        <el-form-item label="邮箱">
          <el-input v-model="employees.email" :value="employees.email" :disabled="true" style="width:500px; heght:30px;"></el-input>
        </el-form-item>
        <el-button type="success" round icon="el-icon-edit" @click="show(employees)">修改</el-button>
      </el-form>

      <el-dialog :title="title" :visible.sync="dis">
        <el-form :model="emps">
          <el-form-item label="编号" hidden="hidden">
            <el-input v-model="emps.eid"></el-input>
          </el-form-item>
          <el-form-item label="用户名">
            <el-input v-model="emps.ename"></el-input>
          </el-form-item>
          <el-form-item label="真实姓名">
            <el-input v-model="emps.realname"></el-input>
          </el-form-item>
          <el-form-item label="身份证号" show-password>
            <el-input v-model="emps.idcard"></el-input>
          </el-form-item>
          <el-form-item label="手机号" show-password>
            <el-input v-model="emps.ephone"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" show-password>
            <el-input v-model="emps.email"></el-input>
          </el-form-item>
        </el-form>
        <el-button type="primary" @click="dis = false">取 消</el-button>
        <el-button type="primary" @click="updateManager(),dis=false">确 定</el-button>
      </el-dialog>
    </div>
</template>

<script>
    export default {
      name: "EmpManager",
      data: () => {
        return {
          dis: false,
          employees: {},
          emps:{},
          title:'',
          imageUrl: ''
        }
      },
      created(){
        this.query();
      },
      methods:{
        'query': function () {
          this.employees = JSON.parse(sessionStorage.getItem("token"));
          this.$http.post(`http://localhost:8088/aiyue/Emp/findByEname?ename=${this.employees.ename}&epwd=${this.employees.epwd}`)
            .then(res => {
              this.employees=res.data;
            })
        },
        'show': function (employees) {
          this.dis = true
          this.title = '修改个人信息'
          this.emps = Object.assign({}, employees)
        },
        'updateManager' : function () {
          this.$http.post(`http://localhost:8088/aiyue/Emp/updateEmpManager?eid=${this.emps.eid}&ename=${this.emps.ename}&realname=${this.emps.realname}&idcard=${this.emps.idcard}&ephone=${this.emps.ephone}&email=${this.emps.email}`)
            .then(res => {
              if(res.data == 1){
                this.query()
              }
          })
        },
        handleAvatarSuccess(res, file) {
          this.employees = JSON.parse(sessionStorage.getItem("token"));
          this.imageUrl = URL.createObjectURL(file.raw);
          this.$http.post(`http://localhost:8088/aiyue/Emp/updateEmpPhoto?ename=${this.employees.eid}&epwd=${this.employees.epwd}&ephoto=${this.imageUrl}`).then((resp)=> {
            if (resp.data == 1){
              this.query();
            }
          });
        },
        beforeAvatarUpload(file) {
          const isJPG = file.type === 'image/jpeg';
          const isLt2M = file.size / 1024 / 1024 < 2;

          if (!isJPG) {
            this.$message.error('上传头像图片只能是 JPG 格式!');
          }
          if (!isLt2M) {
            this.$message.error('上传头像图片大小不能超过 2MB!');
          }
          return isJPG && isLt2M;
        }
      }
    }
</script>

<style scoped>
  .class{
    border-radius: 4px;
    width: 100px;
    height: 100px;
  }
</style>
