<template>
  <div>
    <el-container>
      <el-header style="padding-top: 20px;padding-left: 50px;text-align:left;">
        <div >
          <el-button size="medium" type="danger" icon="el-icon-delete"
            @click="handleDeleteMultiple()"
            :disabled="multipleSelection.length === 0">
                     删 除
          </el-button>
        </div>

      </el-header>
      <!-- <div>{{ multipleSelection }}</div> -->
      <el-aside width="20px">
      </el-aside>
      <el-main style="padding-top: 10px;padding-left: 50px">
        <el-table
          :data="works"
          style="width: 100%"
          height="600"
          :default-sort = "{prop: 'startTime', order: 'descending'}"
          @selection-change="handleSelectionChange">
          <el-table-column
          type="selection"
            width="55">
          </el-table-column>
          <el-table-column
            fixed
            prop="id"
            label="序号"
            width="100"
            sortable >
          </el-table-column>
          <el-table-column
            fit="true"
            prop="workDetail.workTitle"
            label="作业标题"
            width="200">
          </el-table-column>
          <el-table-column
            fit="true"
            prop="student.mClass.className"
            label="发布班级"
            width="200">
          </el-table-column>
          <el-table-column
            fit="true"
            prop="startTime"
            label="发布时间"
            width="200"
            sortable>
          </el-table-column>

          <el-table-column
            fit="true"
            prop="endTime"
            label="截止时间"
            width="200"
            sortable>
          </el-table-column>
          <el-table-column
            fit="true"
            prop="process"
            label="完成进度"
            width="200">
            <el-progress :width="45" :height="45" type="circle" :percentage="processData.process" ></el-progress>
          </el-table-column>
          <el-table-column  fixed="right" label="操作" min-width="200" >

            <template slot-scope="scope">
              <el-button
                size="small"
                @click.native="handleDetail(scope.$index, scope.row)">查看</el-button>
              <el-button
                size="small"
                type="danger"
                @click.native="handleDelete(scope.$index, scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-main>
      <el-footer>
        <el-pagination
          background
          layout="prev, pager, next"
          @current-change="handleCurrentChange"
          :page-size="7"
          style="padding-top: 20px; padding-right: 50px; text-align:right;"
          :total="50">
        </el-pagination>
      </el-footer>
    </el-container>

  </div>
</template>

<script>

export default {
  name: 'TeacherWork',
  components: {},
  data () {
    return {
      input: '',
      works: '',
      keywords: '',
      multipleSelection: [],
      account: localStorage.getItem('account'),
      // 完成进度数据
      processData: {
        process: '25'
      },
      listenLoading: false
    }
  },

  // mounted，组件挂载后，此方法执行后，页面显示
  mounted: function () {
    this.loadWorkInfo()
    console.log('测试' + this.laccount)
    console.log('测试' + this.laccount)
    console.log('测试' + this.laccount)
    console.log('测试' + this.laccount)
    console.log('测试' + this.laccount)
  },

  methods: {

    // 点击跳转到显示详情界面，传递参数过去，在详情界面需要接受
    handleDetail: function (index, row) {
      this.$router.push({
        path: '/workDetailList',
        // name: 'mallList',
        query: {
          data: row
        }
      })
    },
    // 请求加载作业信息
    loadWorkInfo () {
      let _this = this
      // this.$axios.get('/workInfo').then(resp => {
      //   if (resp && resp.status === 200) {
      //     _this.works = resp.data;
      //   }
      // });

      this.$axios
        .post('/getTeacherPersonalWork', {
          keywords: this.account
        }).then(resp => {
          if (resp && resp.status === 200) {
            _this.works = resp.data
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
            _this.works = _this.searchResult

            let tempWorkList = []
            for (let i = 0; i < _this.searchResult.length; i++) {
              if (_this.searchResult[i].teacher.user.account === this.account) {
                tempWorkList.push(_this.searchResult[i])
              }
            }
            _this.works = tempWorkList
          }
        })
    },

    // 删除
    handleDelete: function (index, row) {
      this.$confirm('确认删除该作业吗?', '提示', {
        type: 'warning'
      }).then(() => {
        this.listenLoading = true
        this.$axios // {id: row.id}
          .post('/deleteWork', {
            workDetailId: row.workDetail.id,
            teacherId: row.teacher.id
          }).then(resp => {
            if (resp && resp.data.code === 100) {
              this.listenLoading = false
              this.$message({
                message: '删除成功',
                type: 'success'
              })
              this.loadWorkInfo()
            } else {
              this.$message({
                message: '删除失败',
                type: 'failure'
              })
              this.listenLoading = false
            }
          })
      }
      ).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },

    handleDeleteMultiple: function () {
      this.$confirm('确认删除这些作业吗?', '提示', {
        type: 'warning'
      }).then(() => {
        this.listenLoading = true
        let workIds = []
        let teacherId = 0
        for (let work of this.multipleSelection) {
          workIds.push(work.workDetail.id)
          teacherId = work.teacher.id
        }
        this.$axios // {id: row.id}
          .post('/deleteWorks', {
            workDetailIds: workIds,
            teacherId: teacherId
          }).then(resp => {
            if (resp && resp.data.code === 100) {
              this.listenLoading = false
              this.$message({
                message: '删除成功',
                type: 'success'
              })
              this.loadWorkInfo()
            } else {
              this.$message({
                message: '删除失败',
                type: 'failure'
              })
              this.listenLoading = false
            }
          })
      }
      ).catch((e) => {
        this.$message({
          type: 'info',
          message: e
        })
      })
    },

    handleSelectionChange (val) {
      this.multipleSelection = val
    }
  }
}
</script>

<style scoped>

</style>
