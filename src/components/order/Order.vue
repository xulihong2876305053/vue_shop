<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col :span="10">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable>
            <el-button slot="append" icon="el-icon-search" @click="getOrderList"></el-button>
          </el-input>
        </el-col>
      </el-row>

      <el-table :data="orderList" border stripe>
        <el-table-column label="#" type="index" align="center"></el-table-column>
        <el-table-column label="订单编号" prop="order_number" align="center" width="240px"></el-table-column>
        <el-table-column label="订单价格" prop="order_price" align="center"></el-table-column>
        <el-table-column label="是否付款" prop="pay_status" align="center">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status==='1'">已付款</el-tag>
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send" align="center"></el-table-column>
        <el-table-column label="下单时间" prop="create_time" align="center">
          <template slot-scope="scope">
            {{scope.row.create_time|dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作" align="center">
          <template>
            <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
            <el-button type="warning" icon="el-icon-location" size="mini"></el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum" :page-sizes="[5, 10, 15,20]" :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper" :total="total">
    </el-pagination>
  </div>
</template>
<script>
  export default {
    name: '',
    data() {
      return {
        orderList: [],
        queryInfo: {
          query: '',
          pagenum: 1,
          pagesize: 10
        },
        total: 0,

      }
    },
    created() {
      this.getOrderList()
    },
    methods: {
      async getOrderList() {
        const {
          data: res
        } = await this.$http.get('orders', {
          params: this.queryInfo
        })
        // console.log(res);
        if (res.meta.status !== 200) return this.$message.error("获取用户列表失败!")
        this.orderList = res.data.goods
        this.total = res.data.total
      },
      handleSizeChange(newSize) {
        this.queryInfo.pagesize = newSize
        this.getOrderList()
      },
      //监听页码数变化
      handleCurrentChange(newNum) {
        this.queryInfo.pagenum = newNum
        this.getOrderList()
      },


    },
    components: {

    }
  }

</script>
<style scoped>

</style>
