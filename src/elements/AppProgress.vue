<template>
  <div class="px-2">
    <div class="relative h-2.5 rounded-xl">
      <span class="absolute inset-0 rounded-xl bg-slate-200"></span>
      <span
        class="absolute inset-0 rounded-xl bg-green-400"
        ref="progress"
      ></span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, watch } from "vue";
import { gsap } from "gsap";

const props = defineProps<{
  progress: number;
}>();

const progress = ref(null);
let progressTo: gsap.QuickToFunc;
onMounted(() => {
  if (progress.value) {
    gsap.set(progress.value, {
      transformOrigin: "right center",
    });
    progressTo = gsap.quickTo(progress.value, "scaleX", {
      duration: 1,
    });
    progressTo(props.progress);
  }
});

watch(
  () => props.progress,
  (newValue) => {
    if (progressTo) {
      progressTo(newValue);
    }
  }
);
</script>

<style lang="scss" scoped></style>
