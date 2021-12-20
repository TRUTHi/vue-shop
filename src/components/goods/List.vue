<template>
  <div>
    <h3>List of goods</h3>
    <!-- Breadcrumb navigation -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">Home</el-breadcrumb-item>
      <el-breadcrumb-item>Commodity management</el-breadcrumb-item>
      <el-breadcrumb-item>List of goods</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- Card view area -->
    <el-card>
      <!-- The search area -->
      <el-row :gutter="0" type="flex" justify="space-between">
        <el-col :span="8">
          <el-input placeholder="Please enter the content">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
        <el-col :span="3">
          <el-button class="add-goods-button" v-on:click="goAddPage" type="primary">Add the goods</el-button>
        </el-col>
      </el-row>
      <!-- Table area -->
      <el-table :data="goodsList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="Name of commodity" prop="goods_name"></el-table-column>
        <el-table-column label="Commodity price" prop="goods_price" width="150px"></el-table-column>
        <el-table-column label="Weight of goods" prop="goods_weight" width="150px"></el-table-column>
        <el-table-column label="Creation time" width="190px">
          <template slot-scope="scope">{{scope.row.add_time | dateFormat}}</template>
        </el-table-column>
        <el-table-column label="Operation" width="125px">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit"></el-button>
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              v-on:click="removeGoods(scope.row.goods_id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- Paging areas -->
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
  </div>
</template>

<script>
export default {
  data () {
    return {
      // Query parameters
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // Commodity list information
      goodsList: [],
      // Total number of data items
      total: 0
    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    goAddPage () {
      this.$router.push('/goods/add')
    },
    async removeGoods (goodsId) {
      const confirmResult = await this.$confirm(
        'Do you want to delete this product ?',
        'Delete the prompt !',
        {
          confirmButtonText: 'Yes',
          cancelButtonText: 'No',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') return this.$message.info('Cancelled delete.')
      const { data: result } = await this.$axios.delete(`goods/${goodsId}`)
      if (result.meta.status !== 200) return this.$message.error('Failed to delete an item.')
      this.$message.success('Product Deleted Successfully.')
      this.getGoodsList()
    },
    async getGoodsList () {
      const { data: result } = await this.$axios.get('goods', { params: this.queryInfo })
      // console.log(result)
      if (result.meta.status !== 200) return this.$message.error('Failed to get the item list.')
      this.$message.success('Obtaining the item list succeeded.')
      this.goodsList = result.data.goods
      this.total = result.data.total
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getGoodsList()
    }
  }
}
</script>
