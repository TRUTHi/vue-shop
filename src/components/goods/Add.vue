<template>
  <div>
    <!-- Title -->
    <h3>Add the goods</h3>

    <!-- Breadcrumb navigation -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">Home</el-breadcrumb-item>
      <el-breadcrumb-item>Commodity management</el-breadcrumb-item>
      <el-breadcrumb-item>Add the goods</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- Card view area -->
    <el-card>
      <!-- Message prompt -->
      <el-alert title="Adding product Information" type="info" center show-icon :closable="false"></el-alert>

      <!-- Article steps -->
      <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
        <el-step title="The basic information"></el-step>
        <el-step title="Product parameters"></el-step>
        <el-step title="Commodity attribute"></el-step>
        <el-step title="Commodity images"></el-step>
        <el-step title="Commodity content"></el-step>
        <el-step title="complete"></el-step>
      </el-steps>

      <!-- Area of the form -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px"
        label-position="top"
      >
        <el-tabs
          v-model="activeIndex"
          tab-position="left"
          :before-leave="beforeTabLeave"
          v-on:tab-click="tabClicked"
        >
          <el-tab-pane label="The basic information" name="0">
            <el-form-item label="Name of commodity" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="Commodity prices" prop="goods_price">
              <el-input v-model="addForm.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="Weight of goods" prop="goods_weight">
              <el-input v-model="addForm.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="The number" prop="goods_number">
              <el-input v-model="addForm.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="Classification of goods" prop="goods_cat">
              <!-- Select commodity categories -->
              <el-cascader
                v-model="addForm.goods_cat"
                :options="cateList"
                :props="cateProps"
                v-on:change="handleChange"
                clearable
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="Product parameters" name="1">
            <el-form-item v-for="item in manyTableData" :key="item.attr_id" :label="item.attr_name">
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  v-for="(value, index) in item.attr_vals"
                  :key="index"
                  :label="value"
                  border
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="Commodity attribute" name="2">
            <el-form-item v-for="item in onlyTableData" :key="item.attr_id" :label="item.attr_name">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="Commodity images" name="3">
            <el-upload
              :action="uploadURL"
              :headers="headerObj"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              :on-success="handleSuccess"
              list-type="picture"
            >
              <el-button size="small" type="primary">Click on the upload</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="Commodity content" name="4">
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <el-button type="primary" class="btn" v-on:click="addTheGoods">Add the goods</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <!-- Preview picture dialog box -->
    <el-dialog title="Preview picture" :visible.sync="previewVisible" width="50%">
      <img :src="previewPath" class="previewImg" />
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  data () {
    return {
      // Save the step bar activation item index
      activeIndex: '0',
      // Add the form data object for the item
      addForm: {
        goods_name: '',
        goods_price: '',
        goods_weight: '',
        goods_number: '',
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      addFormRules: {
        goods_name: [
          { required: true, message: 'Please enter the name of the product', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: 'Please enter commodity price', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: 'Please enter the weight of the goods', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: 'Please enter the quantity of goods', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: 'Please select product category', trigger: 'blur' }
        ]
      },
      cateList: [],
      cateProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      manyTableData: [],
      onlyTableData: [],
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      // Request header object
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      // Save the URL of the preview image
      previewPath: '',
      previewVisible: false
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    addTheGoods () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return this.$message.error('Please fill in the necessary form items !')
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        this.manyTableData.forEach(item => {
          form.attrs.push({ attr_id: item.attr_id, attr_value: item.attr_vals.join(' ') })
        })
        this.onlyTableData.forEach(item => {
          form.attrs.push({ attr_id: item.attr_id, attr_value: item.attr_vals })
        })
        const { data: result } = await this.$axios.post('goods', form)
        if (result.meta.status !== 201) {
          return this.$message.error('Failed to add goods.')
        }
        this.$message.success('Adding an item Succeeded.')
        this.$router.push('/goods')
      })
    },
    beforeTabLeave (activeName, oldActiveName) {
      if (oldActiveName === '0') {
        if (this.addForm.goods_cat.length !== 3) {
          this.$message.error('Please select product category.')
          return false
        } else if (this.addForm.goods_name.trim() === '') {
          this.$message.error('Please enter the name of the product.')
          return false
        } else if (this.addForm.goods_price.trim() === '') {
          this.$message.error('Please enter commodity price')
          return false
        } else if (this.addForm.goods_weight.trim() === '') {
          this.$message.error('Please enter the weight of the goods')
          return false
        } else if (this.addForm.goods_number.trim() === '') {
          this.$message.error('Please enter the quantity of goods')
          return false
        }
      }
    },
    async tabClicked () {
      if (this.activeIndex === '1') {
        // Obtaining dynamic parameters
        const { data: result } = await this.$axios.get(`categories/${this.cateId}/attributes`, { params: { sel: 'many' } })
        if (result.meta.status !== 200) return this.$message.error('Failed to get the dynamic parameter list.')
        result.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = result.data
      } else if (this.activeIndex === '2') {
        // Get static properties
        const { data: result } = await this.$axios.get(`categories/${this.cateId}/attributes`, { params: { sel: 'only' } })
        if (result.meta.status !== 200) return this.$message.error('Failed to get the static property list.')
        this.onlyTableData = result.data
      }
    },
    async getCateList () {
      const { data: result } = await this.$axios.get('categories')
      if (result.meta.status !== 200) return this.$message.error('Failed to get commodity classification data')
      this.cateList = result.data
    },
    handleChange () {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
        return null
      }
    },
    handleSuccess (response) {
      console.log(response)
      // The parameter 'response' is the result returned by the server after a successful upload
      // Save the temporary path returned by the server into the 'pics' array of the 'addForm' form
      this.addForm.pics.push({ pic: response.data.tmp_path })
    },
    handlePreview (file) {
      console.log(file)
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    handleRemove (file) {
      console.log(file)
      // Gets the temporary path to the image that the user clicked to delete
      const filePath = file.response.data.tmp_path
      const index = this.addForm.pics.findIndex(item => item.pic === filePath)
      this.addForm.pics.splice(index, 1)
    }
  },
  computed: {
    cateId () {
      return this.addForm.goods_cat[2]
    }
  }
}
</script>

<style lang="less">
.el-checkbox {
  margin-right: 15px !important;
}

.el-checkbox.is-bordered + .el-checkbox.is-bordered {
  margin-left: 0 !important;
}

.previewImg {
  width: 100%;
}

.btn {
  margin-top: 15px !important;
}

.ql-editor {
  min-height: 300px;
}
</style>
