<template>
  <div class="stats-card">
    <h3>{{ title }}</h3>
    <div class="stats-grid">
      <div class="stat-item">
        <div class="stat-label">Average</div>
        <div class="stat-value">{{ avg ? avg.toFixed(1) : '--' }}{{ unit }}</div>
      </div>
      <div class="stat-item">
        <div class="stat-label">Min</div>
        <div class="stat-value">{{ min ? min.toFixed(1) : '--' }}{{ unit }}</div>
      </div>
      <div class="stat-item">
        <div class="stat-label">Max</div>
        <div class="stat-value">{{ max ? max.toFixed(1) : '--' }}{{ unit }}</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    title: String,
    avg: Number,
    min: Number,
    max: Number,
    unit: String
  }
};
</script>

<style scoped>
.stats-card {
  background-color: white;
  border-radius: 8px;
  padding: 1rem;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.stats-card h3 {
  margin-top: 0;
  margin-bottom: 1rem;
  color: #333;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

.stat-item {
  text-align: center;
}

.stat-label {
  font-size: 0.8rem;
  color: #666;
  margin-bottom: 0.25rem;
}

.stat-value {
  font-weight: bold;
  font-size: 1.2rem;
  color: #42b983;
}
</style>