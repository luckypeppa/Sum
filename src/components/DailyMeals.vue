<template>
  <div>
    <MealsTab
      :mealsByPeriodList="mealStore.transFormedTodayMeals.mealsByPeriod"
      @click:item="openModal"
      @toggle="handleToggle"
    >
      <template v-slot:title>
        <h1 class="text-lg text-center font-bold">餐盘</h1>
        <h3 class="text-md text-center">点击卡片修改</h3>
      </template>
    </MealsTab>

    <!-- Progress -->
    <ProgressStats
      :labels="labels"
      :quantities="amounts"
      :goal="TDEEStore.TDEE"
    />
    <!-- Modal -->
    <AppModal
      :isOpen="isOpen"
      @cancel="handleCancel"
      :isLoading="isLoading"
      :error="error"
      title="修改"
    >
      <AmountForm v-model:amount="amount" :materialId="meal?.materialId ?? 1" />
      <template v-slot:footer>
        <BaseButton @click="handleCancel">取消</BaseButton>
        <BaseButton @click="deleteMeal">删除</BaseButton>
        <BaseButton @click="handleOk" :isLoading="isLoading">确定</BaseButton>
      </template>
    </AppModal>
  </div>
</template>

<script setup lang="ts">
import MealsTab from "./MealsTab.vue";
import { ref, computed } from "vue";
import { useMealStore } from "../stores/meal";
import AmountForm from "../elements/AmountForm.vue";
import { useNoti } from "../plugins/useNoti";
import { useTDEEStore } from "../stores/TDEE";
import BaseButton from "../elements/BaseButton.vue";
import type { Meal } from "@/db/mealTypes";
import { cates } from "@/db/mealTypes";
import mealService from "@/services/mealService";
import ProgressStats from "./ProgressStats.vue";

const notier = useNoti();
const mealStore = useMealStore();
const TDEEStore = useTDEEStore();

const TDEESetted = ref<boolean>(false);

if (localStorage.getItem("TDEE")) {
  TDEESetted.value = true;
}

/**
 * Today's statistics
 */

const labels = computed(() => {
  const labels: string[] = [];
  mealStore.transFormedTodayMeals.mealsByPeriod.forEach((mealsByPeriod) => {
    labels.push(cates[mealsByPeriod.category]);
  });
  return labels;
});

const amounts = computed(() => {
  const amounts: number[] = [];
  mealStore.transFormedTodayMeals.mealsByPeriod.forEach((mealsByPeriod) => {
    amounts.push(mealsByPeriod.caloriesByPeriod);
  });
  return amounts;
});

/**
 * Modal
 */
const isOpen = ref(false);
const amount = ref(0);
const isLoading = ref(false);
const error = ref("");
const meal = ref<Meal | null>(null);

function handleCancel() {
  isOpen.value = false;
}

function openModal(mealId: number) {
  const res = mealStore.getById(mealId);
  if (res) {
    meal.value = res;
    amount.value = res.amount;
  }
  isOpen.value = true;
}

function handleToggle(index: number) {
  mealStore.setCurrentCategory(index);
}

function deleteMeal() {
  if (meal.value && meal.value.id) {
    mealService
      .deleteMeal(meal.value.id)
      .then(() => {
        isOpen.value = false;
      })
      .finally(() => {
        isLoading.value = false;
      });
  }
}

function handleOk() {
  if (amount.value < 1) {
    notier.error("重量最少为1克");
  }
  isLoading.value = true;
  if (meal.value && meal.value.id) {
    mealService
      .update(meal.value.id, {
        amount: amount.value,
      })
      .then(() => {
        isOpen.value = false;
      })
      .finally(() => {
        isLoading.value = false;
      });
  }
}
</script>

<style lang="scss" scoped></style>
