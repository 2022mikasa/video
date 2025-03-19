<template>
  <div>
    <el-form :model="ruleForm" ref="ruleForm" label-width="120px" class="demo-ruleForm">
      <el-form-item label="标题" required>
        <el-input v-model="ruleForm.title"></el-input>
      </el-form-item>
      <el-form-item label="描述" required>
        <el-input type="textarea" v-model="ruleForm.summary"></el-input>
      </el-form-item>
      <el-form-item label="添加视频" class="video-upload" required>
        <el-upload
          class="avatar-uploader"
          action=""
          accept='.mp4,.qlv,.qsv,.ogg,.flv,.avi,.wmv,.rmvb'
          :auto-upload="false"
          :show-file-list="false"
          id = 'video'
          :on-change="handleChangeVideo">
          <video
            v-if="Video !=''"
            :src="Video"
            width="350"
            height="180"
            controls="controls"
          >您的浏览器不支持视频播放
          </video>
          <i
            v-else-if="Video ==''"
            class="el-icon-plus avatar-uploader-icon"
          ></i>            <!--没选择视频之前显示-->
        </el-upload>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="checkSubmit('ruleForm')" :disabled="!canSubmit">提交</el-button>
        </el-form-item>
      </el-form>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      ruleForm: {
        title: '',
        summary: '',
        file: ''
      },
      Video: ''
    }
  },
  computed: {
    canSubmit() {
      return this.ruleForm.title && this.ruleForm.summary && this.ruleForm.file
    }
  },
  methods: {
    handleChangeVideo(file) {
      this.Video = URL.createObjectURL(file.raw)
      console.log(file)
      this.ruleForm.file = file.raw
    },
    checkSubmit() {
      const formData = new FormData()
      Object.keys(this.ruleForm).forEach((ele) => {
        formData.append(ele, this.ruleForm[ele])
      })
      axios.post('http://localhost:5000/video/add', formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then(response => {
        // 处理上传成功的逻辑
        this.$router.push({ path: this.redirect || '/video/videolist' })
      }).catch(error => {
        // 处理上传失败的逻辑
        console.error(error)
      })
    }
  }
}
</script>

<style scoped>
.video-upload {
  align-items: center;
}

.file-input {
  display: none;
}

.upload-btn {
  width: 100%;
}
</style>
