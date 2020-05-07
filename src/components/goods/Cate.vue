<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 添加分类区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 商品分类列表 -->
      <tree-table
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
        class="treetable"
      >
        <!-- 是否有效模板 -->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted===false" style="color:lightgreen;"></i>
          <i class="el-icon-error" v-else style="red;"></i>
        </template>
        <!-- 排序模板 -->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level===0" size="mini">一级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level===1" size="mini">二级</el-tag>
          <el-tag type="warning" v-else size="mini">三级</el-tag>
        </template>
        <!-- 操作模板 -->
        <template slot="opt" slot-scope="scope">
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
            @click="editCateDialogShow(scope.row.cat_id)"
          >编辑</el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="removeCateById(scope.row.cat_id)"
          >删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogclosed"
    >
      <!-- 添加分类的表单 -->
      <el-form :model="addCate" :rules="addCateRules" ref="addCateRef" label-width="100px">
        <!-- prop绑定的是验证规则 -->
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCate.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <!-- options用来指定数据源 -->
          <!-- props用来指定配置对象 -->
          <!-- v-model绑定的是存放选中的分类的ID数据的数组 -->
          <!-- clearable是否允许清空 -->
          <!-- change-on-select每一级分类是否可选 -->
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable
            change-on-select
          ></el-cascader>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCateSure">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑分类对话框 -->
    <el-dialog
      title="编辑分类"
      :visible.sync="editCateDialogVisible"
      width="50%"
      @close="editCateDialogClosed"
    >
      <!-- 编辑分类表单 -->
      <el-form
        :model="editCateForm"
        :rules="editCateFormRules"
        ref="editCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCateInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有商品分类数据
      cateList: [],
      // 分页数据
      queryInfo: {
        // 获取几级分类
        type: 3,
        // 当前页码值，默认显示第一页
        pagenum: 1,
        // 每页显示多少条数据，默认5条
        pagesize: 5
      },
      // 总数据条数，默认为0
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 表示将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用的模板名称
          template: 'isok'
        },
        {
          label: '排序',
          // 表示将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用的模板名称
          template: 'order'
        },
        {
          label: '操作',
          // 表示将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用的模板名称
          template: 'opt'
        }
      ],
      // 控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      // 添加分类的表单数据对象
      addCate: {
        // 将要添加的分类的名称
        cat_name: '',
        // 父级分类的id
        cat_pid: 0,
        // 分类的等级，默认要添加的是一级分类
        cat_level: 0
      },
      // 添加分类的表单验证规则对象
      addCateRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 保存父级分类的列表数据
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      // 保存选中的父级分类的ID数据
      selectedKeys: [],
      // 保存需要编辑的分类的数据
      editCateForm: {},
      // 控制编辑分类对话框的显示与隐藏
      editCateDialogVisible: false,
      // 编辑分类的验证规则对象
      editCateFormRules: {
        cat_name: {
          required: true,
          message: '请输入分类名称',
          trigger: 'blur' // 触发验证的时机，失去焦点的时候
        }
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类列表失败！')
      }
      this.cateList = res.data.result
      this.total = res.data.total
    },
    // 监听pagesize改变事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum改变事件
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 添加分类按钮事件
    showAddCateDialog() {
      // 先获取父级分类的数据列表
      this.getParentCateList()
      // 再展示对话框
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类信息失败！')
      }
      this.parentCateList = res.data
    },
    // 选择父级分类的选中项发生变化的处理函数
    parentCateChanged() {
      // 如果selectedKeys这个数组的length大于，证明选中了父级分类
      // 反之，就说明没有选中任何父级分类
      if (this.selectedKeys.length > 0) {
        // 新增分类的父级分类id
        this.addCate.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 新增分类的等级赋值
        this.addCate.cat_level = this.selectedKeys.length
        // eslint-disable-next-line no-useless-return
        return
      } else {
        this.addCate.cat_pid = 0
        this.addCate.cat_level = 0
      }
    },
    // 添加分类对话框确定事件
    addCateSure() {
      this.$refs.addCateRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCate)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }
        this.$message.success('添加分类成功！')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 监听添加分类对话框关闭事件,重置表单数据
    addCateDialogclosed() {
      this.$refs.addCateRef.resetFields()
      this.selectedKeys = []
      this.addCate.cat_level = 0
      this.addCate.cat_pid = 0
    },
    // 点击按钮显示编辑分类对话框,并获取该分类信息数据渲染
    async editCateDialogShow(id) {
      const { data: res } = await this.$http.get('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类名称失败！')
      }
      this.editCateForm = res.data
      this.editCateDialogVisible = true
    },
    // 监听编辑分类对话框关闭事件
    editCateDialogClosed() {
      // this.$refs.editCateFormRef.resetFields()
      this.editCateForm = {}
    },
    // 监听编辑分类对话框确定事件并预验证发起请求
    editCateInfo() {
      this.$refs.editCateFormRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        const { data: res } = await this.$http.put(
          'categories/' + this.editCateForm.cat_id,
          {
            cat_name: this.editCateForm.cat_name
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('更新分类名称失败！')
        }
        this.$message.success('更新分类名称成功！')
        this.getCateList()
        this.editCateDialogVisible = false
      })
    },
    // 删除分类事件
    async removeCateById(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该分类, 是否继续?',
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
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除分类失败！')
      }
      this.$message.success('删除分类成功')
      // 以下代码解决当删除的为最后一页最后一项数据时，列表显示为空，不会自动跳转到前一页的问题
      const totalPage = Math.ceil((this.total - 1) / this.queryInfo.pagesize) // 总页数
      this.queryInfo.pagenum =
        this.queryInfo.pagenum > totalPage ? totalPage : this.queryInfo.pagenum
      this.queryInfo.pagenum =
        this.queryInfo.pagenum < 1 ? 1 : this.queryInfo.pagenum
      this.getCateList()
    }
  }
}
</script>
<style lang="less" scoped>
.treetable {
  margin-top: 15px;
}
</style>
