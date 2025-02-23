<template>
  <div class="card mb-3">
    <div class="card-header bg-info text-white">Provider Weights History</div>
    <div v-if="weightsHistory && weightsHistory.length" class="card-body grid-container">
      <!-- providers x metrics = 3x3 = 9개의 차트 -->
      <div
        v-for="provider in providers"
        :key="provider"
        class="provider-column"
      >
        <div
          v-for="metric in metrics"
          :key="metric"
          class="chart-cell"
        >
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
    const providers = ['NationalWeatherService', 'OpenWeatherMap', 'WeatherBit'];
    const metrics = ['temp', 'rain', 'snow'];

    // canvasRefs: provider-metric => canvas element
    const canvasRefs = {};
    // charts: provider-metric => Chart.js instance
    const charts = {};

    function setCanvasRef(el, provider, metric) {
      if (el) {
        canvasRefs[`${provider}-${metric}`] = el;
      }
    }

    const renderCharts = async () => {
      await nextTick();
      // 기존 차트 destroy (같은 canvas에 새로 그릴 때 메모리 누수 방지)
      Object.values(charts).forEach(chart => {
        if (chart) chart.destroy();
      });

      // weightsHistory에서 마지막 3건만 사용
      const last3 = props.weightsHistory.slice(-3);
      // 차트의 x축 라벨 예시
      const labels = ['2H ago', '1H ago', 'Now'];

      providers.forEach(provider => {
        metrics.forEach(metric => {
          const key = `${provider}-${metric}`;
          const el = canvasRefs[key];
          if (!el) return;

          const ctx = el.getContext('2d');
          // 해당 provider-metric의 최근 3개 데이터
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
                  max: 1 // 가중치 0~1 범위
                }
              }
            }
          });
        });
      });
    };

    // provider & metric별 라인 색상 지정 (원하는 대로 변경 가능)
    function getColor(provider, metric) {
      const colorMap = {
        NationalWeatherService: {
          temp: 'rgba(255, 99, 132, 1)',
          rain: 'rgba(54, 162, 235, 1)',
          snow: 'rgba(75, 192, 192, 1)'
        },
        OpenWeatherMap: {
          temp: 'rgba(255, 159, 64, 1)',
          rain: 'rgba(153, 102, 255, 1)',
          snow: 'rgba(255, 205, 86, 1)'
        },
        WeatherBit: {
          temp: 'rgba(201, 203, 207, 1)',
          rain: 'rgba(100, 149, 237, 1)',
          snow: 'rgba(60, 179, 113, 1)'
        }
      };
      return colorMap[provider]?.[metric] || 'black';
    }

    onMounted(renderCharts);
    // weightsHistory가 바뀔 때마다 차트 다시 그림
    watch(() => props.weightsHistory, renderCharts);

    return { providers, metrics, setCanvasRef };
  }
};
</script>

<style scoped>
.card-header {
  font-weight: bold;
}

.grid-container {
  /* 3개의 provider 열, 가로 공간 1fr로 나눔 */
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  /* padding: 10px;  // 필요하면 추가 */
}

/* provider-column 안에서 metrics 3개가 세로로 쌓임 */
.provider-column {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

/* 차트 컨테이너에 aspect-ratio를 지정해 반응형 유지 */
.chart-cell {
  position: relative;
  width: 100%;
  aspect-ratio: 4 / 3; /* 4:3 비율 예시 */
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 6px;
  padding: 4px;
}

/* 차트(canvas)가 부모 .chart-cell의 크기에 맞게 늘어남 */
canvas {
  width: 100% !important;
  height: 100% !important;
}

/* 화면이 1200px 이하이면 2열로 */
@media (max-width: 1200px) {
  .grid-container {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* 화면이 768px 이하이면 1열로 (모바일) */
@media (max-width: 768px) {
  .grid-container {
    grid-template-columns: 1fr;
  }
}
</style>