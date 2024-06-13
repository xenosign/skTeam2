<template>
  <div class="container">
    <Header />
    <div class="chart-container">
      <DoughnutChart :chartData="chartData" :chartOptions="chartOptions" />
    </div>
    <ul class="item-list">
      <li
        v-for="(item, index) in items"
        :key="index"
        :style="{
          backgroundColor: item.backgroundColor,
          borderColor: item.borderColor,
        }"
      >
        <div class="item-icon">
          <i :class="item.icon"></i>
        </div>
        <div class="item-details">
          <p class="item-name">{{ item.name }}</p>
          <p class="item-amount">{{ formatCurrency(item.amount) }}</p>
        </div>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';
import DoughnutChart from '@/components/DoughnutChart.vue';
import Header from '@/components/Header.vue';
import { useTransactionStore } from '@/stores/transaction';

const transactionStore = useTransactionStore();
const categoryExpenses = computed(() => transactionStore.categoryExpenses);

const chartData = ref({
  labels: [
    '식비',
    '교통비',
    '문화생활',
    '패션/미용',
    '마트/편의점',
    '고정비',
    '기타',
  ],
  datasets: [
    {
      label: 'Category Expenses',
      data: [],
      backgroundColor: [
        '#FF6384',
        '#36A2EB',
        '#FFCE56',
        '#4BC0C0',
        '#9966FF',
        '#FF9F40',
        '#ccc',
      ],
      borderColor: [
        '#FF6384',
        '#36A2EB',
        '#FFCE56',
        '#4BC0C0',
        '#9966FF',
        '#FF9F40',
        '#ccc',
      ],
      borderWidth: 1,
    },
  ],
});

const chartOptions = ref({
  responsive: true,
  plugins: {
    legend: {
      position: 'right',
      labels: {
        padding: 10,
      },
    },
    layout: {
      padding: {
        left: 20,
        right: 20,
        top: 20,
        bottom: 20,
      },
    },
  },
});

const items = ref([
  {
    icon: 'fa-solid fa-utensils',
    name: '식비',
    amount: 0,
    backgroundColor: '#FF6384',
    borderColor: '#FF6384',
  },
  {
    icon: 'fa-solid fa-bus',
    name: '교통비',
    amount: 0,
    backgroundColor: '#36A2EB',
    borderColor: '#36A2EB',
  },
  {
    icon: 'fa-solid fa-gamepad',
    name: '문화생활',
    amount: 0,
    backgroundColor: '#FFCE56',
    borderColor: '#FFCE56',
  },
  {
    icon: 'fa-solid fa-shirt',
    name: '패션/미용',
    amount: 0,
    backgroundColor: '#4BC0C0',
    borderColor: '#4BC0C0',
  },
  {
    icon: 'fa-solid fa-cart-shopping',
    name: '마트/편의점',
    amount: 0,
    backgroundColor: '#9966FF',
    borderColor: '#9966FF',
  },
  {
    icon: 'fa-solid fa-house',
    name: '고정비',
    amount: 0,
    backgroundColor: '#FF9F40',
    borderColor: '#FF9F40',
  },
  {
    icon: 'fa-solid fa-ellipsis',
    name: '기타',
    amount: 0,
    backgroundColor: '#ccc',
    borderColor: '#ccc',
  },
]);

const updateItems = () => {
  items.value.forEach((item) => {
    item.amount = categoryExpenses.value[item.name] || 0;
  });
  // 지출 순서대로 정렬
  items.value.sort((a, b) => b.amount - a.amount);
};

watch(
  categoryExpenses,
  (newExpenses) => {
    chartData.value.datasets[0].data = [
      newExpenses['식비'],
      newExpenses['교통비'],
      newExpenses['문화생활'],
      newExpenses['패션/미용'],
      newExpenses['마트/편의점'],
      newExpenses['고정비'],
      newExpenses['기타'],
    ];
    updateItems();
  },
  { immediate: true }
);

const formatCurrency = (value) => {
  return value.toLocaleString('ko-KR', { style: 'currency', currency: 'KRW' });
};
</script>

<style scoped>
.container {
  max-width: 375px;
  margin: 0 auto;
  padding: 0;
}

.chart-container {
  margin-bottom: 10px;
  text-align: center;
}

.item-list {
  list-style-type: none;
  padding: 0;
  margin: 0;
  height: 350px;
  overflow-y: auto;
}

.item-list::-webkit-scrollbar {
  display: none;
}

.item-list li {
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-width: 1px;
  border-style: solid;
  margin: 0 auto 10px;
  padding: 5px;
  max-width: 350px;
  gap: 10px;
}

.item-list li .item-icon {
  width: 40px;
  margin-right: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 20px;
}

.item-list li .item-details {
  flex: 1;
  display: flex;
  align-items: center;
}

.item-list li .item-details .item-name {
  margin: 0;
  font-weight: bold;
  font-size: 20px;
}

.item-list li .item-details .item-amount {
  margin-left: auto;
  margin-bottom: 0;
  margin-right: 10px;
  font-size: 20px;
  font-weight: bold;
  line-height: 46px;
}

@media (min-width: 375px) {
  .chart-container {
    max-width: 350px;
    margin: 0 auto;
  }
}
</style>
