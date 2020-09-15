<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddGoodPage">添加商品</el-button>
        </el-col>
      </el-row>

      <el-table :data="goodsList" border stripe>
        <el-table-column label="#" type="index" align="center"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column width="90px" label="商品价格(元)" prop="goods_price" align="center"></el-table-column>
        <el-table-column width="70px" label="商品重量" prop="goods_weight" align="center"></el-table-column>
        <el-table-column width="160px" label="创建时间" prop="add_time" align="center">
          <template slot-scope="scope">
            {{scope.row.add_time|dateFormat}}
          </template>
        </el-table-column>
        <el-table-column width="160px" label="操作" align="center">
          <template>
            <el-button type="primary" icon="el-icon-edit" size="mini">
            </el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeGoodsById(scope.row.goods_id)">
            </el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum" :page-sizes="[5, 10, 15,20]" :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper" :total="total" background>
      </el-pagination>
    </el-card>
  </div>
</template>
<script>
  export default {
    name: '',
    data() {
      return {
        //商品列表
        goodsList: [],
        //查询参数
        queryInfo: {
          query: '',
          pagenum: 1,
          pagesize: 10
        },
        //总共条数
        total: 0
      }
    },
    created() {
      this.getGoodsList()
    },
    methods: {
      //商品列表数据
      async getGoodsList() {
        const {
          data: res
        } = await this.$http.get('goods', {
          params: this.queryInfo
        })
        // console.log(res);
        if (res.meta.status !== 200) return this.$message.error("获取商品列表失败!")
        this.goodsList = res.data.goods
        this.total = res.data.total
      },
      //监听每页用户数值变化
      handleSizeChange(newSize) {
        this.queryInfo.pagesize = newSize
        this.getGoodsList()
      },
      //监听页码数变化
      handleCurrentChange(newNum) {
        this.queryInfo.pagenum = newNum
        this.getGoodsList()
      },
      //删除商品
      async removeGoodsById(id) {
        const res = await this.$confirm('此操作将删除该商品, 是否继续?', '删除商品', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if (res !== "confirm") {
          return this.$message.info("已经取消该操作")
        }
        const {
          data: result
        } = await this.$http.delete('goods/' + id)
        if (result.meta.status !== 200)
          return this.$message.error("删除商品失败!")
        this.$message.success("删除商品成功!")
        this.getGoodsList()
      },
      //跳转至添加商品
      goAddGoodPage() {
        this.$router.push('/goods/add')
      },

    },
    components: {

    }
  }

</script>
<style scoped>

</style>
