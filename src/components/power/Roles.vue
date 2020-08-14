<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色 -->
      <el-row>
        <el-button type="primary" @click="addRoles()">添加角色</el-button>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="rolesList" stripe border>
        <el-table-column type="index"></el-table-column>
        <el-table-column type="expand">
          <!-- 权限展开区域 -->
          <template slot-scope="scope">
            <el-row v-for="item in scope.row.children" :key="item.id" class="outer vcenter">
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item.id)">{{item.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <!-- 二级权限 -->
                <el-row v-for="item2 in item.children" :key="item2.id" class="inner vcenter">
                  <el-col :span="6">
                    <el-tag
                      closable
                      type="warning"
                      @close="removeRightById(scope.row, item2.id)"
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="19">
                    <el-tag
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      type="success"
                      @close="removeRightById(scope.row, item3.id)"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作" width="300">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row)"
            >编辑</el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="deleteRoles(scope.row)"
            >删除</el-button>
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showSetRightDialog(scope.row)"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog title="添加角色" :visible.sync="addRolesVisible" width="30%" @close="resetNewRoles">
      <!-- 内容主体区 -->
      <el-form
        :model="newRoles"
        label-position="right"
        label-width="80px"
        ref="newRolesFormRef"
        :rules="newRolesRules"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="newRoles.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="newRoles.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 操作按钮 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRolesVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRolesData">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑角色对话框 -->
    <el-dialog title="编辑角色" :visible.sync="editRolesVisible" width="30%">
      <!-- 内容主体区 -->
      <el-form
        :model="editRoles"
        label-position="right"
        label-width="80px"
        ref="editRolesFormRef"
        :rules="newRolesRules"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRoles.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoles.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 操作按钮 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRolesVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRolesData">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightVisible" width="50%">
      <!-- 树形控件 -->
      <el-tree
        :data="rightsList"
        :props="defaultProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="roleRightsList"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      rolesList: [],
      addRolesVisible: false,
      newRoles: {
        roleName: '',
        roleDesc: ''
      },
      newRolesRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ]
      },
      editRolesVisible: false,
      editRoles: {
        roleName: '',
        roleDesc: '',
        id: ''
      },
      setRightVisible: false,
      rightsList: [],
      roleRightsList: [],
      defaultProps: {
        children: 'children',
        label: 'authName'
      },
      roleId: ''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('/roles')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.rolesList = res.data
    },
    addRoles() {
      this.addRolesVisible = !this.addRolesVisible
    },
    resetNewRoles() {
      this.$refs.newRolesFormRef.resetFields()
    },
    addRolesData() {
      this.$refs.newRolesFormRef.validate(async valid => {
        if (valid) {
          const { data: res } = await this.$http.post('roles', this.newRoles)
          if (res.meta.status !== 201) {
            return this.$message.error(res.meta.msg)
          }
          this.addRolesVisible = false
          this.getRolesList()
        } else {
          this.$message.error('请检查输入内容')
          return false
        }
      })
    },
    showEditDialog(role) {
      this.editRolesVisible = true
      this.editRoles.roleName = role.roleName
      this.editRoles.roleDesc = role.roleDesc
      this.editRoles.id = role.id
      this.newRoles.roleName = role.roleName
      this.newRoles.roleDesc = role.roleDesc
    },
    editRolesData() {
      this.$refs.editRolesFormRef.validate(async valid => {
        if (valid) {
          if (
            this.editRoles.roleName === this.newRoles.roleName &&
            this.editRoles.roleDesc === this.newRoles.roleDesc
          ) {
            return this.$message.warning('角色信息未修改')
          }
          const { data: res } = await this.$http.put(
            'roles/' + this.editRoles.id,
            {
              roleName: this.editRoles.roleName,
              roleDesc: this.editRoles.roleDesc
            }
          )
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          this.editRolesVisible = false
          this.$message.success('修改成功')
          this.getRolesList()
        }
      })
    },
    deleteRoles(role) {
      this.$confirm('此操作将会永久删除角色，是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete('roles/' + role.id)
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          this.$message.success('删除成功')
          this.getRolesList()
        })
        .catch(() => {
          this.$message('已取消')
        })
    },
    removeRightById(role, rightId) {
      this.$confirm('是否删除此权限？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete(
            'roles/' + role.id + '/rights/' + rightId
          )
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          role.children = res.data
          this.$message.success('删除成功')
        })
        .catch(() => {
          this.$message('已取消')
        })
    },
    async showSetRightDialog(role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.rightsList = res.data
      console.log(this.rightsList)
      this.roleRightsList = []
      this.getRoleRightId(role)
      console.log(this.roleRightsList)
      this.setRightVisible = true
    },
    getRoleRightId(role) {
      if (role.children) {
        role.children.forEach(item => {
          this.getRoleRightId(item)
        })
      } else {
        this.roleRightsList.push(role.id)
      }
    },
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(
        'roles/' + this.roleId + '/rights',
        {
          rids: idStr
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
      this.getRolesList()
      this.setRightVisible = false
    }
  }
}
</script>
<style lang="less" scoped>
.outer {
  padding-bottom: 10px;
  padding-top: 10px;
  border-bottom: 1px solid #eee;
  &:first-child {
    border-top: 1px solid #eee;
  }
}
.inner {
  padding-bottom: 10px;
  padding-top: 10px;
  border-bottom: 1px solid #eee;
  &:last-child {
    padding-bottom: 0;
    border-bottom: none;
  }
  &:first-child {
    padding-top: 0;
  }
}
.el-tag {
  margin-bottom: 5px;
  margin-right: 5px;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
