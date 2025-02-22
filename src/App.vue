<template>
  <div>
    <!-- 최상위(루트) 컴포넌트. 여기서 WeatherDashboard를 보여줍니다. -->
    <WeatherDashboard />
  </div>
</template>

<script>
import WeatherDashboard from '@/components/WeatherDashboard.vue';

export default {
  name: 'App',
  components: {
    WeatherDashboard
  }
};
</script>

<style>
/* 전역 스타일 (선택) */
</style>