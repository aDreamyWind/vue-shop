<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片区域 -->
    <el-card>
      <!-- 警告 -->
      <el-alert title="注意：只允许为第三级分类设置相关参数！"
                type="warning"
                :closable="false"
                show-icon>
      </el-alert>

      <!-- 级联选择器 -->
      <el-row class="checkParams">
        <el-col>
          <span class="demonstration">选择商品分类：</span>
          <el-cascader v-model="selecttedCateKeys"
                       :options="categorieList"
                       expandTrigger='hover'
                       :props="cateProps"
                       @change="handleChange">
          </el-cascader>
        </el-col>
      </el-row>

      <!-- 标签页选择 -->
      <el-tabs v-model="activeName"
               @tab-click="handleTabClick">
        <!-- 添加动态参数 -->
        <el-tab-pane label="动态参数"
                     name="many">
          <el-button type="primary"
                     size="mini"
                     :disabled="isNotThree"
                     @click="addParamOpen">添加参数</el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData"
                    border>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染tag标签 -->
                <el-tag v-for="(val, i) in scope.row.attr_vals"
                        :key="i"
                        closable
                        @close="handleClose(i, scope.row)">{{ val }}</el-tag>
                <!-- 渲染添加tag标签 -->
                <el-input class="input-new-tag"
                          v-if="scope.row.inputVisible"
                          v-model="scope.row.inputValue"
                          ref="saveTagInput"
                          size="small"
                          @keyup.enter.native="handleInputConfirm(scope.row)"
                          @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else
                           class="button-new-tag"
                           size="small"
                           @click="showInput(scope.row)">+ New Tag</el-button>

              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"
                             label="#"></el-table-column>
            <el-table-column label="参数名称"
                             prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary"
                           size="mini"
                           icon="el-icon-edit"
                           @click="editParamOpen(scope.row)">编辑</el-button>
                <el-button type="primary"
                           size="mini"
                           icon="el-icon-delete"
                           @click="removeParams(scope.row)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>

        <!-- 添加静态属性 -->
        <el-tab-pane label="静态属性"
                     name="only">
          <el-button type="primary"
                     size="mini"
                     :disabled="isNotThree"
                     @click="addParamOpen">添加属性</el-button>
          <!-- 静态属性表格 -->
          <el-table :data="onlyTableData"
                    border>
            <!-- 展开行 -->
            <el-table-column type="expand">

            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"
                             label="#"></el-table-column>
            <el-table-column label="属性名称"
                             prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary"
                           size="mini"
                           icon="el-icon-edit"
                           @click="editParamOpen(scope.row)">编辑</el-button>
                <el-button type="primary"
                           size="mini"
                           icon="el-icon-delete"
                           @click="removeParams(scope.row)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>

    </el-card>

    <!-- 对话框-添加属性、添加参数 -->
    <el-dialog :title="'添加' + titleText"
               :visible.sync="paramsVisible"
               @close="paramClose"
               width="50%">

      <!-- 添加参数 -->
      <el-form ref="addParamRef"
               label-width="100px"
               :model="addParamJson"
               :rules="addParamRules">
        <el-form-item :label="titleText"
                      prop="attr_name">
          <el-input v-model="addParamJson.attr_name"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer"
            class="dialog-footer">
        <el-button @click="paramsVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 对话框-修改属性、修改参数 -->
    <el-dialog :title="'修改' + titleText"
               :visible.sync="editparamsVisible"
               @close="editparamClose"
               width="50%">

      <!-- 添加参数 -->
      <el-form ref="editParamRef"
               label-width="100px"
               :model="editParamJson"
               :rules="editParamRules">
        <el-form-item :label="titleText"
                      prop="attr_name">
          <el-input v-model="editParamJson.attr_name"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editparamsVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="editParams">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data () {
    return {
      // 初始化数据
      categorieList: [],
      // 级联选择框数据
      selecttedCateKeys: [],
      // 级联选择框数据
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 被激活的页签名称
      activeName: 'many',
      // 动态参数数据
      manyTableData: [],
      // 静态属性数据
      onlyTableData: [],
      // 添加属性、参数弹框展示
      paramsVisible: false,
      addParamJson: {
        attr_name: ''
      },
      addParamRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      editparamsVisible: false,
      editParamJson: {
        attr_name: '',
        attr_id: ''
      },
      editParamRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getCategorieList()
  },
  methods: {
    async getCategorieList () {
      const { data: rel } = await this.$http.get('/categories')
      if (rel.meta.status !== 200) this.$message.error('数据获取失败')
      this.categorieList = rel.data
    },
    // 级联选择框变化时
    async handleChange () {
      this.getParamsData()
    },
    // tab页签点击事件
    handleTabClick () {
      console.log('activeName', this.activeName)
      this.getParamsData()
    },
    // 获取数据
    async getParamsData () {
      // 不是三级分类
      if (this.selecttedCateKeys.length < 3) {
        this.selecttedCateKeys = []
        this.$message.warning('只允许为第三级分类设置相关参数！')
        this.selecttedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      // 根据所选分类id和所处面板，展示下面数据
      const { data: res } = await this.$http.get(`categories/${this.selecttedCateKeys.at(-1)}/attributes`, {
        params: { sel: this.activeName }
      })

      if (res.meta.status !== 200) return this.$message.error('获取数据错误')

      res.data.forEach(item => {
        item.attr_vals =
          item.attr_vals ? item.attr_vals.split(' ') : []
        item.inputVisible = false
        item.inputValue = ''
      })
      console.log(res)
      if (this.activeName === 'many') this.manyTableData = res.data
      else this.onlyTableData = res.data
    },
    // 弹框关闭
    paramClose () {
      this.$refs.addParamRef.resetFields()
      this.paramsVisible = false
      console.log(this.addParamJson.attr_name)
    },
    // 弹框开启
    addParamOpen () {
      this.paramsVisible = true
    },
    // 点击按钮，添加参数
    addParams () {
      this.$refs.addParamRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.selecttedCateKeys.at(-1)}/attributes`, {
          attr_name: this.addParamJson.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) return this.$message.error('添加失败')
        this.$message.success('添加成功')
        this.paramsVisible = false
        this.getParamsData()
      })
    },
    // 编辑参数
    editParamOpen (row) {
      console.log(row)
      this.editParamJson.attr_name = row.attr_name
      this.editParamJson.attr_id = row.attr_id
      this.editparamsVisible = true
    },
    // 弹框关闭
    editparamClose () {
      this.$refs.editParamRef.resetFields()
      this.editparamsVisible = false
      console.log(this.editParamJson.attr_name)
    },
    // 点击按钮，修改参数
    editParams () {
      this.$refs.editParamRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`categories/${this.selecttedCateKeys.at(-1)}/attributes/${this.editParamJson.attr_id}`, {
          attr_name: this.editParamJson.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 200) return this.$message.error('添加失败')
        this.$message.success('添加成功')
        this.editparamsVisible = false
        this.getParanmsData()
      })
    },
    async removeParams (row) {
      const result = await this.$confirm('将会永久删除，是否继续',
        '提示',
        {
          confirmButtonText: '确认',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
      console.log(result)
      if (result === 'cancel') return this.$message.info('已经取消')
      const { data: res } = await this.$http.delete(`categories/${this.selecttedCateKeys.at(-1)}/attributes/${row.attr_id}`)
      if (res.meta.status !== 200) return this.$message.error('失败')
      this.$message.success('已经删除成功')
      this.getParamsData()
    },
    // 文本框失去焦点或按了回车
    async handleInputConfirm (row) {
      row.inputValue = row.inputValue.trim()
      if (row.inputValue.length === 0) {
        row.inputVisible = false
        return
      }
      // 尾巴后添加数据
      row.attr_vals.push(row.inputValue)
      console.log('几次')
      this.saveAttrVals(row)

      row.inputVisible = false
    },
    async saveAttrVals (row) {
      // 发送请求
      const { data: res } = await this.$http.put(`categories/${this.selecttedCateKeys.at(-1)}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        })
      if (res.meta.status !== 200) return this.$message.error('修改失败')
      this.$message.success('修改成功')
      row.inputValue = ''
    },
    // 点击按钮切换文本框
    showInput (row) {
      row.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    handleClose (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    // 如果按钮需要被禁用，则返回true，否则返回false
    isNotThree () {
      if (this.selecttedCateKeys.length !== 3) return true
      return false
    },
    // 返回弹框内容
    titleText () {
      if (this.activeName === 'many') return '动态参数'
      else return '静态属性'
    }

  }
}
</script>

<style lang="less" scoped>
.checkParams,
.el-tabs {
  margin-top: 15px;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 100px;
}
</style>
