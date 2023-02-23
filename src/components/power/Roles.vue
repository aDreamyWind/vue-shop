<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>角色管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <!-- 添加角色按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 添加角色列表 -->
      <el-table :data="rolelist"
                border
                stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row v-for="(item1, i1) in scope.row.children"
                    :key="item1.id"
                    :class="['vcenter', 'bdbottom', i1 === 0 ? 'bdtop' : '']">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable
                        @close="removeRightById(scope.row, item1.id)">{{ item1.authName }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级三级权限 -->
              <el-col :span="19">
                <el-row v-for="(item2, i2) in item1.children"
                        :key="item2.id"
                        :class="['vcenter', i2 !== 0 ? 'bdtop' : '']">
                  <!-- 渲染二级权限 -->
                  <el-col :span="6">
                    <el-tag closable
                            @close="removeRightById(scope.row, item2.id)"
                            type="success">{{ item2.authName }}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 渲染三级权限 -->
                  <el-col :span="18">
                    <el-tag closable
                            @close="removeRightById(scope.row, item3.id)"
                            type="warning"
                            v-for="item3 in item2.children"
                            :key="item3.id">{{ item3.authName }}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>

        <el-table-column type="index"
                         label="#"></el-table-column>
        <el-table-column label="角色名称"
                         prop="roleName"></el-table-column>
        <el-table-column label="角色负责人"
                         prop="roleDesc"></el-table-column>
        <el-table-column label="操作"
                         width="300px">
          <template slot-scope="scope">
            <el-button size="mini"
                       type="primary"
                       icon="el-icon-edit"
                       @click="delete(scope.row)">编辑</el-button>
            <el-button size="mini"
                       type="danger"
                       icon="el-icon-delete"
                       @click="delete(scope.row)">删除</el-button>
            <el-button size="mini"
                       type="warning"
                       icon="el-icon-setting"
                       @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 分配权限弹出框 -->
    <el-dialog title="分配权限"
               :visible.sync="setRightDialogVisible"
               width="50%"
               :before-close="setRightDialogClose">
      <el-tree :data="rightsTree"
               show-checkbox
               node-key="id"
               ref="treeRef"
               default-expand-all
               :default-checked-keys="defKeys"
               :props="defaultProps">
      </el-tree>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="setRightDialogClose">取 消</el-button>
        <el-button type="primary"
                   @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolelist: [],
      // 分配权限弹出框展示与不展示
      setRightDialogVisible: false,
      rightsTree: [],
      defaultProps: {
        children: 'children',
        label: 'authName'
      },
      defKeys: [],
      // 角色id
      roleId: ''
    }
  },
  created () {
    this.getRoles()
  },
  methods: {
    async getRoles () {
      const { data: rel } = await this.$http.get('roles')
      if (rel.meta.status !== 200) return this.$message.error('无法获取角色数据')
      this.rolelist = rel.data
    },
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除')
      }

      const { data: rel } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (rel.meta.status !== 200) return this.$message.error('删除失败')
      this.$message.info('删除成功')
      role.children = rel.data
    },
    async showSetRightDialog (row) {
      // 获取所有权限数据，树形结构
      const { data: rel } = await this.$http.get('/rights/tree')
      if (rel.meta.status !== 200) this.$message.error('获取数据错误')
      this.rightsTree = rel.data
      this.getLeafKeys(row, this.defKeys)
      this.roleId = row.id
      // row.children.forEach(item1 => {
      //   item1.children.forEach(item2 => {
      //     item2.children.forEach(item3 => {
      //       this.defKeys.push(item3.id)
      //     })
      //   })
      // })
      // 展示分配权限的对话框
      this.setRightDialogVisible = true
    },
    setRightDialogClose () {
      this.setRightDialogVisible = false
      this.defKeys = []
      this.roleId = ''
    },
    getLeafKeys (node, arr) {
      // 递归获取三级id
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    async allotRights () {
      // 分配角色权限
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: rel } = await this.$http.post(`roles/${this.roleId}/rights`, {
        rids: idStr
      })
      if (rel.meta.status !== 200) this.$message.error('获取数据错误')
      this.$message.success('成功')
      this.setRightDialogClose()
      this.getRoles()
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
