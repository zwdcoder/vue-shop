<template>
<div>
  <!--面包屑导航区域-->
  <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>数据统计</el-breadcrumb-item>
    <el-breadcrumb-item>数据报表</el-breadcrumb-item>
  </el-breadcrumb>

  <!--卡片试图区域-->
  <el-card>
    <div id="main" style="width: 850px;height: 400px;"></div>
  </el-card>
</div>
</template>

<script>
import echarts from 'echarts'
import _ from 'lodash'
export default {
  data() {
    return {
      //需要合并的数据
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
  created() {},
  mounted() {
    var myChart = echarts.init(document.getElementById('main'))

    this.$axios.get("reports/type/1").then(response => {
      let res = response.data
      if (res.meta.status !== 200)
        return this.$message({
          message: "获取折线图失败！",
          type: "error",
          showClose: true
        });
      const result =  _.merge(res.data,this.options)
      myChart.setOption(result);
    })
  },
  methods: {}
}
</script>

<style lang="less" scoped>
</style>
