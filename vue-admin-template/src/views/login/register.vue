<template>
  <div>
    <el-button type="primary" @click="dialogVisible = true">注册</el-button>
    <el-dialog title="注册" :visible.sync="dialogVisible" width="400px">
      <el-form ref="registerForm" :model="registerForm" :rules="registerRules" label-position="left" label-width="80px" class="register-form">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="registerForm.username" placeholder="请输入用户名" />
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="registerForm.password" type="password" placeholder="请输入密码" />
        </el-form-item>
        <el-form-item label="确认密码" prop="confirmPassword">
          <el-input v-model="registerForm.confirmPassword" type="password" placeholder="请再次输入密码" />
        </el-form-item>
        <div v-if="message !== ''" class="error-message">{{ message }}</div>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="submitForm('registerForm')">提交</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  data() {
    return {
      dialogVisible: false,
      registerForm: {
        username: '',
        password: '',
        confirmPassword: ''
      },
      registerRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 4, message: '密码长度不能少于4位', trigger: 'blur' }
        ],
        confirmPassword: [
          { required: true, message: '请确认密码', trigger: 'blur' },
          { validator: this.checkConfirmPassword, trigger: 'blur' }
        ]
      },
      message: ''
    }
  },
  methods: {
    checkConfirmPassword(rule, value, callback) {
      if (value === '') {
        callback(new Error('请再次输入密码'))
      } else if (value !== this.registerForm.password) {
        callback(new Error('两次输入的密码不一致'))
      } else {
        callback()
      }
    },
    submitForm(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          console.log('发送注册请求')
          axios.post('http://localhost:5000/register', this.registerForm).then(response => {
            if (response.data.code === 200) {
              this.message = response.data.message
              this.username = ''
              this.password = ''
              this.confirmPassword = ''
              this.dialogVisible = false
            } else {
              this.message = response.data.message
            }
            console.log(response)
          }).catch(error => {
            console.error(error)
          })
        } else {
          return false
        }
      })
    }
  }
}
</script>

<style scoped>
.register-form {
  margin-top: 20px;
}
.el-form-item__label {
  font-weight: bold;
  color: #606266;
}
.error-message {
  margin-top: 10px;
  color: #f56c6c;
}
.dialog-footer {
  padding: 10px 20px;
  text-align: right;
}
</style>
