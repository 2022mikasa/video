<template>
  <div>
    <div class="table-header">
      <el-row>
        <el-col :span="6">
          <el-input v-model="keyword" placeholder="请输入搜索关键词" prefix-icon="el-icon-search"></el-input>
        </el-col>
      </el-row>
    </div>
    <el-table :data="filteredData" style="width: 100%" border>
      <el-table-column prop="id" label="ID" sortable></el-table-column>
      <el-table-column prop="username" label="用户名" sortable></el-table-column>
      <el-table-column prop="nickname" label="昵称" sortable></el-table-column>
      <el-table-column prop="status" label="状态">
        <template slot-scope="scope">
          <el-tag :type="getTagType(scope.row.role)">
            {{ getTagText(scope.row.role) }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="create_time" label="创建时间" sortable></el-table-column>
      <el-table-column prop="update_time" label="更新时间" sortable></el-table-column>
      <el-table-column label="操作" v-if="role == 'SuperAdmin'">
        <!-- <el-button type="warning">修改</el-button> -->
        <template slot-scope="scope">
          <update-user :user=scope.row @updateSuccess="getData" @deleteSuccess="getData"></update-user>
        </template>
        <!-- <el-button type="danger">删除</el-button> -->
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[2, 5, 10, 20, 50]"
      :page-size="pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
  </div>
</template>

<script>
import axios from 'axios'
import UpdateUser from '@/views/updateuser/index.vue'
import { mapGetters } from 'vuex'

export default {
  components: {
    UpdateUser
  },
  data() {
    return {
      keyword: '',
      userData: [], // 用户数据
      currentPage: 1, // 当前页码
      pageSize: 5, // 每页显示的条数
      total: 0 // 数据总数
    }
  },
  computed: {
    ...mapGetters([
      'name',
      'role'
    ]),
    filteredData() {
      return this.userData.filter(item => {
        return item.username.includes(this.keyword) ||
        item.nickname.toString().includes(this.keyword) ||
        item.create_time.includes(this.keyword) ||
        item.update_time.includes(this.keyword)
      })
    }
  },
  mounted() {
    // 初始化数据
    this.getData()
  },
  methods: {
    // 获取用户数据
    getData() {
      axios.get('http://localhost:5000/user/list', {
        params: {
          page: this.currentPage,
          pageSize: this.pageSize
        }
      }).then(res => {
        this.userData = res.data.data
        this.total = res.data.total
        console.log('get info')
        // console.log(this.userData)
      }).catch(err => {
        console.log(err)
      })
    },
    // 每页显示条数变化时调用
    handleSizeChange(val) {
      this.pageSize = val
      this.getData()
    },
    // 当前页码变化时调用
    handleCurrentChange(val) {
      this.currentPage = val
      this.getData()
    },
    getTagType(role) {
      if (role === 0) {
        return 'info'
      } else if (role === 1) {
        return 'success'
      } else if (role === 2) {
        return 'danger'
      } else {
        return 'danger'
      }
    },
    getTagText(role) {
      if (role === 1) {
        return '管理员'
      } else if (role === 2) {
        return '超级管理员'
      } else {
        return '用户'
      }
    }
  }
}
</script>

<style scoped>
.table-header {
  /* display: flex; */
  /* justify-content: space-between; */
  align-items: center;
  /* margin-bottom: 20px; */
}
</style>
