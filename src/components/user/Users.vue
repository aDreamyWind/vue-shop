<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card class="box-card">

      <el-row :gutter="20">
        <el-col :span="7">
          <!-- 搜索与添加区域 -->
          <el-input placeholder="请输入内容">
            <el-button slot="append"
                       icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary"
                     @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>

      <!-- 用户列表区 -->
      <el-table :data="userlist"
                style="width: 100%"
                border
                stripe>
        <el-table-column type="index">
        </el-table-column>
        <el-table-column prop="username"
                         label="姓名">
        </el-table-column>
        <el-table-column prop="email"
                         label="邮箱">
        </el-table-column>
        <el-table-column prop="mobile"
                         label="电话">
        </el-table-column>
        <el-table-column label="角色"
                         prop="role_name">
        </el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state"
                       @change="userStateChange(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <!-- {{ scope.row }} -->
            <!-- 修改 -->
            <el-button type="primary"
                       icon="el-icon-edit"
                       circle
                       size="mini"
                       @click="showEditDialog(scope.row)" />
            <!-- 删除 -->
            <el-button type="danger"
                       icon="el-icon-delete"
                       circle
                       size="mini"
                       @click="deleteUser(scope.row.id)" />
            <!-- 分配角色 -->
            <el-tooltip effect="dark"
                        content="分配角色"
                        placement="top"
                        :enterable="false">
              <el-button type="warning"
                         icon="el-icon-setting"
                         circle
                         size="mini"
                         @click="setRoles(scope.row)" />
            </el-tooltip>

          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区 -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[1, 2, 5, 10]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>

      <!-- 添加用户对话框 -->
      <el-dialog title="添加用户"
                 :visible.sync="addDialogVisible"
                 width="50%"
                 @close="addDialogClose">
        <!-- 内容主题区 -->
        <span>
          <el-form label-position="right"
                   label-width="70px"
                   :model="addForm"
                   :rules="addFormRules"
                   ref="addFormRef">
            <el-form-item label="用户名"
                          prop="username">
              <el-input v-model="addForm.username"></el-input>
            </el-form-item>
            <el-form-item label="密码"
                          prop="password">
              <el-input v-model="addForm.password"></el-input>
            </el-form-item>
            <el-form-item label="邮箱"
                          prop="email">
              <el-input v-model="addForm.email"></el-input>
            </el-form-item>
            <el-form-item label="手机"
                          prop="mobile">
              <el-input v-model="addForm.mobile"></el-input>
            </el-form-item>
          </el-form>
        </span>
        <!-- 底部区 -->
        <span slot="footer"
              class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary"
                     @click="adduser">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 修改用户对话框 -->
      <el-dialog title="修改用户"
                 :visible.sync="editeDialogVisible"
                 width="50%"
                 @close="editeDialogClose">
        <!-- 内容主题区 -->
        <span>
          <el-form label-position="right"
                   label-width="70px"
                   :model="editForm"
                   :rules="addFormRules"
                   ref="editFormRef">
            <el-form-item label="用户名">
              <el-input v-model="editForm.username"
                        disabled></el-input>
            </el-form-item>
            <el-form-item label="邮箱"
                          prop="email">
              <el-input v-model="editForm.email"></el-input>
            </el-form-item>
            <el-form-item label="手机"
                          prop="mobile">
              <el-input v-model="editForm.mobile"></el-input>
            </el-form-item>
          </el-form>
        </span>
        <!-- 底部区 -->
        <span slot="footer"
              class="dialog-footer">
          <el-button @click="editeDialogVisible = false">取 消</el-button>
          <el-button type="primary"
                     @click="edituser">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 分配角色对话框 -->
      <el-dialog title="分配角色"
                 :visible.sync="setRoleDialogVisible"
                 width="50%"
                 @close="handleSetRoleDialogClose">
        <div>
          <p>当前的用户：{{ this.currentUser.username }}</p>
          <p>当前的角色：{{ this.currentUser.role_name }}</p>
          <p>
            分配新角色：
            <el-select v-model="seletedRoleId"
                       placeholder="请选择">
              <el-option v-for="item in rolesList"
                         :key="item.id"
                         :label="item.roleName"
                         :value="item.id">
              </el-option>
            </el-select>
          </p>
        </div>
        <span slot="footer"
              class="dialog-footer">
          <el-button @click="handleSetRoleDialogClose">取 消</el-button>
          <el-button type="primary"
                     @click="saveRoleInfo">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    // 验证邮箱的规则
    // const checkEmail = (rule, value, cb) => {
    //   // 验证邮箱的正则
    //   const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
    //   if (regEmail.test(value)) {
    //     return cb()
    //   }
    //   cb(new Error('请输入合法的邮箱'))
    // }

    // // 验证手机号的规则
    // const checkMobile = (rule, value, cb) => {
    //   // 验证手机号的正则
    //   const regMobile = /^([0|86|17951])?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}&/
    //   if (regMobile.test(value)) {
    //     return cb()
    //   }
    //   cb(new Error('请输入合法的手机号'))
    // }

    return {
      // 获取用户列表参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      userlist: [],
      total: 0,
      // 控制添加用户显示与隐藏
      addDialogVisible: false,
      editeDialogVisible: false,
      addForm: {
        // 添加用户表单数据
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editForm: {
        // 添加用户表单数据
        username: '',
        email: '',
        mobile: ''
      },
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名3-10', trigger: 'blur' }
        ],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' }
          // { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' }
          // { validator: checkMobile, trigger: 'blur' }
        ]
      },
      setRoleDialogVisible: false,
      currentUser: {},
      // 所有角色的数据列表
      rolesList: [],
      // 选中的角色
      seletedRoleId: ''
    }
  },
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取用户列表失败')
      this.total = res.data.total
      this.userlist = res.data.users
    },
    handleSizeChange (val) {
      // 监听pagesize、
      this.queryInfo.pagesize = val
      this.queryInfo.pagenum = 1
      this.getUserList()
      console.log(`每页 ${val} 条`)
    },
    handleCurrentChange (val) {
      this.queryInfo.pagenum = val
      this.getUserList()
      // 监听页码值
      console.log(`当前页: ${val}`)
    },
    async userStateChange (row) {
      const { data: rel } = await this.$http.put(`users/${row.id}/state/${row.mg_state}`)
      if (rel.meta.status !== 200) {
        row.mg_state = !row.mg_state
        this.$message.error('更新用户状态失败')
      }
    },
    addDialogClose () {
      // 添加用户对话框的关闭
      this.$refs.addFormRef.resetFields()
    },
    adduser () {
      this.$refs.addFormRef.validate(async valid => {
        if (valid) {
          const { data: rel } = await this.$http.post('users', this.addForm)
          console.log(rel)
          if (rel.meta.status !== 201) {
            return this.$message.error('添加用户失败')
          }
          this.$message.success('添加用户成功')
          this.addDialogVisible = false
          this.getUserList()
        }
      })
    },
    async showEditDialog (row) {
      // 调用接口获取用户数据
      const { data: rel } = await this.$http.get(`users/${row.id}`)
      if (rel.meta.status !== 200) return this.$message.error('无法获取用户数据')
      // 展示编辑用户对话框
      this.editeDialogVisible = true
      this.editForm = rel.data
      // this.editForm.username = row.username
      // this.editForm.email = row.username
      // this.editForm.mobile = row.username
    },
    editeDialogClose () {
      this.$refs.editFormRef.resetFields()
    },
    edituser () {
      this.$refs.editFormRef.validate(async valid => {
        if (valid) {
          const { data: rel } = await this.$http.put(`users/${this.editForm.id}`,
            { email: this.editForm.email, mobile: this.editForm.mobile }
          )
          if (rel.meta.status !== 200) return this.$message.error('更新数据出错')
          this.editeDialogVisible = false
          this.getUserList()
          this.$message.success('更新数据成功')
        }
      })
    },
    async deleteUser (id) {
      // 删除用户
      const result = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)

      if (result !== 'confirm') return this.$message.info('已经取消删除')

      const { data: res } = await this.$http.delete(`users/${id}`)
      if (res.meta.status !== 200) { return this.$message.success('删除失败') }
      this.getUserList()
      this.$message.success('成功删除')
    },
    handleSetRoleDialogClose () {
      // 分配角色视图关闭
      this.setRoleDialogVisible = false
      this.currentUser = {}
      this.rolesList = []
      this.seletedRoleId = ''
    },
    async setRoles (row) {
      // 分配角色视图打开并初始化数据
      this.currentUser = row
      // 在对话框之前，获取所有角色列表
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取列表失败')

      this.rolesList = res.data

      this.setRoleDialogVisible = true
    },
    async saveRoleInfo () {
      // 点击按钮分配角色
      if (!this.seletedRoleId) return this.$message.error('请选择用户角色')
      const { data: res } = await this.$http.put(`users/${this.currentUser.id}/role`, {
        rid: this.seletedRoleId
      })
      if (res.meta.status !== 200) return this.$message.error('分配角色失败')
      this.setRoleDialogVisible = false
      // this.handleSetRoleDialogClose()
      this.$message.success('分配角色成功')
      this.getUserList()
    }
  },
  created () {
    this.getUserList()
  }

}
</script>

<style lang="less" scoped>
</style>
