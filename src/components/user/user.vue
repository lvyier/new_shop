<template>
  <div class="user">
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>活动管理</el-breadcrumb-item>
      <el-breadcrumb-item>活动列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card class="box-card">
      <!-- 搜索与确认框 -->
      <el-row :gutter="20">
        <el-col :span="9">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="dialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <el-table :data="userList" border style="width: 100%">
        <el-table-column type="index"> </el-table-column>
        <el-table-column prop="username" label="姓名"> </el-table-column>
        <el-table-column prop="email" label="邮箱"> </el-table-column>
        <el-table-column prop="mobile" label="电话"> </el-table-column>
        <el-table-column prop="role_name" label="角色"> </el-table-column>
        <el-table-column prop="mg_state" label="状态">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)"> </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <!-- 修改 -->
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="editUserForms(scope.row.id)"></el-button>
            <!-- 删除 -->
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteUser(scope.row.id)"></el-button>
            <!-- 权限 -->
            <el-tooltip class="item" effect="dark" :enterable="false" content="分配角色" placement="top-start">
              <el-button type="warning" icon="el-icon-setting" size="mini" @click="selectRole(scope.row)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 添加用户弹框 -->
    <el-dialog title="添加用户" :visible.sync="dialogVisible" @close="addDialogClosed" width="50%">
      <el-form :model="addUserForm" :rules="addRules" ref="addRuleForm" label-width="70px">
        <el-form-item label="姓名" prop="username">
          <el-input v-model="addUserForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addUserForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addUserForm.email"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="addUserForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="resetForm">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户弹框 -->
    <el-dialog title="修改用户" :visible.sync="editdialogVisible" @close="editDialogClosed" width="50%">
      <el-form :model="EditUserForm" :rules="EditFormRules" ref="EditFormRef" label-width="70px">
        <el-form-item label="姓名">
          <el-input v-model="EditUserForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="EditUserForm.mobile"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="EditUserForm.email"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="resetEditForm">取 消</el-button>
        <el-button type="primary" @click="EditUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配角色弹框 -->
    <el-dialog title="分配角色" :visible.sync="RoledialogVisible" width="50%" @close="roleClose">
      <div>
        <p>当前的用户：{{ userInfo.username }}</p>
        <p>当前的角色：{{ userInfo.role_name }}</p>
        <p>
          分配新角色:
          <el-select v-model="selectedRoleId" placeholder="请选择">
            <el-option v-for="item in roleList" :key="item.id" :label="item.roleName" :value="item.id"> </el-option>
          </el-select>
        </p>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="RoledialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: 'user',
  data() {
    // 自定义验证 邮箱
    var checkEmail = (rule, value, cb) => {
      const regEmail = /^\w+@\w+(\.\w+)+$/
      if (regEmail.test(value)) {
        // 验证成功 返回一个回调函数
        return cb()
      }
      // 返回错误提示信息
      cb(new Error('请输入合法的邮箱'))
    }
    // 自定义验证 电话
    var checkMobile = (rule, value, cb) => {
      const regMobile = /^1[34578]\d{9}$/
      if (regMobile.test(value)) {
        // 验证成功 返回一个回调函数
        return cb()
      }
      // 返回错误提示信息
      cb(new Error('请输入合法的电话'))
    }
    return {
      // 列表数据
      userList: [],
      // 参数
      queryInfo: {
        query: '',
        // 当前页
        pagenum: 1,
        // 每页的数据
        pagesize: 2
      },
      // 总数量
      total: 0,
      // 弹框参数
      dialogVisible: false,
      // 添加用户数据
      addUserForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加用户表单验证信息
      addRules: {
        // 用户名验证
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 ~ 10个字符', trigger: 'blur' }
        ],
        // 密码验证
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 ~ 18 个字符', trigger: 'blur' }
        ],
        // 验证邮箱
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { min: 6, max: 18, message: '长度在6~18个字符', trigger: 'blur' },
          {
            validator: checkEmail,
            required: true,
            message: '邮箱格式不正确请重新输入',
            trigger: 'blur'
          }
        ],
        // 验证电话
        mobile: [
          { required: true, message: '请输入电话', trigger: 'blur' },
          { min: 8, max: 11, message: '长度在8~12个字符', trigger: 'blur' },
          {
            validator: checkMobile,
            message: '电话格式不正确请重新输入',
            trigger: 'blur'
          }
        ]
      },
      // 编辑用户验证信息
      EditFormRules: {
        // 验证邮箱
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { min: 6, max: 18, message: '长度在6~18个字符', trigger: 'blur' },
          {
            validator: checkEmail,
            required: true,
            message: '邮箱格式不正确请重新输入',
            trigger: 'blur'
          }
        ],
        // 验证电话
        mobile: [
          { required: true, message: '请输入电话', trigger: 'blur' },
          { min: 8, max: 11, message: '长度在8~12个字符', trigger: 'blur' },
          {
            validator: checkMobile,
            message: '电话格式不正确请重新输入',
            trigger: 'blur'
          }
        ]
      },
      // 修改弹框参数
      editdialogVisible: false,
      // 编辑用户信息
      EditUserForm: {},
      // 分配角色弹框参数
      RoledialogVisible: false,
      // 角色信息
      userInfo: {},
      // 所有角色列表
      roleList: [],
      // 保存用户选中的id
      selectedRoleId: ''
    }
  },
  created() {
    // 获取用户列表
    this.getUserList()
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) return this.$message.error('获取列表失败')
      this.userList = res.data.users
      this.total = res.data.total
    },
    // 页数发生变化事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 当前页发生变化
    handleCurrentChange(newnum) {
      this.queryInfo.pagenum = newnum
      this.getUserList()
    },
    // swith切换事件
    async userStateChange(row) {
      const { data: res } = await this.$http.put(`users/${row.id}/state/${row.mg_state}`)
      if (res.meta.status !== 200) {
        row.mg_state = !row.mg_state
        return this.$message.error('用户状态修改失败')
      }
      this.$message.success('用户状态修改成功')
      this.getUserList()
    },
    // 重置添加用户表单
    resetForm() {
      this.$refs.addRuleForm.resetFields()
      this.dialogVisible = false
    },
    // 添加用户
    addUser() {
      this.$refs.addRuleForm.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addUserForm)
        if (res.meta.status !== 201) return this.$message.error('添加用户失败')
        this.$message.success('添加用户成功')
        this.dialogVisible = false
        this.getUserList()
      })
    },
    // 添加用户弹框关闭
    addDialogClosed() {
      this.$refs.addRuleForm.resetFields()
    },
    // 编辑用户弹框关闭
    editDialogClosed() {
      this.$refs.EditFormRef.resetFields()
    },
    // 取消编辑用户信息
    resetEditForm() {
      this.editdialogVisible = false
      this.$refs.EditFormRef.resetFields()
    },
    // 修改用户操作
    async editUserForms(id) {
      console.log('用户id', id)
      this.editdialogVisible = true
      const { data: res } = await this.$http.get(`users/${id}`)
      if (res.meta.status !== 200) return this.message.error('获取用户信息失败')
      this.EditUserForm = res.data
    },
    // 确定编辑用户信息
    EditUser() {
      this.$refs.EditFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`users/${this.EditUserForm.id}`, {
          mobile: this.EditUserForm.mobile,
          email: this.EditUserForm.email
        })
        if (res.meta.status !== 200) return this.$message.error('更新用户失败')
        this.$message.success('更新用户成功')
        this.getUserList()
        this.editdialogVisible = false
      })
    },
    // 删除用户
    async deleteUser(id) {
      const comfirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (comfirmResult !== 'confirm') {
        return this.$message.info('已经取消删除！')
      }
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.info('删除用户失败')
      }
      this.$message.success('删除用户成功')
      this.getUserList()
    },
    // 分配角色事件
    async selectRole(userInfo) {
      // 分配角色
      this.userInfo = userInfo
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.roleList = res.data
      this.RoledialogVisible = true
    },
    // 确定分配角色事件
    async saveRoleInfo() {
      if (!this.selectedRoleId) {
        return this.$message.error('请选择要分配的角色')
      }
      // 发送请求，保存用户角色
      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, {
        rid: this.selectedRoleId
      })
      if (res.meta.status !== 200) {
        return this.$message.error('分配角色失败')
      }
      this.$message.success('分配角色成功')
      this.getUserList()
      this.RoledialogVisible = false
    },
    roleClose() {
      this.userInfo = {}
      this.selectedRoleId = ''
    }
  }
}
</script>
<style lang="less" scoped>
.el-pagination {
  margin-top: 20px;
}
</style>
