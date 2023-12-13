<template>
  <section class="flex items-center justify-between mb-10">
    <h1 class="text-4xl font-extrabold">Summary</h1>
    <div>
      <USelectMenu :options="transactionViewOptions" v-model="viewSelected" />
    </div>
  </section>

  <section
    class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 sm:gap-16 mb-10"
  >
    <Trend
      color="green"
      title="Income"
      :amount="incomeTotal"
      :last-amount="3000"
      :loading="isLoading"
    />
    <Trend
      color="red"
      title="Expense"
      :amount="expenseTotal"
      :last-amount="2000"
      :loading="isLoading"
    />
    <Trend
      color="green"
      title="Investments"
      :amount="4000"
      :last-amount="3000"
      :loading="isLoading"
    />
    <Trend
      color="red"
      title="Saving"
      :amount="4000"
      :last-amount="4100"
      :loading="isLoading"
    />
  </section>

  <section class="flex justify-between mb-10">
    <div>
      <h2 class="text-2xl font-extrabold">Transactions</h2>
      <div class="text-gray-500 dark:text-gray-400">
        You have {{ incomeCount }} incomes and {{ expenseCount }} expenses this
        period
      </div>
    </div>
    <div>
      <TransactionModal v-model="isOpen" @saved="refreshTransactions" />
      <UButton
        icon="i-heroicons-plus-circle"
        color="white"
        variant="solid"
        label="Add"
        @click="isOpen = true"
      />
    </div>
  </section>

  <section v-if="!isLoading">
    <div
      v-for="(transactionOnDay, date) in transactionsGroupedByDate"
      :key="date"
      class="mb-10"
    >
      <DailyTransactionSummary
        :date="typeof date === 'string' ? date : ''"
        :transactions="transactionOnDay"
      />
      <Transaction
        v-for="transaction in transactionOnDay"
        :key="transaction.id"
        :transaction="transaction"
        @deleted="refreshTransactions()"
      />
    </div>
  </section>
  <section v-else>
    <USkeleton class="h-8 w-full mb-2" v-for="n in 4" :key="n" />
  </section>
</template>

<script setup lang="ts">
import { transactionViewOptions } from "~/constants";
import type { Transaction } from "~/interfaces";

const supabase = useSupabaseClient();
const transactions = ref<Transaction[]>([]);
const viewSelected = ref<string>(transactionViewOptions[1]);
const isLoading = ref<boolean>(false);
const isOpen = ref<boolean>(false);

const income = computed((): Transaction[] =>
  transactions.value.filter((t) => t.type === "Income")
);
const expense = computed((): Transaction[] =>
  transactions.value.filter((t) => t.type === "Expense")
);

const incomeCount = computed((): number => income.value.length);
const expenseCount = computed((): number => expense.value.length);

const incomeTotal = computed((): number =>
  income.value.reduce((sum, transaction) => {
    return sum + transaction.amount;
  }, 0)
);

const expenseTotal = computed((): number =>
  expense.value.reduce((sum, transaction) => {
    return sum + transaction.amount;
  }, 0)
);

const fetchTransactions = async (): Promise<Transaction[]> => {
  isLoading.value = true;

  try {
    const { data, pending } = await useAsyncData<Transaction[]>(
      "transactions",
      async () => {
        const { data, error } = await supabase
          .from("transactions")
          .select()
          .order("created_at", { ascending: false });

        if (error) return [];
        return data;
      }
    );

    return data.value || [];
  } finally {
    isLoading.value = false;
  }
};

const refreshTransactions = async () =>
  (transactions.value = await fetchTransactions());
transactions.value = await fetchTransactions();

const transactionsGroupedByDate = computed(() => {
  type Grouped = {
    [key: string]: Transaction[];
  };

  let grouped: Grouped = {};

  for (const transaction of transactions.value) {
    const date = new Date(transaction.created_at).toISOString().split("T")[0];

    if (!grouped[date]) grouped[date] = [];

    grouped[date].push(transaction);
  }

  // const sortedKeys = Object.keys(grouped).sort().reverse();

  // const sortedGrouped: Grouped = {};

  // sortedKeys.forEach((key) => (sortedGrouped[key] = grouped[key]));

  return grouped;
});
</script>
