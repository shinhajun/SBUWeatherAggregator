<!-- WeatherDashboard.vue-->
<template>
  <div class="container my-4">
    <h2 class="mb-3 text-center">StonyBrookWeather.com</h2>
    <h5 class="mb-3 text-center">Team SkyWolves</h5>

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

// 자식 컴포넌트들 임포트
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
      finalForecast: {
        temperature: 0,
        rainProbability: 0,
        snowProbability: 0
      },
      hourlyForecast: null,
      weeklyForecast: [],
      accuracyData: [],
      providerWeights: {},
      weightsHistory: [],
      dailyOverview: null, // 혹시 필요하면 사용

      // Google Map 설정
      googleMapSrc: '',

      // 자동 새로고침용 카운트다운
      countdown: 5,
      autoRefreshInterval: null,

      // Heroku 백엔드 URL
      backendBaseUrl: 'https://sbuweather-backend-85bc4d585c0d.herokuapp.com'
    };
  },
  methods: {
    async fetchAllData() {
      try {
        // 1) /api/forecast
        const forecastResp = await axios.get(`${this.backendBaseUrl}/api/forecast`);
        const data = forecastResp.data;

        // 백엔드 응답 구조에 맞게 필드 매칭
        // data.hourlyForecastWithBreakdown가 undefined면 dummy로 대체
        this.hourlyForecast = data.hourlyForecastWithBreakdown || this.getDummyHourlyForecast();

        this.finalForecast = data.finalForecast || {
          temperature: 0,
          rainProbability: 0,
          snowProbability: 0
        };

        this.weeklyForecast = data.weeklyForecast || [];
        this.providerWeights = data.weights || {};
        this.dailyOverview = data.dailyOverview || null;

        // 2) /api/accuracy
        const accResp = await axios.get(`${this.backendBaseUrl}/api/accuracy`);
        this.accuracyData = accResp.data || [];

        // 3) /api/weights-history
        const weightsHistoryResp = await axios.get(`${this.backendBaseUrl}/api/weights-history`);
        this.weightsHistory = weightsHistoryResp.data || [];

      } catch (err) {
        console.error('Error fetching data:', err);
      }
    },

    // fallback dummy
    getDummyHourlyForecast() {
      return {
        temperature: 10.5,
        rainProbability: 0.2,
        snowProbability: 0.1,
        calculation: {
          temperature: [
            { provider: 'NationalWeatherService', value: 10, weight: 0.33, contribution: 3.3 },
            { provider: 'OpenWeatherMap', value: 11, weight: 0.33, contribution: 3.63 },
            { provider: 'WeatherBit', value: 10.5, weight: 0.34, contribution: 3.57 }
          ],
          rainProbability: [
            { provider: 'NationalWeatherService', value: 0.1, weight: 0.33, contribution: 0.033 },
            { provider: 'OpenWeatherMap', value: 0.2, weight: 0.33, contribution: 0.066 },
            { provider: 'WeatherBit', value: 0.25, weight: 0.34, contribution: 0.085 }
          ],
          snowProbability: [
            { provider: 'NationalWeatherService', value: 0.05, weight: 0.33, contribution: 0.0165 },
            { provider: 'OpenWeatherMap', value: 0.1, weight: 0.33, contribution: 0.033 },
            { provider: 'WeatherBit', value: 0.15, weight: 0.34, contribution: 0.051 }
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
    // Google Map API 키를 Vite 환경변수에서 가져와 설정
    const gKey = import.meta.env.VITE_GOOGLE_MAPS_KEY;
    this.googleMapSrc = `https://www.google.com/maps/embed/v1/place?key=${gKey}&q=Stony+Brook,NY`;

    // 초기 데이터 로드 및 자동 새로고침
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