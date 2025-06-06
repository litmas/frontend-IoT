<template>
  <div class="dashboard">
    <div class="dashboard-header">
      <h2>Sensor Data Overview</h2>
      <div class="time-range-selector">
        <button @click="setTimeRange(1)">1 Hour</button>
        <button @click="setTimeRange(24)">24 Hours</button>
        <button @click="setTimeRange(168)">7 Days</button>
        <button @click="setTimeRange(720)">30 Days</button>
        <input type="datetime-local" v-model="startTime" @change="fetchData">
        <span>to</span>
        <input type="datetime-local" v-model="endTime" @change="fetchData">
      </div>
    </div>

    <div class="dashboard-content">
      <div class="latest-readings">
        <LatestReading :data="latestData" />
      </div>
      
      <div class="stats-cards">
        <StatsCard 
          title="Temperature Stats" 
          :avg="stats.avgTemp"
          :min="stats.minTemp"
          :max="stats.maxTemp"
          unit="°C"
        />
        <StatsCard 
          title="Humidity Stats" 
          :avg="stats.avgHumidity"
          :min="stats.minHumidity"
          :max="stats.maxHumidity"
          unit="%"
        />
      </div>
      
      <div class="charts">
        <DataChart 
          title="Temperature Over Time" 
          :data="chartData"
          dataKey="temperature"
          color="#FF6384"
          unit="°C"
        />
        <DataChart 
          title="Humidity Over Time" 
          :data="chartData"
          dataKey="humidity"
          color="#36A2EB"
          unit="%"
        />
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue';
import DataChart from './DataChart.vue';
import LatestReading from './LatestReading.vue';
import StatsCard from './StatsCard.vue';
import { format, subHours, parseISO } from 'date-fns';

export default {
  components: { DataChart, LatestReading, StatsCard },
  setup() {
    const chartData = ref([]);
    const latestData = ref(null);
    const stats = ref({});
    const startTime = ref('');
    const endTime = ref('');
    const loading = ref(false);

    const formattedData = computed(() => {
      if (!chartData.value || chartData.value.length === 0) return [];
      
      const isAggregated = chartData.value[0].temperature && typeof chartData.value[0].temperature === 'object';
      
      return chartData.value.map(item => {
        const timestamp = item.timestamp || item.date;
        
        if (isAggregated) {
          return {
            timestamp: new Date(timestamp),
            temperature: item.temperature.avg,
            humidity: item.humidity.avg,
            minTemp: item.temperature.min,
            maxTemp: item.temperature.max,
            minHumidity: item.humidity.min,
            maxHumidity: item.humidity.max
          };
        }
        
        return {
          ...item,
          timestamp: new Date(timestamp)
        };
      }).sort((a, b) => a.timestamp - b.timestamp);
    });

    const fetchData = async () => {
      try {
        loading.value = true;
        
        if (!startTime.value || !endTime.value) {
          setTimeRange(24);
          return;
        }
        
        const url = `https://cscloud7-130.lnu.se/backend/api/data?start=${encodeURIComponent(startTime.value)}&end=${encodeURIComponent(endTime.value)}`;
        const statsUrl = `https://cscloud7-130.lnu.se/backend/api/data/stats?start=${encodeURIComponent(startTime.value)}&end=${encodeURIComponent(endTime.value)}`;
        
        const [dataRes, statsRes] = await Promise.all([
          fetch(url),
          fetch(statsUrl)
        ]);
        
        if (!dataRes.ok || !statsRes.ok) {
          throw new Error('Failed to fetch data');
        }
        
        const [data, statsData] = await Promise.all([
          dataRes.json(),
          statsRes.json()
        ]);
        
        chartData.value = data;
        stats.value = statsData;
        
        const endDate = new Date(endTime.value);
        const hoursDiff = (new Date() - endDate) / (1000 * 60 * 60);
        
        if (hoursDiff < 2) {
          const latestRes = await fetch('https://cscloud7-130.lnu.se/backend/api/data/latest');
          if (latestRes.ok) {
            latestData.value = await latestRes.json();
          }
        }
      } catch (error) {
        console.error('Error fetching data:', error);
      } finally {
        loading.value = false;
      }
    };

    const setTimeRange = (hours) => {
      const now = new Date();
      endTime.value = format(now, "yyyy-MM-dd'T'HH:mm");
      startTime.value = format(subHours(now, hours), "yyyy-MM-dd'T'HH:mm");
      fetchData();
    };

    onMounted(() => {
      setTimeRange(24);
      
      setInterval(async () => {
        try {
          const response = await fetch('https://cscloud7-130.lnu.se/backend/api/data/latest');
          if (response.ok) {
            const data = await response.json();
            latestData.value = data;
            
            const endDate = new Date(endTime.value);
            const hoursDiff = (new Date() - endDate) / (1000 * 60 * 60);
            
            if (hoursDiff < 2 && data && data.timestamp) {
              const newDataPoint = {
                ...data,
                timestamp: new Date(data.timestamp)
              };
              
              const exists = chartData.value.some(
                item => new Date(item.timestamp).getTime() === newDataPoint.timestamp.getTime()
              );
              
              if (!exists) {
                chartData.value = [...chartData.value, newDataPoint]
                  .sort((a, b) => new Date(a.timestamp) - new Date(b.timestamp));
              }
            }
          }
        } catch (error) {
          console.error('Error fetching latest data:', error);
        }
      }, 10000);
    });

    return {
      chartData: formattedData,
      latestData,
      stats,
      startTime,
      endTime,
      loading,
      fetchData,
      setTimeRange
    };
  }
};
</script>

<style scoped>
.dashboard {
  max-width: 1200px;
  margin: 0 auto;
}

.dashboard-header {
  margin-bottom: 1rem;
}

.time-range-selector {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  margin-top: 0.5rem;
}

.time-range-selector button {
  padding: 0.25rem 0.5rem;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.time-range-selector button:hover {
  background-color: #3aa876;
}

.time-range-selector input {
  padding: 0.25rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.dashboard-content {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

.latest-readings {
  background-color: #f9f9f9;
  padding: 1rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.stats-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1rem;
}

.charts {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

@media (min-width: 768px) {
  .dashboard-content {
    grid-template-columns: 300px 1fr;
  }
  
  .latest-readings {
    grid-row: span 2;
  }
  
  .stats-cards {
    grid-column: 2;
  }
  
  .charts {
    grid-column: 2;
  }
}
</style>