<template>
  <el-container>
    <!-- 头部 -->
    <el-header>
      <div>
        <img src="@/assets/heima.png"
             alt="">
        <span>电商后台管理系统</span>
      </div>
      <el-button class="logout"
                 type="info"
                 @click="logout">退出登陆</el-button>
    </el-header>
    <!-- 页面主题 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <!-- 侧边栏菜单区 -->
        <el-menu background-color="#333744"
                 text-color="#fff"
                 active-text-color="#409eff"
                 :unique-opened="true"
                 :collapse="isCollapse"
                 :collapse-transition="false"
                 :router="true"
                 :default-active='activePath'>
          <!-- 一级菜单 -->
          <el-submenu :index="'/' + menu.path"
                      v-bind:key='menu.id'
                      v-for="menu in menuList">
            <!-- 一级菜单模板区 -->
            <template slot="title">
              <i :class="iconsObj[menu.id]"></i>
              <span>{{menu.authName}}</span>
            </template>
            <!-- 二级菜单 -->

            <el-menu-item :index="'/' + me.path"
                          v-bind:key="me.id"
                          v-for="me in menu.children"
                          @click="saveNavState('/' + me.path)">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{ me.authName }}</span>
              </template>
            </el-menu-item>

          </el-submenu>
        </el-menu>
        <div class="toggle-button"
             @click="toggleCollanse">
          ||| </div>
      </el-aside>
      <!-- 内容主题 -->
      <el-container>
        <el-main>
          <!-- 路由占位符 -->
          <router-view></router-view>
        </el-main>
      </el-container>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      menuList: [],
      iconsObj: {
        125: 'iconfont icon-users',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'

      },
      isCollapse: false,
      // 被激活的地址
      activePath: ''
    }
  },
  created () {
    this.listMenus()
    this.activePath = window.sessionStorage.getItem('navState')
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async listMenus () {
      const { data: menus } = await this.$http.get('/menus')
      if (menus.meta.status !== 200) return this.$message.error(menus.meta.msg)
      this.menuList = menus.data
    },
    toggleCollanse () {
      // 点击按钮切换菜单折叠和展开
      this.isCollapse = !this.isCollapse
    },
    saveNavState (rel) {
      // 保存链接激活状态,并切换
      window.sessionStorage.setItem('navState', rel)
      this.activePath = rel
    }
  }
}
</script>

<style lang="less" scoped>
.el-header {
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  background-color: #373d41;
  align-items: center;
  color: white;
  font-size: 20px;
  > div {
    display: flex;
    align-items: center;
    span {
      margin-left: 15px;
    }
  }
}
.el-aside {
  background-color: #333744;
  position: relative;
  .el-menu {
    border-right: none;
  }
  > .toggle-button {
    background-color: #4a5064;
    width: 100%;
    font-size: 10px;
    color: white;
    line-height: 24px;
    position: absolute;
    text-align: center;
    bottom: 10px;
    left: 50%;
    transform: translate(-50%);
    letter-spacing: 0.4em;
    cursor: pointer;
  }
}
.el-main {
  background-color: #eaedf1;
}
.el-container {
  width: 100%;
  height: 100%;
}
.iconfont {
  margin-right: 10px;
}
</style>
