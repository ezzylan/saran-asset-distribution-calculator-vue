<script setup lang="ts">
import CalcForm from "@/components/CalcForm.vue";
import ResultsChart from "@/components/ResultsChart.vue";
import { Toaster } from "@/components/ui/sonner";
import { ref } from "vue";

const showDialog = ref(false);
const percentages = ref([
  { name: "Parents", percentage: 0 },
  { name: "Spouse", percentage: 0 },
  { name: "Children", percentage: 0 },
]);
</script>

<template>
  <main class="container mx-auto p-8">
    <h2
      class="scroll-m-20 pb-2 text-3xl font-semibold tracking-tight transition-colors first:mt-0"
    >
      Asset Distribution Calculator
    </h2>
    <p>Please fill in the following details.</p>

    <CalcForm
      @show-results="
        (results) => {
          showDialog = true;
          percentages[0].percentage = results.parentsPerc;
          percentages[1].percentage = results.spousePerc;
          percentages[2].percentage = results.childrenPerc;
        }
      "
    />

    <ResultsChart
      :showDialog
      :percentages
      @close-results="showDialog = false"
    />
  </main>

  <Toaster richColors />
</template>
