<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>数据统计</el-breadcrumb-item>
      <el-breadcrumb-item>数据报表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 2.为 ECharts 准备一个具备大小（宽高）的 DOM -->
      <div id="main" style="width: 800px; height: 400px"></div>
    </el-card>
  </div>
</template>

<script>
// 1.导入echarts
import echarts from "echarts";
import _ from "lodash";

export default {
  data() {
    return {
      // 指定图表的配置项和数据
      options: {
        title: {
          text: "用户来源",
        },
        tooltip: {
          trigger: "axis",
          axisPointer: {
            type: "cross",
            label: {
              backgroundColor: "#E9EEF3",
            },
          },
        },
        grid: {
          left: "3%",
          right: "4%",
          bottom: "3%",
          containLabel: true,
        },
        xAxis: [
          {
            boundaryGap: false,
          },
        ],
        yAxis: [
          {
            type: "value",
          },
        ],
      },
    };
  },
  created() {},
  // 当页面上的DOM元素渲染完成，立即调用mounted
  async mounted() {
    // this.$nextTick(async () => {
    //   this.init();
    // 基于准备好的dom，初始化echarts实例
    var myChart = echarts.init(document.getElementById("main"));
    //请求数据
    const { data: res } = await this.$http.get("reports/type/1");
    if (res.meta.status !== 200) {
      return this.$message.error("获取报表数据失败！");
    }
    // this.reportData = res.data;
    const result = _.merge(this.options, res.data);
    // 使用刚指定的配置项和数据显示图表。
    myChart.setOption(result);
    // console.log(this.reportData);
    // console.log(res.data);
    // });
  },
  methods: {
    // init() {
    // 基于准备好的dom，初始化echarts实例
    // var myChart = echarts.init(document.getElementById("main"));
    // 使用刚指定的配置项和数据显示图表。
    // myChart.setOption(this.reportData);
    // },
  },
};
</script>

<style lang="less">
</style>>