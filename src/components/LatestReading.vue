<template>
  <div class="latest-reading" v-if="data">
    <h3>Latest Reading</h3>
    <div class="reading-time">
      {{ formatTime(data.timestamp) }}
    </div>
    <div class="reading-values">
      <div class="reading-value">
        <span class="label">Temperature:</span>
        <span class="value">{{ data.temperature.toFixed(1) }}°C</span>
      </div>
      <div class="reading-value">
        <span class="label">Humidity:</span>
        <span class="value">{{ data.humidity.toFixed(1) }}%</span>
      </div>
    </div>
  </div>
</template>

<script>
import { format } from 'date-fns';

export default {
  props: {
    data: Object
  },
  methods: {
    formatTime(timestamp) {
      if (!timestamp) return '';
      return format(new Date(timestamp), 'MMM d, yyyy HH:mm:ss');
    }
  }
};
</script>

<style scoped>
.latest-reading {
  padding: 1rem;
}

.reading-time {
  font-size: 0.9rem;
  color: #666;
  margin-bottom: 1rem;
}

.reading-values {
  display: grid;
  gap: 0.5rem;
}

.reading-value {
  display: flex;
  justify-content: space-between;
}

.label {
  font-weight: bold;
}

.value {
  color: #42b983;
  font-weight: bold;
}
</style>