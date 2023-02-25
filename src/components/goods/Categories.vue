<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片区 -->
    <el-card>

      <!-- 按钮 -->
      <el-button type="primary"
                 @click="handleAddCategorie">添加分类</el-button>

      <!-- 数据 -->
      <tree-table class="treeTable"
                  :columns="columns"
                  show-index
                  index-text="#"
                  :selection-type="false"
                  :expand-type="false"
                  border
                  :show-row-hover="false"
                  :data="categorieList">
        <!-- 操作模板 -->
        <template slot="deleted"
                  slot-scope="scope">
          <!-- 是否有效标签 -->
          <i v-if="scope.row.cat_deleted"
             style="color: lightcoral;"
             class="el-icon-error"></i>
          <i v-else
             style="color: lightgreen;"
             class="el-icon-success"></i>
        </template>

        <!-- 排序模板 -->
        <template slot="level"
                  slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0"
                  size="mini">一级</el-tag>
          <el-tag v-else-if="scope.row.cat_level === 1"
                  size="mini"
                  type='success'>二级</el-tag>
          <el-tag v-else-if="scope.row.cat_level === 2"
                  size="mini"
                  type='warning'>三级</el-tag>
        </template>

        <!-- 操作模板 -->
        <template slot="opt"
                  slot-scope="scope">
          <el-button type="primary"
                     size="mini"
                     icon="el-icon-edit">编辑</el-button>
          <el-button type="danger"
                     size="mini"
                     icon="el-icon-delete"
                     @click="deleteCategorie(scope.row.cat_id)">删除</el-button>
        </template>
      </tree-table>

      <!-- 分页 -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[3, 5, 10, 15]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>

    </el-card>

    <!-- 添加分类对话框 -->
    <el-dialog title="添加分类"
               :visible.sync="addCategorieVisible"
               width="50%"
               @close="addCategorieClose">

      <!-- 添加分类表单 -->
      <el-form ref="addCategorieRef"
               :model="addCategorieJson"
               label-width="100px">
        <el-form-item label="分类名称："
                      prop="cat_name">
          <el-input v-model="addCategorieJson.cat_name"></el-input>
        </el-form-item>

        <el-form-item label="父级分类：">
          <!-- options指定数据源 -->
          <el-cascader v-model="checkdKeys"
                       :options="parentCateList"
                       :props="cascaderProps"
                       @change="handleParendChange"
                       clearable
                       change-on-select></el-cascader>

          <!-- <el-select v-model="form"
                     placeholder="请选择">
            <el-option v-for="item in parentCateList"
                       :key="item.cat_id"
                       :label="item.cat_name"
                       :value="item.cat_level"></el-option>
          </el-select> -->
        </el-form-item>
      </el-form>

      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addCategorieVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addCategorie">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data () {
    return {
      // 商品分类数组
      categorieList: [],
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      // 为table指定列
      columns: [
        {
          prop: 'cat_name',
          label: '分类名称'
        },
        {
          label: '是否有效',
          // 当前列定义为模板列
          type: 'template',
          // 当前列使用的模板名称
          template: 'deleted'
        },
        {
          label: '排序',
          // 当前列定义为模板列
          type: 'template',
          // 当前列使用的模板名称
          template: 'level'
        },
        {
          label: '操作',
          // 当前列定义为模板列
          type: 'template',
          // 当前列使用的模板名称
          template: 'opt'
        }
      ],
      // 添加分类对话框展开与关闭
      addCategorieVisible: false,
      addCategorieJson: {
        // 父级分类id
        cat_pid: 0,
        cat_level: 0,
        cat_name: ''
      },
      // 父级分类数组
      parentCateList: [],
      cascaderProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      // 选中的数组
      checkdKeys: []

    }
  },
  created () {
    this.getCategorieList()
  },
  methods: {
    async getCategorieList () {
      const { data: rel } = await this.$http.get('/categories', {
        params: this.queryInfo
      })
      if (rel.meta.status !== 200) this.$message.error('数据获取失败')
      this.categorieList = rel.data.result
      // 总数据条数
      this.total = rel.data.total
    },
    handleSizeChange (newSize) {
      // 改变了size
      this.queryInfo.pagesize = newSize
      this.queryInfo.pagenum = 1
      this.getCategorieList()
    },
    handleCurrentChange (currentNum) {
      // console.log('currentNum', currentNum)
      // 改变当前页
      this.queryInfo.pagenum = currentNum
      this.getCategorieList()
    },
    // 添加对话框关闭
    addCategorieClose () {
      console.log(this.$refs.addCategorieRef)
      this.$refs.addCategorieRef.resetFields()

      console.log(this.$refs.addCategorieRef)
      this.addCategorieVisible = false
      this.addCategorieJson.cat_pid = 0
      this.addCategorieJson.cat_level = 0
      this.checkdKeys = []
    },
    // 添加分类-打开对话框
    handleAddCategorie () {
      this.getparentCateList()
      this.addCategorieVisible = true
    },
    // 添加分类弹框-确定
    async addCategorie () {
      console.log(this.addCategorieJson)
      const { data: res } = await this.$http.post('categories', this.addCategorieJson)
      if (res.meta.status !== 201) return this.$message.error('添加数据失败')
      this.$message.success('添加数据成功')
      this.getCategorieList()
      this.addCategorieClose()
    },
    async getparentCateList () {
      // 获取父级分类的数据列表
      const { data: res } = await this.$http.get('/categories', {
        params: {
          type: 2
        }
      })

      if (res.meta.status !== 200) return this.$message.error('获取数据失败')
      this.parentCateList = res.data
      console.log(this.parentCateList)
    },
    // 选择项发生变化时触发
    handleParendChange () {
      if (this.checkdKeys.length > 0) {
        // 当前分类等级
        this.addCategorieJson.cat_level = this.checkdKeys.length
        // 父级分类id
        this.addCategorieJson.cat_pid = this.checkdKeys.at(-1)
      } else {
        this.addCategorieJson.cat_level = 0
        this.addCategorieJson.cat_pid = 0
      }
    }

  }
}
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
