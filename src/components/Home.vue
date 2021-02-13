<template>
  <el-container class='home-container'>
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="">
        <span>电商管理系统</span>
      </div>
      <el-button type='info' @click='logout'>退出</el-button>
    </el-header>
    <el-container>
      <el-aside :width="isCollapse ? '64px':'200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu background-color="#333744" text-color="#fff" :collapse-transition='false' router
          :default-active="'/'+ this.activeItem" :collapse='isCollapse' unique-opened active-text-color="#409eff">
          <!-- 一级菜单 -->
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" @click='saveNavState(subItem.path)'
              :key='subItem.id'>
              <i class="el-icon-menu"></i>
              <span>{{subItem.authName}}</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
  export default {
    data() {
      return {
        menuList: [],
        iconsObj: {
          125: 'iconfont icon-user',
          103: 'iconfont icon-tijikongjian',
          101: 'iconfont icon-shangpin',
          102: 'iconfont icon-danju',
          145: 'iconfont icon-baobiao'
        },
        isCollapse: false,
        // 被激活的链接地址
        activeItem: ''
      }
    },
    created() {
        this.getMemuList()
        this.activeItem = window.sessionStorage.getItem('activeItem')
    },
    methods: {
      logout() {
        window.sessionStorage.clear()
        this.$router.push('/login')
      },
      async getMemuList() {
        const {
          data: res
        } = await this.$http.get('menus')
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        this.menuList = res.data
      },
      toggleCollapse() {
        this.isCollapse = !this.isCollapse
      },
      saveNavState(activeItem) {
        window.sessionStorage.setItem('activeItem', activeItem)
        this.activeItem = activeItem
      }
    }
  }

</script>

<style lang='less' scoped>
  .home-container {
    height: 100%;
  }

  .el-header {
    background-color: #333744;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-left: 0;
    color: #fff;
    font-size: 20px;

    >div {
      display: flex;
      align-items: center;

      >span {
        margin-left: 10px;
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
    background-color: #fff;
  }

  .iconfont {
    margin-right: 10px;
  }

  .toggle-button {
    background-color: #5a5064;
    font-size: 10px;
    display: flex;
    justify-content: center;
    align-items: center;
    line-height: 24px;
    color: #fff;
    letter-spacing: 0.2em;
    cursor: pointer;
  }

</style>
