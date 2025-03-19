<template>
  <div class="video-list">
    <el-row>
      <el-col :span="6">
        <el-input v-model="keyword" placeholder="请输入搜索关键词" prefix-icon="el-icon-search"></el-input>
      </el-col>
    </el-row>
    <el-table :data="filteredData" style="width: 100%" border>
      <el-table-column prop="title" label="标题" sortable />
      <!-- <el-table-column prop="summary" label="描述"></el-table-column> -->
      <el-table-column prop="create_time" label="上传时间" sortable>
        <template slot-scope="scope">{{ formatDate(scope.row.create_time) }}</template>
      </el-table-column>
      <el-table-column prop="update_time" label="更新时间" sortable>
        <template slot-scope="scope">{{ formatDate(scope.row.update_time) }}</template>
      </el-table-column>
      <el-table-column prop="video_length" label="时长" sortable>
        <template slot-scope="scope">
          <p>{{ formatTimeSecond(getProp(scope.row, 'video_length', 0).toFixed(0)) }}</p>
        </template>
      </el-table-column>
      <el-table-column prop="video_size" label="大小" sortable>
        <template slot-scope="scope">
          <p>{{ getProp(scope.row, 'video_size', 0).toFixed(0) }}MB</p>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button type="primary" size="medium" round @click="showVideo(scope.row)">详情</el-button>
          <!-- <el-button type="primary" @click="dialogVisible = true" size="medium" round>详情</el-button> -->
          <el-button type="danger" size="medium" round @click="deleteVideo(scope.row.id)" v-if="role != 'User'">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[5, 10, 20, 50]"
      :page-size="pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    <el-dialog title="视频详情" :visible.sync="dialogVisible" width="70%" height="70%" @close="closeDialog">
      <p><b class="video-title">{{ selectedVideo.title }}</b></p>
      <p><i class="el-icon-time" />{{ formatDate(selectedVideo.create_time) }}</p>
      <div class="video-player">
        <video :src="selectedVideo.video_url" controls autoplay :volume="0.5" style="width: 100%; height: auto;" />
      </div>
      <div class="video-info">
        <p><b>描述：</b>{{ selectedVideo.summary }}</p>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">关闭</el-button>
        <router-link :to="{ name: 'analyse', params: { id: selectedVideo.id } }">
          <!-- <el-button @click="dialogVisible = false">分析</el-button> -->
          <el-button>分析</el-button>
        </router-link>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import axios from 'axios'
import { mapGetters } from 'vuex'
export default {
  data() {
    return {
      keyword: '',
      videos: [],
      dialogVisible: false,
      selectedVideo: {},
      currentPage: 1, // 当前页码
      pageSize: 5, // 每页显示的条数
      total: 0 // 数据总数
    }
  },
  computed: {
    filteredData() {
      return this.videos.filter(item => {
        return item.title.includes(this.keyword) ||
        item.create_time.includes(this.keyword) ||
        item.update_time.includes(this.keyword)
      })
    },
    ...mapGetters([
      'name',
      'role'
    ])
  },
  watch: {
    dialogVisible(newVal) {
      if (!newVal) {
        const videoPlayer = document.querySelector('.video-player video')
        if (videoPlayer) {
          videoPlayer.currentTime = 0
        }
      }
    }
  },
  mounted() {
    this.fetchVideos()
  },
  methods: {
    showVideo(video) {
      this.selectedVideo = video
      this.dialogVisible = true
    },
    fetchVideos() {
      axios.get('http://localhost:5000/video/list', {
        params: {
          page: this.currentPage,
          pageSize: this.pageSize
        }
      }).then(response => {
        this.videos = response.data.data
        this.total = response.data.total
        console.log(this.videos)
      }).catch(error => {
        console.error(error)
      })
    },
    deleteVideo(id) {
      if (window.confirm('您确定要删除这个视频吗？')) {
        axios.get('http://localhost:5000/video/delete?id=' + id).then(response => {
          this.$emit('deleteSuccess')
          this.fetchVideos()
        }).catch(error => {
          console.error(error)
        })
      }
    },
    // 每页显示条数变化时调用
    handleSizeChange(val) {
      this.pageSize = val
      this.fetchVideos()
    },
    // 当前页码变化时调用
    handleCurrentChange(val) {
      this.currentPage = val
      this.fetchVideos()
    },
    getProp(obj, prop, defaultVal = '') {
      return obj && Object.prototype.hasOwnProperty.call(obj, prop) ? obj[prop] : defaultVal
    },
    formatDate(dateString) {
      const date = new Date(dateString)
      const year = date.getFullYear()
      const month = (date.getMonth() + 1).toString().padStart(2, '0')
      const day = date.getDate().toString().padStart(2, '0')
      // const hours = date.getHours().toString().padStart(2, '0')
      // const minutes = date.getMinutes().toString().padStart(2, '0')
      return `${year}-${month}-${day}`
    },
    formatTimeSecond(seconds) {
      const hours = Math.floor(seconds / 3600)
      const minutes = Math.floor((seconds % 3600) / 60)
      const secondsRemainder = seconds % 60
      const formattedTime = [hours, minutes, secondsRemainder]
        .map((val) => val.toString().padStart(2, '0'))
        .join(':')
      return formattedTime
    },
    handleCloseDialog(done) {
      // 停止视频播放
      this.$refs.video.pause()
      // 执行关闭 Dialog 的操作
      done()
    },
    closeDialog() {
      const videoPlayer = document.querySelector('.video-player video')
      if (videoPlayer) {
        videoPlayer.pause()
        videoPlayer.removeAttribute('src')
      }
      this.selectedVideo = {}
    }
  }
}
</script>

<style scoped>
.video-title {
  font-size: 36px;
}
</style>
