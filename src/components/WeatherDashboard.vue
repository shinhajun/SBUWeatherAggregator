<template>
  <div class="container my-4">
    <h2 class="mb-3 text-center">Stony Brook Weather Dashboard</h2>

    <!-- Google Map -->
    <GoogleMap :src="googleMapSrc" />

    <!-- Next 1 Hour Forecast -->
    <HourlyForecast :forecast="hourlyForecast" />

    <!-- Weekly Forecast -->
    <WeeklyForecast :forecast="weeklyForecast" />

    <!-- Provider Accuracies -->
    <!-- 정확도는 accuracyData 중 최신값 사용, computeAccuracy 대신 서버 데이터를 활용 -->
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
      <!-- 카운트다운 표시 -->
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
      // 서버에서 가져올 주요 데이터들
      forecasts: null,
      finalForecast: { temperature: 0, rainProbability: 0, snowProbability: 0 },
      hourlyForecast: {},
      weeklyForecast: [],
      accuracyData: [],      // 서버의 accuracyHistory
      providerWeights: {},   // 서버의 현재 가중치
      weightsHistory: [],    // 서버의 가중치 변화 이력

      // 구글 맵
      googleMapSrc: '',

      // 카운트다운 자동 새로고침
      countdown: 5,
      autoRefreshInterval: null
    };
  },
  methods: {
    /**
     * 서버로부터 모든 데이터를 가져오는 함수
     */
    async fetchAllData() {
      try {
        // /api/forecast => forecasts, finalForecast, hourlyForecast, weeklyForecast, weights
        const forecastResp = await axios.get('http://localhost:3000/api/forecast');
        const data = forecastResp.data;
        this.forecasts = data.forecasts;
        this.finalForecast = data.finalForecast;
        this.hourlyForecast = data.hourlyForecast || this.getDummyHourlyForecast();
        this.weeklyForecast = data.weeklyForecast;
        this.providerWeights = data.weights;

        // 정확도 히스토리 (/api/accuracy)
        const accResp = await axios.get('http://localhost:3000/api/accuracy');
        this.accuracyData = accResp.data;

        // 가중치 히스토리 (/api/weights-history)
        const weightsHistoryResp = await axios.get('http://localhost:3000/api/weights-history');
        this.weightsHistory = weightsHistoryResp.data;
      } catch (err) {
        console.error('Error fetching data:', err);
      }
    },
    /**
     * 데이터가 없을 때 보여줄 더미 예보
     */
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
    /**
     * 수동으로 새로고침 버튼을 눌렀을 때
     */
    manualRefresh() {
      this.fetchAllData();
      this.countdown = 5; // 카운트다운 리셋
    },
    /**
     * 카운트다운 자동 새로고침
     */
    startAutoRefresh() {
      // 혹시 기존 인터벌이 있으면 정리
      if (this.autoRefreshInterval) {
        clearInterval(this.autoRefreshInterval);
      }
      // 1초마다 카운트다운
      this.autoRefreshInterval = setInterval(() => {
        if (this.countdown > 0) {
          this.countdown--;
        } else {
          // 카운트가 0이면 새로고침 후 다시 5로 세팅
          this.fetchAllData();
          this.countdown = 5;
        }
      }, 1000);
    }
  },
  mounted() {
    // 구글 맵 설정
    const gKey = import.meta.env.VITE_GOOGLE_MAPS_KEY;
    this.googleMapSrc = `https://www.google.com/maps/embed/v1/place?key=${gKey}&q=Stony+Brook,NY`;

    // 최초 로딩 시 데이터 가져오고, 자동 새로고침 시작
    this.fetchAllData();
    this.startAutoRefresh();
  },
  beforeUnmount() {
    // 컴포넌트 언마운트 시 인터벌 해제
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