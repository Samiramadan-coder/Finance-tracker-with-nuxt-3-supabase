<template>
  <div
    class="grid grid-cols-2 py-4 border-b border-gray-200 dark:border-gray-800 text-gray-500 dark:text-gray-400 font-bold"
  >
    <div class="flex items-center justify-between">
      {{ date }}
    </div>

    <div class="flex items-center justify-end space-x-2 mr-10">
      {{ currency }}
    </div>
  </div>
</template>

<script setup lang="ts">
import type { Transaction } from "~/interfaces";

const props = defineProps({
  date: {
    type: String,
    required: true,
  },
  transactions: {
    type: Array as PropType<Transaction[]>,
    required: true,
  },
});

const sum = computed((): number => {
  let sum = 0;

  for (const transaction of props.transactions) {
    if (transaction.type === "Income") {
      sum += transaction.amount;
    } else {
      sum -= transaction.amount;
    }
  }

  return sum;
});

const { currency } = useCurrency(sum.value);
</script>
