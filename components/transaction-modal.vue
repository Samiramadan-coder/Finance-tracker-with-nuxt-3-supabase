<template>
  <UModal v-model="isOpen">
    <UCard>
      <template #header> Add Transaction </template>

      <UForm :state="state" :schema="schema" ref="form" @submit.prevent="save">
        <UFormGroup
          :required="true"
          label="Transaction Type"
          name="type"
          class="mb-4"
        >
          <USelect
            placeholder="Select the transaction type"
            :options="types"
            v-model="state.type"
          />
        </UFormGroup>

        <UFormGroup label="Amount" :required="true" name="amount" class="mb-4">
          <UInput
            type="number"
            placeholder="Amount"
            v-model.number="state.amount"
          />
        </UFormGroup>

        <UFormGroup
          label="Transaction date"
          :required="true"
          name="created_at"
          class="mb-4"
        >
          <UInput
            type="date"
            icon="i-heroicons-calendar-days-20-solid"
            v-model="state.created_at"
          />
        </UFormGroup>

        <UFormGroup
          label="Descriptions"
          hint="Optional"
          name="description"
          class="mb-4"
        >
          <UInput placeholder="Descriptions" v-model="state.description" />
        </UFormGroup>

        <UFormGroup
          :required="true"
          label="Category"
          name="category"
          class="mb-4"
          v-if="state.type === 'Expense'"
        >
          <USelect
            placeholder="Category"
            :options="categories"
            v-model="state.category"
          />
        </UFormGroup>

        <UButton
          type="submit"
          color="black"
          variant="solid"
          label="Save"
          :loading="isLoading"
        />
      </UForm>
    </UCard>
  </UModal>
</template>

<script setup lang="ts">
import { z } from "zod";
import { categories, types } from "~/constants";

const defaultSchema = z.object({
  amount: z.number().positive("Amounts needs to be more than 0"),
  created_at: z.string(),
  description: z.string(),
});

const incomeSchema = z.object({
  type: z.literal("Income"),
});

const expenseSchema = z.object({
  type: z.literal("Expense"),
  category: z.enum(categories),
});

const investmentSchema = z.object({
  type: z.literal("Investment"),
});

const savingSchema = z.object({
  type: z.literal("Saving"),
});

const schema = z.intersection(
  z.discriminatedUnion("type", [
    incomeSchema,
    expenseSchema,
    investmentSchema,
    savingSchema,
  ]),
  defaultSchema
);

const emit = defineEmits(["update:modelValue", "saved"]);

const props = defineProps({
  modelValue: Boolean,
});

const isOpen = computed({
  get: (): boolean => props.modelValue,
  set: (value: boolean): void => {
    if (!value) resetForm();
    emit("update:modelValue", value);
  },
});

const form = ref();
const isLoading = ref(false);
const supabase = useSupabaseClient();
const toast = useToast();

const initialState = {
  type: undefined,
  amount: 0,
  created_at: undefined,
  description: undefined,
  category: undefined,
};

const state = ref({
  ...initialState,
});

const resetForm = (): void => {
  Object.assign(state.value, initialState);
  // form.value.clear;
};

const save = async () => {
  // console.log(form.value);
  // if (form.value.error.length) return;

  // const error = await form.value.validate();
  // console.log(error);

  // return;

  isLoading.value = true;
  try {
    const { error } = await supabase
      .from("transactions")
      .upsert({ ...state.value });

    if (!error) {
      toast.add({
        title: "Transaction Saved",
        icon: "i-heroicons-check-circle",
      });

      isOpen.value = false;

      emit("saved");

      return;
    }

    throw error;
  } catch (e) {
    toast.add({
      title: "Transaction not Saved",
      description: e.message,
      icon: "i-heroicons-check-circle",
      color: "red",
    });
  } finally {
    isLoading.value = false;
  }
};
</script>
