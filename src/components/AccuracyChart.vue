<template>
  <div class="card mb-3">
    <div class="card-header bg-info text-white">Accuracy Chart</div>
    <div v-if="data && data.length" class="card-body">
      <div class="chart-container">
        <div v-for="provider in providers" :key="provider" class="chart-cell">
          <canvas :ref="el => setCanvasRef(el, provider)"></canvas>
        </div>
      </div>
    </div>
    <div v-else class="p-3">
      No accuracy data yet.
    </div>
  </div>
</template>

<script>
import { onMounted, watch, nextTick } from 'vue';
import Chart from 'chart.js/auto';

export default {
  name: 'AccuracyChart',
  props: {
    data: {
      type: Array,
      default: () => []
    }
  },
  setup(props) {
    const providers = ['NationalWeatherService', 'OpenWeatherMap', 'WeatherBit'];
    const canvasRefs = {};
    const charts = {};

    function setCanvasRef(el, provider) {
      if (el) canvasRefs[provider] = el;
    }

    const renderCharts = async () => {
      await nextTick();
      // 기존 차트가 있으면 파괴
      Object.values(charts).forEach(chart => chart && chart.destroy());

      // "최근 3건"만 사용
      const last3 = props.data.slice(-3);
      // x축 라벨
      const labels = ['2 hours ago', '1 hour ago', 'Now'];

      providers.forEach(provider => {
        const tempData = last3.map(item => parseFloat(item[provider]?.tempAcc || 0));
        const rainData = last3.map(item => parseFloat(item[provider]?.rainAcc || 0));
        const snowData = last3.map(item => parseFloat(item[provider]?.snowAcc || 0));

        const canvasEl = canvasRefs[provider];
        if (!canvasEl) return;
        const ctx = canvasEl.getContext('2d');

        charts[provider] = new Chart(ctx, {
          type: 'line',
          data: {
            labels,
            datasets: [
              { label: 'Temp Accuracy', data: tempData, borderColor: 'red', fill: false },
              { label: 'Rain Accuracy', data: rainData, borderColor: 'blue', fill: false },
              { label: 'Snow Accuracy', data: snowData, borderColor: 'green', fill: false }
            ]
          },
          options: {
            responsive: true,
            maintainAspectRatio: false,
            animation: false,
            scales: {
              y: {
                beginAtZero: true,
                max: 100
              }
            }
          }
        });
      });
    };

    onMounted(renderCharts);
    watch(() => props.data, renderCharts);

    return {
      providers,
      setCanvasRef
    };
  }
};
</script>

<style scoped>
.chart-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: 16px;
}

.chart-cell {
  /* 고정된 비율을 유지하면서 크기를 조정 */
  width: 100%;
  height: 240px;
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 6px;
  padding: 4px;
}

canvas {
  width: 100% !important;
  height: 100% !important;
}
</style>