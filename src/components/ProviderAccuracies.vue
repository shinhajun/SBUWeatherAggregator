<template>
  <div class="card mb-3">
    <div class="card-header bg-secondary text-white">Provider Details</div>
    <div class="card-body">
      <div class="table-responsive">
        <table class="table table-bordered custom-table">
          <thead>
            <tr>
              <th>Provider</th>
              <th>Temperature Accuracy (%)</th>
              <th>Rain Accuracy (%)</th>
              <th>Snow Accuracy (%)</th>
              <th>Temp Weight</th>
              <th>Rain Weight</th>
              <th>Snow Weight</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="provider in providerList" :key="provider">
              <td>{{ provider }}</td>
              <td>{{ getLatestAccuracy(provider, 'tempAcc') }}</td>
              <td>{{ getLatestAccuracy(provider, 'rainAcc') }}</td>
              <td>{{ getLatestAccuracy(provider, 'snowAcc') }}</td>
              <td>{{ (weights[provider]?.temp || 0).toFixed(2) }}</td>
              <td>{{ (weights[provider]?.rain || 0).toFixed(2) }}</td>
              <td>{{ (weights[provider]?.snow || 0).toFixed(2) }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ProviderAccuracies',
  props: {
    accuracyData: {
      type: Array,
      default: () => []
    },
    weights: {
      type: Object,
      default: () => ({})
    }
  },
  data() {
    return {
      providerList: ['providerNWS', 'providerOWM', 'providerWB']
    };
  },
  methods: {
    getLatestAccuracy(provider, field) {
      if (!this.accuracyData.length) return 0;
      const latest = this.accuracyData[this.accuracyData.length - 1];
      if (!latest[provider] || !latest[provider][field]) return 0;
      return latest[provider][field];
    }
  }
};
</script>

<style scoped>
/* 표 전체 크기를 조정하고, 폰트 및 셀 패딩을 줄여서 비율에 맞게 표시 */
.table-responsive {
  width: 100%;
  overflow-x: auto;
}

.custom-table {
  font-size: 0.9rem;
}

.custom-table th,
.custom-table td {
  padding: 0.4rem;
}
</style>