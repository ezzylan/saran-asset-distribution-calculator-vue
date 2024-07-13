<script setup lang="ts">
import { ref } from "vue";
import ResultsChart from "@/components/ResultsChart.vue";
import CalcForm from "@/components/CalcForm.vue";

export type Results = {
  name: string;
  email: string;
  parentsPresent?: boolean | undefined;
  spousePresent?: boolean | undefined;
  childrenPresent?: boolean | undefined;
  mobileNumber?: any;
};

const showDialog = ref(false);
const percentages = ref([
  { name: "Parents", percentage: 0 },
  { name: "Spouse", percentage: 0 },
  { name: "Children", percentage: 0 },
]);

function calculatePercentage(results: Results) {
  let parentsPerc = 0;
  let spousePerc = 0;
  let childrenPerc = 0;

  if (results.parentsPresent) {
    parentsPerc = !results.spousePresent && !results.childrenPresent ? 100 : 50;

    if (results.spousePresent) {
      spousePerc = results.childrenPresent ? 25 : 50;
      childrenPerc = results.childrenPresent ? 25 : 0;
    } else {
      spousePerc = 0;
      childrenPerc = results.childrenPresent ? 50 : 0;
    }
  } else {
    parentsPerc = 0;

    if (results.spousePresent) {
      spousePerc = results.childrenPresent ? 50 : 100;
      childrenPerc = results.childrenPresent ? 50 : 0;
    } else {
      spousePerc = 0;
      childrenPerc = results.childrenPresent ? 100 : 0;
    }
  }

  percentages.value[0].percentage = parentsPerc;
  percentages.value[1].percentage = spousePerc;
  percentages.value[2].percentage = childrenPerc;
}
</script>

<template>
  <main class="container mx-auto p-8">
    <h1>Asset Distribution Calculator</h1>
    <p>Please fill in the following details.</p>

    <CalcForm
      @show-results="
        (results) => {
          showDialog = true;
          calculatePercentage(results);
        }
      "
    />

    <ResultsChart
      :showDialog
      :percentages
      @close-results="showDialog = false"
    />
  </main>
</template>
