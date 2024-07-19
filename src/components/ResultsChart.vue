<script setup lang="ts">
import { Button } from "@/components/ui/button";
import { DonutChart } from "@/components/ui/chart-donut";
import {
  Dialog,
  DialogClose,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog";
import {
  Drawer,
  DrawerClose,
  DrawerContent,
  DrawerDescription,
  DrawerFooter,
  DrawerHeader,
  DrawerTitle,
  DrawerTrigger,
} from "@/components/ui/drawer";
import { useMediaQuery } from "@vueuse/core";
import { Baby, HandHeart, Users } from "lucide-vue-next";

defineProps<{
  showDialog: boolean;
  percentages: { name: string; percentage: number }[];
}>();

const isDesktop = useMediaQuery("(min-width: 768px)");
</script>

<template>
  <Dialog v-if="isDesktop" :open="showDialog">
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

  <Drawer v-else :open="showDialog">
    <DrawerContent>
      <DrawerHeader>
        <DrawerTitle>Here is your result!</DrawerTitle>
        <DrawerDescription class="flex justify-between">
          <span v-for="{ name, percentage } in percentages">
            <div class="flex items-center gap-2">
              <Users v-if="name === 'Parents'" class="h-4 w-4" />
              <HandHeart v-if="name === 'Spouse'" class="h-4 w-4" />
              <Baby v-if="name === 'Children'" class="h-4 w-4" />
              {{ name }}: {{ percentage }}%
            </div>
          </span>
        </DrawerDescription>
      </DrawerHeader>

      <DonutChart
        index="name"
        :category="'percentage'"
        :data="percentages"
        :type="'pie'"
      />

      <DrawerFooter class="sm:justify-start">
        <DrawerClose as-child>
          <Button
            @click="$emit('closeResults')"
            type="button"
            variant="secondary"
          >
            Close
          </Button>
        </DrawerClose>
      </DrawerFooter>
    </DrawerContent>
  </Drawer>
</template>
