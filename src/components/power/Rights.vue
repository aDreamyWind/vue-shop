<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <el-table :data="rightsList"
                border
                stripe
                style="width: 100%">
        <el-table-column type="index"
                         label="#">
        </el-table-column>
        <el-table-column prop="authName"
                         label="权限名称">
        </el-table-column>
        <el-table-column prop="path"
                         label="路径">
        </el-table-column>
        <el-table-column prop="level"
                         label="权限等级">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.level === '0'">一级</el-tag>
            <el-tag v-if="scope.row.level === '1'"
                    type="success">二级</el-tag>
            <el-tag v-if="scope.row.level === '2'"
                    type="warning">三级</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 权限列表
      rightsList: []
    }
  },
  created () {
    // 获取所有权限
    this.getRightsList()
  },
  methods: {
    async getRightsList () {
      const { data: rel } = await this.$http.get('rights/list')
      if (rel.meta.status !== 200) return this.$message.error('获取数据错误')
      this.rightsList = rel.data
    }
  }

}
</script>

<style lang="less" scoped>
</style>
