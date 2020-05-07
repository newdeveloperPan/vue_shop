<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 头部警告区域 -->
      <el-alert show-icon title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false"></el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 商品分类级联选择框 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="cateList"
            :props="cateProps"
            @change="CateChanged"
            clearable
            change-on-select
          ></el-cascader>
        </el-col>
      </el-row>
      <!-- tab页签区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 添加动态参数的面板 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="showAddDialog"
          >添加参数</el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染tag标签 -->
                <el-tag
                  v-for="(item,i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClose(i,scope.row)"
                >{{item}}</el-tag>
                <!-- 输入文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <!-- 添加按钮 -->
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="showEditDialog(scope.row.attr_id)"
                >编辑</el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="removeParams(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性的面板 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="showAddDialog"
          >添加属性</el-button>
          <!-- 静态属性表格 -->
          <el-table :data="onlyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染tag标签 -->
                <el-tag
                  v-for="(item,i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClose(i,scope.row)"
                >{{item}}</el-tag>
                <!-- 输入文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <!-- 添加按钮 -->
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="showEditDialog(scope.row.attr_id)"
                >编辑</el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="removeParams(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数/属性对话框 -->
    <el-dialog
      :title="'添加'+titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @closed="addDialogClosed"
    >
      <!-- 添加参数/属性的表单 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <!-- prop绑定的是验证规则 -->
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑参数/属性对话框 -->
    <el-dialog
      :title="'修改'+titleText"
      :visible.sync="EditDialogVisible"
      width="50%"
      @closed="editDialogClosed"
    >
      <!-- 添加参数/属性的表单 -->
      <el-form :model="EditForm" :rules="EditFormRules" ref="EditFormRef" label-width="100px">
        <!-- prop绑定的是验证规则 -->
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="EditForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="EditDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="EditParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 所有商品分类列表数据
      cateList: [],
      // 级联选择框配置对象
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      // 保存选中的分类的ID值
      selectedCateKeys: [],
      // 被激活的页签名称
      activeName: 'many',
      // 保存动态参数的数据
      manyTableData: [],
      // 保存静态属性的数据
      onlyTableData: [],
      // 控制添加对话框的显示与隐藏
      addDialogVisible: false,
      // 保存添加参数/属性的表单数据对象
      addForm: {
        attr_name: ''
      },
      // 保存添加参数/属性的表单验证规则对象
      addFormRules: {
        attr_name: [
          {
            required: true,
            message: '请输入参数名称',
            trigger: 'blur' // 触发验证的时机，失去焦点的时候
          }
        ]
      },
      // 控制编辑对话框的显示与隐藏
      EditDialogVisible: false,
      // 保存要修改的参数数据对象
      EditForm: {
        attr_name: ''
      },
      // 修改表单的验证规则对象
      EditFormRules: {
        attr_name: [
          {
            required: true,
            message: '请输入参数名称',
            trigger: 'blur' // 触发验证的时机，失去焦点的时候
          }
        ]
      }
    }
  },
  created() {
    // 获取所有的商品分类列表
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类信息失败！')
      }
      this.cateList = res.data
    },
    // 级联选择框选中分类改变后的触发的事件
    CateChanged() {
      this.getParamsData()
    },
    // tab页签点击事件的处理函数
    handleTabClick() {
      this.getParamsData()
    },
    async getParamsData() {
      // 证明选择的不是三级分类
      if (this.selectedCateKeys.length !== 3) {
        // 禁用添加按钮
        this.selectedCateKeys = []
        // 清空动态参数表格
        this.manyTableData = []
        // 清空静态属性表格
        this.onlyTableData = []
        // eslint-disable-next-line no-useless-return
        return
      }
      // 证明选择的是三级分类
      // 根据所选分类的id和当前所处的面板，获取对应的参数
      const { data: res } = await this.$http.get(
        `categories/${this.catdId}/attributes`,
        {
          params: { sel: this.activeName }
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败！')
      }
      res.data.forEach(item => {
        // 为获取到的可选项字符串分割成数组并重新赋值,用于渲染到参数扩展行下的可选项
        // 判断item.attr_vals是否为空，如果为空，则不分割
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 为每一项数据新增属性，控制文本框的显示与隐藏
        item.inputVisible = false
        //  为每一项数据新增属性，保存文本框中输入的值
        item.inputValue = ''
      })
      // 判断获取到的数据是动态参数还是静态属性
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // 点击按钮显示添加对话框
    showAddDialog() {
      this.addDialogVisible = true
    },
    // 监听添加对话框的关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 点击确定预验证添加内容并发起请求
    addParams() {
      this.$refs.addFormRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        const { data: res } = await this.$http.post(
          `categories/${this.catdId}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败！')
        }
        this.$message.success('添加参数成功！')
        this.getParamsData()
        this.addDialogVisible = false
      })
    },
    // 展示编辑对话框
    // eslint-disable-next-line camelcase
    async showEditDialog(attr_id) {
      // 查询当前参数的信息
      const { data: res } = await this.$http.get(
        // eslint-disable-next-line camelcase
        `categories/${this.catdId}/attributes/+${attr_id}`,
        { params: { attr_sel: this.activeName } }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数失败！')
      }
      this.EditForm = res.data
      this.EditDialogVisible = true
    },
    // 监听编辑对话框的关闭事件
    editDialogClosed() {
      // 重置编辑表单数据
      this.$refs.EditFormRef.resetFields()
    },
    // 点击确定按钮修改参数信息
    EditParams() {
      this.$refs.EditFormRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        const { data: res } = await this.$http.put(
          `categories/${this.catdId}/attributes/${this.EditForm.attr_id}`,
          {
            attr_name: this.EditForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数失败！')
        }
        this.$message.success('修改参数成功！')
        this.getParamsData()
        this.EditDialogVisible = false
      })
    },
    // 根据id删除对应参数
    // eslint-disable-next-line camelcase
    async removeParams(attr_id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该参数, 是否继续?',
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
      const { data: res } = await this.$http.delete(
        // eslint-disable-next-line camelcase
        `categories/${this.catdId}/attributes/${attr_id}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败！')
      }
      this.$message.success('删除参数成功')
      this.getParamsData()
    },
    // 可选项输入文本框失去焦点或enter键抬起触发的事件
    handleInputConfirm(row) {
      // 如果文本框中输入的内容不合法，则重置文本框
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        // eslint-disable-next-line no-useless-return
        return
      }
      // 如果内容合法，将文本框的值添加到row.attr_vals中
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      // 需要发起请求，保存这次操作
      this.saveAttrVals(row)
    },
    // 点击按钮显示输入文本框
    showInput(row) {
      row.inputVisible = true
      // 让文本框自动获得焦点
      // $nextTick方法的作用，就是当页面上元素被重新渲染之后，才执行回调函数中的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 删除可选项处理事件
    handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    },
    // 将对attr_vals的操作保存到数据库的处理函数
    async saveAttrVals(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.catdId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('修改参数项失败！')
      }
      this.$message.success('修改参数项成功！')
    }
  },
  computed: {
    // 如果添加参数和添加属性的按钮需要被禁用，则返回true，否则返回false
    isBtnDisabled() {
      if (this.selectedCateKeys.length !== 3) {
        return true
      }
      return false
    },
    // 当前选中的三级分类的id
    catdId() {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    },
    // 动态计算添加对话框标题的文本
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>
<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin: 0 10px;
}
.input-new-tag {
  width: 120px;
}
</style>
