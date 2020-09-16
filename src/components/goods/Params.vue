<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-alert title="注意:只允许为第三级分类设置相关参数" type="warning" show-icon :closable="false">
      </el-alert>

      <el-row>
        <el-col :span="4">
          <span>选择商品分类:</span>
        </el-col>
        <el-col :span="20">
          <el-cascader ref="cascader" v-model="selectedCateKeys" :options="cateList" :props="cateProps"
            @change="parentCateChanged" clearable>
          </el-cascader>
        </el-col>
      </el-row>

      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 动态参数面板 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加参数</el-button>
          <el-table :data="manyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column align="center" type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable @close="handleClose(scope.row,i)">
                  {{item}}</el-tag>
                <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue"
                  ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag
                </el-button>
              </template>
            </el-table-column>
            <!-- 索引行 -->
            <el-table-column align="center" type="index" label="#"></el-table-column>
            <el-table-column align="center" label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column align="center" label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑
                </el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除
                </el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 静态属性面板 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加参数</el-button>
          <el-table :data="onlyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column align="center" type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable @close="handleClose(scope.row,i)">
                  {{item}}</el-tag>
                <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue"
                  ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag
                </el-button>
              </template>
            </el-table-column>
            <!-- 索引行 -->
            <el-table-column align="center" type="index" label="#"></el-table-column>
            <el-table-column align="center" label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column align="center" label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑
                </el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除
                </el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加对话框 -->
    <el-dialog :title="'添加'+titleText" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改对话框 -->
    <el-dialog :title="'修改'+titleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
  export default {
    name: '',
    data() {
      return {
        //商品分类
        cateList: [],
        //级联选择的数据
        selectedCateKeys: [],
        paramId: '',
        cateProps: {
          value: 'cat_id',
          label: 'cat_name',
          children: 'children',
          expandTrigger: 'hover',
          checkStrictly: true
        },
        //被激活的页签名称
        activeName: 'many',
        //动态参数的数据
        manyTableData: [],
        //静态属性的数据
        onlyTableData: [],
        //添加对话框显示
        addDialogVisible: false,
        //添加表单数据
        addForm: {
          attr_name: ''
        },
        //添加表单校验
        addFormRules: {
          attr_name: [{
            required: true,
            message: '请输入参数名称',
            trigger: 'blur'
          }]
        },
        //编辑对话框显示
        editDialogVisible: false,
        editForm: {
          attr_name: ''
        },
        //添加表单校验
        editFormRules: {
          attr_name: [{
            required: true,
            message: '请输入参数名称',
            trigger: 'blur'
          }]
        },


      }
    },
    created() {
      this.getCateList()
    },
    computed: {
      // 按钮的禁用
      isBtnDisabled() {
        if (this.selectedCateKeys.length !== 3) return true
        return false
      },
      //选中物品的id
      cateId() {
        if (this.selectedCateKeys.length === 3) return this.selectedCateKeys[2]
        return null
      },
      //对话框文本
      titleText() {
        if (this.activeName === "many")
          return '动态参数'
        else return '静态属性'
      }
    },
    methods: {
      //获取分类
      async getCateList() {
        const {
          data: res
        } = await this.$http.get('categories')
        if (res.meta.status !== 200)
          return this.$message.error(res.meta.msg)
        this.cateList = res.data
      },
      async getParamsData() {
        const {
          data: res
        } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: {
            sel: this.activeName
          }
        })
        if (res.meta.status !== 200)
          return this.$message.error(res.meta.msg)
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
          item.inputVisible = false
          item.inputValue = ''
        })

        if (this.activeName === 'many') {
          this.manyTableData = res.data
        } else {
          this.onlyTableData = res.data
        }
      },
      //级联选择变化
      parentCateChanged() {
        if (this.selectedCateKeys.length !== 3) {
          this.selectedCateKeys = []
          this.manyTableData = []
          this.onlyTableData = []
          return this.$message.error("请选择三级分类")
        }

        this.getParamsData()
      },
      //标签页发生变化
      handleTabClick() {
        this.getParamsData()
      },
      //添加对话框关闭
      addDialogClosed() {
        this.$refs.addFormRef.resetFields()
      },
      //提交推荐表单数据
      addParams() {
        this.$refs.addFormRef.validate(async valid => {
          if (!valid) {
            return this.$message.error("请填写相关信息!")
          }
          const {
            data: res
          } = await this.$http.post(`categories/${this.cateId}/attributes`, {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          })
          if (res.meta.status !== 201) {
            return this.$message.error("添加参数失败!")
          }
          this.$message.success("添加参数成功!")
          //隐藏对话框
          this.addDialogVisible = false
          //获取新的角色列表
          this.getParamsData()
        })
      },
      async showEditDialog(id) {
        this.paramId = id
        if (this.activeName === "many") {
          // console.log(this.manyTableData);
          const res = this.manyTableData.filter(item => item.attr_id === id)
          this.editForm.attr_name = res[0].attr_name
          // console.log(this.editForm.attr_name);

        } else {
          const res = this.onlyTableData.filter(item => item.attr_id === id)
          this.editForm.attr_name = res[0].attr_name
          // console.log(this.editForm.attr_name);
        }
        this.editDialogVisible = true
      },
      //修改对话框关闭
      editDialogClosed() {
        this.$refs.editFormRef.resetFields()
      },
      //修改参数
      editParams() {
        // console.log(this.paramId);
        this.$refs.editFormRef.validate(async valid => {
          if (!valid) {
            return this.$message.error("请填写相关信息!")
          }
          const {
            data: res
          } = await this.$http.put(`categories/${this.cateId}/attributes/${this.paramId}`, {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName
          })
          if (res.meta.status !== 200) {
            return this.$message.error("修改参数失败!")
          }
          this.$message.success("修改参数成功!")
          //隐藏对话框
          this.editDialogVisible = false
          //获取新的角色列表
          this.getParamsData()
        })
      },
      //删除参数val
      async removeParams(id) {
        const res = await this.$confirm('此操作将删除该参数, 是否继续?', '删除参数', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if (res !== "confirm") {
          return this.$message.info("已经取消该操作")
        }
        const {
          data: result
        } = await this.$http.delete(`categories/${this.cateId}/attributes/${id}`)
        // console.log(res);
        if (result.meta.status !== 200)
          return this.$message.error("删除参数失败!")
        this.$message.success("删除参数成功!")
        this.getParamsData()
      },
      //添加参数项
      async handleInputConfirm(row) {
        if (!row.inputValue.trim()) {
          row.inputValue = ''
          row.inputVisible = false
          return
        }
        row.attr_vals.push(row.inputValue)
        row.inputValue = ''
        row.inputVisible = false
        const {
          data: result
        } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' '),
        })
        // console.log(res);
        if (result.meta.status !== 200)
          return this.$message.error("添加参数项失败!")
        this.$message.success("添加参数项成功!")
      },
      //tag标签button与input切换
      showInput(row) {
        row.inputVisible = true
        this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus();
        });
      },
      //删除参数项
      async handleClose(row, i) {
        row.attr_vals.splice(i, 1)
        const {
          data: result
        } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' '),
        })
        // console.log(res);
        if (result.meta.status !== 200)
          return this.$message.error("删除参数项失败!")
        this.$message.success("删除参数项成功!")
      }
    },
    watch: {
      selectedCateKeys() {
        if (this.$refs.cascader) {
          this.$refs.cascader.dropDownVisible = false;
        }
      }
    },
    components: {

    }
  }

</script>
<style scoped>
  .el-alert {
    margin-bottom: 15px;
  }

  .el-col {
    line-height: 36px;
  }

  .el-cascader {
    width: 50%;
  }

  .el-radio {
    color: #606266;
    cursor: pointer;
    visibility: hidden !important;
    margin-right: 30px;
  }

  .el-tag {
    margin-right: 20px;
  }

  .input-new-tag {
    width: 120px;
  }

</style>
