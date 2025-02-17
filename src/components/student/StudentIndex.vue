<template>
    <div>
      <el-container>
        <el-header style="padding-top: 20px;padding-left: 50px;text-align:left;">
        <div >

          <el-input
            @keyup.enter.native="searchClick"
            placeholder="请输入发布教师、作业标题"
            prefix-icon="el-icon-search"
            size="medium"
            style="width: 400px;margin-right: 10px"
            v-model="keywords">
          </el-input>
          <el-button size="medium" type="primary" icon="el-icon-search"
                     style="background-color: #545c64" @click="searchClick">搜 索</el-button>

<!--          <el-input style="width: 300px" v-model="input" placeholder="请输入内容"></el-input>-->
<!--          <el-button type="primary" style="background-color: #545c64" v-on:click="getUsers">查询</el-button>-->

        </div>

        </el-header>

        <el-aside width="20px">
        </el-aside>
        <el-main style="padding-top: 10px;padding-left: 50px">
          <el-col :span="11">
            <h1 style="text-align: left;">待做作业</h1>
            <el-collapse v-model="activeName" accordion>
            <el-collapse-item v-for="(work, index) in to_do_works" :key="index">
              <template slot="title">
                <div class="work_title">
                  <span><strong>{{ work.workDetail.workTitle }}</strong></span>
                  <span :class="state-lable">{{ work.state }}</span>
                </div>
              </template>
              <el-row :gutter="20" justify="space-between">
                <el-col :span="12" style="text-align: left;">
                  <p><strong>教师：</strong>{{ work.workDetail.teacher.name }}<strong style="margin-left: 20px;">截止日期：</strong>{{ work.workDetail.endTime }}</p>
                </el-col>
                <el-col :span="12" style="text-align: right;">
                  <el-button @click="handleDetail(index, work)">提交作业</el-button>
                </el-col>
              </el-row>
              <div>
                <el-card class="work-content">
                  <div>{{ work.workDetail.publishContent }}</div>
                </el-card>
              </div>
            </el-collapse-item>
          </el-collapse>
          </el-col>
          <el-col :span="11" :offset="1">
            <h1 style="text-align: left;">已完成</h1>
            <el-collapse v-model="activeName" accordion>
            <el-collapse-item v-for="(work, index) in done_works" :key="index">
              <template slot="title">
                <div class="work_title">
                  <span><strong>{{ work.workDetail.workTitle }}</strong></span>
                  <span :class="state-lable">{{ work.state }}</span>
                </div>
              </template>
              <el-row :gutter="20" justify="space-between">
                <el-col :span="12" style="text-align: left;">
                  <p><strong>教师：</strong>{{ work.workDetail.teacher.name }}<strong style="margin-left: 20px;">截止日期：</strong>{{ work.workDetail.endTime }}</p>
                </el-col>
                <el-col :span="12" style="text-align: right;">
                  <el-button type="warning" @click="handleDetail(index, work)" v-if="work.state != '已完成'">提交作业</el-button>
                  <el-button v-else type="success" disabled>分数:{{ work.score }}</el-button>
                </el-col>
              </el-row>
              <div>
                <el-card class="work-content">
                  <div>{{ work.workDetail.publishContent }}</div>
                </el-card>
              </div>
            </el-collapse-item>
          </el-collapse>
          </el-col>
        </el-main>
      </el-container>

      <!--查看答案界面-->
      <el-dialog title="作业答案" :visible.sync="checkFormVisible" :append-to-body='true'>
        <el-form :model="checkForm" label-width="80px" :rules="checkFormRules" >

          <el-form-item label="作业内容" >
            <el-input
              type="textarea"
              v-model="checkForm.content"
              auto-complete="off"
              :readonly="true"
              :autosize="{ minRows: 8,maxRows:10}"></el-input>
          </el-form-item>
          <el-form-item label="参考答案">
            <el-input
              type="textarea"
              v-model="checkForm.answer"
              auto-complete="off"
              :readonly="true"
              :autosize="{ minRows: 8}"></el-input>
          </el-form-item>
        </el-form>
      </el-dialog>

    </div>
</template>

<script>

export default {
  name: 'StudentIndex',
  components: {
  },
  data () {
    return {
      input: '',
      account: localStorage.getItem('account'),
      works: [],
      to_do_works: [],
      done_works: [],
      keywords: '',
      searchResult: [],
      listenLoading: false,

      checkFormVisible: false, // 查看答案界面是否显示
      checkLoading: false,
      checkFormRules: {
      },
      // 查看答案界面数据
      checkForm: {
        // content:'作业内容作业内容作业内容作业内容作业内容作业内容作业内容作业内容' +
        //   '作业内容作业内容作业内容作业内容作业内容作业内容作业内容作业内容' +
        //   '作业内容作业内容作业内容作业内容作业内容作业内容作业内容作业内容' +
        //   '作业内容作业内容作业内容作业内容作业内容作业内容作业内容作业内容' +
        //   '作业内容作业内容作业内容作业内容作业内容作业内容作业内容作业内容' +
        //   '作业内容作业内容作业内容作业内容作业内容作业内容作业内容作业内容' +
        //   '作业内容作业内容作业内容作业内容作业内容作业内容作业内容作业内容' +
        //   '作业内容作业内容作业内容作业内容作业内容作业内容作业内容作业内容' +
        //   '作业内容作业内容作业内容作业内容作业内容作业内容作业内容作业内容' +
        //   '作业内容作业内容作业内容作业内容作业内容作业内容作业内容作业内容' +
        //   '作业内容作业内容作业内容作业内容作业内容作业内容作业内容作业内容',
        // answer:'答案内容答案内容答案内容答案内容答案内容答案内容答案内容答案内容',

      }

    }
  },

  // mounted，组件挂载后，此方法执行后，页面显示
  mounted: function () {
    this.loadWorkInfo()
  },

  methods: {
    // 请求加载学生作业信息
    loadWorkInfo () {
      let _this = this
      // this.$axios.get('/workInfo').then(resp => {
      //   if (resp && resp.status === 200) {
      //     _this.works = resp.data;
      //   }
      // });

      this.$axios
        .post('/getStudentPersonalWork', {
          keywords: this.account
        }).then(resp => {
          if (resp && resp.status === 200) {
            _this.works = resp.data
            let today = new Date()
            today.setHours(0, 0, 0, 0)
            for (let work of this.works) {
              let endDate = new Date(work.workDetail.endTime)
              if (endDate >= today) {
                console.log('bigger')
                if (work.state === '未提交') {
                  this.to_do_works.push(work)
                } else {
                  this.done_works.push(work)
                }
              }
            }
          }
        })
    },

    // 查询
    searchClick () {
      let _this = this
      this.$axios
        .post('/searchWork', {
          keywords: this.keywords
        }).then(resp => {
          if (resp && resp.status === 200) {
            _this.searchResult = resp.data

            let tempWorkList = []
            for (let i = 0; i < _this.searchResult.length; i++) {
              if (_this.searchResult[i].student.user.account === this.account) {
                tempWorkList.push(_this.searchResult[i])
              }
            }
            _this.works = tempWorkList
          }
        })
    },

    // 显示查看作业答案界面
    handleCheck: function (index, row) {
      this.checkFormVisible = true
      // this.checkForm = Object.assign({}, row);
      this.checkForm = {
        content: row.workDetail.publishContent,
        answer: row.workDetail.answer
      }
    },

    // 点击跳转到显示详情界面，传递参数过去，在详情界面需要接受
    handleDetail: function (index, row) {
      this.$router.push({
        path: '/workDetail',
        // name: 'mallList',
        query: {
          data: row
        }
      })
    }
  }
}
</script>

<style scoped>
.work_title{
  display: flex;
  justify-content: space-between;
  width: 50%;
}

.state-lable{
  display: inline-flex;
  align-items: center;
  color: red;
  width: 120px;
  text-align: right;
}
</style>
