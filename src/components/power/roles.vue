<template>
  <div class="role">
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <el-row>
        <el-button type="primary" @click="addRoleDialogVisible = true">添加角色</el-button>
      </el-row>
      <!-- 表格区域 -->
      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom', li1 == 0 ? 'bdtop' : '']" v-for="(item, li1) in scope.row.children" :key="item.id">
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag type="primary" @close="removeRightById(scope.row, item.id)" closable>{{ item.authName }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二三级权限 -->
              <el-col :span="19">
                <el-row :class="[li2 === 0 ? '' : 'bdtop']" v-for="(items, li2) in item.children" :key="items.id">
                  <!--  二级权限 -->
                  <el-col :span="6">
                    <el-tag type="success" @close="removeRightById(scope.row, items.id)" closable>{{ item.authName }}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="18">
                    <el-row v-for="it in items.children" :key="it.id">
                      <el-col>
                        <el-tag type="warning" @close="removeRightById(scope.row, it.id)" closable>{{ item.authName }}</el-tag>
                        <i class="el-icon-caret-right"></i>
                      </el-col>
                    </el-row>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"> </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"> </el-table-column>
        <el-table-column label="操作" width="300">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="small" @click="edit(scope.row.id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="small" @click="delRole(scope.row.id)">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="small" @click="showRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 角色弹框 -->
    <el-dialog title="添加角色" @close="resetRole" :visible.sync="addRoleDialogVisible" width="50%">
      <el-form :model="RoleForm" :rules="addRoleRules" ref="addRoleRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="RoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="RoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑弹框 -->
    <el-dialog title="修改角色" :visible.sync="editRoleDialogVisible" width="50%">
      <el-form :model="editForm" ref="addRoleRef" label-width="80px">
        <el-form-item label="角色名称">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog @close="roleClose" title="分配权限" :visible.sync="SetRightdialogVisible" width="30%">
      <el-tree :data="rolesum" ref="treeRef" default-expand-all show-checkbox node-key="id" :default-checked-keys="defKey" :props="RoleProps"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="SetRightdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: 'role',
  data() {
    return {
      // 角色列表参数
      rolesList: [],
      // 添加弹框参数
      addRoleDialogVisible: false,
      editRoleDialogVisible: false,
      RoleForm: {
        roleName: '',
        roleDesc: ''
      },
      // 编辑弹框参数
      editForm: {},
      // 验证规则
      addRoleRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 3, max: 8, message: '长度在3~8个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 3, max: 8, message: '长度在3~8个字符', trigger: 'blur' }
        ]
      },
      // 分配权限参数
      SetRightdialogVisible: false,
      // 角色id
      roleId: '',
      // 权限列表
      rolesum: [],
      // 树形结构参数
      RoleProps: {
        children: 'children',
        label: 'authName'
      },
      // 树形结构默认展开节点
      defKey: []
    }
  },
  created() {
    // 获取角色列表
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('/roles')
      console.log('角色列表', res)
      if (res.meta.status !== 200) return this.$message.error('获取角色权限失败')
      this.rolesList = res.data
    },
    // 添加角色
    async addRole() {
      this.addRoleDialogVisible = false
      const { data: res } = await this.$http.post('/roles', this.RoleForm)
      if (res.meta.status !== 201) return this.$message.error('添加角色失败')
      this.$message.success('添加角色成功')
      this.getRolesList()
    },
    // 取消
    resetRole() {
      this.addRoleDialogVisible = false
      //   this.RoleForm = {}
      this.$refs.addRoleRef.resetFields()
    },
    // 编辑
    async editRole(id) {
      const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, this.editForm)
      if (res.meta.status !== 200) return this.$message.error('编辑用户权限失败')
      this.$message.success('编辑用户成功')
      this.getRolesList()
      this.editRoleDialogVisible = false
    },
    async edit(id) {
      this.editRoleDialogVisible = true
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('获取角色信息失败')
      this.editForm = res.data
    },
    // 删除用户权限
    async delRole(id) {
      const res = await this.$confirm('此操作将永久删除该用户权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (res === 'cancel') return this.$message.info('已经取消删除操作')
      const { data: dat } = await this.$http.delete('roles/' + id)
      if (dat.meta.status !== 200) this.$message.error('删除用户权限失败')
      this.$message.success('删除用户权限成功')
      this.getRolesList()
    },
    // 删除指定权限
    async removeRightById(role, rightId) {
      const data = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (data !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        this.$message.error('删除角色权限失败')
      }
      this.$message.success('删除角色权限成功')
      role.children = res.data
    },
    // 分配权限
    async showRightDialog(role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.info('获取权限列表失败！')
      }
      this.rolesum = res.data
      this.getDefKey(role, this.defKey)
      this.SetRightdialogVisible = true
    },
    // 树形结构展开节点
    getDefKey(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getDefKey(item, arr))
    },
    // 权限列表弹框关闭
    roleClose() {
      this.defKey = []
    },
    // 确定权限事件
    async allRights() {
      const keys = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('更新用户权限失败')
      }
      this.$message.success('分配权限成功')
      // 刷新用户权限列表
      this.getRolesList()
      this.SetRightdialogVisible = false
    }
  }
}
</script>
<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.bdtop {
  border-top: 1px solid #eee;
}
</style>
