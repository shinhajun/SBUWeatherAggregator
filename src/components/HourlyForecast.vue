<template>
  <div class="card mb-3">
    <div class="card-header bg-warning text-dark">Next 1 Hour Forecast</div>
    <div class="card-body">
      <div v-if="forecast && forecast.calculation">
        <p>
          <strong>Temperature:</strong> {{ forecast.temperature.toFixed(2) }} °C<br>
          <strong>Rain Probability:</strong> {{ (forecast.rainProbability * 100).toFixed(2) }} %<br>
          <strong>Snow Probability:</strong> {{ (forecast.snowProbability * 100).toFixed(2) }} %
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