<template>
  <div class="card mb-3">
    <div class="card-header bg-secondary text-white">Provider Details</div>
    <div class="card-body">
      <table class="table table-bordered">
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
            <!-- 최신 accuracyData가 있으면 마지막 요소를 참고 -->
            <td>{{ getLatestAccuracy(provider, 'tempAcc') }}</td>
            <td>{{ getLatestAccuracy(provider, 'rainAcc') }}</td>
            <td>{{ getLatestAccuracy(provider, 'snowAcc') }}</td>
            <!-- 가중치도 소수점 2자리 등으로 표시 -->
            <td>{{ (weights[provider]?.temp || 0).toFixed(2) }}</td>
            <td>{{ (weights[provider]?.rain || 0).toFixed(2) }}</td>
            <td>{{ (weights[provider]?.snow || 0).toFixed(2) }}</td>
          </tr>
        </tbody>
      </table>
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
    /**
     * accuracyData 중 최신 항목에서 해당 provider의 accuracy 가져오기
     */
    getLatestAccuracy(provider, field) {
      if (!this.accuracyData.length) return 0;
      const latest = this.accuracyData[this.accuracyData.length - 1];
      if (!latest[provider] || !latest[provider][field]) return 0;
      return latest[provider][field];
    }
  }
};
</script>