<template>
  <div class="card mb-3">
    <div class="card-header bg-warning text-dark d-flex justify-content-between align-items-center">
      <span>Next 1 Hour Forecast</span>
      <!-- 온도 단위 스위치 -->
      <div class="form-check form-switch">
        <input class="form-check-input" type="checkbox" id="tempUnitSwitch" v-model="isFahrenheit">
        <label class="form-check-label" for="tempUnitSwitch">
          {{ isFahrenheit ? '°F' : '°C' }}
        </label>
      </div>
    </div>
    <div class="card-body">
      <div v-if="forecast && forecast.calculation">
        <p>
          <strong>Temperature:</strong> {{ convertedTemperature.toFixed(2) }} {{ isFahrenheit ? '°F' : '°C' }}
          <img v-if="forecast.temperature" src="temp.png" alt="Temperature" style="height: 24px; margin-left: 8px;">
          <br>
          <strong>Rain Probability:</strong> {{ (forecast.rainProbability * 100).toFixed(2) }} %
          <img v-if="forecast.rainProbability > 0" src="rain.png" alt="Rain" style="height: 24px; margin-left: 8px;">
          <br>
          <strong>Snow Probability:</strong> {{ (forecast.snowProbability * 100).toFixed(2) }} %
          <img v-if="forecast.snowProbability > 0" src="snow.png" alt="Snow" style="height: 24px; margin-left: 8px;">
        </p>
        <div v-for="(calc, key) in forecast.calculation" :key="key" class="mb-3">
          <p class="mb-1"><strong>{{ getLabel(key) }} Calculation:</strong></p>
          <ul class="list-group">
            <li v-for="item in calc" :key="item.provider" class="list-group-item d-flex justify-content-between align-items-center">
              {{ item.provider }}: ({{ item.value }} x {{ item.weight }}) = <span>{{ item.contribution.toFixed(2) }}</span>
            </li>
          </ul>
        </div>
      </div>
      <div v-else class="text-center">
        <p>Loading hourly forecast...</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HourlyForecast',
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
    convertedTemperature() {
      // forecast.temperature는 기본적으로 섭씨 값으로 가정
      if (!this.forecast || typeof this.forecast.temperature !== 'number') {
        return 0;
      }
      return this.isFahrenheit
        ? (this.forecast.temperature * 9) / 5 + 32
        : this.forecast.temperature;
    }
  },
  methods: {
    getLabel(key) {
      switch (key) {
        case 'temperature': return 'Temperature';
        case 'rainProbability': return 'Rain Probability';
        case 'snowProbability': return 'Snow Probability';
        default: return key;
      }
    }
  }
};
</script>

<style scoped>
.card-header {
  /* 헤더 내부 스위치 정렬 */
}
</style>