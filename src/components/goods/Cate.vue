<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" class="addbtn" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <tree-table :data="cateList" show-index :columns="columns" :selection-type="false" :expand-type="false" border>
        <template slot="isOk" slot-scope="scope">
          <i style="color: lightgreen;" v-if="scope.row.cat_deleted===false" class="el-icon-success"></i>
          <i style="color: red;" v-else class="el-icon-error"></i>
        </template>

        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level===0" size="mini">一级</el-tag>
          <el-tag v-else-if="scope.row.cat_level===1" type="success" size="mini">二级</el-tag>
          <el-tag v-else type="warning" size="mini">三级</el-tag>
        </template>

        <template slot="option">
          <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum" :page-sizes="[3,5,10,15]" :page-size="queryInfo.pagesize"
        layout="total,sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="120px">
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <el-cascader size="medium" v-model="selectedKeys" :options="parentCateList" :props="cascadeProps"
            @change="parentCateChanged" clearable>
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCateDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
  export default {
    name: '',
    data() {
      return {
        //商品数据
        cateList: [],
        //商品总数
        total: 0,
        //查询条件
        queryInfo: {
          type: 3,
          pagenum: 1,
          pagesize: 5
        },
        //表格每一列的配置信息
        columns: [{
          label: '分类名称',
          prop: 'cat_name',
          width: 300
        }, {
          label: '是否有效',
          type: 'template',
          template: 'isOk',
          align: 'center',
          headerAlign: 'center',
          width: 250
        }, {
          label: '排序',
          type: 'template',
          template: 'order',
          align: 'center',
          headerAlign: 'center',
          width: 250
        }, , {
          label: '操作',
          type: 'template',
          template: 'option',
          align: 'center',
          headerAlign: 'center',
        }],
        //添加分类对话框
        addCateDialogVisible: false,
        //添加分类表单数据
        addCateForm: {
          cat_pid: 0,
          cat_name: '',
          cat_level: 0
        },
        //添加表单校验
        addCateFormRules: {
          cat_name: [{
            required: true,
            message: '请输入分类名称',
            trigger: 'blur'
          }]
        },
        //父级分类列表
        parentCateList: [],
        //级联选择器的配置
        cascadeProps: {
          value: 'cat_id',
          label: 'cat_name',
          children: 'children',
          expandTrigger: 'hover',
          checkStrictly: true
        },
        //选中的父级分类
        selectedKeys: []
      }
    },
    created() {
      this.getCateList()
    },
    methods: {
      //获取商品分类
      async getCateList() {
        const {
          data: res
        } = await this.$http.get('categories', {
          params: this.queryInfo
        })
        if (res.meta.status !== 200)
          return this.$message.error(res.meta.msg)
        this.cateList = res.data.result
        this.total = res.data.total
      },
      //获取父级分类的数据列表
      async getParentCateList() {
        const {
          data: res
        } = await this.$http.get('categories', {
          params: {
            type: 2
          }
        })
        if (res.meta.status !== 200)
          return this.$message.error(res.meta.msg)
        this.parentCateList = res.data
        // console.log(this.parentCateList);
      },
      //监听pagesize变化
      handleSizeChange(newSize) {
        this.queryInfo.pagesize = newSize
        this.getCateList()
      },
      //监听pagenum变化
      handleCurrentChange(newPage) {
        this.queryInfo.pagenum = newPage
        this.getCateList()
      },
      //展示添加分类对话框
      showAddCateDialog() {
        this.getParentCateList()
        this.addCateDialogVisible = true
      },
      //重置添加分类对话框
      addCateDialogClosed() {
        this.$refs.addCateFormRef.resetFields();
      },
      parentCateChanged() {

      },

    },
    components: {

    }
  }

</script>
<style scoped>
  .addbtn {
    margin-bottom: 15px;
  }

  .el-cascader {
    width: 100%;
  }

</style>
