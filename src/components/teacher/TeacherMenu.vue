
<!--白红配色导航-->
<!--background-color="white"-->
<!--text-color="#222"-->
<!--active-text-color="red"-->

<template>
  <el-menu class="nav-menu"
    router :default-active="$route.path"
    mode="horizontal"

    background-color="#1ABC9C"
    text-color="#fff"
    active-text-color="#ffd04b"
    style="min-width: 1300px;max-height: 120px"

  >
    <el-menu-item v-for="(item,i) in navList" :key="i" :index="item.name">
      {{ item.navItem }}
    </el-menu-item>

<!--    <span style="color:#fff;position: absolute;padding-top: 20px;right: 43%;font-size: 20px;font-weight: bold">作业批改系统</span>-->
  </el-menu>
</template>

<script>
export default {
  name: 'TeacherMenu',
  data () {
    return {
      navList: [
        {name: '/teacherWork', navItem: '我的作业'},
        {name: '/workPublish', navItem: '发布作业'}
      ]
    }
  }
  // methods:{
  //   handleSelect(key,keypath){
  //     if (key==='/teacherInfo') {this.$router.push('/teacherInfo')
  //     }else if(key==='/studentInfo') {this.$router.push('/studentInfo')
  //     }else if(key==='/classInfo') {this.$router.push('/classInfo')
  //     }else if(key==='/course') {this.$router.push('/course')
  //     }
  //   }
  // }
}
</script>

<style scoped>
  a{
    text-decoration: none;
  }

  span {
    pointer-events: none;
  }
  .nav-menu{
    margin-top: 80px;
  }
</style>
