<template>
  <el-container>
    <el-header>
      <el-page-header @back="goBack" content="作业详情" style="margin-top: 10px">
      </el-page-header>
    </el-header>

    <el-main style="padding-top: 10px;padding-left: 50px">
      <div>{{ previewList }}</div>
      <el-row :gutter="30" type="flex">
        <el-col :span="10" style="display: flex; flex-direction: column; height: 100%;">
          <div class="grid-content bg-purple">
            <span style="font-size: 20px;font-weight: bold">作业内容</span>
            <el-input :autosize="{ minRows: 20 }" type="textarea" :readonly="true" prefix="题目题目题目"
              v-model="workData.workDetail.publishContent" resize="none"></el-input>
          </div>
        </el-col>
        <el-col :span="10" style="display: flex; flex-direction: column; height: 100%;">
          <div class="grid-content bg-purple">
            <el-col><span style="font-size: 20px;font-weight: bold">答题区域</span></el-col>
            <el-input :autosize="{ minRows: 20 }" type="textarea" v-model="submitContent" resize="none"></el-input>
          </div>
        </el-col>
        <el-col :span="4" style="align-self: baseline;">
          <p><strong>上传图片</strong></p>
          <el-upload action="/api/postImages" :on-remove="handleRemove" list-type="picture" :limit="3"
            :on-success="handleUploadSuccess" :file-list="fileList" :data="uploadData"
            :on-preview="handlePicturePreview">
            <el-button size="medium" type="primary">点击上传</el-button>
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过10Mb</div>
            <el-image-viewer
              v-if="showImageViewer"
              :on-close="onCloseImageViewer"
              :url-list="previewList"
              :z-index="3000"
              :initial-index="initialIndex"
            />
          </el-upload>
        </el-col>
      </el-row>
    </el-main>
    <el-footer>
      <el-footer style="padding-right: 100px;text-align:right;">

        <el-col :span="1" :offset="1"><el-button type="primary" style="background-color: #545c64;"
            v-on:click="workSubmit" size="medium">提 交</el-button></el-col>
      </el-footer>
    </el-footer>
  </el-container>
</template>

<script>
import ElImageViewer from 'element-ui/packages/image/src/image-viewer'
export default {
  name: 'WorkDetail',
  components: {
    'el-image-viewer': ElImageViewer
  },
  data () {
    return {
      input: '',
      questionTextarea: '',
      submitContent: '',
      fileList: [],
      previewList: [],
      exsitedFiles: '',
      uploadData: {
        username: localStorage.getItem('name')
      },
      showImageViewer: false,
      initialIndex: 0,
      // 详情界面接收作业列表传过来的数据
      workData: this.$route.query.data,
      workId: this.$route.query.data.id,
      listenLoading: false
    }
  },
  // mounted，组件挂载后，此方法执行后，页面显示
  mounted: function () {
    this.judgeState()
  },
  methods: {
    goBack () {
      window.history.back()
      console.log('go back')
    },

    judgeState () {
      if (this.$route.query.data.state === '已提交') {
        this.submitContent = this.$route.query.data.submitContent
        let files = this.$route.query.data.submitFiles.split('|')
        this.exsitedFiles = '已使用图片:'
        this.fileList = []
        for (let file of files) {
          this.exsitedFiles += ' ' + file
          this.fileList.push({
            name: file,
            url: `/api/images/${this.uploadData.username}/thumb_${file}`
          })
          this.previewList.push(`/api/images/${this.uploadData.username}/${file}`)
        }
      }
    },
    // 提交作业
    workSubmit: function () {
      this.$confirm('确认提交吗？', '提示', {}).then(() => {
        this.listenLoading = true
        let fileList = []
        for (let file of this.fileList) {
          fileList.push(file.name)
        }
        this.$axios
          .post('/submitAnswer', {
            workId: this.workId,
            submitContent: this.submitContent,
            files: fileList
          }).then(resp => {
            if (resp.data === '') {
              this.$message({
                message: '提交失败',
                type: 'failure'
              })
              this.listenLoading = false
            }
            if (resp.data != null) {
              this.$message({
                message: '提交成功',
                type: 'success'
              })
              this.listenLoading = false
              // this.loadWorkInfo();
              this.$emit('onSubmit')
            }
          })
      })
    },
    handleUploadSuccess (response, file, fileList) {
      this.fileList = fileList
      this.exsitedFiles = '已使用图片:'
      for (let file of fileList) {
        this.exsitedFiles += ' ' + file.name
        this.previewList.push(file.url)
      }
    },
    beforeRemove (file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`)
    },
    handleRemove (file, fileList) {
      this.fileList = fileList
      this.exsitedFiles = '已使用图片:'
      for (let file of fileList) {
        this.exsitedFiles += ' ' + file.name
      }
    },
    handlePicturePreview (file) {
      let index = this.fileList.indexOf(file)
      if (index >= 0) {
        this.initialIndex = index
      }
      this.showImageViewer = true
    },
    onCloseImageViewer () {
      this.showImageViewer = false
    }
  }
}

</script>

<style scoped>
span {
  pointer-events: none;
}
</style>
