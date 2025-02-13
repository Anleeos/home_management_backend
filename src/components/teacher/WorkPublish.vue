<template>
  <el-container>
    <el-main

      style="padding-top: 30px;padding-left: 200px ;  " >
      <el-row gutter="1px">
        <el-col :span="10"><div class="grid-content bg-purple">
          <span style="font-size: 20px;font-weight: bold">作业内容</span>
            <el-input
              type="textarea"
              :rows="30"
              placeholder="请输入作业内容..."
              v-model="textarea"
              style="margin-top: 10pt;align-self: center;">
            </el-input>
        </div></el-col>
        <el-col :span="10">
          <div >
            <div style="margin-top:40px;">
              <el-input
                @keyup.enter.native="searchClick"
                placeholder="请输入作业标题..."

                size="primary"
                style="width: 400px;margin-left: -10px"
                v-model="workTitle">
              </el-input>
            </div>

            <div  style="margin-top:40px;">
            <el-select v-model="classOptionsValue"
                       placeholder="选择发布班级"
                       style="width: 400px;margin-left: -10px"
                       @change="classOptionValueChange">
              <el-option
                v-for="item in classOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
            </div>

            <div style="margin-top:40px;">

                <el-date-picker
                  v-model="endTimePicker"
                  type="datetime"
                  placeholder="选择截止时间"
                  style="width: 400px;margin-left: -10px"
                  value-format="yyyy-MM-dd HH:mm:ss">
                </el-date-picker>
            </div>

            <div style="margin-top:40px;margin-left: -10px">
              <el-button type="primary" style="background-color: #545c64;" @click.native="handlePublish" :loading="listenLoading">发 布</el-button>

            </div>
          </div>
        </el-col>
      </el-row>
    </el-main>
    <el-footer
      align="center"
      style="padding-right: 300px;text-align:right;">

<!--        <div class="block">-->
<!--          <el-input v-model="input" placeholder="请输入作业标题"></el-input>-->
<!--          <el-select v-model="value" placeholder="选择发布班级">-->
<!--            <el-option-->
<!--              v-for="item in options"-->
<!--              :key="item.value"-->
<!--              :label="item.label"-->
<!--              :value="item.value">-->
<!--            </el-option>-->
<!--          </el-select>-->
<!--          -->
<!--          <el-date-picker-->
<!--            v-model="endTime_picker"-->
<!--            type="datetime"-->
<!--            placeholder="选择截止时间">-->
<!--          </el-date-picker>-->
<!--          <el-button type="primary" style="background-color: #545c64;" v-on:click="handlePublish">发 布</el-button>-->
<!--        </div>-->

<!--      <div >-->

<!--        <el-input-->
<!--          @keyup.enter.native="searchClick"-->
<!--          placeholder="请输入作业标题..."-->

<!--          size="primary"-->
<!--          style="width: 300px;margin-right: 10px"-->
<!--          v-model="workTitle">-->
<!--        </el-input>-->
<!--                  <el-select v-model="classOptionsValue" placeholder="选择发布班级" @change="classOptionValueChange">-->
<!--                    <el-option-->
<!--                      v-for="item in classOptions"-->
<!--                      :key="item.value"-->
<!--                      :label="item.label"-->
<!--                      :value="item.value">-->
<!--                    </el-option>-->
<!--                  </el-select>-->

<!--                  <el-date-picker-->
<!--                    v-model="endTimePicker"-->
<!--                    type="datetime"-->
<!--                    placeholder="选择截止时间"-->
<!--                    value-format="yyyy-MM-dd HH:mm:ss">-->
<!--                  </el-date-picker>-->
<!--                  <el-button type="primary" style="background-color: #545c64;" @click.native="handlePublish" :loading="listenLoading">发 布</el-button>-->

<!--      </div>-->

    </el-footer>
  </el-container>
</template>

<script>
export default {
  name: 'WorkPublish',
  data () {
    return {
      account: localStorage.getItem('account'),
      teacherId: '',
      workTitle: '',
      questionTextarea: '', // 题目文本
      answerTextarea: '', // 答案文本

      endTimePicker: '', // 截止时间
      classes: [],
      classOptions: [],
      classOptionsValue: '', // 选中选择之后的值

      listenLoading: false,
      textarea: '',

      worksDetail: {
        id: '',
        workTitle: '',
        publishContent: '',
        answer: ''
      },

      Work: {
        id: '',
        workDetail: '',
        teacherId: '',
        classId: '',
        endTime: ''
      }

    }
  },
  mounted: function () {
    this.loadBankInfo()
  },
  methods: {
    // 请求加载题库信息
    loadBankInfo () {
      let user2 = {
        account: this.account
      }
      this.$axios
        .post('/getTeacher', {
          user: user2
        }).then(resp => {
          if (resp && resp.status === 200) {
            this.teacherId = resp.data.id
          }
        })
      // 请求加载班级信息
      let toClasses = `/teacherClasses/${this.account}`
      this.$axios.get(toClasses).then(resp => {
        if (resp && resp.status === 200) {
          this.classes = resp.data

          let tempClasses = []
          for (let i = 0; i < this.classes.length; ++i) {
            let tempClassOption = {
              value: '',
              label: ''
            }
            // tempCourseOption.value = "选项"+ii;
            tempClassOption.value = this.classes[i].id
            tempClassOption.label = this.classes[i].className
            tempClasses.push(tempClassOption)
          }
          this.classOptions = tempClasses
        }
      })
    },

    classOptionValueChange (value) {
      this.classOptionsValue = value
    },

    handleWork: function () {
      this.worksDetail.publishContent = this.textarea
      this.worksDetail.answer = ''
    },

    // 处理发布作业
    handlePublish: function () {
      this.handleWork()

      this.$confirm('确认发布?', '提示', {
        type: 'warning'
      }).then(() => {
        this.listenLoading = true
        this.$axios
          .post('/publishWork', {
            id: '',
            worksDetailId: '',
            worksDetailWorkTitle: this.workTitle,
            worksDetailPublishContent: this.worksDetail.publishContent,
            worksDetailAnswer: this.worksDetail.answer,
            teacherId: this.teacherId,
            classId: this.classOptionsValue,
            endTime: this.endTimePicker
          }).then(resp => {
            if (resp && resp.data.code === 100) {
              this.listenLoading = false
              this.$message({
                message: '发布成功',
                type: 'success'
              })
              this.loadBankInfo()
              this.workTitle = ''
              this.classOptionsValue = ''
              this.endTimePicker = ''
            } else {
              this.$message({
                message: '该班级没有学生，发布失败',
                type: 'failure'
              })
              this.listenLoading = false
            }
          })
      }
      ).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消发布'
        })
      })
    }

  }
}
</script>

<style scoped>

</style>
