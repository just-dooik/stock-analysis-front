<template>
  <div v-if="companyId">
    <canvas ref="PriceChart"></canvas>
    <canvas ref="RSIChart"></canvas>
  </div>
</template>

<script>
import { Chart, registerables } from 'chart.js';
import zoomPlugin from 'chartjs-plugin-zoom';
import 'chartjs-adapter-date-fns';
import axios from 'axios';

Chart.register(...registerables, zoomPlugin);

export default {
  props: {
    companyId: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      priceChart: null, // 가격 차트를 저장할 변수
      rsiChart: null, // RSI 차트를 저장할 변수
      type: 'line',
      priceData: {
        labels: [],
        datasets: [
          {
            label: 'Adjusted Close',
            data: [],
            borderColor: 'rgba(225, 0, 0, 1)',
            borderWidth: 0.5,
            pointRadius: 0,
            pointHoverRadius: 5,
            fill: false,
          },
          {
            label: 'SMA',
            data: [],
            borderColor: 'rgba(0, 225, 0, 1)',
            borderWidth: 0.5,
            pointRadius: 0,
            pointHoverRadius: 5,
            fill: false,
          },
          {
            label: 'BBands Upper',
            data: [],
            borderColor: 'rgba(0, 0, 225, 1)',
            borderWidth: 0.5,
            pointRadius: 0,
            pointHoverRadius: 5,
            fill: false,
          },
          {
            label: 'BBands Lower',
            data: [],
            borderColor: 'rgba(225, 225, 0, 1)',
            borderWidth: 0.5,
            pointRadius: 0,
            pointHoverRadius: 5,
            fill: false,
          },
        ]
      },
      rsiData: {
        labels: [],
        datasets: [
          {
            label: 'RSI',
            data: [],
            borderColor: 'rgba(0, 0, 225, 1)',
            borderWidth: 0.5,
            pointRadius: 0,
            pointHoverRadius: 5,
            fill: false,
          },
        ]
      },
      sharedZoomOptions: {
        zoom: {
          wheel: {
            enabled: true,
          },
          pinch: {
            enabled: true
          },
          mode: 'x', // x축 방향으로만 줌
          onZoomComplete: ({ chart }) => {
            this.syncCharts(chart); // 줌 동기화
          }
        },
        pan: {
          enabled: true,
          mode: 'x', // x축 방향으로만 팬
          onPanComplete: ({ chart }) => {
            this.syncCharts(chart); // 팬 동기화
          }
        }
      }
    };
  },
  computed: {
    priceOptions() {
      return {
        scales: {
          x: {
            type: 'time',
            time: {
              unit: 'month'
            }
          },
          y: {
            beginAtZero: false
          }
        },
        plugins: {
          zoom: this.sharedZoomOptions
        }
      };
    },
    rsiOptions() {
      return {
        scales: {
          x: {
            type: 'time',
            time: {
              unit: 'month'
            }
          },
          y: {
            beginAtZero: true,
            max: 100, // RSI는 일반적으로 0에서 100 사이의 값을 가짐
          }
        },
        plugins: {
          zoom: this.sharedZoomOptions
        }
      };
    }
  },
  watch: {
    companyId: {
      immediate: true,
      handler(newVal) {
        if (newVal) {
          this.resetCharts();
          this.fetchData(newVal);
        }
      }
    }
  },
  methods: {
    resetCharts() {
      if (this.priceChart) {
        this.priceChart.destroy(); // 기존 가격 차트를 삭제
        this.priceChart = null;
      }
      if (this.rsiChart) {
        this.rsiChart.destroy(); // 기존 RSI 차트를 삭제
        this.rsiChart = null;
      }
    },
    async fetchData(company_id) {
      await this.createAdjusted(company_id);
      await this.createSMA(company_id, 20); // window 크기를 20으로 예시
      await this.createBBands(company_id, 20, 2); // window 크기 20, 표준편차 2로 예시
      await this.createRSI(company_id, 14); // RSI window 크기를 14로 예시

      this.$nextTick(() => {
        if (this.$refs.PriceChart) {
          this.createPriceChart();
        }
        if (this.$refs.RSIChart) {
          this.createRSIChart();
        }
      });
    },
    async createAdjusted(company_id) {
      const apiUrl = `${process.env.VUE_APP_API_URL}/stockprice/${company_id}`;
      console.log('Fetching Adjusted Close data from:', apiUrl);
      try {
        const response = await axios.get(apiUrl);
        const serverData = response.data;

        this.priceData.labels = serverData.map(data => new Date(data.date));
        this.priceData.datasets[0].data = serverData.map(data => data.adjusted_close);
      } catch (error) {
        console.error('Error fetching Adjusted Close data:', error);
      }
    },
    async createSMA(company_id, window) {
      const apiUrl = `${process.env.VUE_APP_API_URL}/tech_indicator/sma/${company_id}/${window}`;
      console.log('Fetching SMA data from:', apiUrl);
      try {
        const response = await axios.get(apiUrl);
        const serverData = response.data;

        this.priceData.datasets[1].data = serverData.map(data => ({
          x: new Date(data.date),
          y: data.SMA
        }));
      } catch (error) {
        console.error('Error fetching SMA data:', error);
      }
    },
    async createBBands(company_id, window, num_stdev) {
      const apiUrl = `${process.env.VUE_APP_API_URL}/tech_indicator/bbands/${company_id}/${window}/${num_stdev}`;
      console.log('Fetching BBands data from:', apiUrl);
      try {
        const response = await axios.get(apiUrl);
        const serverData = response.data;

        this.priceData.datasets[2].data = serverData.map(data => ({
          x: new Date(data.date),
          y: data.upper_band
        }));

        this.priceData.datasets[3].data = serverData.map(data => ({
          x: new Date(data.date),
          y: data.lower_band
        }));
      } catch (error) {
        console.error('Error fetching BBands data:', error);
      }
    },
    async createRSI(company_id, window) {
      const apiUrl = `${process.env.VUE_APP_API_URL}/tech_indicator/rsi/${company_id}/${window}`;
      console.log('Fetching RSI data from:', apiUrl);
      try {
        const response = await axios.get(apiUrl);
        const serverData = response.data;

        this.rsiData.labels = serverData.map(data => new Date(data.date));
        this.rsiData.datasets[0].data = serverData.map(data => data.RSI);
      } catch (error) {
        console.error('Error fetching RSI data:', error);
      }
    },
    createPriceChart() {
      if (!this.priceChart && this.$refs.PriceChart) {
        this.priceChart = new Chart(this.$refs.PriceChart, {
          type: this.type,
          data: this.priceData,
          options: this.priceOptions
        });
      }
    },
    createRSIChart() {
      if (!this.rsiChart && this.$refs.RSIChart) {
        this.rsiChart = new Chart(this.$refs.RSIChart, {
          type: this.type,
          data: this.rsiData,
          options: this.rsiOptions
        });
      }
    },
    syncCharts(sourceChart) {
      // sourceChart의 줌 이벤트가 발생할 때 동기화할 대상 차트를 결정합니다.
      const targetChart = sourceChart === this.priceChart ? this.rsiChart : this.priceChart;

      if (targetChart && sourceChart) {
        const sourceScale = sourceChart.scales.x;
        const targetScale = targetChart.scales.x;

        if (sourceScale.min !== targetScale.min || sourceScale.max !== targetScale.max) {
          targetScale.options.min = sourceScale.min;
          targetScale.options.max = sourceScale.max;

          targetChart.update('none'); // 업데이트 적용
        }
      }
    }
  },
  beforeUnmount() {
    this.resetCharts(); // 컴포넌트가 파괴되기 전에 차트를 제거
  }
}
</script>

<style scoped>
/* 스타일링을 원하는 대로 추가하세요 */
</style>
