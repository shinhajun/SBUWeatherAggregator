<!-- HourlyForecast.vue-->
<template>
  <div class="card mb-3">
    <!-- 카드 헤더 (온도 단위 스위치) -->
    <div class="card-header bg-warning text-dark d-flex justify-content-between align-items-center">
      <span>Next 1 Hour Forecast</span>
      <div class="form-check form-switch">
        <input
          class="form-check-input"
          type="checkbox"
          id="tempUnitSwitch"
          v-model="isFahrenheit"
        />
        <label class="form-check-label" for="tempUnitSwitch">
          {{ isFahrenheit ? "°F" : "°C" }}
        </label>
      </div>
    </div>

    <!-- 카드 바디 (2열 레이아웃) -->
    <div class="card-body">
      <div v-if="forecast && forecast.calculation">
        <div class="row">
          <!-- 왼쪽 텍스트 영역 -->
          <div class="col-8">
            <p>
              <strong>Temperature:</strong>
              {{ convertedTemperature.toFixed(2) }} {{ isFahrenheit ? "°F" : "°C" }}
            </p>
            <p>
              <strong>Rain Probability:</strong>
              {{ (forecast.rainProbability * 100).toFixed(2) }} %
            </p>
            <p>
              <strong>Snow Probability:</strong>
              {{ (forecast.snowProbability * 100).toFixed(2) }} %
            </p>
          </div>

          <!-- 오른쪽 아이콘 영역 -->
          <div class="col-4 d-flex align-items-center justify-content-center">
            <img
              :src="forecastIcon"
              alt="Forecast Icon"
              class="big-forecast-icon"
            />
          </div>
        </div>

        <!-- 계산 내역 -->
        <div v-for="(calc, key) in forecast.calculation" :key="key" class="mt-3">
          <p class="mb-1"><strong>{{ getLabel(key) }} Calculation:</strong></p>
          <ul class="list-group">
            <li
              v-for="item in calc"
              :key="item.provider"
              class="list-group-item d-flex justify-content-between align-items-center"
            >
              {{ item.provider }}: ({{ item.value }} x {{ item.weight }}) =
              <span>{{ item.contribution.toFixed(2) }}</span>
            </li>
          </ul>
        </div>
      </div>
      <!-- 예보 데이터가 없을 때 -->
      <div v-else class="text-center">
        <p>Loading hourly forecast...</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "HourlyForecast",
  props: {
    forecast: {
      type: Object,
      default: () => ({})
    }
  },
  data() {
    return {
      isFahrenheit: false
    };
  },
  computed: {
    // 온도 변환
    convertedTemperature() {
      if (!this.forecast || typeof this.forecast.temperature !== "number") {
        return 0;
      }
      return this.isFahrenheit
        ? (this.forecast.temperature * 9) / 5 + 32
        : this.forecast.temperature;
    },
    // 아이콘 표시 로직
    // (눈 확률 > 0) => 눈 아이콘
    // (비 확률 > 0) => 비 아이콘
    // 그 외 => 온도 아이콘
    forecastIcon() {
      if (!this.forecast) return "";
      if (this.forecast.snowProbability > 0) {
        return "https://img.icons8.com/fluency/48/000000/snow.png";
      } else if (this.forecast.rainProbability > 0) {
        return "https://img.icons8.com/fluency/48/000000/rain.png";
      } else {
        return "https://img.icons8.com/?size=100&id=15352&format=png&color=000000";
      }
    }
  },
  methods: {
    getLabel(key) {
      switch (key) {
        case "temperature":
          return "Temperature";
        case "rainProbability":
          return "Rain Probability";
        case "snowProbability":
          return "Snow Probability";
        default:
          return key;
      }
    }
  }
};
</script>

<style scoped>
.big-forecast-icon {
  max-width: 150px; /* 아이콘이 너무 커지지 않도록 제한 */
  width: 80%;       /* 전체 영역 대비 80% 정도 사용 */
  height: auto;
}
</style>