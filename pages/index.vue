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
      :amount="4000"
      :last-amount="3000"
      :loading="isLoading"
    />
    <Trend
      color="red"
      title="Expense"
      :amount="4000"
      :last-amount="5000"
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

const fetchTransactions = async () => {
  isLoading.value = true;

  try {
    const { data, pending } = await useAsyncData<Transaction[]>(
      "transactions",
      async () => {
        const { data, error } = await supabase.from("transactions").select();

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

  return grouped;
});
</script>
