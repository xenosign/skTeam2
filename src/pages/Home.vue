<template>
  <div class="container">
    <Header />

    <!-- 월 변경 바 -->
    <div class="calendar_header">
      <button @click="prevMonth" style="font-size: 20px">&lt;</button>
      <div class="calendar_month">{{ year }}년 {{ month }}월</div>
      <button @click="nextMonth" style="font-size: 20px">&gt;</button>
    </div>

    <SummaryStats
      :income="monthlyIncome"
      :expense="monthlyExpense"
      :filters="filters"
    />

    <div class="transactions">
      <div v-for="(transactions, date) in groupedTransactions" :key="date">
        <div class="transaction-date">
          <span>
            <button class="bold-date">
              {{ formatDateWithoutMonth(date) }} ({{ formatDayOfWeek(date) }})
            </button>
          </span>
        </div>
        <div
          v-for="transaction in transactions"
          :key="transaction.id"
          class="transaction"
        >
          <div class="transaction-details">
            <div class="method" style="flex-basis: 80px; flex-grow: 0">
              {{ transaction.category }}
            </div>
            <div
              class="description"
              style="flex-basis: 100px; padding-left: 10px"
            >
              {{ transaction.content }}
            </div>
            <div
              :class="{
                blue: transaction.type === 'income',
                red: transaction.type === 'expense',
              }"
            >
              {{ transaction.type === 'income' ? '+' : '-' }}
              {{ formatAmount(parseInt(transaction.amount)) }}원
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="footer">
      <button @click="showModal = true"><i class="fa fa-plus"></i></button>
    </div>

    <!-- 모달 창 -->
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <AddTransaction @close="modalHandler" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue';
import { useTransactionStore } from '@/stores/transaction';
import { useRouter, useRoute } from 'vue-router';
import AddTransaction from './AddTransaction.vue';
import Header from '@/components/Header.vue';
import SummaryStats from '@/components/SummaryStats.vue';

const router = useRouter();
const route = useRoute();
const transactionStore = useTransactionStore();
const currentDate = ref(new Date());
const showModal = ref(false);
const filters = ref({
  showIncome: true,
  showExpense: true,
  showBalance: true,
});

function modalHandler(data) {
  showModal.value = data;
}

const totalIncome = computed(() => transactionStore.totalIncome);
const totalExpense = computed(() => transactionStore.totalExpense);
const year = ref(new Date().getFullYear());
const month = ref(new Date().getMonth() + 1);

const transactions = computed(() => transactionStore.total);

function prevMonth() {
  month.value -= 1;
  if (month.value < 1) {
    month.value = 12;
    year.value -= 1;
  }
  updatePage();
}

function nextMonth() {
  month.value += 1;
  if (month.value > 12) {
    month.value = 1;
    year.value += 1;
  }
  updatePage();
}

const formatDateWithoutMonth = (date) => {
  const [year, month, day] = date.split('-');
  return `${parseInt(day)}일`;
};

const formatDayOfWeek = (date) => {
  const dayOfWeek = new Date(date).getDay();
  const days = [
    '일요일',
    '월요일',
    '화요일',
    '수요일',
    '목요일',
    '금요일',
    '토요일',
  ];
  return days[dayOfWeek];
};

const formatAmount = (amount) => {
  if (!amount) return '';
  return amount.toLocaleString();
};

const groupedTransactions = computed(() => {
  const filteredTransactions = transactions.value.filter((transaction) => {
    const transactionDate = new Date(transaction.date);
    return (
      transactionDate.getFullYear() === year.value &&
      transactionDate.getMonth() + 1 === month.value
    );
  });

  return filteredTransactions.reduce((groups, transaction) => {
    const date = transaction.date;
    if (!groups[date]) {
      groups[date] = [];
    }
    groups[date].push(transaction);
    return groups;
  }, {});
});

const monthlyIncome = computed(() => {
  return transactions.value
    .filter(
      (transaction) =>
        transaction.type === 'income' &&
        new Date(transaction.date).getFullYear() === year.value &&
        new Date(transaction.date).getMonth() + 1 === month.value
    )
    .reduce((sum, transaction) => sum + parseInt(transaction.amount), 0);
});

const monthlyExpense = computed(() => {
  return transactions.value
    .filter(
      (transaction) =>
        transaction.type === 'expense' &&
        new Date(transaction.date).getFullYear() === year.value &&
        new Date(transaction.date).getMonth() + 1 === month.value
    )
    .reduce((sum, transaction) => sum + parseInt(transaction.amount), 0);
});

function updatePage() {
  console.log('페이지 업데이트!');
}

watch([year, month], () => {
  updatePage();
});

watch(
  () => route.path,
  (newPath, oldPath) => {
    updatePage();
  }
);

onMounted(() => {
  updatePage();
});
</script>

<style scoped src="@/assets/Home.css"></style>
<style scoped>
* {
  font-size: 20px;
}

.calendar_header {
  display: flex;
  height: 80px;
  font-size: 20px;
  justify-content: space-between;
  align-items: center;
  padding: 10px 20px 0 20px;
  font-size: 20px;
}

.calendar_month {
  font-weight: bold;
}
</style>
