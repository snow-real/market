<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item :to="{ path: '/rights' }"
        >权限管理</el-breadcrumb-item
      >
      <el-breadcrumb-item :to="{ path: '/roles' }">权限列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddRole">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="rolelist" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag
                  type="success"
                  closable
                  @close="removeRightById(scope.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级三级权限 -->
              <el-col :span="19">
                <!-- 通过for循环，嵌套渲染二级权限 -->
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <!-- eslint-disable -->
                    <el-tag
                      v-for="(item3, i3) in item2.children"
                      :key="item3.id"
                      type="warning"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                    <!-- eslint-enable -->
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"> </el-table-column>
        <el-table-column label="角色名称" prop="roleName"> </el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"> </el-table-column>
        <el-table-column label="操作">
          <!-- eslint-disable-next-line -->
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showEditRole(scope.row.id, scope.row)"
              >编辑</el-button
            >
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="deleteRole(scope.row.id)"
              >删除</el-button
            >
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog
      title="提示"
      :visible.sync="addRoleVisible"
      width="40%"
      @close="addRoleClosed"
    >
      <!-- 内容主体 -->
      <el-form
        ref="addRoleRef"
        :model="addRoleForm"
        label-width="100px"
        :rules="addRoleRules"
      >
        <!-- 用户名 -->
        <el-form-item prop="roleName" label="角色名称">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <!-- 用户名 -->
        <el-form-item prop="roleDesc" label="角色描述">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="editRoleVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <!-- 内容主体 -->
      <el-form
        ref="editRoleRef"
        :model="editRoleForm"
        label-width="100px"
        :rules="addRoleRules"
      >
        <!-- 用户名 -->
        <el-form-item prop="roleName" label="角色名称">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <!-- 用户名 -->
        <el-form-item prop="roleDesc" label="角色描述">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog title="提示" :visible.sync="setRightVisible" width="30%">
      <el-tree
        :data="rightsList"
        :props="treeProps"
        show-checkbox=""
        node-key="id"
        :default-checked-keys="defkeys"
        :default-expand-all="true"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有角色列表数据
      rolelist: [],
      // 控制添加、编辑、分配权限对话框的显示
      addRoleVisible: false,
      editRoleVisible: false,
      setRightVisible: false,
      // 所有权限的数据
      rightsList: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点ID值数组
      defkeys: [],
      // 当前即将分配权限的角色id
      roleId: '',
      // 添加角色的表单
      addRoleForm: {
        roleName: '',
        roleDesc: ''
      },
      // 编辑角色的表单
      editRoleForm: {
        roleName: '',
        roleDesc: '',
        roleId: ''
      },
      // 添加角色的验证规则
      addRoleRules: {
        // 验证用户名是否合法
        roleName: [
          { required: true, message: '请输入角色名', tigger: 'blur' }
        ],
        // 验证角色描述是否合法
        roleDesc: [
          { required: true, message: '请输入角色描述', tigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      console.log(res.data)
      this.rolelist = res.data
    },
    // 显示添加用户对话框
    showAddRole () {
      this.addRoleVisible = true
    },
    addRole () {
      this.$refs.addRoleRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addRoleForm)
        if (res.meta.status !== 201) return this.$message.error('添加角色失败！')
        this.$message.success('添加成功')
        // 隐藏添加用户的对话框
        this.addRoleVisible = false
        this.getRolesList()
      })
    },
    addRoleClosed () {
      this.$refs.addRoleRef.resetFields()
    },
    // 显示编辑角色对话框
    async showEditRole (id) {
      this.editRoleVisible = true
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('获取角色信息失败')
      this.editRoleForm = res.data
    },
    // 监听修改角色对话框关闭事件
    editRoleClosed () {
      this.$refs.editRoleRef.resetFields()
    },
    // 提交角色修改
    async editRole () {
      this.editRoleVisible = false
      const { data: res } = await this.$http.put('roles/' + this.editRoleForm.roleId, { roleName: this.editRoleForm.roleName, roleDesc: this.editRoleForm.roleDesc })
      if (res.meta.status !== 200) return this.$message.error('修改用户信息失败')
      this.getRolesList()
    },
    // 删除角色
    async deleteRole (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }
      ).catch(err => err)
      // 如果用户确认删除，则返回字符串confirm
      // 如果用户取消删除，则返回字符串cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已经取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除角色失败')
      this.getRolesList()
      console.log(res)
      console.log(id)
    },
    // 权限删除触发事件
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除')
      }
      console.log(role)
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)

      if (res.meta.status !== 200) {
        return this.message.error('删除权限失败')
      }

      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/' + 'tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限失败！')
      }
      // 把获取到的权限数据保存到 data 中
      this.rightsList = res.data
      console.log(this.rightsList)

      // 递归获取三级节点的id
      this.getLeafKeys(role, this.defkeys)
      this.setRightVisible = true
    },
    // 通过递归的形式，获取角色下所有三级2权限的id,
    // 并保存到数组中
    getLeafKeys (node, arr) {
      if (!node.children) {
        // 如果当前node属性不包含children属性，则是三级节点
        return arr.push(node.id)
      }

      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed () {
      this.defkeys = []
    },
    // 点击为角色分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]

      const idStr = keys.join(',')
      // eslint-disable-next-line
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        console.log(res)
        return this.$message.error('分配权限失败')
      }

      this.$message.success('分配权限成功！')
      this.getRolesList()
      this.setRightVisible = false
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
</style>
