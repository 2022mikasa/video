<template>
  <div>
    <el-button type="primary" @click="dialogVisible = true" size="medium" round>编辑</el-button>
    <el-button v-if="form.role != '2'" type="danger" @click="deleteUser" size="medium" round>删除</el-button>
    <el-dialog title="编辑用户" :visible.sync="dialogVisible" width="30%">
      <el-form :model="form" label-width="100px">
        <el-form-item label="用户名">
          <el-input v-model="form.username" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="昵称">
          <el-input v-model="form.nickname"></el-input>
        </el-form-item>
        <el-form-item label="状态" v-if="form.role != 2">
          <el-radio-group v-model="form.role">
            <el-radio :label="0">普通用户</el-radio>
            <el-radio :label="1">管理员</el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <div slot="footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="updateUser">修改</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import axios from 'axios'
import { mapGetters } from 'vuex'
export default {
  props: {
    user: Object // 定义一个名为 user 的对象类型的属性
  },
  data() {
    return {
      dialogVisible: false,
      form: Object.assign({}, this.user)
    }
  },
  computed: {
    ...mapGetters([
      'name',
      'role'
    ])
  },
  methods: {
    updateUser() {
      // 发送请求更新用户
      console.log(this.form)
      axios.post('http://localhost:5000/user/update', this.form).then(response => {
        // 添加成功，关闭对话框并刷新用户列表
        this.dialogVisible = false
        this.$emit('updateSuccess')
      }).catch(error => {
        console.error(error)
      })
    },
    deleteUser() {
      if (window.confirm('您确定要删除这个用户吗？')) {
        axios.get('http://localhost:5000/user/delete?id=' + this.form.id).then(response => {
          this.$emit('deleteSuccess')
        }).catch(error => {
          console.error(error)
        })
      }
    }
  }
}
</script>
