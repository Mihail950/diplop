<template>
  <div class="mt-8">
    <h3 class="text-xl font-semibold mb-2">График функции 𝜓̃(x)</h3>
    <canvas ref="chartCanvas" />
  </div>
</template>

<script setup>
import { ref, nextTick, watchEffect } from 'vue';
import {
  Chart, LineController, LineElement, PointElement,
  LinearScale, Title, CategoryScale
} from 'chart.js';

Chart.register(LineController, LineElement, PointElement, LinearScale, Title, CategoryScale);

const props = defineProps({
  computePsi: Function // Передаётся функция напрямую
});

const chartCanvas = ref(null);
const chartInstance = ref(null);

function generateDataPoints() {
  const xs = [];
  const ys = [];
  const psi = props.computePsi;

  if (typeof psi !== 'function') return { xs, ys };

  for (let i = 0; i <= 100; i++) {
    const x = i * 0.1;
    let y = 0;
    try {
      y = psi(x);
      if (!isFinite(y)) y = 0;
    } catch (e) {
      y = 0;
    }
    xs.push(Number(x.toFixed(2)));
    ys.push(y);
  }

  return { xs, ys };
}

async function renderChart() {
  await nextTick();
  const { xs, ys } = generateDataPoints();

  if (chartInstance.value) {
    chartInstance.value.destroy();
  }

  chartInstance.value = new Chart(chartCanvas.value, {
    type: 'line',
    data: {
      labels: xs,
      datasets: [{
        label: '𝜓̃(x)',
        data: ys,
        borderColor: 'rgba(75,192,192,1)',
        borderWidth: 2,
        pointRadius: 0,
        fill: false
      }]
    },
    options: {
      responsive: true,
      plugins: {
        title: {
          display: true,
          text: 'График функции 𝜓̃(x)'
        }
      },
      scales: {
        x: { title: { display: true, text: 'x' } },
        y: {
          title: { display: true, text: '𝜓̃(x)' },
          beginAtZero: false,
          suggestedMin: Math.min(...ys) * 1.1,
          suggestedMax: Math.max(...ys) * 1.1
        }
      }
    }
  });
}

// Автоматический запуск при любом изменении зависимости
watchEffect(() => {
  renderChart();
});
</script>

<style scoped>
canvas {
  max-width: 100%;
}
</style>
