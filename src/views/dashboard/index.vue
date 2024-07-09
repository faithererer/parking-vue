<template>
  <div class="app-container">
    <el-card type="flex" justify="center" align="middle">
      <h1>欢迎使用智慧车位管理系统</h1>
    </el-card>
    <!-- 块类统计 -->
    <el-row :gutter="20" type="flex" justify="center">
      <el-col :span="8">
        <el-card class="box-card">
          <h3>总车位数</h3>
          <div>
            <i class="el-icon-s-shop" style="color: purple"></i>
            <span>{{ parkAmount }}</span>
          </div>
        </el-card>
      </el-col>
      <el-col :span="8">
        <el-card class="box-card">
          <h3>剩余车位数</h3>
          <div>
            <i class="el-icon-s-shop" style="color: green"></i>
            <span>{{ parkSpare }}</span>
          </div>
        </el-card>
      </el-col>
      <el-col :span="8">
        <el-card class="box-card">
          <h3>收入总额</h3>
          <div>
            <i class="el-icon-s-flag" style="color: red"></i>
            <span>{{ totalIncome }}元</span>
          </div>
        </el-card>
      </el-col>
    </el-row>
    <!-- ECharts 绘图区域 -->
    <div ref="chart" class="echarts-chart"></div>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import parkOrderApi from "@/api/parkOrder";
import parkInfoApi from "@/api/parkManage";
import echarts from 'echarts';

export default {
  data() {
    return {
      parkAmount: "", // 总车位
      parkSpare: "", // 剩余车位
      totalIncome: "", // 总收入
      chartData: [], // 曲线图数据
    }
  },
  methods: {
    // 获取每日缴费数据，实际应用中应从后端获取数据
      fetchChartData() {
        // 这里假设 startDate 和 endDate 是你需要的日期范围参数
        const startDate = '2023-01-01';
        const endDate = '2024-06-03';
        parkOrderApi.getDailyIncome(startDate, endDate).then((res) => {
          this.chartData = res.data;
          this.renderChart(); // 获取数据后渲染图表
        });

      },
    // 获取总收入
    getTotalIncome() {
      parkOrderApi.getTotalIncome(this.name).then((res) => {
        this.totalIncome = res.data;
      });
    },
    // 获取总车位和剩余车位
    getParkSpace() {
      parkInfoApi.getParkSpace(this.name).then((res) => {
        this.parkAmount = res.data.parkAmount;
        this.parkSpare = res.data.parkSpare;
      });
    },
    // 渲染图表
    renderChart() {
      const chartDom = this.$refs.chart;
      const myChart = echarts.init(chartDom);

      // 指定图表的配置项和数据
      const option = {
        title: {
          text: '每日缴费曲线图',
        },
        tooltip: {
          trigger: 'axis',
        },
        xAxis: {
          type: 'category',
          data: this.chartData.map(item => item.date), // x 轴数据为日期
          //横轴长度保持不变，即使只有一个数据
          axisLabel: {
            interval: 0,
            rotate: 30,
          },
          
        },
        yAxis: {
          type: 'value',
          name: '缴费金额（元）',
        },
        series: [{
          data: this.chartData.map(item => item.amount), // 曲线数据为缴费金额
          type: 'line',
          smooth: true,
        }],
      };

      // 使用刚指定的配置项和数据显示图表
      myChart.setOption(option);
    },
  },
  computed: {
    ...mapGetters([
      "name", // 从 Vuex 获取的属性
      // 其他属性
    ]),
  },
  created() {
    this.getTotalIncome(); // 获取总收入
    this.getParkSpace(); // 获取车位信息
  },
  mounted() {
    this.fetchChartData(); // 在挂载后获取图表数据并渲染图表
  },
};
</script>

<style scoped>
.app-container {
  margin: 30px;
}

.box-card {
  text-align: center;
  padding: 20px;
}

.echarts-chart {
  height: 400px; /* 设置合适的高度 */
}
</style>
