<template>
  <el-container class="home-container">
    <el-header>
      <div>
        <img src="../assets/logo.png" />
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出登录</el-button>
    </el-header>
    <el-container>
      <el-aside :style="isCollapse?'width:64px':'width:200px'">
        <!-- 折叠按钮 -->
        <div class="toggle-button-back">
          <div
            class="toggle-button el-icon-arrow-left"
            :class="isCollapse?'left':''"
            @click="toggle"
          ></div>
        </div>

        <!-- 菜单栏 -->
        <el-menu
          background-color="#323844"
          text-color="#fff"
          active-text-color="#409eff"
          :unique-opened="true"
          :collapse="isCollapse"
          :collapse-transition="false"
          :router="true"
          :default-active="activePath"
        >
          <!-- 一级菜单 -->
          <el-submenu v-for="(item,index) in menuList" :key="item.id" :index="index+''">
            <template slot="title">
              <i :class="iconList[index+'']"></i>
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item
              :index="'/'+subItem.path"
              v-for="subItem in item.children"
              :key="subItem.id"
              @click="saveNavState(subItem.path)"
            >
              <i class="el-icon-menu"></i>
              <span>{{subItem.authName}}</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-container>
        <el-main>
          <!-- 路由占位符 -->
          <router-view />
        </el-main>
      </el-container>
    </el-container>
  </el-container>
</template>
<script>
export default {
  // 生命周期函数
  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('nav')
  },
  data() {
    return {
      // 左侧菜单数据
      menuList: [],
      iconList: [
        'el-icon-user-solid',
        'el-icon-s-cooperation',
        'el-icon-s-goods',
        'el-icon-s-order',
        'el-icon-s-marketing'
      ],
      isCollapse: false,
      activePath: ''
    }
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 获取所有菜单
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      // console.log(res.data)
      if (res.meta.status !== 200) {
        if (res.meta.status === 400) {
          return this.$router.push('login')
        }
        return this.$message.error(res.meta.msg)
      }
      this.menuList = res.data
    },
    toggle() {
      this.isCollapse = !this.isCollapse
    },
    saveNavState(activePath) {
      this.activePath = activePath
      window.sessionStorage.setItem('nav', activePath)
    }
  }
}
</script>
<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  display: flex;
  background-color: #31383b;
  height: 60px;
  line-height: 60px;
  justify-content: space-between;
  padding: 0 10px;
  align-items: center;
  img {
    float: left;
    height: 50px;
    width: 50px;
    border: 5px solid #333;
    border-radius: 50%;
    margin-right: 10px;
  }
  span {
    float: left;
    display: block;
    font-size: 30px;
    color: #ccc;
  }
}
.el-aside {
  background-color: #323844;
  transition: 0.5s all;
  .el-menu {
    border: none;
  }
  .toggle-button {
    height: 50px;
    width: 100%;
    cursor: pointer;
    text-align: center;
    line-height: 50px;
    color: #fff;
    font-size: 22px;
    transition: 0.5s transform;
  }
  .toggle-button.left {
    transform: rotate(180deg);
  }
  .toggle-button-back:hover {
    background-color: #282d36;
  }
}
.el-main {
  background-color: #e5e9ed;
}
</style>
