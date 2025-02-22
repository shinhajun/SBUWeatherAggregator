<template>
  <div class="container my-4">
    <h2 class="mb-3 text-center">Stony Brook Weather Dashboard</h2>
    <h5 class="mb-3 text-center">Team:something</h5>

    <!-- Google Map -->
    <GoogleMap :src="googleMapSrc" />

    <!-- Next 1 Hour Forecast -->
    <HourlyForecast :forecast="hourlyForecast" />

    <!-- Weekly Forecast -->
    <WeeklyForecast :forecast="weeklyForecast" />

    <!-- Provider Accuracies -->
    <ProviderAccuracies
      :accuracyData="accuracyData"
      :weights="providerWeights"
    />

    <!-- Accuracy Chart (최근 3건 그래프) -->
    <AccuracyChart :data="accuracyData" />

    <!-- Provider Weights History Chart (최근 3건 그래프) -->
    <ProviderWeightsHistoryChart :weightsHistory="weightsHistory" />

    <!-- Refresh Button with Countdown -->
    <div class="text-center mb-3">
      <button class="btn btn-outline-primary" @click="manualRefresh">
        Refresh ({{ countdown }})
      </button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import GoogleMap from './GoogleMap.vue';
import HourlyForecast from './HourlyForecast.vue';
import WeeklyForecast from './WeeklyForecast.vue';
import ProviderAccuracies from './ProviderAccuracies.vue';
import AccuracyChart from './AccuracyChart.vue';
import ProviderWeightsHistoryChart from './ProviderWeightsHistoryChart.vue';

export default {
  name: 'WeatherDashboard',
  components: {
    GoogleMap,
    HourlyForecast,
    WeeklyForecast,
    ProviderAccuracies,
    AccuracyChart,
    ProviderWeightsHistoryChart
  },
  data() {
    return {
      // 백엔드에서 가져올 데이터들
      forecasts: null,
      finalForecast: { temperature: 0, rainProbability: 0, snowProbability: 0 },
      hourlyForecast: {},
      weeklyForecast: [],
      accuracyData: [],
      providerWeights: {},
      weightsHistory: [],
      // Google Map 설정
      googleMapSrc: '',
      // 자동 새로고침용 카운트다운
      countdown: 5,
      autoRefreshInterval: null,
      // Heroku 백엔드 URL (여기서 자신의 Heroku 앱 URL을 입력)
      backendBaseUrl: 'https://sbuweather-backend-85bc4d585c0d.herokuapp.com'
    };
  },
  methods: {
    async fetchAllData() {
      try {
        // Heroku 백엔드의 /api/forecast 호출
        const forecastResp = await axios.get(`${this.backendBaseUrl}/api/forecast`);
        const data = forecastResp.data;
        this.forecasts = data.forecasts;
        this.finalForecast = data.finalForecast;
        this.hourlyForecast = data.hourlyForecast || this.getDummyHourlyForecast();
        this.weeklyForecast = data.weeklyForecast;
        this.providerWeights = data.weights;

        // Heroku 백엔드의 /api/accuracy 호출
        const accResp = await axios.get(`${this.backendBaseUrl}/api/accuracy`);
        this.accuracyData = accResp.data;

        // Heroku 백엔드의 /api/weights-history 호출
        const weightsHistoryResp = await axios.get(`${this.backendBaseUrl}/api/weights-history`);
        this.weightsHistory = weightsHistoryResp.data;
      } catch (err) {
        console.error('Error fetching data:', err);
      }
    },
    getDummyHourlyForecast() {
      return {
        temperature: 10.5,
        rainProbability: 0.2,
        snowProbability: 0.1,
        calculation: {
          temperature: [
            { provider: 'NWS', value: 10, weight: 0.33, contribution: 3.3 },
            { provider: 'OWM', value: 11, weight: 0.33, contribution: 3.63 },
            { provider: 'WB', value: 10.5, weight: 0.34, contribution: 3.57 }
          ],
          rainProbability: [
            { provider: 'NWS', value: 0.1, weight: 0.33, contribution: 0.033 },
            { provider: 'OWM', value: 0.2, weight: 0.33, contribution: 0.066 },
            { provider: 'WB', value: 0.25, weight: 0.34, contribution: 0.085 }
          ],
          snowProbability: [
            { provider: 'NWS', value: 0.05, weight: 0.33, contribution: 0.0165 },
            { provider: 'OWM', value: 0.1, weight: 0.33, contribution: 0.033 },
            { provider: 'WB', value: 0.15, weight: 0.34, contribution: 0.051 }
          ]
        }
      };
    },
    manualRefresh() {
      this.fetchAllData();
      this.countdown = 5;
    },
    startAutoRefresh() {
      if (this.autoRefreshInterval) {
        clearInterval(this.autoRefreshInterval);
      }
      this.autoRefreshInterval = setInterval(() => {
        if (this.countdown > 0) {
          this.countdown--;
        } else {
          this.fetchAllData();
          this.countdown = 5;
        }
      }, 1000);
    }
  },
  mounted() {
    // Google Map API 키를 환경변수에서 가져와 설정
    const gKey = import.meta.env.VITE_GOOGLE_MAPS_KEY;
    this.googleMapSrc = `https://www.google.com/maps/embed/v1/place?key=${gKey}&q=Stony+Brook,NY`;

    // 초기 데이터 로드 및 자동 새로고침 시작
    this.fetchAllData();
    this.startAutoRefresh();
  },
  beforeUnmount() {
    if (this.autoRefreshInterval) {
      clearInterval(this.autoRefreshInterval);
    }
  }
};
</script>

<style scoped>
.container {
  max-width: 1200px;
}
</style>