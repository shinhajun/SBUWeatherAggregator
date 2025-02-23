<!-- WeeklyForecast.vue-->
<template>
  <div class="card mb-3">
    <!-- 헤더에 스위치 추가 -->
    <div class="card-header d-flex justify-content-between align-items-center bg-info text-white">
      <span>Weekly Forecast</span>
      <div class="form-check form-switch">
        <input
          class="form-check-input"
          type="checkbox"
          id="tempUnitSwitchWeekly"
          v-model="isFahrenheit"
        />
        <label class="form-check-label" for="tempUnitSwitchWeekly">
          {{ isFahrenheit ? '°F' : '°C' }}
        </label>
      </div>
    </div>
    <div class="card-body">
      <div v-if="forecast && forecast.length" class="row">
        <div
          v-for="(day, index) in forecast"
          :key="index"
          class="col-12 col-sm-6 col-md-4 col-lg-3 mb-3"
        >
          <div class="card h-100 shadow-sm">
            <div class="card-header text-center bg-primary text-white">
              {{ getDayName(index) }}
            </div>
            <div class="card-body">
              <p>
                <strong>Temperature:</strong>
                {{ convertTemperature(day.temperature).toFixed(2) }}
                {{ isFahrenheit ? '°F' : '°C' }}
              </p>
              <p>
                <strong>Rain Probability:</strong>
                {{ (day.rainProbability * 100).toFixed(2) }}%
              </p>
              <p>
                <strong>Snow Probability:</strong>
                {{ (day.snowProbability * 100).toFixed(2) }}%
              </p>
            </div>
          </div>
        </div>
      </div>
      <div v-else class="text-center">
        <p>No weekly forecast data available.</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "WeeklyForecast",
  props: {
    forecast: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      isFahrenheit: false
    };
  },
  methods: {
    getDayName(index) {
      const today = new Date();
      today.setDate(today.getDate() + index);
      return today.toLocaleDateString(undefined, { weekday: "long" });
    },
    convertTemperature(tempC) {
      // 기본 온도 값은 섭씨로 가정
      if (this.isFahrenheit) {
        return (tempC * 9) / 5 + 32;
      }
      return tempC;
    }
  }
};
</script>

<style scoped>
/* 필요한 경우 추가 스타일링 */
</style>