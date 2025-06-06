<template>
  <div class="chart-container">
    <h3>{{ title }}</h3>
    <div class="chart-wrapper">
      <canvas ref="chartCanvas"></canvas>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, watch, nextTick } from 'vue';
import { Chart, registerables } from 'chart.js';
import 'chartjs-adapter-date-fns';

Chart.register(...registerables);

export default {
  props: {
    title: String,
    data: Array,
    dataKey: String,
    color: String,
    unit: String
  },
  setup(props) {
    const chartCanvas = ref(null);
    let chartInstance = null;

    const createChart = () => {
      if (!chartCanvas.value || !props.data || props.data.length === 0) {
        if (chartInstance) {
          chartInstance.destroy();
          chartInstance = null;
        }
        return;
      }

      const ctx = chartCanvas.value.getContext('2d');
      
      if (chartInstance) {
        chartInstance.destroy();
      }

      const hasRangeData = props.data[0][`min${props.dataKey.charAt(0).toUpperCase() + props.dataKey.slice(1)}`] !== undefined;
      
      const chartData = {
        labels: props.data.map(item => item.timestamp),
        datasets: [
          {
            label: props.dataKey,
            data: props.data.map(item => ({
              x: item.timestamp,
              y: item[props.dataKey]
            })),
            borderColor: props.color,
            backgroundColor: props.color + '20',
            borderWidth: 2,
            fill: true,
            tension: 0.1
          }
        ]
      };

      if (hasRangeData) {
        const minKey = `min${props.dataKey.charAt(0).toUpperCase() + props.dataKey.slice(1)}`;
        const maxKey = `max${props.dataKey.charAt(0).toUpperCase() + props.dataKey.slice(1)}`;
        
        chartData.datasets.push({
          label: `${props.dataKey} Range`,
          data: props.data.map(item => ({
            x: item.timestamp,
            y: [item[minKey], item[maxKey]]
          })),
          backgroundColor: props.color + '40',
          borderColor: props.color + '80',
          borderWidth: 1,
          type: 'bar',
          barPercentage: 1.0,
          categoryPercentage: 1.0
        });
      }

      chartInstance = new Chart(ctx, {
        type: 'line',
        data: chartData,
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            x: {
              type: 'time',
              time: {
                unit: props.data.length > 24 * 7 ? 'day' : 'hour',
                tooltipFormat: 'MMM d, yyyy HH:mm',
                displayFormats: {
                  hour: 'HH:mm',
                  day: 'MMM d'
                }
              },
              title: {
                display: true,
                text: 'Time'
              }
            },
            y: {
              title: {
                display: true,
                text: props.unit
              }
            }
          },
          plugins: {
            tooltip: {
              callbacks: {
                label: function(context) {
                  if (context.datasetIndex === 0) {
                    return `${context.dataset.label}: ${context.parsed.y.toFixed(2)}${props.unit}`;
                  }
                  return null;
                }
              }
            },
            legend: {
              display: false
            }
          }
        }
      });
    };

    onMounted(() => {
      nextTick(() => {
        createChart();
      });
    });

    watch(() => props.data, (newData) => {
      if (newData && newData.length > 0) {
        nextTick(() => {
          createChart();
        });
      }
    }, { deep: true });

    return { chartCanvas };
  }
};
</script>

<style scoped>
.chart-container {
  background-color: white;
  border-radius: 8px;
  padding: 1rem;
  margin-bottom: 1rem;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.chart-wrapper {
  position: relative;
  height: 300px;
  width: 100%;
}
</style>