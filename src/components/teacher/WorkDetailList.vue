<template>
  <el-container>
    <el-header>
      <el-page-header @back="goBack" content="作业详情列表" style="margin-top: 10px">
      </el-page-header>
    </el-header>

    <el-main style="margin-left: auto; margin-right: auto;">
        <el-table
          :data="works"
          style="width: 1400px"
          height="450">
          <el-table-column
            fixed
            prop="student.id"
            label="序号"
            width="200" >
          </el-table-column>
          <el-table-column
            fit="true"
            prop="student.user.account"
            label="学号"
            width="200" >
          </el-table-column>
          <el-table-column
            fit="true"
            prop="student.name"
            label="学生姓名"
            width="200" >
          </el-table-column>
          <el-table-column
            fit="true"
            prop="score"
            label="分数"
            width="200" >
          </el-table-column>
          <el-table-column  fixed="right" label="操作" min-width="200" >

            <template slot-scope="scope">
              <el-button
                size="small"
                @click="handleCorrect(scope.$index, scope.row)">批改</el-button>
            </template>
          </el-table-column>
        </el-table>

    </el-main>
    <!--作业批改界面-->
    <el-dialog title="作业详情" :visible.sync="correctFormVisible" :append-to-body='true'>
      <el-form :model="correctForm" label-width="80px" :rules="correctFormRules" ref="correctForm">
        <el-form-item label="作业内容" >
          <el-input
            type="textarea"
            v-model="correctForm.content"
            auto-complete="off"
            :readonly="true"
            :autosize="{ minRows: 8,maxRows:10}"></el-input>
        </el-form-item>
        <el-form-item label="学生答案">
          <el-input
            type="textarea"
            v-model="correctForm.submit_content"
            auto-complete="off"
            readonly="true"
            :autosize="{ minRows: 8,maxRows:10}"></el-input>
        </el-form-item>
        <el-form-item label="作业图片" >
          <el-image style="width: 100px; height: 100px" :src="correctForm.thumbImage" :preview-src-list="correctForm.submitFiles"></el-image>
        </el-form-item>
        <el-form-item label="分数" prop="score">
          <el-input v-model="correctForm.score" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="correctFormVisible = false">取 消</el-button>
        <el-button type="primary" style="background-color: #545c64" @click.native="correctSubmit" :loading="listenLoading">保存</el-button>
      </div>
    </el-dialog>
  </el-container>

</template>

<script>
export default {
  name: 'WorkDetailList',
  data () {
    return {
      account: localStorage.getItem('account'),
      query: this.$route.query,
      // 详情界面接收作业列表传过来的数据
      workDetailId: this.$route.query.data.id,
      workDetail: this.$route.query.data,
      works: [], // 个人账号里面发布的所有作业
      input: '',
      url: 'https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg',
      srcList: [
        'https://fuss10.elemecdn.com/8/27/f01c15bb73e1ef3793e64e6b7bbccjpeg.jpeg'
      ],
      correctFormVisible: false, // 新增界面是否显示
      correctLoading: false,
      correctFormRules: {
      },
      // 新增界面数据
      correctForm: {
        workTitle: '未定义',
        workId: 0,
        content: '',
        submit_content: '',
        score: '',
        submitFiles: [],
        thumbImage: ''
      },
      listenLoading: false
    }
  },
  // mounted，组件挂载后，此方法执行后，页面显示
  mounted: function () {
    this.loadWorkInfo()
  },
  methods: {
    // 请求加载学生作业信息
    loadWorkInfo () {
      this.$axios
        .post('/getWorks', {
          keywords: this.workDetailId
        }).then(resp => {
          if (resp && resp.status === 200) {
            this.works = resp.data

            let tempWorkList = []
            for (let i = 0; i < this.works.length; i++) {
              if (this.workDetailId === this.works[i].workDetail.id) {
                tempWorkList.push(this.works[i])
              }
            }
            this.works = tempWorkList
          }
        })
    },

    // 详细界面返回按钮
    goBack () {
      window.history.back()
      console.log('go back')
    },

    // 显示批改界面
    handleCorrect: function (index, row) {
      this.correctFormVisible = true
      let images = row.submitFiles.split('|')
      let urls = []
      for (let imageFile of images) {
        urls.push(`/api/images/${row.student.name}/${imageFile}`)
      }
      let thumb = ''
      if (images.length >= 1) {
        thumb = `/api/images/${row.student.name}/thumb_${images[0]}`
      }
      this.correctForm = {
        workTitle: row.title,
        workId: row.id,
        content: row.workDetail.publishContent,
        submit_content: row.submitContent,
        score: row.score,
        submitFiles: urls,
        thumbImage: thumb
      }
    },

    // 批改
    correctSubmit: function () {
      this.$refs.correctForm.validate((valid) => {
        if (valid) {
          this.$confirm('确认提交吗？', '提示', {}).then(() => {
            this.listenLoading = true

            this.$axios
              .post('/checkWork', {

                workId: this.correctForm.workId,
                score: this.correctForm.score
              }).then(resp => {
                if (resp.data === '') {
                  this.$message({
                    message: '批改失败',
                    type: 'failure'
                  })
                  this.listenLoading = false
                  this.correctForm = false
                }
                if (resp.data != null) {
                  this.$message({
                    message: '批改成功',
                    type: 'success'
                  })
                  this.listenLoading = false
                  this.correctFormVisible = false
                  this.loadWorkInfo()
                  this.$emit('onSubmit')
                }
              })
          })
        }
      })
    }
  }
}

</script>

<style scoped>

</style>
