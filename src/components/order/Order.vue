<template>
  <div>
    <!-- Title -->
    <h3>The order list</h3>

    <!-- Breadcrumb navigation -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">Home</el-breadcrumb-item>
      <el-breadcrumb-item>The order management</el-breadcrumb-item>
      <el-breadcrumb-item>The order list</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- Card view area -->
    <el-card>
      <!-- Search bar -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="Please enter the content" v-model="queryInfo.query" clearable>
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- The order form -->
      <el-table :data="orderList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="Order number" prop="order_number"></el-table-column>
        <el-table-column label="Order price" prop="order_price"></el-table-column>
        <el-table-column label="Payment or not">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status === '1'">Paid</el-tag>
            <el-tag type="danger" v-else>Unpaid</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="Ship or not" prop="is_send"></el-table-column>
        <el-table-column label="Order time">
          <template slot-scope="scope">{{ scope.row.create_time | dateFormat }}</template>
        </el-table-column>
        <el-table-column label="Operation" width="125px">
          <template slot-scope>
            <el-button v-on:click="showEditAddress" size="mini" type="primary" icon="el-icon-edit"></el-button>
            <el-button v-on:click="showProgress" size="mini" type="success" icon="el-icon-location"></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- Pagination -->
      <el-pagination
        v-on:size-change="handleSizeChange"
        v-on:current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        :total="total"
        layout="total, sizes, prev, pager, next, jumper"
      ></el-pagination>
    </el-card>

    <!-- Modify address dialog box -->
    <el-dialog
      title="Modify The Shipping Address"
      :visible.sync="addressVisible"
      width="50%"
      v-on:close="addressDialogClosed"
    >
      <el-form
        :model="addressForm"
        :rules="addressFormRules"
        ref="addressFormRef"
        label-width="100px"
      >
        <el-form-item
          label-width="auto"
          label="Provinces, Cities, Districts and Counties"
          prop="address1"
        >
          <el-cascader :options="cityData" v-model="addressForm.address1"></el-cascader>
        </el-form-item>
        <el-form-item label-width="auto" label="Detailed Address" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button v-on:click="addressVisible = false">Cancel</el-button>
        <el-button type="primary" v-on:click="addressVisible = false">OK</el-button>
      </span>
    </el-dialog>
    <!-- Logistics information progress dialog box -->
    <el-dialog title="The logistics process" :visible.sync="progressVisible" width="50%">
      <!-- Time line component -->
      <el-timeline>
        <el-timeline-item
          v-for="(activity, index) in progress"
          :key="index"
          :timestamp="activity.time"
        >{{ activity.context }}</el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>

<script>
import cityData from './citydata.js'
import progress from './progress.js'

export default {
  data () {
    return {
      // Query conditions
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // Order list data
      orderList: [],
      // Total number of data items
      total: 0,
      addressVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [{ required: true, message: 'Please select a province, city, district or county', trigger: 'blur' }],
        address2: [{ required: true, message: 'Please enter the detailed address', trigger: 'blur' }]
      },
      cityData,
      progressVisible: false,
      // Logistics information
      progressInfo: [],
      progress
    }
  },
  created () {
    this.getOrderList()
  },
  methods: {
    // Send the request: Get order list data
    async getOrderList () {
      const { data: result } = await this.$axios.get('orders', { params: this.queryInfo })
      if (result.meta.status !== 200) return this.$message.error('Failed to get order list data!')
      this.total = result.data.total
      this.orderList = result.data.goods
    },
    // The number of data items displayed per page changes
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    // Page number switching
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    showEditAddress () {
      this.addressVisible = true
    },
    addressDialogClosed () {
      this.$refs.addressFormRef.resetFields()
    },
    // Send request: Get logistics data
    async showProgress () {
      this.progressVisible = true
    }
  }
}
</script>

<style lang="less">
.el-cascader {
  width: 100%;
  margin: 0;
}
</style>
