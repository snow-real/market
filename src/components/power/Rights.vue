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
      <template>
        <el-table :data="rightsList" style="width: 100%" border="" stripe="">
          <el-table-column type="index"> </el-table-column>
          <el-table-column prop="authName" label="权限名称" width="180">
          </el-table-column>
          <el-table-column prop="path" label="路径" width="180">
          </el-table-column>
          <el-table-column prop="level" label="权限等级">
            <!-- eslint-disable-next-line -->
            <template slot-scope="scope">
              <el-tag v-if="scope.row.level === '0'">标签一</el-tag>
              <el-tag type="success" v-else-if="scope.row.level === '1'"
                >标签二</el-tag
              >
              <el-tag type="warning" v-else>标签三</el-tag>
            </template>
          </el-table-column>
        </el-table>
      </template>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rightsList: []
    }
  },
  // 获取所有的权限
  created () {
    this.getRightsList()
  },
  methods: {
    // 获取权限列表
    async getRightsList () {
      const { data: res } = await this.$http.get('rights/' + 'list')
      this.rightsList = res.data
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败')
      }
    }
  }
}
</script>

<style lang="less" scoped>
</style>
