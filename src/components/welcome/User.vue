<template>
  <div>
    <div class="hander">
      <h4>用户列表</h4>
      <router-link to="/users" class="link">查看详细信息</router-link>
    </div>
    <div class="body">
      <el-table :data="tableData" stripe style="width: 100%">
        <el-table-column prop="username" label="用户名" min-width="100">
        </el-table-column>
        <el-table-column prop="mobile" label="手机号" min-width="150">
        </el-table-column>
        <el-table-column prop="email" label="邮箱"> </el-table-column>
      </el-table>
    </div>
  </div>
</template>
<script>
export default {
  props: [],
  data() {
    return {
      tableData: []
    }
  },
  created() {
    this.getUserInfo()
  },
  mounted() {
    document.querySelectorAll('.has-gutter')[0].style.color = '#000'
  },
  methods: {
    async getUserInfo() {
      const { data: res } = await this.$http.get('/users', {
        params: { pagenum: 1, pagesize: 4 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户失败')
      }
      this.tableData = res.data.users
    }
  }
}
</script>
<style lang="less" scoped>
.hander {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
h4 {
  font-size: 20px;
  font-weight: bold;
  margin: 10px 0;
}
.link {
  font-size: 14px;
  color: #1f87fa;
  text-decoration: none;
}
.el-table {
  color: #000;
}
</style>
