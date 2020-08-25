<template>
  <!--login外框-->
  <div class="_login">
    <!--login内框-->
    <div class="login_inner">
      <el-form class="form" ref="form" :model="formLogin">
        <el-form-item>
          <h2 class="title">爱阅小说管理系统</h2>
        </el-form-item>
        <el-form-item>
          <el-input v-model="formLogin.loginName" placeholder="账号"></el-input>
        </el-form-item>
        <el-form-item>
          <el-input v-model="formLogin.password" placeholder="密码" show-password></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="login">登陆</el-button>
          <div v-show="this.errorInfo.isShowError" class="error">{{this.errorInfo.text}}</div>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>
<style lang="scss">
._login {
  // border:1px solid red;
  height: 100%;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  .login_inner {
    // border:1px solid green;
    width: 460px;
    height: 300px;
    margin-top: -150px;
    display: flex;
    justify-content: center;
    box-shadow: 0 0 25px #cac6c6;
    .form {
      // border:1px solid blue;
      width: 300px;
      margin-top: 30px;
      text-align: center;
      .title {
        color: #505458;
      }
    }
    .error {
      color: red;
    }
  }
}
</style>
<script>
export default {
  name: "login",
  data() {
    return {
      formLogin: {
        loginName: "daihui",
        password: "66668888"
      },
      errorInfo: {
        text: "登陆失败,请重试",
        isShowError: false //显示错误提示
      }
    };
  },
   mounted() {
        document.onkeydown = (event) => {
            var router=this.$route.path;
            var e = event || window.event || arguments.callee.caller.arguments[0];
            if (e && e.keyCode == 13&&this.$route.path=='/login') { // enter 键
                this.login();
            }
        };
    },
  methods: {
    login() {
      var param = {
        loginName: this.formLogin.loginName,
        password: this.formLogin.password
      };
      this.$http
        .post(`http://localhost:8088/aiyue/Emp/login?ename=${this.formLogin.loginName}&epwd=${this.formLogin.password}`)
        .then(response => {
          console.log("成功报文:", response);
          var json = response.data;
          if (json.length > 0) {
            var userInfo = json[0];
            var position = "";
            this.$http.post("http://localhost:8088/aiyue/Post/findByPid?pid="+userInfo.postid)
              .then((resp)=>{
                position = resp.data;
              });

            setTimeout(function(){
              //保存登陆信息
              sessionStorage.setItem("userName", userInfo.ename); //用户名
              sessionStorage.setItem("token", JSON.stringify(userInfo)); //保存用户

              sessionStorage.setItem("position", position); //用户职位
              //登陆成功跳转主页
            },2000);
            this.$router.replace({ path: "/index" });

          } else {
            this.errorInfo.isShowError = true;
            this.errorInfo.text = "用户名或者密码错误";
          }
        })
        .catch(error => {
          console.log("失败报文:", error);
          this.errorInfo.isShowError = true;
          this.errorInfo.text = "系统接口异常";
        });
    }
  }
};
</script>
