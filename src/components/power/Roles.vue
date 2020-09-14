<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddRoleDialog">添加角色</el-button>
        </el-col>
      </el-row>

      <el-table :data="rolesList" border stripe>
        <el-table-column align="center" type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom','vcenter',i1===0?'bdtop':'']" v-for="(item1,i1) in scope.row.children"
              :key="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级权限 -->
              <el-col :span="19">
                <el-row :class="['vcenter',i2===0?'':'bdtop']" v-for="(item2,i2) in item1.children" :key="item2.id">
                  <!-- 二级权限 -->
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="18">
                    <el-tag v-for="(item3) in item2.children" :key="item3.id" type="warning" closable
                      @close="removeRightById(scope.row,item3.id)">
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column align="center" label="#" type="index"></el-table-column>
        <el-table-column align="center" label="角色名称" prop="roleName"></el-table-column>
        <el-table-column align="center" label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column align="center" label="操作" width="320px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditRightDialog(scope.row)">编辑
            </el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="delRoleById(scope.row)">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加角色对话框 -->
    <el-dialog title="添加角色" :visible.sync="addRoleDialogVisible" width="50%" @close="addRoleDialogClosed">
      <el-form :model="addRoleForm" ref="addRoleFormRef" :rules="addRoleFormRules" label-width="80px">
        <el-form-item label="角色名:" prop="roleName">
          <el-input v-model="addRoleForm.roleName" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="角色描述:" prop="roleDesc">
          <el-input v-model="addRoleForm.roleDesc" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑角色对话框 -->
    <el-dialog title="修改角色" :visible.sync="editRoleDialogVisible" width="50%" @close="editRoleDialogClosed">
      <el-form :model="currentRole" ref="editRoleFormRef" :rules="editRoleFormRules" label-width="80px">
        <el-form-item label="角色序号:">
          <el-input v-model="roleId" disabled autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="角色名:" prop="roleName">
          <el-input v-model="currentRole.roleName" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="角色描述:" prop="roleDesc">
          <el-input v-model="currentRole.roleDesc" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配权限对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%">
      <el-tree ref="treeRef" :data="rightsList" :props="treeProps" show-checkbox node-key="id"
        :default-expand-all="true" :default-checked-keys="defKeys">

      </el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="setRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
  export default {
    name: '',
    data() {
      return {
        //角色列表
        rolesList: [],
        //所有的权限列表
        rightsList: [],
        //角色默认已有的权限id
        defKeys: [],
        //当前角色对应的id
        roleId: '',
        //select下拉框对应数据
        treeProps: {
          label: 'authName',
          children: 'children'
        },
        //分配权限对话框
        setRightDialogVisible: false,
        //添加角色对话框
        addRoleDialogVisible: false,
        //添加角色表单数据
        addRoleForm: {
          roleName: '',
          roleDesc: ''
        },
        //添加角色表单校验
        addRoleFormRules: {
          roleName: [{
            required: true,
            message: '请输入角色名',
            trigger: 'blur'
          }],
          roleDesc: [{
            required: true,
            message: '请输入角色描述',
            trigger: 'blur'
          }]
        },
        //修改角色信息对话框
        editRoleDialogVisible: false,
        //当前角色信息
        currentRole: {},
        //修改角色表单校验
        editRoleFormRules: {
          roleName: [{
            required: true,
            message: '请输入角色名',
            trigger: 'blur'
          }],
          roleDesc: [{
            required: true,
            message: '请输入角色描述',
            trigger: 'blur'
          }]
        },
      }
    },
    created() {
      this.getRolesList()
    },
    methods: {
      //获取所有角色列表
      async getRolesList() {
        const {
          data: res
        } = await this.$http.get('roles')
        if (res.meta.status !== 200)
          return this.$message.error(res.meta.msg)
        this.rolesList = res.data
      },
      //添加角色对话框
      showAddRoleDialog() {
        this.addRoleDialogVisible = true
      },
      //添加用户框重置
      addRoleDialogClosed() {
        this.$refs.addRoleFormRef.resetFields();
      },
      //修改用户框重置
      editRoleDialogClosed() {
        this.$refs.editRoleFormRef.resetFields();
      },
      addRole() {
        this.$refs.addRoleFormRef.validate(async valid => {
          if (!valid) {
            // console.log('失败');
            return this.$message.error("请填写相关信息!")
          }
          const {
            data: res
          } = await this.$http.post('roles', this.addRoleForm)
          if (res.meta.status !== 201) {
            return this.$message.error("添加角色失败!")
          }
          this.$message.success("添加角色成功!")
          //隐藏对话框
          this.addRoleDialogVisible = false
          //获取新的角色列表
          this.getRolesList()
        })
      },
      //删除某人某项权限
      async removeRightById(role, id) {
        const res = await this.$confirm('此操作将删除该权限, 是否继续?', '删除用户', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if (res !== "confirm") {
          return this.$message.info("已经取消删除")
        }
        // console.log(role);
        // console.log(id);
        const {
          data: result
        } = await this.$http.delete(`roles/${role.id}/rights/${id}`)
        // console.log(`roles/${role.id}/rights/${id}`);
        // // console.log(res);
        if (result.meta.status !== 200)
          return this.$message.error("删除权限失败!")
        this.$message.success("删除权限成功!")
        // this.getRolesList()
        role.children = result.data
      },
      //获取用户已有的权限
      async showSetRightDialog(role) {
        this.defKeys = []
        this.roleId = role.id
        const {
          data: res
        } = await this.$http.get('rights/tree')
        if (res.meta.status !== 200)
          return this.$message.error(res.meta.msg)
        this.rightsList = res.data
        // console.log(this.rightsList);
        // console.log(this.rolesList);
        this.getLeafKeys(role, this.defKeys)
        // console.log(this.defKeys);
        this.setRightDialogVisible = true
      },
      getLeafKeys(node, arr) {
        if (!node.children) {
          return arr.push(node.id)
        }
        node.children.forEach(item => {
          this.getLeafKeys(item, arr)
        })
      },
      //分配权限
      async setRights() {
        const key = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]
        const idStr = key.join(',')
        const {
          data: res
        } = await this.$http.post(`roles/${this.roleId}/rights`, {
          rids: idStr
        })
        if (res.meta.status !== 200)
          return this.$message.error(res.meta.msg)
        this.$message.success("分配权限成功")
        this.getRolesList()
        this.setRightDialogVisible = false
      },
      //获取当前角色信息
      async showEditRightDialog(role) {
        this.roleId = role.id
        const {
          data: res
        } = await this.$http.get('roles/' + role.id)
        if (res.meta.status !== 200)
          return this.$message.error(res.meta.msg)
        //console.log(res.data);
        this.currentRole = {
          roleId: res.data.roleId,
          roleName: res.data.roleName,
          roleDesc: res.data.roleDesc
        }
        this.editRoleDialogVisible = true
      },
      //提交修改后的角色信息
      async editRole() {
        this.$refs.editRoleFormRef.validate(async valid => {
          if (!valid) {
            // console.log('失败');
            return this.$message.error("请填写相关信息!")
          }
          // console.log(this.roleId);
          // console.log(this.currentRole);
          const {
            data: res
          } = await this.$http.put(`roles/${this.roleId}`, this.currentRole)
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          this.$message.success("修改角色成功!")
          //隐藏对话框
          this.getRolesList()
          this.editRoleDialogVisible = false

        })
      },
      async delRoleById(role) {
        this.roleId = role.id
        const res = await this.$confirm('此操作将删除该角色, 是否继续?', '删除角色', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if (res !== "confirm") {
          return this.$message.info("已经取消该操作")
        }
        const {
          data: result
        } = await this.$http.delete('roles/' + this.roleId)
        // console.log(res);
        if (result.meta.status !== 200)
          return this.$message.error("删除角色失败!")
        this.$message.success("删除角色成功!")
        this.getRolesList()
      }
    },
    components: {

    }
  }

</script>
<style scoped>
  .el-tag {
    margin: 7px;
  }

  .vcenter {
    display: flex;
    align-items: center;
  }

  .bdtop {
    border-top: 1px solid #eee;
  }

  .bdbottom {
    border-bottom: 1px solid #eee;
  }

</style>
