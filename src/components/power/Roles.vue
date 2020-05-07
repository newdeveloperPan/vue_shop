<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 添加角色区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRuleDialogVisible=true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="roleList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom',i1===0? 'bdtop':'','vcenter']"
              v-for="(item1,i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <!-- 通过for循环嵌套渲染二级权限 -->
                <el-row
                  :class="[i2===0?'':'bdtop','vcenter']"
                  v-for="(item2,i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row,item2.id)"
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      :class="[i3===0?'':'bdtop']"
                      v-for="(item3,i3) in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row,item3.id)"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <!-- 编辑按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditRoleDialog(scope.row.id)"
            >编辑</el-button>
            <!-- 删除按钮 -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeRoleById(scope.row.id)"
            >删除</el-button>
            <!-- 分配权限按钮 -->
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showSetRightDialog(scope.row)"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addRuleDialogVisible"
      width="50%"
      @close="addRuleDialogClosed"
    >
      <!-- 内容主体区域 -->
      <el-form :model="addRole" :rules="addRoleRules" ref="addRoleRef" label-width="100px">
        <el-form-item label="角色名称" prop="roleName">
          <!-- prop绑定的是验证规则 -->
          <el-input v-model="addRole.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRole.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRuleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoles">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改角色对话框 -->
    <el-dialog
      title="修改角色"
      :visible.sync="editRoleDialogVisible"
      width="50%"
      @close="editRoleDialogClosed"
    >
      <el-form :model="editRole" :rules="editRoleRules" ref="editRoleRef" label-width="100px">
        <el-form-item label="角色名称">
          <el-input v-model="editRole.roleName" disabled></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRole.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <!-- 这是树形控件 -->
      <el-tree
        :data="rightList"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="treeref"
      ></el-tree>
      <!--node-key绑定选中状态下选中的值,default-expand-all默认展开所有节点,:default-checked-keys默认勾选的节点-->
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="setRoleRight">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 所有角色的数据
      roleList: [],
      // 控制添加角色对话框的显示与隐藏
      addRuleDialogVisible: false,
      // 添加角色的数据
      addRole: {
        roleName: '',
        roleDesc: ''
      },
      // 添加角色的校验规则对象
      addRoleRules: {
        roleName: [
          {
            required: true,
            message: '请输入角色名称',
            trigger: 'blur' // 触发验证的时机，失去焦点的时候
          },
          {
            min: 1,
            max: 10,
            message: '角色名称的长度在1~10个字符之间',
            trigger: 'blur'
          }
        ],
        roleDesc: [
          {
            required: true,
            message: '请输入角色描述',
            trigger: 'blur' // 触发验证的时机，失去焦点的时候
          },
          {
            min: 1,
            max: 10,
            message: '角色描述的长度在1~10个字符之间',
            trigger: 'blur'
          }
        ]
      },
      // 控制修改角色对话框的显示与隐藏
      editRoleDialogVisible: false,
      // 修改角色的数据对象
      editRole: {},
      // 修改角色的验证规则对象
      editRoleRules: {
        roleDesc: [
          {
            required: true,
            message: '请输入角色描述',
            trigger: 'blur' // 触发验证的时机，失去焦点的时候
          },
          {
            min: 1,
            max: 10,
            message: '角色描述的长度在1~10个字符之间',
            trigger: 'blur'
          }
        ]
      },
      // 控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      // 所有权限列表数据
      rightList: [],
      // 树形控件的树形绑定对象
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认选中的节点ID值
      defKeys: [],
      // 即将分配权限的角色id
      roleId: ''
    }
  },
  created() {
    this.getRoleList()
  },
  methods: {
    // 获取所有角色的数据
    async getRoleList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.roleList = res.data
    },
    // 监听添加角色对话框关闭事件
    addRuleDialogClosed() {
      this.$refs.addRoleRef.resetFields()
    },
    addRoles() {
      this.$refs.addRoleRef.validate(async valid => {
        // valid是校验结果，true或者false
        if (!valid) return
        // 可以发起添加角色的网络请求
        const { data: res } = await this.$http.post('roles', this.addRole)
        if (res.meta.status !== 201) {
          this.$message.error('添加角色状失败！')
        }
        this.$message.success('添加角色成功！')
        // 添加角色成功之后关闭对话框
        this.addRuleDialogVisible = false
        // 重新获取用户列表数据
        this.getRoleList()
      })
    },
    // 展示修改角色的对话框
    async showEditRoleDialog(id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询角色信息失败！')
      }
      this.editRole = res.data
      this.editRoleDialogVisible = true
    },
    // 监听修改角色对话框关闭事件
    editRoleDialogClosed() {
      this.$refs.editRoleRef.resetFields()
    },
    // 修改角色预验证
    editRoleInfo() {
      this.$refs.editRoleRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          'roles/' + this.editRole.roleId,
          {
            roleName: this.editRole.roleName,
            roleDesc: this.editRole.roleDesc
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('更新角色信息失败！')
        }
        this.editRoleDialogVisible = false
        this.getRoleList()
        this.$message.success('更新角色信息成功！')
      })
    },
    // 根据角色ID删除对应的角色信息
    async removeRoleById(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
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
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败！')
      }
      this.$message.success('删除角色成功')
      this.getRoleList()
    },
    // 根据id删除对应的权限
    async removeRightById(role, rightId) {
      // 弹框提示用户是否要删除
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已经取消删除')
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败！')
      }
      this.$message.success('删除权限成功')
      // 重新获取角色列表会完全重新渲染页面，不推荐
      // this.getRoleList()
      // 返回的data, 是当前角色下最新的权限数据,所以重新赋值给scope.row即可
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      // 保存当前角色id
      this.roleId = role.id
      // 获取所有权限列表
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限失败！')
      }
      // 获取到的权限数据保存到data中
      this.rightList = res.data
      // 递归获取三级节点的Id
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形势获取当前角色三级权限ID，并保存到defkeys数组中
    getLeafKeys(node, arr) {
      // 如果当前node节点不包含children属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 监听分配权限对话框关闭事件
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 点击分配权限对话框确定按钮分配角色权限
    async setRoleRight() {
      const keys = [
        ...this.$refs.treeref.getCheckedKeys(),
        ...this.$refs.treeref.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }
      this.$message.success('分配权限成功')
      this.setRightDialogVisible = false
      this.getRoleList()
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
// 纵向居中对齐
.vcenter {
  display: flex;
  align-items: center;
}
</style>
