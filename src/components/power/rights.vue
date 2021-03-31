<template>
  <div name="right">
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡边区域 -->
    <el-card>
      <el-table :data="getRightList" border stripe="">
        <el-table-column type="index"> </el-table-column>
        <el-table-column prop="authName" label="权限名称"> </el-table-column>
        <el-table-column prop="path" label="路径"> </el-table-column>
        <el-table-column prop="level" label="权限等级">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.level === '0'" type="success">一等级</el-tag>
            <el-tag v-if="scope.row.level === '1'" type="warning">二等级</el-tag>
            <el-tag v-if="scope.row.level === '2'" type="danger">三等级</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>
<script>
export default {
  name: 'right',
  data() {
    return {
      getRightList: []
    }
  },
  created() {
    this.getRight()
  },
  methods: {
    async getRight() {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) return this.$message.error('获取权限列表失败')
      this.getRightList = res.data
      console.log('权限数据', res)
    }
  }
}
</script>
<style lang="less" scoped></style>
