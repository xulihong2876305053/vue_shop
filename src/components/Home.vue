<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <span class="iconfont icon-xitong"></span>
        <span>电商后台管理系统</span>
      </div>
      <el-button type='info' @click='logout'>退出</el-button>
    </el-header>
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse?'64px':'200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu unique-opened router background-color="#333744" text-color="#fff" active-text-color="#409EFF"
          :collapse="isCollapse" :collapse-transition="false" :default-active="activePath">
          <!-- 一级菜单 -->
          <el-submenu :index="item.id+''" v-for="item in menuList" :key="item.id">

            <template slot="title">
              <!-- 图标 -->
              <i :class="iconsObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id"
              @click="saveNavState('/'+subItem.path)">
              <template slot="title">
                <!-- 图标 -->
                <i class="el-icon-menu"></i>
                <!-- 文本 -->
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 内容主体 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>
<script>
  export default {
    name: '',
    data() {
      return {
        menuList: [],
        iconsObj: {
          '125': 'iconfont icon-yonghu',
          '103': 'iconfont icon-tijikongjian',
          '101': 'iconfont icon-shangpin',
          '102': 'iconfont icon-danju-tianchong',
          '145': 'iconfont icon-baobiao',
        },
        isCollapse: false,
        activePath: ''
      }
    },

    methods: {
      logout() {
        window.sessionStorage.clear()
        this.$router.push('/login')
      },
      async getMenuList() {
        const {
          data: res
        } = await this.$http.get('menus')
        if (res.meta.status !== 200)
          return this.$message.error(res.meta.msg)
        this.menuList = res.data
        // console.log(res);
      },
      toggleCollapse() {
        this.isCollapse = !this.isCollapse
      },
      saveNavState(activePath) {
        this.activePath = activePath
        window.sessionStorage.setItem('activePath', activePath)
      }
    },
    components: {

    },
    created() {
      this.getMenuList()
      this.activePath = window.sessionStorage.getItem('activePath')
    },
    destroyed() {
      window.sessionStorage.clear()
    },
  }

</script>
<style lang="less" scoped>
  .home-container {
    height: 100%;
  }

  .el-header {
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: white;
    font-size: 20px;

    >div {
      display: flex;
      align-items: center;

      span:first-child {
        font-size: 30px;
        color: gray;
      }

      span:last-child {
        margin-left: 15px;
      }
    }
  }

  .el-aside {
    background-color: #333744;

    .el-menu {
      border-right: none;
    }
  }

  .el-main {
    background-color: #eaedf1;
  }

  .iconfont {
    margin-right: 15px;
  }

  .toggle-button {
    background-color: #4A5064;
    color: white;
    font-size: 10px;
    line-height: 24px;
    text-align: center;
    letter-spacing: 0.6em;
    cursor: pointer;
  }

</style>
