<template>
  <div class="card mb-3">
    <div class="card-header bg-info text-white">Weekly Forecast</div>
    <div class="card-body">
      <div v-if="forecast && forecast.length" class="row">
        <div v-for="(day, index) in forecast" :key="index" class="col-12 col-sm-6 col-md-4 col-lg-3 mb-3">
          <div class="card h-100 shadow-sm">
            <div class="card-header text-center bg-primary text-white">
              {{ getDayName(index) }}
            </div>
            <div class="card-body">
              <p><strong>Temperature:</strong> {{ day.temperature.toFixed(2) }} °C</p>
              <p><strong>Rain Probability:</strong> {{ (day.rainProbability * 100).toFixed(2) }}%</p>
              <p><strong>Snow Probability:</strong> {{ (day.snowProbability * 100).toFixed(2) }}%</p>
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
  name: 'WeeklyForecast',
  props: {
    forecast: {
      type: Array,
      default: () => []
    }
  },
  methods: {
    getDayName(index) {
      const today = new Date();
      today.setDate(today.getDate() + index);
      return today.toLocaleDateString(undefined, { weekday: 'long' });
    }
  }
};
</script>