<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容"
                    clearable
                    @clear="queryOrders"
                    v-model="queryInfo.query">
            <el-button slot="append"
                       icon="el-icon-search"
                       @click="queryOrders"></el-button>
          </el-input>
        </el-col>
      </el-row>

      <!-- 表格 -->
      <!-- 数据表格 -->
      <el-table :data="orderList"
                stripe
                border
                style="width: 100%">
        <el-table-column type="index"
                         label="#"></el-table-column>
        <el-table-column prop="order_number"
                         label="订单编号">
        </el-table-column>
        <el-table-column prop="order_price"
                         label="订单价格"
                         width="100">
        </el-table-column>
        <el-table-column label="是否付款"
                         width="100">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.pay_status === '1'"
                    type="success">已付款</el-tag>
            <el-tag v-else
                    type="danger">未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send"
                         label="是否发货"
                         width="100">
        </el-table-column>
        <el-table-column prop="create_time"
                         label="创建时间"
                         width="150">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
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
            <el-button type="success"
                       size="mini"
                       icon="el-icon-location"
                       @click="showBox(scope.row)"
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

    <el-dialog title="修改地址"
               :visible.sync="addressVisible"
               width="50%"
               @close="addressClose">
      <el-form :model="addressForm"
               :rules="addressRules"
               label-width="80px"
               ref="addressRef">
        <el-form-item label="省市区县"
                      prop="address1">
          <el-cascader :options="cityData"
                       v-model="addressForm.address1"></el-cascader>
        </el-form-item>
        <el-form-item label="活动区域"
                      prop="address2">
          <el-input v-model="addressForm.address2"
                    placeholder="请选择活动区域">
          </el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addressVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addressVisible = false">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
import cityData from './citydata.js'

export default {
  data () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 5

      },
      orderList: [],
      total: 0,
      addressVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressRules: {
        address1: [
          { required: true, message: '请输入省市区县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请选择活动区域', trigger: 'blur' }
        ]
      },
      cityData
    }
  },
  created () {
    this.queryOrders()
  },
  methods: {
    async queryOrders () {
      const { data: res } = await this.$http.get('/orders', {
        params: this.queryInfo
      })
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error('数据响应错误')
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange (res) {
      this.queryInfo.pagenum = 1
      this.queryInfo.pagesize = res
      this.queryOrders()
    },
    handleCurrentChange (res) {
      this.queryInfo.pagenum = res
      this.queryOrders()
    },
    showBox (row) {
      this.addressVisible = true
    },
    addressClose () {
      this.$refs.addressRef.resetFields()
    }
  }
}
</script>

<style lang="less" scoped>
</style>
