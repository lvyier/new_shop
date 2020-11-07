<template>
  <div class="Home">
    <el-container>
      <!-- 头部区域 -->
      <el-header>
        <div class="home_top">
          <div class="img">
            <img src="../assets/heima.png" alt="vue" />
          </div>
          <span>电商后台管理系统</span>
        </div>
        <el-button type="info" @click="logout">退出</el-button>
      </el-header>
      <el-container>
        <!-- 左侧区域 -->
        <el-aside :width="isCollapse ? '64px' : '200px'">
          <div class="toggle-button" @click="toggleCollapse">|||</div>
          <el-menu
            :default-active="activePath"
            :collapse="isCollapse"
            :collapse-transition="false"
            router
            :unique-opened="true"
            background-color="#333744"
            text-color="#fff"
            active-text-color="#409EFF"
          >
            <!-- 一级菜单项 -->
            <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
              <template slot="title">
                <i :class="iconsObj[item.id]"></i>
                <span>{{ item.authName }}</span>
              </template>
              <!-- 二级菜单项 -->
              <el-menu-item @click="saveActive('/' + subItem.path)" :index="'/' + subItem.path" v-for="subItem in item.children" :key="subItem.id">
                <template>
                  <i class="el-icon-menu"></i>
                  <span slot="title">{{ subItem.authName }}</span>
                </template>
              </el-menu-item>
            </el-submenu>
          </el-menu>
        </el-aside>
        <!-- 主体区域 -->
        <el-main>
          <router-view></router-view>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>
<script>
export default {
  name: 'Home',
  data() {
    return {
      // 菜单项数据
      menuList: [],
      // 图标数据
      iconsObj: {
        125: 'iconfont icon-user',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      // 链接状态
      activePath: '',
      isCollapse: false
    }
  },
  created() {
    // 左侧菜单接口
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    // 退出
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 获取左侧菜单项数据
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.status)
      this.menuList = res.data
      //   console.log(this.menuList)
    },
    // 链接
    saveActive(path) {
      window.sessionStorage.setItem('activePath', path)
      this.activePath = path
    },
    // 缩小
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    }
  }
}
</script>
<style lang="less" scoped>
.Home {
  height: 100%;
}
.home_top {
  display: flex;
  align-items: center;
}
.home_top img {
  margin-right: 15px;
}
.el-menu {
  border-right: none;
}
.el-main {
  background-color: #eaedf1;
}
.iconfont {
  margin-right: 10px;
}
.toggle-button {
  background-color: #4a5064;
  font-size: 10px;
  line-height: 24px;
  color: #ffffff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
