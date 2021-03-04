<template>
  <div>
    <el-row :gutter="10" class="order">
      <!-- 热销商品 -->
      <el-col :span="12">
        <el-card><router-view name="hot" /></el-card>
      </el-col>
      <!-- 商品订单详情 -->
      <el-col :span="12" class="details">
        <router-view
          name="details"
          :total="total"
          :pendingPay="pendingPay"
          :pendingSend="pendingSend"
          :customerPrice="customerPrice"
        ></router-view>
      </el-col>
    </el-row>
    <el-row :gutter="10" class="rank">
      <!-- 用户列表 -->
      <el-col :span="12">
        <el-card><router-view name="user" /></el-card>
      </el-col>
      <!-- 商品成交金额分布 -->
      <el-col :span="12">
        <el-card>
          <router-view name="price" :price="pricePie"></router-view>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>
<script>
export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      orderList: [],
      pendingPay: 0,
      pendingSend: 0,
      customerPrice: 0,
      // <30, 30-100, 100-300, >300
      pricePie: [
        { value: 0, name: '<30' },
        { value: 0, name: '30-100' },
        { value: 0, name: '100-300' },
        { value: 0, name: '>300' }
      ]
    }
  },
  created() {
    this.getOrderList().then(() => {
      this.queryInfo.pagesize = this.total
      this.getOrderList().then(() => {
        this.getOtherInfo()
      })
    })
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get('/orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单失败')
      }
      console.log(res.data)
      this.total = res.data.total
      this.orderList = res.data.goods
    },
    getOtherInfo() {
      this.pendingPay = 0
      this.pendingSend = 0
      this.customerPrice = 0
      let totalPrice = 0
      this.orderList.forEach((item, index) => {
        if (item.is_send === '否') {
          this.pendingSend += 1
        }
        if (item.pay_status === '0') {
          this.pendingPay += 1
        }
        totalPrice += item.order_price
        if (item.order_price <= 30) {
          this.pricePie[0].value += 1
        } else if (item.order_price <= 100) {
          this.pricePie[1].value += 1
        } else if (item.order_price <= 300) {
          this.pricePie[2].value += 1
        } else {
          this.pricePie[3].value += 1
        }
      })
      this.customerPrice = totalPrice / this.total
    }
  },
  computed: {},
  watch: {}
}
</script>
<style lang="less" scoped>
.el-row {
  margin-bottom: 10px;
  &:last-child {
    margin-bottom: 0;
  }
}
.order {
  height: 400px;
  .el-col {
    height: 100%;
  }
}
.rank {
  height: 350px;
  .el-col {
    height: 100%;
  }
}
.el-card {
  height: 100%;
}
</style>
