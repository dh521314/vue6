<template>
    <div align="center">
      <el-form label-width="100px" :model="employees" label-suffix="：">
        <el-form-item label="用户名">
          <el-input v-model="employees.ename" :value="employees.ename" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="真实姓名">
          <el-input v-model="employees.realname" :value="employees.realname" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="身份证号">
          <el-input v-model="employees.idcard" :value="employees.idcard" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="手机号">
          <el-input v-model="employees.ephone" :value="employees.ephone" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="邮箱">
          <el-input v-model="employees.email" :value="employees.email" :disabled="true"></el-input>
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
        <el-button type="primary" @click="updateManager">确 定</el-button>
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
          title:''
        }
      },
      created(){
        this.employees = JSON.parse(sessionStorage.getItem("token"));
        console.log(this.employees);
      },
      methods:{
        'show': function (employees) {
          this.dis = true
          this.title = '修改个人信息'
          this.emps = Object.assign({}, employees)
        },
        'updateManager' : function () {
          this.$http.post(`http://localhost:8088/aiyue/Emp/updateEmpManager?eid=${this.emps.eid}&ename=${this.emps.ename}&realname=${this.emps.realname}&idcard=${this.emps.idcard}&ephone=${this.emps.ephone}&email=${this.emps.email}`)
            .then(res => {
              console.info(res)
              this.$router.replace('/index')
          })
        }
      }
    }
</script>

<style scoped>

</style>
