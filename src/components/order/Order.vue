<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>

      <!-- 订单列表数据 -->
      <el-table :data="orderList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column label="订单价格" prop="order_price"></el-table-column>
        <el-table-column label="是否付款" prop="pay_status">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status==='1'">已付款</el-tag>
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send"></el-table-column>
        <el-table-column label="下单时间" prop="create_time">
          <template slot-scope="scope">{{scope.row.create_time | dateFormat}}</template>
        </el-table-column>
        <el-table-column label="操作">
          <template>
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showBox"></el-button>
            <el-button type="success" icon="el-icon-location" size="mini" @click="showProgressBox"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background
      ></el-pagination>
    </el-card>
    <!-- 修改地址对话框 -->
    <el-dialog title="修改地址" :visible.sync="addressVisible" width="50%">
      <!-- 内容主体区域 -->
      <el-form
        :model="addressForm"
        :rules="addressFormRules"
        ref="addressFormRef"
        label-width="100px"
      >
        <el-form-item label="省市区/县" prop="address1">
          <!-- prop绑定的是验证规则 -->
          <el-cascader :options="regionData" v-model="addressForm.address1" @change="handleChange"></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addressVisible = false">取 消</el-button>
        <el-button type="primary" @click="addressVisible = false">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 物流状态对话框 -->
    <el-dialog
      title="物理进度"
      :visible.sync="progressVisible"
      width="50%"
      @close="addressDialogClosed"
    >
      <!-- 内容主体区域 -->
      <el-timeline>
        <el-timeline-item
          v-for="(activity, index) in progressInfo"
          :key="index"
          :timestamp="activity.time"
        >{{activity.context}}</el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>
<script>
// 导入省市区三级联动数据组件
import { regionData } from 'element-china-area-data'
export default {
  data() {
    return {
      // 保存查询条件
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 保存获取到的订单列表数据
      orderList: [],
      // 保存总数据条数
      total: 0,
      addressVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [
          {
            required: true,
            message: '请选择省市区/县',
            trigger: 'blur' // 触发验证的时机，失去焦点的时候
          }
        ],
        address2: [
          {
            required: true,
            message: '请填写详细地址',
            trigger: 'blur' // 触发验证的时机，失去焦点的时候
          }
        ]
      },
      // 省市区数据源
      regionData: regionData,
      // 控制物流进度对话框的显示与隐藏
      progressVisible: false,
      // 保存物流信息数据
      progressInfo: []
    }
  },
  created() {
    this.getOrderList()
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表失败！')
      }
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    // 编辑点击事件
    showBox() {
      this.addressVisible = true
    },
    // 省市区选择改变事件
    handleChange() {
      console.log(this.addressForm)
    },
    // 监听修改地址对话框关闭事件
    addressDialogClosed() {
      // 解决dom元素还没有加载完时使用resetFields置空的报错
      if (this.$refs.addressFormRef !== undefined) {
        this.$refs.addressFormRef.resetFields()
      }
    },
    // 查询物流点击事件
    async showProgressBox() {
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      if (res.meta.status !== 200) {
        return this.$message.error('获取物流信息失败！')
      }
      this.progressInfo = res.data
      this.progressVisible = true
    }
  }
}
</script>
<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
