
<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-50">
    <div class="p-6 max-w-[1000px] w-full bg-gray-100 rounded-lg shadow">
      <h2 class="text-2xl font-bold mb-4 text-center">Интерактивная формула с полиномами Лагерра</h2>

      <!-- Формула -->
      <div
        class="mb-6 border p-4 bg-white rounded text-lg overflow-x-auto"
        style="text-align: center; white-space: normal; word-break: break-word"
        v-html="renderedFormula"
      ></div>

      <!-- Поля ввода -->
      <div class="grid grid-cols-2 gap-4">
        <label>Функция ψ₁(x): <input v-model="psiInput" placeholder="например, sin(x)" /></label>
        <label>k: <input type="number" v-model.number="k" /></label>
        <label>n: <input type="number" v-model.number="n" /></label>
        <label>ℓ (эль): <input type="number" v-model.number="ell" /></label>
        <label>x: <input type="number" v-model.number="x" /></label>
      </div>

      <!-- Кнопка по центру -->
      <div class="col-span-2 flex justify-center mt-6">
        <button class="px-6 py-2 bg-blue-600 text-white rounded" @click="calculate">Вычислить</button>
      </div>

      <!-- Результат -->
      <p class="mt-6 text-xl font-semibold text-center">Результат: {{ result }}</p>
<PsiChart :computePsi="psiFunction" :key="result" />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';
import { compile } from 'mathjs';
import PsiChart from './PsiChart.vue';

const psiInput = ref('sin(x)');
const k = ref(1);
const n = ref(3);
const ell = ref(1);
const x = ref(2);
const result = ref(0);
const renderedFormula = ref('');
const psiFunction = ref(() => 0); // реактивная функция для графика

function updateLatex() {
  const psi = psiInput.value;
  renderedFormula.value = String.raw`
\[
\begin{aligned}
\tilde{\psi}(${x.value}) &= ${psi}_1(${x.value} - 2 \cdot ${n.value} \cdot \ell) - \sum_{i=1}^{${n.value}} \mathcal{R}_i(2 \cdot ${k.value} \cdot ${x.value}) 
\cdot \frac{1}{(2i+1) \cdot ${ell.value}} 
\int_0^1 t^{i-1} e^{${k.value}(t-1)${x.value}} 
\cdot ${psi}_1(${x.value} - 2 \cdot ${n.value} \cdot \ell) \, dt \\
&\quad + \sum_{j=1}^{${n.value - 1}} \sum_{i=1}^{j} \mathcal{R}_i(2 \cdot ${k.value} \cdot ${x.value}) 
\cdot \frac{1}{(2j-1) \cdot ${ell.value}} \cdot \int_0^1 t^{i-1} e^{${k.value}(t-1)${x.value}} 
\cdot ${psi}_1(t - 2j \ell) \, dt
\end{aligned}
\]
`;

  setTimeout(() => window.MathJax?.typesetPromise(), 0);
}

function factorial(m) {
  if (m <= 1) return 1;
  return m * factorial(m - 1);
}

function laguerreR(i, y) {
  const r = 2 * i;
  const coeff = Math.pow(2 * i, r) / factorial(r - 1);
  let poly = 0;
  for (let m = 0; m <= i - 1; m++) {
    const term = (Math.pow(-1, m) * factorial(i - 1 + r)) /
      (factorial(i - 1 - m) * factorial(r + m) * factorial(m));
    poly += term * Math.pow(y, m);
  }
  return coeff * poly;
}

function numericalIntegral(f, a, b, steps = 100) {
  const h = (b - a) / steps;
  let sum = 0.5 * (f(a) + f(b));
  for (let i = 1; i < steps; i++) {
    sum += f(a + i * h);
  }
  return sum * h;
}

function calculate() {
  const psiExpr = compile(psiInput.value);
  const y = 2 * k.value * x.value;
  let total = psiExpr.evaluate({ x: x.value - 2 * n.value * ell.value });

  for (let i = 1; i <= n.value; i++) {
    const R = laguerreR(i, y);
    const integral = numericalIntegral(t =>
      Math.pow(t, i - 1) * Math.exp(k.value * (t - 1) * x.value) *
      psiExpr.evaluate({ x: x.value - 2 * n.value * ell.value }), 0, 1);
    total -= R * integral / ((2 * i + 1) * ell.value);
  }

  for (let j = 1; j <= n.value - 1; j++) {
    for (let i = 1; i <= j; i++) {
      const R = laguerreR(i, y);
      const integral = numericalIntegral(t =>
        Math.pow(t, i - 1) * Math.exp(k.value * (t - 1) * x.value) *
        psiExpr.evaluate({ x: t - 2 * j * ell.value }), 0, 1);
      total += R * integral / ((2 * j - 1) * ell.value);
    }
  }

  result.value = total;
  psiFunction.value = getPsiFunction();
}

function getPsiFunction() {
  const psiExpr = compile(psiInput.value);
  return (xVal) => {
    try {
      const y = 2 * k.value * xVal;
      let total = psiExpr.evaluate({ x: xVal - 2 * n.value * ell.value });

      for (let i = 1; i <= n.value; i++) {
        const R = laguerreR(i, y);
        const integral = numericalIntegral(t =>
          Math.pow(t, i - 1) * Math.exp(k.value * (t - 1) * xVal) *
          psiExpr.evaluate({ x: xVal - 2 * n.value * ell.value }), 0, 1);
        total -= R * integral / ((2 * i + 1) * ell.value);
      }

      for (let j = 1; j <= n.value - 1; j++) {
        for (let i = 1; i <= j; i++) {
          const R = laguerreR(i, y);
          const integral = numericalIntegral(t =>
            Math.pow(t, i - 1) * Math.exp(k.value * (t - 1) * xVal) *
            psiExpr.evaluate({ x: t - 2 * j * ell.value }), 0, 1);
          total += R * integral / ((2 * j - 1) * ell.value);
        }
      }

      return isFinite(total) ? total : 0;
    } catch (e) {
      console.warn("Ошибка при вычислении ψ̃(x):", e);
      return 0;
    }
  };
}

watch([psiInput, k, n, ell, x], updateLatex, { immediate: true });
onMounted(updateLatex);
</script>

<style scoped>
input {
  border: 1px solid #ccc;
  padding: 6px;
  width: 100%;
}
</style>
