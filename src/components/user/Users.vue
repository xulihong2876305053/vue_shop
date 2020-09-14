<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row :gutter="20">
        <el-col :span="10">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
        </el-col>
      </el-row>
      <el-table :data="userList" border stripe>
        <el-table-column label="#" type="index" align="center"></el-table-column>
        <el-table-column label="姓名" prop="username" align="center"></el-table-column>
        <el-table-column label="邮箱" prop="email" align="center"></el-table-column>
        <el-table-column label="电话" prop="mobile" align="center"></el-table-column>
        <el-table-column label="角色" prop="role_name" align="center"></el-table-column>
        <el-table-column label="状态" align="center">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="200px" align="center">
          <template slot-scope="scope">
            <el-tooltip effect="dark" content="修改用户" placement="top" :enterable="false">
              <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">
              </el-button>
            </el-tooltip>
            <el-tooltip effect="dark" content="删除用户" placement="top" :enterable="false">
              <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)">
              </el-button>
            </el-tooltip>
            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini" @click="setRole(scope.row)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum" :page-sizes="[2, 5, 10]" :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>

    <!-- 添加用户对话框 -->
    <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="80px">
        <el-form-item label="用户名:" prop="username">
          <el-input v-model="addForm.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="密码:" prop="password">
          <el-input v-model="addForm.password" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="邮箱:" prop="email">
          <el-input v-model="addForm.email" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="电话:" prop="mobile">
          <el-input v-model="addForm.mobile" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户对话框 -->
    <el-dialog title="修改用户信息" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="80px">
        <el-form-item label="用户名:">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱:" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="电话:" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配角色对话框 -->
    <el-dialog title="分配角色" :visible.sync="setRoleDialogVisible" width="40%" @close="setRoleDialogClosed">
      <div>
        <el-row class="bdbottom">
          <el-col :span="4">当前用户:</el-col>
          <el-col :span="20">{{userInfo.username}}</el-col>
        </el-row>
        <el-row class="bdbottom">
          <el-col :span="4">当前角色:</el-col>
          <el-col :span="20">{{userInfo.role_name}}</el-col>
        </el-row>
        <el-row>
          <el-col :span="4" class="vcenter">角色选择:</el-col>
          <el-col :span="20">
            <el-select v-model="selectedRoleId" placeholder="请选择">
              <el-option v-for="item in rolesList" :key="item.id" :label="item.roleName" :value="item.id">
              </el-option>
            </el-select>
          </el-col>
        </el-row>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
  export default {
    name: '',
    data() {
      //验证密码
      var checkPwd = (rule, value, cb) => {
        const regPwd = /^(?=.*\d)(?=.*[a-z])(?=.*[A-Z])[a-zA-Z0-9]{6,15}$/
        if (regPwd.test(value)) {
          return cb()
        }
        cb(new Error('密码必须包含大小写字母和数字，不能使用特殊字符，长度在 8-10 之间'))
      }
      //验证邮箱
      var checkEmail = (rule, value, cb) => {
        const regEmail = /^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$/
        if (regEmail.test(value)) {
          return cb()
        }
        cb(new Error('邮箱格式不正确'))
      }
      //验证手机号码
      var checkMobile = (rule, value, cb) => {
        const regMobile = /^(13[0-9]|14[5|7]|15[0|1|2|3|4|5|6|7|8|9]|18[0|1|2|3|5|6|7|8|9])\d{8}$/
        if (regMobile.test(value)) {
          return cb()
        }
        cb(new Error('手机号码格式不正确'))
      }

      return {
        //查询
        queryInfo: {
          query: '',
          pagenum: 1,
          pagesize: 2
        },
        //所有用户列表
        userList: [],
        //总共人数
        total: 0,
        //添加用户对话框状态
        addDialogVisible: false,
        //添加用户信息
        addForm: {
          username: '',
          password: '',
          email: '',
          mobile: '',
        },
        //添加用户校验
        addFormRules: {
          username: [{
            required: true,
            message: '请输入用户名',
            trigger: 'blur'
          }, {
            min: 3,
            max: 10,
            message: '用户名长度在 3 到 10 个字符',
            trigger: 'blur'
          }],
          password: [{
            required: true,
            message: '请输入密码',
            trigger: 'blur'
          }, {
            validator: checkPwd,
            trigger: 'blur'
          }],
          email: [{
            required: true,
            message: '请输入邮箱',
            trigger: 'blur'
          }, {
            validator: checkEmail,
            trigger: 'blur'
          }],
          mobile: [{
            required: true,
            message: '请输入电话',
            trigger: 'blur'
          }, {
            validator: checkMobile,
            trigger: 'blur'
          }]
        },
        //修改用户信息对话框
        editDialogVisible: false,
        //修改用户信息
        editForm: {},
        //修改用户校验
        editFormRules: {
          email: [{
            validator: checkEmail,
            trigger: 'blur'
          }],
          mobile: [{
            validator: checkMobile,
            trigger: 'blur'
          }]
        },
        //分配角色对话框
        setRoleDialogVisible: false,
        //当前用户信息
        userInfo: {},
        //角色列表
        rolesList: [],
        //当前选择角色id
        selectedRoleId: '',
      }
    },
    created() {
      this.getUserList()
    },
    methods: {
      //获取用户列表
      async getUserList() {
        const {
          data: res
        } = await this.$http.get('users', {
          params: this.queryInfo
        })
        // console.log(res);
        if (res.meta.status !== 200) return this.$message.error("获取用户列表失败!")
        this.userList = res.data.users
        this.total = res.data.total
      },
      //监听每页用户数值变化
      handleSizeChange(newSize) {
        this.queryInfo.pagesize = newSize
        this.getUserList()
      },
      //监听页码数变化
      handleCurrentChange(newNum) {
        this.queryInfo.pagenum = newNum
        this.getUserList()
      },
      //监听用户状态改变
      async userStateChange(userInfo) {
        const {
          data: res
        } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
        if (res.meta.status !== 200) {
          userInfo.mg_state = !userInfo.mg_state
          return this.$message.error("更新用户状态失败!")
        }
        this.$message.success("更新用户状态成功!")
      },
      //添加用户对话框重置
      addDialogClosed() {
        this.$refs.addFormRef.resetFields();
      },
      //添加用户的校验
      addUser() {
        this.$refs.addFormRef.validate(async valid => {
          if (!valid) return this.$message.error("请填写相关信息!")
          const {
            data: res
          } = await this.$http.post('users', this.addForm)
          if (res.meta.status !== 201) {
            return this.$message.error("添加用户失败!")
          }
          this.$message.success("添加用户成功!")
          //隐藏对话框
          this.addDialogVisible = false
          //获取新的用户列表
          this.getUserList()
        })
      },
      //修改用户获取数据
      async showEditDialog(id) {
        // console.log(id);
        const {
          data: res
        } = await this.$http.get('users/' + id)
        // console.log(res);
        if (res.meta.status !== 200)
          return this.$message.error("查询用户信息失败!")
        this.editForm = res.data
        this.editDialogVisible = true
      },
      //修改对话框重置
      editDialogClosed() {
        this.$refs.editFormRef.resetFields();
      },
      //修改用户信息
      editUser() {
        this.$refs.editFormRef.validate(async valid => {
          if (!valid) return
          const {
            data: res
          } = await this.$http.put('users/' + this.editForm.id, {
            email: this.editForm.email,
            mobile: this.editForm.mobile
          })
          if (res.meta.status !== 200)
            return this.$message.error("修改用户信息失败!")
          this.editDialogVisible = false
          this.getUserList()
          this.$message.success("修改用户信息成功!")
        })
      },
      //删除用户
      async removeUserById(id) {
        const res = await this.$confirm('此操作将删除该用户, 是否继续?', '删除用户', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if (res !== "confirm") {
          return this.$message.info("已经取消该操作")
        }
        const {
          data: result
        } = await this.$http.delete('users/' + id)
        // console.log(res);
        if (result.meta.status !== 200)
          return this.$message.error("删除用户失败!")
        this.$message.success("删除用户成功!")
        this.getUserList()
      },
      //分配角色对话框
      async setRole(userInfo) {
        this.userInfo = userInfo
        const {
          data: res
        } = await this.$http.get('roles')
        if (res.meta.status !== 200)
          return this.$message.error("获取角色失败!")
        this.rolesList = res.data
        // console.log(this.rolesList);
        this.setRoleDialogVisible = true
      },
      //分配角色
      async saveRoleInfo() {
        if (!this.selectedRoleId) {
          return this.$message.error("请选择需要分配的角色")
        }
        const {
          data: res
        } = await this.$http.put(`users/${this.userInfo.id}/role`, {
          rid: this.selectedRoleId
        })
        if (res.meta.status !== 200)
          return this.$message.error("分配角色失败!")
        this.$message.success("分配角色成功!")
        this.getUserList()
        this.setRoleDialogVisible = false
      },
      setRoleDialogClosed() {
        this.selectedRoleId = ""
        this.userInfo = ''
      }
    },
    components: {

    }
  }

</script>
<style scoped>
  .vcenter {
    line-height: 40px;
  }

  .bdbottom {
    margin-bottom: 15px;
  }

</style>
