<template>
  <div>
    <h3>Classification of Goods</h3>
    <!-- Breadcrumb navigation -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">HOME</el-breadcrumb-item>
      <el-breadcrumb-item>Commodity Management</el-breadcrumb-item>
      <el-breadcrumb-item>Classification of Goods</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- Card view area -->
    <el-card>
      <!-- Add category button area -->
      <el-row>
        <el-col>
          <el-button type="primary" v-on:click="showAddCateDialog">Add the Classification</el-button>
        </el-col>
      </el-row>
      <!-- Commodity classification list data -->
      <tree-table
        class="treeTable"
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
      >
        <!-- The Validity Of -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen;"
          ></i>
          <i class="el-icon-error" v-else style="color: red;"></i>
        </template>
        <!-- The Sorting -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">Level 1</el-tag>
          <el-tag size="mini" type="success" v-else-if="scope.row.cat_level===1">Level 2</el-tag>
          <el-tag size="mini" type="warning" v-else>Level 3</el-tag>
        </template>
        <!-- Operation -->
        <template slot="opt">
          <el-button size="mini" type="primary" icon="el-icon-edit">Edit</el-button>
          <el-button size="mini" type="danger" icon="el-icon-delete">Delete</el-button>
        </template>
      </tree-table>
      <!-- Pagination -->
      <el-pagination
        v-on:size-change="handleSizeChange"
        v-on:current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <!-- Add category dialog box -->
    <el-dialog
      title="Add The Classification"
      :visible.sync="addCateDialogVisible"
      width="50%"
      v-on:close="addCateDialogClosed"
    >
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRuleForm"
        label-width="auto"
      >
        <el-form-item label="Category Name" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="The Parent Category">
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            v-on:change="parentCateChange"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button v-on:click="addCateDialogVisible = false">Cancel</el-button>
        <el-button type="primary" v-on:click="addCate">Determine</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // List of commodity classification data
      cateList: [],
      // Conditions for querying classified data
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // Total number of data items
      total: 0,
      columns: [
        { label: 'Category Name', prop: 'cat_name' },
        { label: 'The Validity Of', type: 'template', template: 'isok' },
        { label: 'The Sorting', type: 'template', template: 'order' },
        { label: 'Operation', type: 'template', template: 'opt' }
      ],
      addCateDialogVisible: false,
      addCateForm: {
        // Category name
        cat_name: '',
        // Example Add the parent ID of a category, 0 indicates that the parent level is 0, Add a level 1 category
        cat_pid: 0,
        // Add a classification level, 0 indicates that a level 1 category is added
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [{ required: true, message: 'Please enter a category name', trigger: 'blur' }]
      },
      parentCateList: [],
      selectedKeys: [],
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover',
        checkStrictly: true
      }
    }
  },
  methods: {
    // Get commodity classification data
    async getCateList () {
      const { data: res } = await this.$axios.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('Failed to get the itemized data list.')
      console.log(res)
      this.cateList = res.data.result
      this.total = res.data.total
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    showAddCateDialog () {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // Gets the list of parent classification data
    async getParentCateList () {
      const { data: result } = await this.$axios.get('categories', { params: { type: 2 } })
      if (result.meta.status !== 200) return this.$message.error('Failed to get commodity category list data.')
      this.parentCateList = result.data
    },
    parentCateChange () {
      console.log(this.selectedKeys)
      // If the user selects the parent category
      if (this.selectedKeys.length > 0) {
        // Sets the last item in the array to the parent category
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCateForm.cat_level = this.selectedKeys.length
        return null
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
        return null
      }
    },
    addCateDialogClosed () {
      this.$refs.addCateFormRuleForm.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    addCate () {
      console.log(this.addCateForm)
      this.$refs.addCateFormRuleForm.validate(async valid => {
        if (!valid) return this.$message.error('Check the filling   information.')
        const { data: result } = await this.$axios.post('categories', this.addCateForm)
        if (result.meta.status !== 201) return this.$message.error('Description Failed to add a category.')
        this.$message.success('Succeeded in adding a category.')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    }
  },
  created () {
    this.getCateList()
  }
}
</script>

<style lang="less">
.treeTable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
.el-cascader-panel {
  height: 300px;
}
</style>
