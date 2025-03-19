<template>
  <div>
    <el-row>
      <el-button type="primary" @click="startAnalysis">开始分析</el-button>
      <el-button type="primary" @click="stopAnalysis">停止分析</el-button>
    </el-row>
    <el-row>
      <img :src="image1" style="width: 50%; height: auto;">
      <img :src="image2" style="width: 50%; height: auto;">
    </el-row>
  </div>
</template>

<script>
import axios from 'axios'
import io from 'socket.io-client'

export default {
  data() {
    return {
      image1: '',
      image2: ''
    }
  },
  mounted() {
    const socket = io.connect('http://localhost:5000')
    // 接收从Flask发送过来的字符串，并将其转换成图像
    socket.on('camera_image', (data) => {
      // console.log('Received image from server.')
      this.image1 = 'data:image/jpeg;base64,' + data.img1
      this.image2 = 'data:image/jpeg;base64,' + data.img2
    })
  },
  methods: {
    startAnalysis() {
      axios.get('http://localhost:5000/camera/analyse').then(response => {
        // 接收从 Flask 发送过来的字符串，并将其转换成图像
        // this.image = 'data:image/jpeg;base64,' + response.data
      }).catch(error => {
        console.log(error)
      })
    },
    stopAnalysis() {
      axios.get('http://localhost:5000/camera/stop').then(response => {
        // 接收从 Flask 发送过来的字符串，并将其转换成图像
        // this.image = 'data:image/jpeg;base64,' + response.data
        this.image1 = ''
        this.image2 = ''
      }).catch(error => {
        console.log(error)
      })
    }
  }
}
</script>
