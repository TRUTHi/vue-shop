<template>
  <div>
    <!-- Breadcrumb navigation -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">Home page</el-breadcrumb-item>
      <el-breadcrumb-item>Rights management</el-breadcrumb-item>
      <el-breadcrumb-item>Permissions list</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- The role list -->
    <el-table row-key="id" :data="roleList" stripe border>
      <!-- Expand row -->
      <el-table-column type="expand">
        <template slot-scope="scope">
          <el-row
            :class="['vcenter', 'bdbottom', indexOne == 0 ? 'bdtop' : '']"
            v-for="(itemOne, indexOne) in scope.row.children"
            :key="itemOne.id"
          >
            <!-- 1 level authority -->
            <el-col :span="5">
              <el-tag
                closable
                v-on:close="removeRightById(scope.row, itemOne.id)"
              >{{ itemOne.authName }}</el-tag>
              <i class="el-icon-caret-right"></i>
            </el-col>
            <!-- Level 2 and 3 authority -->
            <el-col :span="19">
              <el-row
                :class="['vcenter', indexTwo == 0 ? '' : 'bdtop']"
                v-for="(itemTwo, indexTwo) in itemOne.children"
                :key="itemTwo.id"
              >
                <el-col :span="6">
                  <el-tag
                    closable
                    v-on:close="removeRightById(scope.row, itemTwo.id)"
                    type="success"
                  >{{ itemTwo.authName }}</el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <el-col :span="18">
                  <el-tag
                    type="warning"
                    v-for="itemThree in itemTwo.children"
                    :key="itemThree.id"
                    closable
                    v-on:close="removeRightById(scope.row, itemThree.id)"
                  >{{ itemThree.authName }}</el-tag>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
        </template>
      </el-table-column>
      <el-table-column label="#" type="index"></el-table-column>
      <el-table-column label="Character name" prop="roleName"></el-table-column>
      <el-table-column label="Role description" prop="roleDesc"></el-table-column>
      <el-table-column label="operation" width="385px">
        <template slot-scope="scope">
          <!-- Edit button -->
          <el-button
            size="mini"
            type="primary"
            icon="el-icon-edit"
            @click="editRoleInfo(scope.row.id)"
          >Edit</el-button>
          <!-- Delete button -->
          <el-button size="mini" type="danger" icon="el-icon-delete">Delete</el-button>
          <!-- Assign Permissions button -->
          <el-button
            v-on:click="showSetRightDialog(scope.row)"
            size="mini"
            type="warning"
            icon="el-icon-setting"
          >Assign Permissions</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- Edit role information dialog box -->
    <el-dialog
      title="Edit Role"
      :visible.sync="editRoleDialog"
      width="30%"
      v-on:close="roleDialogClosed"
    >
      <!-- The main body -->
      <el-form
        ref="editRoleFormRef"
        :rules="roleFormRules"
        :model="editRoleForm"
        label-width="100px"
      >
        <el-form-item label="ID">
          <el-input v-model="editRoleForm.roleId" disabled></el-input>
        </el-form-item>
        <el-form-item label="Name" prop="roleName">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="Describe" prop="roleDesc">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- The footer -->
      <span slot="footer" class="dialog-footer">
        <el-button v-on:click="editRoleDialog = false">Cancel</el-button>
        <el-button type="primary" v-on:click="editRoleInfoDone">Determine</el-button>
      </span>
    </el-dialog>

    <!-- Assign Role rights dialog box -->
    <el-dialog
      title="Assigning Role Rights"
      :visible.sync="setRightDialogVisible"
      width="50%"
      v-on:close="setRightDialogClose"
    >
      <!-- The main body -->
      <el-tree
        :data="rightsList"
        node-key="id"
        show-checkbox
        default-expand-all
        :default-checked-keys="defKeys"
        :props="treeProps"
        ref="treeRef"
      ></el-tree>
      <!-- The footer -->
      <span slot="footer" class="dialog-footer">
        <el-button v-on:click="setRightDialogVisible = false">Cancel</el-button>
        <el-button type="primary" v-on:click="allotRights">Determine</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // Role list data
      roleList: [],
      // Edit role information object
      editRoleForm: {},
      // Edit role information dialog box
      editRoleDialog: false,
      // Edit role form rule object
      roleFormRules: {
        roleName: [
          { required: true, message: 'Please enter role name', trigger: 'blur' },
          { min: 3, max: 10, message: 'The length is between 3 and 10 characters', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: 'Please enter role description', trigger: 'blur' },
          { min: 3, max: 10, message: 'The length is between 3 and 10 characters', trigger: 'blur' }
        ]
      },
      setRightDialogVisible: false,
      rightsList: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      defKeys: [],
      roleId: 0
    }
  },
  methods: {
    // Send request: get the role list data
    async getRoleList () {
      const { data: result } = await this.$axios.get('roles')
      if (result.meta.status !== 200) return this.$message.error('Failed to obtain role list data.')
      this.roleList = result.data
      // console.log(result.data)
    },
    async editRoleInfo (id) {
      const { data: result } = await this.$axios.get('roles/' + id)
      if (result.meta.status !== 200) this.$message.error('Failed to query role information.')
      this.editRoleForm = result.data
      this.editRoleDialog = true
    },
    roleDialogClosed () {
      this.$refs.editRoleFormRef.resetFields()
    },
    editRoleInfoDone () {
      this.$refs.editRoleFormRef.validate(async valid => {
        if (!valid) return this.$message.error('Please confirm whether the fields filled in are correct')
        const { data: result } = await this.$axios.put('roles/' + this.editRoleForm.roleId, this.editRoleForm)
        if (result.meta.status !== 200) return this.$message.error('Failed to edit role.')
        this.$message.success('Edit role succeeded')
        this.editRoleDialog = false
        this.getRolesList()
      })
    },
    async removeRightById (role, rightId) {
      // Pop-up prompts
      const confirmResult = await this.$confirm(
        'Do you want to delete this permission ?',
        'Delete The Prompt',
        {
          confirmButtonText: 'Yes',
          cancelButtonText: 'No',
          type: 'warning'
        }
      ).catch(error => error)
      // If the user clicks confirm, confirmResult is 'confirm'
      // If the user clicks cancel, the confirmResult gets the Catch error message 'cancel'.
      if (confirmResult !== 'confirm') return this.$message.info('The deletion permission operation has been cancelled.')
      const { data: result } = await this.$axios.delete(`roles/${role.id}/rights/${rightId}`)
      if (result.meta.status !== 200) return this.$message.error('Failed to delete permission.')
      // You no longer need to reload all permissions, You only need to update the existing role permissions
      role.children = result.data
    },
    async showSetRightDialog (role) {
      this.roleId = role.id
      const { data: result } = await this.$axios.get('rights/tree')
      if (result.meta.status !== 200) return this.$message.error('Failed to obtain permission tree')
      this.rightsList = result.data
      this.getLeafKeys(role)
      console.log(role)
      console.log(this.defKeys)
      this.setRightDialogVisible = true
    },
    getLeafKeys (node) {
      if (!node.children) return this.defKeys.push(node.id)
      node.children.forEach(item => this.getLeafKeys(item))
    },
    async allotRights () {
      // Get all selected and semi selected contents
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      // Convert the array to a string spliced with ","
      const idStr = keys.join(',')
      const { data: result } = await this.$axios.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (result.meta.status !== 200) return this.$message.error('Failed to assign permissions')
      this.$message.success('Permission assignment succeeded')
      this.getRoleList()
      this.setRightDialogVisible = false
    },
    setRightDialogClose () {
      this.defKeys = []
    }
  },
  created () {
    this.getRoleList()
  }
}
</script>

<style lang="less">
.el-tag {
  margin: 10px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>
