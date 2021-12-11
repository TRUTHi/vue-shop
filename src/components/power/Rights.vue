<template>
  <div>
    <!-- Breadcrumb navigation -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">Home page</el-breadcrumb-item>
      <el-breadcrumb-item>Rights management</el-breadcrumb-item>
      <el-breadcrumb-item>Permissions list</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- Permissions list -->
    <el-table :data="rightsList" stripe border>
      <el-table-column label="#" type="index"></el-table-column>
      <el-table-column label="Permission To Name" prop="authName"></el-table-column>
      <el-table-column label="The Path" prop="path"></el-table-column>
      <el-table-column label="Access Level">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.level == '0'">1 level authority</el-tag>
          <el-tag v-else-if="scope.row.level == '1'" type="success">2 level authority</el-tag>
          <el-tag type="warning" v-else>3 level authority</el-tag>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // Permission list data
      rightsList: []
    }
  },
  methods: {
    // Send the request: get permission list data
    async getRightsList () {
      const { data: result } = await this.$axios.get('rights/list')
      if (result.meta.status !== 200) return this.$message.error('Failed to obtain permission list data.')
      this.rightsList = result.data
    }
  },
  created () {
    this.getRightsList()
  }
}
</script>
