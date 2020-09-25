<template>
  <div class="wrapper login-index-model">
    <!--顶部-->
    <div class="part-header">
      <div class="part-header-main">
        <span class="hd-title">我的电商</span>
        <span class="hd-text">欢迎登录</span>
      </div>
    </div>
    <!--注册-->
    <div class="part-main">
      <div class="part-content">
        <div class="info-headline">
          登录
          <div class="info-headline-float">
            <span class="info-headline-text">
              还没有账号？免费
              <el-button type="text" class="info-headline-register">注册>></el-button>
            </span>
          </div>
        </div>
        <!--登录-->
        <el-form
          :label-position="labelPosition"
          label-width="80px"
          :model="form"
          ref="loginRef"
          type="text"
          class="content-form"
        >
          <el-form-item label="账号:" :rules="loginrules">
            <el-input v-model="form.username" placeholder="请输入账号"></el-input>
          </el-form-item>
          <el-form-item label="密码:">
            <el-input v-model="form.password" type="password" placeholder="5~15位字符"></el-input>
          </el-form-item>
          <el-form-item>
            <el-checkbox v-model="checked">记住账号密码</el-checkbox>
            <el-button type="text" class="el-button pull-right forget-pwd float--text">忘记密码</el-button>
          </el-form-item>
          <el-form-item>
            <div class="content-form-model">
              <el-button type="primary" class="content-form-btn" @click="login">登录</el-button>
            </div>
          </el-form-item>
        </el-form>
        <div class="service-tip-box">
          <p class="pwd-tip">*登录账号admin登录密码默认为123456</p>
          <p class="service-tip">有任何问题可以直接联系qq:1448933153</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "login",
  data() {
    return {
      form: {
        username: "",
        password: "",
      },
      labelPosition: "right",
      checked: true,
      loginrules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
          { min: 5, max: 15, message: "长度在 5 到 15 个字符", trigger: "blur" }
        ],
        password: [
          { required: true, message: "请输入密码", trigger: "blur" },
          { min: 5, max: 15, message: "长度在 5到 15 个字符", trigger: "blur" }
        ]
      }
    };
  },
  methods: {
    login() {
      this.$refs.loginRef.validate(valid => {
        if (!valid) return;
        this.$axios.post("login", this.form).then(res => {
          let res1 = res.data;
          if (res1.meta.status !== 200)
            return this.$message({
              message: "登陆失败！",
              type: "error"
            });
          this.$message({
            message: "登陆成功！",
            type: "success"
          });
          sessionStorage.setItem("token", res1.data.token);
          this.$router.push("/home");
        });
      });
    }
  },
};
</script>
<style>
.el-form .el-checkbox__input.is-checked + .el-checkbox__label {
  color: #606266;
}
</style>
<style lang="less" scoped>
.wrapper {
  margin: 0 auto;
}
.wrapper:after {
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: #f5f5f5;
  z-index: -1;
}
/*顶部*/
.part-header .part-header-main .hd-title {
  display: inline-block;
  margin-right: 20px;
  padding-right: 20px;
  border-right: 1px solid #fff;
  font-size: 30px;
  font-weight: 700;
  color: #fff;
  vertical-align: middle;
}
.part-header .part-header-main .hd-text {
  display: inline-block;
  font-size: 20px;
  color: #fff;
  vertical-align: middle;
}
.main-box {
  width: 400px;
  text-align: center;
}
/*注册*/
.part-header {
  position: relative;
  margin-bottom: 30px;
  padding: 25px 0;
  height: 90px;
  line-height: 40px;
  background: #3084c7;
  box-sizing: border-box;
}
.part-header .part-header-main {
  margin: 0 auto;
  padding: 0 15px;
  max-width: 780px;
}
.part-main {
  margin: 0 auto;
  max-width: 780px;
}
.part-main .part-content {
  border: 1px solid #ddd;
  background-color: #fff;
  border-radius: 2px;
  box-shadow: 0 4px 4px 0 rgba(0, 0, 0, 0.1);
}
.part-main .part-content .info-headline {
  margin-bottom: 40px;
  padding: 10px 20px;
  border-bottom: 1px solid #eee;
  line-height: 47px;
  font-size: 18px;
  font-weight: 700;
  background-color: #f5f5f5;
  box-sizing: border-box;
}
.part-main .part-content .info-headline .info-headline-float {
  padding-bottom: 5px;
  float: right;
}
.part-main .part-content .info-headline .info-headline-text {
  font-size: 16px;
  font-weight: 400;
  color: #555;
}
.part-main
  .part-content
  .info-headline
  .info-headline-text
  .info-headline-register {
  overflow: hidden;
  font-size: 16px;
}
/*登录*/
.content-form {
  width: 100%;
  max-width: 550px;
  margin: 0 auto 30px;
}
.el-form-item {
  margin-bottom: 22px;
}
label {
  display: inline-block;
  max-width: 100%;
  margin-bottom: 5px;
  font-weight: 700;
}
.content-form .forget-pwd {
  margin-right: 15px;
  color: #999;
  font-size: 14px;
}
.float--text {
  border: none;
  color: #20a0ff;
  background: 0 0;
  padding-left: 0;
  padding-right: 0;
  float: right;
}
.content-form-model {
  display: block;
  width: 100%;
  position: relative;
  margin: 20px 0;
}
.content-form-btn {
  overflow: hidden;
  display: block;
  width: 240px;
  border-radius: 4px;
  font-size: 14px;
  box-sizing: border-box;
  height: 40px;
  text-align: center;
  position: absolute;
  margin-left: 15%;
  left: 0;
  right: 0;
}
.service-tip-box,
.service-tip {
  font-size: 13px;
  color: #999;
  text-align: center;
  margin-bottom: 5px;
}
.service-tip-box .pwd-tip {
  font-size: 13px;
  color: #999;
  text-align: center;
  margin-bottom: 5px;
  margin-top: 100px;
}
.service-tip-box .service-tip {
  margin-bottom: 20px;
}
</style>
