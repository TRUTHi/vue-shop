<template>
  <div>
    <!-- Breadcrumb navigation -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">Home page</el-breadcrumb-item>
      <el-breadcrumb-item>User management</el-breadcrumb-item>
      <el-breadcrumb-item>List of users</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- Card view area -->
    <el-card>
      <!-- Layout components -->
      <el-row :gutter="20">
        <el-col :span="7">
          <!-- The search area -->
          <el-input
            placeholder="Please enter the content"
            v-model="queryInfo.query"
            clearable
            v-on:clear="getUserList"
          >
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <!-- Add area -->
          <el-button type="primary" v-on:click="addDialogVisible = true">Add user</el-button>
        </el-col>
      </el-row>
    </el-card>
    <!-- List of users (Table) area -->
    <el-table :data="userList" border stripe>
      <el-table-column type="index"></el-table-column>
      <el-table-column prop="username" label="Name"></el-table-column>
      <el-table-column prop="email" label="Email"></el-table-column>
      <el-table-column prop="mobile" label="Phone"></el-table-column>
      <el-table-column prop="role_name" label="Role"></el-table-column>
      <el-table-column label="State">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.mg_state"
            v-on:change="userStateChanged(scope.row)"
            active-color="#13ce66"
            inactive-color="#ff4949"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="Operation" width="180px">
        <template slot-scope="scope">
          <!-- Modify -->
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
            v-on:click="showModifyDialog(scope.row.id)"
          ></el-button>
          <!-- Delete -->
          <el-button
            v-on:click="removeUserById(scope.row.id)"
            type="danger"
            icon="el-icon-delete"
            size="mini"
          ></el-button>
          <!-- Assign roles -->
          <el-tooltip
            class="item"
            effect="dark"
            content="Assign roles"
            placement="top"
            :enterable="false"
          >
            <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
          </el-tooltip>
        </template>
      </el-table-column>
    </el-table>
    <!-- Paging navigation -->
    <el-pagination
      :current-page="queryInfo.pagenum"
      :page-size="queryInfo.pagesize"
      :page-sizes="[1, 2, 5, 10]"
      :total="total"
      v-on:size-change="handleSizeChange"
      v-on:current-change="handleCurrentChange"
      layout="total, sizes, prev, pager, next, jumper"
    ></el-pagination>
    <!-- Dialog area -->
    <el-dialog
      v-on:close="addDialogClosed"
      title="Add user"
      :visible.sync="addDialogVisible"
      width="50%"
    >
      <!-- The body area -->
      <el-form ref="addFormRef" :rules="addFormRules" :model="addForm" label-width="70px">
        <el-form-item label="The user name" prop="username" label-width="auto">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="Password" prop="password" label-width="auto">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="Email" prop="email" label-width="auto">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="The phone" prop="mobile" label-width="auto">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- The footer area -->
      <span slot="footer" class="dialog-footer">
        <el-button v-on:click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" v-on:click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- Modify The User dialog box -->
    <el-dialog
      title="Modify the user"
      v-on:close="modifyDialogClosed"
      :visible.sync="modifyDialogVisible"
      width="50%"
    >
      <!-- The body area -->
      <el-form ref="modifyFormRef" :rules="modifyFormRules" :model="modifyForm" label-width="70px">
        <el-form-item label="The user name" label-width="auto">
          <el-input v-model="modifyForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="Email" prop="email" label-width="auto">
          <el-input v-model="modifyForm.email"></el-input>
        </el-form-item>
        <el-form-item label="The phone" prop="mobile" label-width="auto">
          <el-input v-model="modifyForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- The footer area -->
      <span slot="footer" class="dialog-footer">
        <el-button v-on:click="modifyDialogVisible = false">Cancel</el-button>
        <el-button type="primary" v-on:click="modifyUser">Determine</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    // Rules for validating mailboxes
    var checkEmail = (rule, value, callback) => {
      const regEmail = /^\w+@\w+(\.\w+)+$/
      if (regEmail.test(value)) return callback()
      callback(new Error('Please enter a valid email address'))
    }
    // Rules for verifying mobile phone numbers
    var checkMobile = (rule, value, callback) => {
      const regMobile = /^1[34578]\d{9}$/
      if (regMobile.test(value)) return callback()
      callback(new Error('Please enter a valid mobile phone number'))
    }
    return {
      queryInfo: { // Request user list object
        query: '',
        pagenum: 1, // What page
        pagesize: 2 // Each page of a few
      },
      userList: [], // List of users
      total: 0, // A total of a few pages
      addDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      addFormRules: {
        username: [
          { required: true, message: 'Please enter a user name', trigger: 'blur' },
          { min: 3, max: 10, message: 'The value contains 3 to 10 characters', trigger: 'blur' }
        ],
        password: [
          { required: true, message: 'Please enter your password', trigger: 'blur' },
          { min: 6, max: 15, message: 'The value contains 6 to 15 characters', trigger: 'blur' }
        ],
        email: [
          { required: true, message: 'Please enter email address', trigger: 'blur' },
          { validator: checkEmail, message: 'The email format is not correct. Please re-enter it', trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: 'Please enter your mobile phone number', trigger: 'blur' },
          { validator: checkMobile, message: 'The mobile number is incorrect. Please enter it again', trigger: 'blur' }
        ]
      },
      // Controls whether to display the modify user dialog box
      modifyDialogVisible: false,
      // Modify the user form object
      modifyForm: {
        username: '',
        email: '',
        mobile: ''
      },
      // Modify the validation rule object for the user form
      modifyFormRules: {
        email: [
          { required: true, message: 'Please enter email address', trigger: 'blur' },
          { validator: checkEmail, message: 'The email format is not correct. Please re-enter it', trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: 'Please enter your mobile phone number', trigger: 'blur' },
          { validator: checkMobile, message: 'The mobile number is incorrect. Please enter it again', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    async getUserList () { // Send the request: get user list data
      const { data: result } = await this.$axios.get('users', {
        params: this.queryInfo
      })
      if (result.meta.status !== 200) return this.$message.error('Failed to obtain user list data.')
      this.userList = result.data.users
      this.total = result.data.total
      // console.log(result)
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    async userStateChanged (userInfo) {
      // Send the request: modifying the status
      const { data: result } = await this.$axios.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
      if (result.meta.status !== 200) {
        userInfo.mg_state = !userInfo.mg_state
        return this.$message.error('Failed to change status.')
      }
      this.$message.success('Update status succeeded.')
    },
    addDialogClosed () {
      // Reset the form
      this.$refs.addFormRef.resetFields()
    },
    addUser () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return this.$message.error('Please check whether the information filled in is correct')
        const { data: result } = await this.$axios.post('users', this.addForm)
        console.log(result)
        if (result.meta.status !== 201) return this.$message.error('Failed to add a User')
        this.$message.success('Adding a User Succeeded')
        this.addDialogVisible = false
        this.getUserList()
      })
    },
    async showModifyDialog (id) {
      const { data: result } = await this.$axios.get('users/' + id)
      if (result.meta.status !== 200) return this.$message.error('Failed to obtain user information. Procedure')
      this.modifyForm = result.data
      this.modifyDialogVisible = true
    },
    modifyDialogClosed () {
      this.$refs.modifyFormRef.resetFields()
      // console.log(this.modifyForm)
    },
    modifyUser () {
      this.$refs.modifyFormRef.validate(async valid => {
        if (!valid) return this.$message.error('Please check whether the information filled in is correct')
        const { data: result } = await this.$axios.put('users/' + this.modifyForm.id, this.modifyForm)
        if (result.meta.status !== 200) return this.$message.error('Failed to Modify a User')
        this.$message.success('Succeeded in modifying a User')
        this.modifyDialogVisible = false
        this.getUserList()
      })
    },
    async removeUserById (id) {
      const confirmResult = await this.$confirm(
        'Do you want to delete the user permanently?',
        'Delete The Prompt',
        {
          confirmButtonText: 'Yes',
          cancelButtonText: 'No',
          type: 'warning'
        }
      ).catch(error => error)
      // If the user clicks OK, 'confirmResult' is 'confirm'. If the user clicks Cancel, 'confirmResult' is 'cancel'.
      if (confirmResult !== 'confirm') return this.$message.info('The deletion has been cancelled.')
      const { data: result } = await this.$axios.delete('users/' + id)
      if (result.meta.status !== 200) return this.$message.error('Failed to Delete a User.')
      this.$message.success('Deleting a User Succeeded.')
      this.getUserList()
    }
  },
  created () {
    this.getUserList()
  }
}
</script>

<style lang="less">
</style>
