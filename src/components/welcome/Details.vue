<template>
  <div>
    <el-row :gutter="10">
      <el-col :span="12">
        <el-card>
          <h4>订单量（个）</h4>
          <div class="number">{{ this.total }}</div>
          <router-link to="/orders" class="link">查看详细信息</router-link>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card>
          <h4>待付款（个）</h4>
          <div class="number">{{ this.pendingPay }}</div>
          <router-link to="/orders" class="link">查看详细信息</router-link>
        </el-card>
      </el-col>
    </el-row>
    <el-row :gutter="10">
      <el-col :span="12">
        <el-card>
          <h4>待发货（个）</h4>
          <div class="number">{{ this.pendingSend }}</div>
          <router-link to="/orders" class="link">查看详细信息</router-link>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card>
          <h4>客单价（元）</h4>
          <div class="number">{{ this.customerPrice.toFixed(2) }}</div>
          <router-link to="/orders" class="link">查看详细信息</router-link>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>
<script>
export default {
  props: ['total', 'pendingPay', 'pendingSend', 'customerPrice'],
  data() {
    return {}
  }
}
</script>
<style lang="less" scoped>
.el-row {
  margin-bottom: 10px;
  height: 195px;
  &:last-child {
    margin-bottom: 0;
  }
  .el-col {
    height: 100%;
  }
}
.el-card {
  height: 100%;
}

h4 {
  font-size: 20px;
  font-weight: bold;
  margin: 10px 0;
}
.number {
  font-size: 40px;
  font-weight: bold;
  margin: 20px 0;
}
.link {
  font-size: 14px;
  color: #1f87fa;
  text-decoration: none;
}
</style>
