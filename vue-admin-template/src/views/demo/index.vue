<template>
  <div class="login-container">
    <el-card class="login-box">
      <h3 class="title">Login</h3>
      <el-form ref="loginForm" :model="loginForm" :rules="loginRules" label-width="80px" class="login-form">
        <el-form-item label="Username" prop="username">
          <el-input v-model="loginForm.username" placeholder="Please enter your username"></el-input>
        </el-form-item>
        <el-form-item label="Password" prop="password">
          <el-input v-model="loginForm.password" type="password" placeholder="Please enter your password"></el-input>
        </el-form-item>
        <el-form-item class="remember-me">
          <el-checkbox v-model="loginForm.rememberMe">Remember me</el-checkbox>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="login">Login</el-button>
        </el-form-item>
      </el-form>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      loginForm: {
        username: '',
        password: '',
        rememberMe: false
      },
      loginRules: {
        username: [
          { required: true, message: 'Please enter your username', trigger: 'blur' }
        ],
        password: [
          { required: true, message: 'Please enter your password', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    login() {
      // 在这里处理登录逻辑
      // ...
    }
  }
}
</script>

<style scoped>
.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f5f7fa;
}
.login-box {
  width: 400px;
  padding: 40px;
  border-radius: 5px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
}
.title {
  font-size: 24px;
  font-weight: 600;
  text-align: center;
  margin-bottom: 30px;
}
.login-form {
  margin-top: 20px;
}
.remember-me {
  text-align: right;
  margin-bottom: 20px;
}
</style>