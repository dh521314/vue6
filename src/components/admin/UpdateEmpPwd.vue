<template>
  <div>
    <el-form label-width="100px" :model="pwd" label-suffix="：">
      <el-form-item label="原密码">
        <el-input v-model="pwd.yuan" show-password></el-input>
      </el-form-item>
      <el-form-item label="新密码">
        <el-input v-model="pwd.xin" show-password></el-input>
      </el-form-item>
      <el-form-item label="确认密码">
        <el-input v-model="pwd.que" show-password></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="success"  @click="updatePwd">确认</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: 'UpdateEmpPwd',
  data: () => {
    return {
      employee: '',
      pwd:{ }
    }
  },
  methods:{
    'updatePwd':function(){
      this.employee = JSON.parse(sessionStorage.getItem("token"))
      if (this.pwd.yuan == this.employee.epwd){
        if(this.pwd.xin == this.pwd.que){
          this.$http.post('http://localhost:8088/aiyue/Emp/updateEmpPwd?eid=' + this.employee.eid + '&epwd=' + this.pwd.que).then(res => {
            console.info(res)
            this.$router.replace('/login');
          })
        }else {
          this.$message('新密码于确认密码不一致');
        }
      }else{
        this.$message('原密码错误');
      }
    }
  }
}
</script>

<style scoped>

</style>
