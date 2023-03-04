<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容"
                    clearable
                    @clear="queryGoods"
                    v-model="queryInfo.query">
            <el-button slot="append"
                       icon="el-icon-search"
                       @click="queryGoods"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary"
                     @click="goAddpage">添加商品</el-button>
        </el-col>
      </el-row>

      <!-- 数据表格 -->
      <el-table :data="goods"
                stripe
                border
                style="width: 100%">
        <el-table-column type="index"
                         label="#"></el-table-column>
        <el-table-column prop="goods_name"
                         label="商品名称">
        </el-table-column>
        <el-table-column prop="goods_price"
                         label="商品价格(元)"
                         width="100">
        </el-table-column>
        <el-table-column prop="goods_weight"
                         label="商品重量"
                         width="100">
        </el-table-column>
        <el-table-column prop="add_time"
                         label="创建时间"
                         width="150">
          <template slot-scope="scope">
            {{ scope.row.add_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作"
                         width="90">
          <template slot-scope="scope">
            <el-button type="primary"
                       size="mini"
                       icon="el-icon-edit"
                       @click="modifyGoods(scope.row)"
                       circle></el-button>
            <el-button type="danger"
                       size="mini"
                       icon="el-icon-delete"
                       @click="deleteGoods(scope.row)"
                       circle></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页 -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[5, 10, 15, 20]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     background
                     :total="total">
      </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      goods: []
    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    // 根据分页获取对应商品列表
    async getGoodsList () {
      const { data: res } = await this.$http.get('/goods', {
        params: this.queryInfo
      })
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error('获取数据失败')
      this.goods = res.data.goods
      this.total = res.data.total
    },
    // 点击查询时
    queryGoods () {
      this.queryInfo.pagenum = 1
      this.getGoodsList()
    },
    // 当size改变
    handleSizeChange (val) {
      this.queryInfo.pagesize = val
      this.queryInfo.pagenum = 1
      this.getGoodsList()
    },
    // 当前页改变
    handleCurrentChange (val) {
      this.queryInfo.pagenum = val
      this.getGoodsList()
    },
    // 删除商品
    async deleteGoods (row) {
      const confi = await this.$confirm('将会删除商品是否继续', '提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confi !== 'confirm') {
        return this.$message.info('取消了删除')
      }
      const { data: res } = await this.$http.delete(`/goods/${row.goods_id}`)
      if (res.meta.status !== 200) return this.$message.error('删除出错')
      this.$message.success('成功删除')
      this.getGoodsList()
    },
    // 跳转到添加商品
    goAddpage () {
      this.$router.push('/goods/add')
    }
  }
}
</script>

<style lang="less" scoped>
</style>
