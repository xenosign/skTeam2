<template>
  <div class="container">
    <Header />

    <!-- 월 변경 바 -->
    <div class="calendar_header">
      <button @click="prevMonth">&lt;</button>
      <div class="calendar_month">{{ year }}년 {{ month }}월</div>
      <button @click="nextMonth">&gt;</button>
    </div>

    <SummaryStats
      :income="totalIncome"
      :expense="totalExpense"
      :filters="filters"
    />

    <div class="transactions">
      <div v-for="(transactions, date) in groupedTransactions" :key="date">
        <div class="transaction-date">
          <span
            ><button class="bold-date">
              {{ formatDateWithoutMonth(date) }} ({{ formatDayOfWeek(date) }})
            </button></span
          >
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
            <div class="description" style="flex-basis: 100px">
              {{ transaction.content }}
            </div>
            <div
              :class="{
                blue: transaction.type === 'income',
                red: transaction.type === 'expense',
              }"
            >
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
import { useRouter } from 'vue-router';
import AddTransaction from './AddTransaction.vue';
import Header from '@/components/Header.vue'; // Header 컴포넌트 가져오기
import SummaryStats from '@/components/SummaryStats.vue'; // SummaryStats 컴포넌트 가져오기

const router = useRouter();
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
const income = computed(() => transactionStore.income);
const expense = computed(() => transactionStore.expense);
const year = ref(new Date().getFullYear());
const month = ref(new Date().getMonth() + 1);

const transactions = computed(() => transactionStore.total);

const currentYearMonth = computed(() =>
  currentDate.value.toLocaleString('default', {
    month: 'long',
    year: 'numeric',
  })
);

function prevMonth() {
  month.value -= 1;
  if (month.value < 1) {
    month.value = 12;
    year.value -= 1;
  }
}

function nextMonth() {
  month.value += 1;
  if (month.value > 12) {
    month.value = 1;
    year.value += 1;
  }
}

const routerPush = (path) => {
  router.push(path);
};

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

let groupedTransactions = [];

function updatePage() {
  groupedTransactions = computed(() => {
    return transactions.value.reduce((groups, transaction) => {
      const date = transaction.date;
      if (!groups[date]) {
        groups[date] = [];
      }
      groups[date].push(transaction);
      return groups;
    }, {});
  });
}

watch([year, month], () => {
  updatePage();
});

onMounted(() => {
  updatePage();
});
</script>

<style src="@/assets/Home.css"></style>
<style scoped>
.calendar_header {
  display: flex;
  height: 80px;
  font-size: 17px;
  justify-content: space-between;
  align-items: center;
  padding: 10px 20px 0 20px;
}

.calendar_month {
  font-weight: bold;
}
</style>
