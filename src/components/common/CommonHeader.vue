<template>
  <el-card class="common-header">
    <a >
      <img src="../../assets/icon2.png" alt="" width="55px" style="float: left;margin-top: -5px;">
    </a>
    <span style="font-size: 32px;font-weight: bold;position:absolute;left: 100px; color: grey">作业批改系统</span>

    <el-dropdown  show-timeout="10" hide-timeout="50" style="float: right;padding-right: 10px">
      <div class="el-dropdown-link" >
        <span >
          你好，{{$store.getters.name}}
        </span>
        <span style="float: right;margin-left: 10px">
          <el-avatar src="https://img0.baidu.com/it/u=4075908010,2021094905&fm=253&fmt=auto&app=120&f=JPEG" size="80"></el-avatar>
        </span>

        <el-dropdown-menu slot="dropdown" >
          <!-- <el-dropdown-item divided="true" @click.native="passwordModify">修改密码</el-dropdown-item> -->
          <el-dropdown-item divided="true" @click.native="logOut">退出登录</el-dropdown-item>
          <!--          <el-dropdown-item divided @click.native="logout">退出登录</el-dropdown-item>-->
        </el-dropdown-menu>

        <i class="el-icon-arrow-down el-icon--right"></i>
      </div>
    </el-dropdown>

  </el-card>
</template>

<script>
export default {
  name: 'commonHeader',
  data () {
    return {
      username: ''
    }
  },
  methods: {
    passwordModify: function () {
      this.$router.push({
        path: '/passwordModify'
      })
    },
    // 退出登录
    logOut: function () {
      var _this = this
      this.$confirm('确认退出吗?', '提示', {
        // type: 'warning'
      }).then(() => {
        _this.$store.commit('logout')
        // _this.$router.push('/login');
        this.$router.push({
          path: '/login'
        })
      }).catch(() => {

      })
    }
  }
}
</script>

<style scoped>
  .common-header {

    height: 80px;
    opacity: 0.85;
    line-height: 40px;
  }
  .el-dropdown-link {
       cursor: pointer;
       color: #409EFF;
       text-size:20px;
     }
  .el-icon-arrow-down {
    font-size: 12px;
  }
</style>
