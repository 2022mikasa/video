<template>
  <div class="video-analysis">
    <el-row>
      <el-col :span="18">
        <video :src="video.video_url" controls autoplay style="width: 100%; height: auto;" />
      </el-col>
      <el-col :span="6" class="right-column">
        <el-row>
          <el-col :span="24">
            <h4>视频信息</h4>
            <p>标题：{{ video.title }}</p>
            <p>时长：{{ formatTimeSecond(getProp(video, 'video_length', 0).toFixed(0)) }}秒</p>
            <p>大小：{{ getProp(video, 'video_size', 0).toFixed(2) }}MB</p>
            <p>上传时间：{{ formatDate(video.create_time) }}</p>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="24">
            <h4>选择算法</h4>
            <el-select v-model="selectedAlgorithm" placeholder="请选择算法">
              <el-option v-for="algorithm in algorithms" :key="algorithm.value" :label="algorithm.label" :value="algorithm.value" />
            </el-select>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="24">
            <h4>选择分析方式</h4>
            <el-radio-group v-model="selectedParsingMethod" style="display: flex; flex-direction: column;">
              <el-radio label="method1" style="margin-top: 0px;">简单分析</el-radio>
              <el-radio label="method2" style="margin-top: 10px;">完全分析</el-radio>
            </el-radio-group>
          </el-col>
        </el-row>
        <p>保存视频帧率：<el-input-number v-model="frameRate" :min="0" :max="120" :step="5" /> fps<br>(默认为0表示以原视频为准)</p>
        <p>置信度：{{ confidenceLevel }}% <el-slider v-model="confidenceLevel" :min="10" :max="90" /></p>
        <el-row>
          <el-col :span="24">
            <br>
            <el-button type="primary" @click="startAnalysis(0)">分析</el-button>
            <el-button type="primary" @click="startAnalysis(1)">分析并保存结果</el-button>
          </el-col>
        </el-row>
        <!-- <el-row v-if="showAdvancedSettings">
          <el-col :span="24">
            <h2>高级设置</h2>
            <p>帧率：<el-input-number v-model="frameRate" :min="1" :max="120" step="1" /> fps</p>
            <p>置信度：<el-slider v-model="confidenceLevel" :min="0" :max="100" /> {{ confidenceLevel }}%</p>
          </el-col>
        </el-row> -->
      </el-col>
    </el-row>
    <el-dialog title="分析结果" :visible.sync="showAnalyseResult" width="80%">
      <span>请耐心等待......</span>
      <div>
        <img :src="image1" style="width: 50%; height: auto;">
        <img :src="image2" style="width: 50%; height: auto;">
      </div>
    </el-dialog>
  </div>
</template>

<script>
import axios from 'axios'
import io from 'socket.io-client'

export default {
  data() {
    return {
      video: {},
      algorithms: [
        { label: '物体识别', value: 'algorithm1' },
        { label: '算法2', value: 'algorithm2' },
        { label: '算法3', value: 'algorithm3' }
      ],
      selectedAlgorithm: 'algorithm1',
      selectedParsingMethod: 'method1',
      image1: '',
      image2: '',
      frameRate: 0,
      confidenceLevel: 50,
      showAnalyseResult: false
    }
  },
  created() {
    this.getVideo()
  },
  mounted() {
    console.log('mounted() called.')
    const socket = io.connect('http://localhost:5000')
    console.log(socket)
    // 接收从Flask发送过来的字符串，并将其转换成图像
    socket.on('image', (data) => {
      // console.log('Received image from server.')
      this.image1 = 'data:image/jpeg;base64,' + data.img1
      this.image2 = 'data:image/jpeg;base64,' + data.img2
    })
  },
  methods: {
    getVideo() {
      const id = this.$route.params.id
      axios.get(`http://localhost:5000/video/${id}`).then(response => {
        console.log(response.data)
        this.video = response.data.data[0]
        console.log(this.video[0])
      }).catch(error => {
        console.error(error)
      })
    },
    getProp(obj, prop, defaultVal = '') {
      return obj && Object.prototype.hasOwnProperty.call(obj, prop) ? obj[prop] : defaultVal
    },
    formatDate(dateString) {
      const date = new Date(dateString)
      const year = date.getFullYear()
      const month = (date.getMonth() + 1).toString().padStart(2, '0')
      const day = date.getDate().toString().padStart(2, '0')
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
    startAnalysis(choose) {
      this.showAnalyseResult = true
      const formData = new FormData()
      formData.append('video_url', this.video.video_url)
      formData.append('path', this.video.path)
      formData.append('title', this.video.title)
      formData.append('method', this.selectedParsingMethod)
      formData.append('fps', this.frameRate)
      formData.append('confidence', this.confidenceLevel * 0.01)
      formData.append('save', choose)
      console.log(formData)
      axios.post('http://localhost:5000/video/analyse', formData)
        .then(response => {
          // 接收从 Flask 发送过来的字符串，并将其转换成图像
          // this.image = 'data:image/jpeg;base64,' + response.data
        })
        .catch(error => {
          console.log(error)
        })
    }
  }
}
</script>

<style>
.video-analysis {
  padding: 20px;
}

.right-column {
  padding-left: 20px;
}

</style>
