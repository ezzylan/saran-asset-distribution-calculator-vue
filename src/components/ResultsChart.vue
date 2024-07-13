<script setup lang="ts">
import {
  Dialog,
  DialogContent,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogDescription,
  DialogClose,
} from "@/components/ui/dialog";
import { Button } from "@/components/ui/button";
import { DonutChart } from "@/components/ui/chart-donut";

import { Baby, HandHeart, Users } from "lucide-vue-next";

type Percentages = { name: string; percentage: number }[];

defineProps<{ showDialog: boolean; percentages: Percentages }>();
</script>

<template>
  <Dialog :open="showDialog">
    <DialogContent>
      <DialogHeader>
        <DialogTitle>Here is your result!</DialogTitle>
        <DialogDescription class="flex justify-between">
          <span v-for="{ name, percentage } in percentages">
            <div class="flex items-center gap-2">
              <Users v-if="name === 'Parents'" class="h-4 w-4" />
              <HandHeart v-if="name === 'Spouse'" class="h-4 w-4" />
              <Baby v-if="name === 'Children'" class="h-4 w-4" />
              {{ name }}: {{ percentage }}%
            </div>
          </span>
        </DialogDescription>
      </DialogHeader>

      <DonutChart
        index="name"
        :category="'percentage'"
        :data="percentages"
        :type="'pie'"
      />

      <DialogFooter class="sm:justify-start">
        <DialogClose as-child>
          <Button
            @click="$emit('closeResults')"
            type="button"
            variant="secondary"
          >
            Close
          </Button>
        </DialogClose>
      </DialogFooter>
    </DialogContent>
  </Dialog>
</template>
