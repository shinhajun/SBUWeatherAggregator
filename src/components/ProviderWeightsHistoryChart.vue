<template>
  <div class="card mb-3">
    <div class="card-header bg-info text-white">Provider Weights History (Line Charts)</div>
    <div v-if="weightsHistory && weightsHistory.length" class="card-body grid-container">
      <div v-for="provider in providers" :key="provider">
        <div v-for="metric in metrics" :key="metric" class="chart-cell">
          <canvas :ref="el => setCanvasRef(el, provider, metric)"></canvas>
        </div>
      </div>
    </div>
    <div v-else class="p-3">
      No weights history data yet.
    </div>
  </div>
</template>

<script>
import { onMounted, watch, nextTick } from 'vue';
import Chart from 'chart.js/auto';

export default {
  name: 'ProviderWeightsHistoryChart',
  props: {
    weightsHistory: {
      type: Array,
      default: () => []
    }
  },
  setup(props) {
    const providers = ['providerNWS', 'providerOWM', 'providerWB'];
    const metrics = ['temp', 'rain', 'snow'];
    const canvasRefs = {};
    const charts = {};

    function setCanvasRef(el, provider, metric) {
      if (el) {
        canvasRefs[`${provider}-${metric}`] = el;
      }
    }

    const renderCharts = async () => {
      await nextTick();
      Object.values(charts).forEach(chart => chart && chart.destroy());

      // 마지막 3건
      const last3 = props.weightsHistory.slice(-3);
      const labels = ['2 hours ago', '1 hour ago', 'Now'];

      providers.forEach(provider => {
        metrics.forEach(metric => {
          const key = `${provider}-${metric}`;
          const el = canvasRefs[key];
          if (!el) return;

          const ctx = el.getContext('2d');
          const dataPoints = last3.map(snapshot => snapshot[provider]?.[metric] || 0);

          charts[key] = new Chart(ctx, {
            type: 'line',
            data: {
              labels,
              datasets: [
                {
                  label: `${provider} - ${metric}`,
                  data: dataPoints,
                  borderColor: getColor(provider, metric),
                  fill: false,
                  tension: 0.1
                }
              ]
            },
            options: {
              responsive: true,
              maintainAspectRatio: false,
              animation: false,
              scales: {
                y: {
                  beginAtZero: true,
                  max: 1
                }
              }
            }
          });
        });
      });
    };

    function getColor(provider, metric) {
      const colorMap = {
        providerNWS: {
          temp: 'rgba(255, 99, 132, 1)',
          rain: 'rgba(54, 162, 235, 1)',
          snow: 'rgba(75, 192, 192, 1)'
        },
        providerOWM: {
          temp: 'rgba(255, 159, 64, 1)',
          rain: 'rgba(153, 102, 255, 1)',
          snow: 'rgba(255, 205, 86, 1)'
        },
        providerWB: {
          temp: 'rgba(201, 203, 207, 1)',
          rain: 'rgba(100, 149, 237, 1)',
          snow: 'rgba(60, 179, 113, 1)'
        }
      };
      return colorMap[provider]?.[metric] || 'black';
    }

    onMounted(renderCharts);
    watch(() => props.weightsHistory, renderCharts);

    return { providers, metrics, setCanvasRef };
  }
};
</script>

<style scoped>
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 30px;
}
.chart-cell {
  position: relative;
  width: 320px;
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