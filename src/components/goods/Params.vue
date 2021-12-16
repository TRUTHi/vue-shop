<template>
  <div>
    <h3>Classification Parameters</h3>

    <!-- Breadcrumb navigation -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">Home</el-breadcrumb-item>
      <el-breadcrumb-item>Commodity Management</el-breadcrumb-item>
      <el-breadcrumb-item>Classification Parameters</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- Card view area -->
    <el-card>
      <!-- Warning area -->
      <el-alert
        title="Note: You can only set parameters for the third level classification"
        type="warning"
        :closable="false"
        show-icon
      ></el-alert>

      <!-- Select commodity classification area -->
      <el-row class="selectGoodsCategory">
        <el-col>
          <span>Select product category:</span>
          <!-- Select the cascading selection box for the product category -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="cateList"
            :props="cateProps"
            v-on:change="handleChange"
            clearable
          ></el-cascader>
        </el-col>
      </el-row>

      <!-- Tabs area -->
      <el-tabs v-model="activeName" v-on:tab-click="handleTabClick">
        <!-- Panel for adding dynamic parameters -->
        <el-tab-pane label="The Dynamic Parameters" name="many">
          <el-button
            class="add-button"
            size="mini"
            type="primary"
            :disabled="isButtonDisabled"
            v-on:click="addDialogVisible = true"
          >Add Parameters</el-button>
          <el-table :data="manyTableData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  v-for="(item, index) in scope.row.attr_vals"
                  :key="index"
                  closable
                >{{ item }}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  v-on:keyup.enter.native="handleInputConfirm(scope.row)"
                  v-on:blur="handleInputConfirm(scope.row)"
                ></el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  v-on:click="showInput(scope.row)"
                >Add New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="The Parameter Name" prop="attr_name"></el-table-column>
            <el-table-column label="Operation">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  v-on:click="showEditDialog(scope.row.attr_id)"
                >Edit</el-button>
                <el-button
                  v-on:click="removeParams(scope.row.attr_id)"
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                >Delete</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- Add panel for static properties -->
        <el-tab-pane label="Static Attributes" name="only">
          <el-button
            class="add-button"
            size="mini"
            type="primary"
            :disabled="isButtonDisabled"
            v-on:click="addDialogVisible = true"
          >Add Attribute</el-button>
          <el-table :data="onlyTableData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  v-for="(item, index) in scope.row.attr_vals"
                  :key="index"
                  closable
                >{{ item }}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  v-on:keyup.enter.native="handleInputConfirm(scope.row)"
                  v-on:blur="handleInputConfirm(scope.row)"
                ></el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  v-on:click="showInput(scope.row)"
                >Add New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="The Attribute Name" prop="attr_name"></el-table-column>
            <el-table-column label="Operation">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  v-on:click="showEditDialog(scope.row.attr_id)"
                >Edit</el-button>
                <el-button
                  v-on:click="removeParams(scope.row.attr_id)"
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                >Delete</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>

      <!-- Public dialog box for adding parameters and properties -->
      <el-dialog
        :title="'Add'+ titleText"
        :visible.sync="addDialogVisible"
        width="50%"
        v-on:close="addDialogClosed"
      >
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
          <el-form-item :label="titleText" prop="attr_name">
            <el-input v-model="addForm.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button v-on:click="addDialogVisible = false">Cancel</el-button>
          <el-button type="primary" v-on:click="addParams">Ok</el-button>
        </span>
      </el-dialog>

      <!-- Public dialog box for modifying parameters and properties -->
      <el-dialog
        :title="'Edit' + titleText"
        :visible.sync="editDialogVisible"
        width="50%"
        v-on:close="editDialogClosed"
      >
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
          <el-form-item :label="titleText" prop="attr_name">
            <el-input v-model="editForm.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button v-on:click="editDialogVisible = false">Cancel</el-button>
          <el-button type="primary" v-on:click="editParams">Ok</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // Classification list
      cateList: [],
      // Id of a category selected by the user in the cascading drop-down list
      selectedCateKeys: [],
      // How to display data in cascade Menu
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      // TAB Activates the displayed TAB
      activeName: 'many',
      // Dynamic parameter data
      manyTableData: [],
      // Static attribute data
      onlyTableData: [],
      addDialogVisible: false,
      addForm: {
        attr_name: ''
      },
      addFormRules: {
        attr_name: [{ required: true, message: 'Please enter a name', trigger: 'blur' }]
      },
      editDialogVisible: false,
      editForm: {
        attr_name: ''
      },
      editFormRules: {
        attr_name: [{ required: true, message: 'Please enter a name', trigger: 'blur' }]
      }
    }
  },
  methods: {
    async getCateList () {
      // Gets a categorized list of all items
      const { data: result } = await this.$axios.get('categories')
      if (result.meta.status !== 200) return this.$message.error('Failed to get commodity category list data.')
      this.cateList = result.data
      // console.log(result)
    },
    async handleChange () {
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return null
      }
      console.log(this.selectedCateKeys)
      const { data: result } = await this.$axios.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      if (result.meta.status !== 200) return this.$message.error('Failed to get parameter list data.')
      // Convert the 'attr_vals' string in the data to an array
      result.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // Add a Boolean value to control whether the text box is displayed or hidden
        item.inputVisible = false
        // Add an 'inputValue' to hold the text box value
        item.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyTableData = result.data
      } else if (this.activeName === 'only') {
        this.onlyTableData = result.data
      }
    },
    handleTabClick () {
      console.log(this.activeName)
      this.handleChange()
    },
    addParams () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$axios.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName,
          attr_vals: 'a,b,c'
        })
        console.log(res)
        if (res.meta.status !== 201) return this.$message.error('Add' + this.titleText + 'failure')
        this.$message.success('Add' + this.titleText + 'successful')
        this.addDialogVisible = false
        this.getCateList()
      })
    },
    addDialogClosed () { },
    async showEditDialog (attrId) {
      const { data: result } = await this.$axios.get(`categories/${this.cateId}/attributes/${attrId}`, {
        params: { attr_sel: this.activeName }
      })
      if (result.meta.status !== 200) return this.$message.error('failure')
      this.editForm = result.data
      this.editDialogVisible = true
    },
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    editParams () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$axios.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
          attr_name: this.editForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 200) return this.$message.error('failure')
        this.$message.success('Edit' + this.titleText + 'successful')
        this.editDialogVisible = false
        this.handleChange()
      })
    },
    async removeParams (attrId) {
      const confirmResult = await this.$confirm('You want to delete this ' + this.titleText, 'Delete the prompt', {
        confirmButtonText: 'Yes',
        cancelButtonText: 'No',
        type: 'warning'
      }).catch(error => error)
      if (confirmResult !== 'confirm') return this.$message.info('Cancelled delete')
      const { data: result } = await this.$axios.delete(`categories/${this.cateId}/attributes/${attrId}`)
      if (result.meta.status !== 200) return this.$message.error('Failed to delete ' + this.titleText)
      this.$message.success('Succeeded in deleting ' + this.titleText)
      this.handleChange()
    },
    handleInputConfirm (row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return false
      }
    },
    showInput (row) {
      row.inputVisible = true
      // Code that calls the callback function after the elements on the page have been rerendered
      this.$nextTick(() => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    }
  },
  computed: {
    // Gets the ID of the selected level 3 category
    cateId () {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[this.selectedCateKeys.length - 1]
      } else {
        return null
      }
    },
    isButtonDisabled () {
      return this.selectedCateKeys.length !== 3
    },
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  },
  created () {
    this.getCateList()
  }
}
</script>

<style lang="less">
.selectGoodsCategory {
  margin: 15px 0;
}

.el-cascader {
  width: 300px;
  margin-left: 15px;
}

.add-button {
  margin-bottom: 15px !important;
}
.button-new-tag {
  margin: 10px !important;
}
.input-new-tag {
  margin: 10px !important;
  width: 150px !important;
  height: 34px !important;
}
</style>
