<template>
  <div>
    <!-- Title -->
    <h3>Data Report</h3>

    <!-- Breadcrumb navigation -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">Home</el-breadcrumb-item>
      <el-breadcrumb-item>Data Statistics</el-breadcrumb-item>
      <el-breadcrumb-item>Data Report</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- Card view area -->
    <el-card>
      <div id="main" style="width:750px;height:400px;"></div>
    </el-card>
  </div>
</template>

<script>
import * as echarts from 'echarts'
import _ from 'lodash'
export default {
  data () {
    return {
      // 需要跟请求的折线图数据合并的 options
      options: {
        title: {
          text: '用户来源'
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            label: {
              backgroundColor: '#E9EEF3'
            }
          }
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: [
          {
            boundaryGap: false
          }
        ],
        yAxis: [
          {
            type: 'value'
          }
        ]
      }
    }
  },
  async mounted () {
    // 在页面 dom 元素加载完毕之后执行的钩子函数 mounted
    // 基于准备好的 dom，初始化 echarts 实例
    var myChart = echarts.init(document.getElementById('main'))
    // 准备数据和配置项
    // 发送请求获取折线图数据
    const { data: res } = await this.$axios.get('reports/type/1')

    if (res.meta.status !== 200) {
      return this.$message.error('获取折线图数据失败')
    }

    // 合并res.data和this.options
    const result = _.merge(res.data, this.options)

    // 使用获取的数据展示图表
    myChart.setOption(result)
  }
}
</script>
