<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddpage">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- table表格区域 -->
      <el-table :data="goodsList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column label="商品价格（元）" prop="goods_price" width="110px"></el-table-column>
        <el-table-column label="商品重量" prop="goods_weight" width="70px"></el-table-column>
        <el-table-column label="创建时间" prop="add_time" width="140px">
          <template slot-scope="scope">{{scope.row.add_time | dateFormat}}</template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditGoods(scope.row.goods_id)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeById(scope.row.goods_id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background
      ></el-pagination>
    </el-card>
    <!-- 编辑商品对话框 -->
    <el-dialog
      title="修改商品"
      :visible.sync="editGoodsDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <el-form
        :model="editGoodsForm"
        :rules="editGoodsFormRules"
        ref="editGoodsFormRef"
        label-width="100px"
      >
        <el-form-item label="商品名称" prop="goods_name">
          <el-input v-model="editGoodsForm.goods_name"></el-input>
        </el-form-item>
        <el-form-item label="商品价格" prop="goods_price">
          <el-input v-model="editGoodsForm.goods_price" type="number"></el-input>
        </el-form-item>
        <el-form-item label="商品重量" prop="goods_weight">
          <el-input v-model="editGoodsForm.goods_weight" type="number"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editGoodsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editGoods">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
import _ from 'lodash'
export default {
  data() {
    return {
      // 查询参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 商品列表数据
      goodsList: [],
      // 总数据条数
      total: 0,
      // 控制编辑商品对话框的显示与隐藏
      editGoodsDialogVisible: false,
      // 保存编辑商品数据对象
      editGoodsForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        attrs: []
      },
      // 保存选中的商品分类的动态参数列表数据
      manyTableData: [],
      // 保存选中的商品分类的静态属性列表数据
      onlyTableData: [],
      // 保存编辑商品表单验证对象
      editGoodsFormRules: {
        goods_name: [
          {
            required: true,
            message: '请输入商品名称',
            trigger: 'blur' // 触发验证的时机，失去焦点的时候
          }
        ],
        goods_price: [
          {
            required: true,
            message: '请输入商品价格',
            trigger: 'blur' // 触发验证的时机，失去焦点的时候
          }
        ],
        goods_weight: [
          {
            required: true,
            message: '请输入商品重量',
            trigger: 'blur' // 触发验证的时机，失去焦点的时候
          }
        ]
      }
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    // 根据分页获取对应的商品列表
    async getGoodsList() {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表信息失败！')
      }
      this.goodsList = res.data.goods
      this.total = res.data.total
    },
    // 每页数据条数发生改变时的处理函数
    handleSizeChange(newsize) {
      this.queryInfo.pagesize = newsize
      this.getGoodsList()
    },
    // 当前页发生改变时的处理函数
    handleCurrentChange(newpage) {
      this.queryInfo.pagenum = newpage
      this.getGoodsList()
    },
    // 根据id删除商品
    async removeById(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该商品, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 如果用户确认删除，则返回值为字符串confirm
      // 如果用户取消删除，则返回值为字符串cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已经取消删除')
      }
      const { data: res } = await this.$http.delete(`goods/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除商品失败！')
      }
      this.$message.success('删除商品成功！')
      // 以下代码解决当删除的为最后一页最后一项数据时，列表显示为空，不会自动跳转到前一页的问题
      const totalPage = Math.ceil((this.total - 1) / this.queryInfo.pagesize) // 总页数
      this.queryInfo.pagenum =
        this.queryInfo.pagenum > totalPage ? totalPage : this.queryInfo.pagenum
      this.queryInfo.pagenum =
        this.queryInfo.pagenum < 1 ? 1 : this.queryInfo.pagenum
      this.getGoodsList()
    },
    goAddpage() {
      this.$router.push('/goods/add')
    },
    // 点击按钮编辑商品信息
    async showEditGoods(id) {
      const { data: res } = await this.$http.get(`goods/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品信息失败！')
      }
      this.editGoodsForm = res.data
      this.editGoodsDialogVisible = true
    },
    // 监听编辑商品对话框关闭事件
    editDialogClosed() {
      this.$refs.editGoodsFormRef.resetFields()
      this.editGoodsDialogVisible = false
    },
    // 点击确定发起修改商品请求
    editGoods() {
      this.$refs.editGoodsFormRef.validate(async valid => {
        if (!valid) return
        const form = _.cloneDeep(this.editGoodsForm)
        // 处理动态参数
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.editGoodsForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.editGoodsForm.attrs.push(newInfo)
        })
        form.attrs = this.editGoodsForm.attrs
        const { data: res } = await this.$http.put(
          `goods/${this.editGoodsForm.goods_id}`,
          form
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改商品失败')
        }
        this.$message.success('修改商品成功')
        this.getGoodsList()
        this.editGoodsDialogVisible = false
      })
    }
  }
}
</script>
<style lang="less" scoped>
</style>
