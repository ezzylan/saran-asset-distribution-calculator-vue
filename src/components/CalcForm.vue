<script setup lang="ts">
import { Baby, HandHeart, Users } from "lucide-vue-next";
import isMobilePhone from "validator/lib/isMobilePhone";
import { useForm } from "vee-validate";
import { toast } from "vue-sonner";
import * as z from "zod";

import { Button } from "@/components/ui/button";
import { Checkbox } from "@/components/ui/checkbox";
import {
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from "@/components/ui/form";
import { Input } from "@/components/ui/input";
import { toTypedSchema } from "@vee-validate/zod";
import { useFetch } from "@vueuse/core";

const emit = defineEmits<{
  (
    e: "showResults",
    value: {
      parentsPerc: number;
      spousePerc: number;
      childrenPerc: number;
    },
  ): void;
}>();

function calculatePercentage(results: { items: string[] }) {
  let parentsPerc = 0;
  let spousePerc = 0;
  let childrenPerc = 0;

  if (results.items.includes("parentsPresent")) {
    parentsPerc =
      !results.items.includes("spousePresent") &&
      !results.items.includes("childrenPresent")
        ? 100
        : 50;

    if (results.items.includes("spousePresent")) {
      spousePerc = results.items.includes("childrenPresent") ? 25 : 50;
      childrenPerc = results.items.includes("childrenPresent") ? 25 : 0;
    } else {
      spousePerc = 0;
      childrenPerc = results.items.includes("childrenPresent") ? 50 : 0;
    }
  } else {
    parentsPerc = 0;

    if (results.items.includes("spousePresent")) {
      spousePerc = results.items.includes("childrenPresent") ? 50 : 100;
      childrenPerc = results.items.includes("childrenPresent") ? 50 : 0;
    } else {
      spousePerc = 0;
      childrenPerc = results.items.includes("childrenPresent") ? 100 : 0;
    }
  }

  return { parentsPerc, spousePerc, childrenPerc };
}

const items = [
  { id: "parentsPresent", label: "Parents Present" },
  { id: "spousePresent", label: "Spouse Present" },
  { id: "childrenPresent", label: "Children Present" },
] as const;

const formSchema = toTypedSchema(
  z.object({
    name: z.string().min(3, {
      message: "Name must be at least 3 characters.",
    }),
    email: z.string().email({ message: "Invalid email address" }),
    items: z.array(z.string()).refine((value) => value.some((item) => item), {
      message: "You have to select at least one item.",
    }),
    childrenNumber: z.coerce.number().int().gte(1).lte(10).optional(),
    mobileNumber: z.string().min(9).max(10).refine(isMobilePhone),
  }),
);

const { handleSubmit } = useForm({
  validationSchema: formSchema,
  initialValues: {
    items: ["parentsPresent"],
  },
});

const onSubmit = handleSubmit((values) => {
  const { parentsPerc, spousePerc, childrenPerc } = calculatePercentage(values);

  let bodyText = `
    New client!
    Name: ${values.name}
    Email: ${values.email}
    Mobile Number: +60${values.mobileNumber}
    Parents: ${parentsPerc}%
    Spouse: ${spousePerc}%
    Children: ${childrenPerc}%
    `;

  if (values.items.includes("childrenPresent")) {
    bodyText += `Number of Children: ${values.childrenNumber}`;
  }

  let telegramBotUrl = `https://api.telegram.org/bot${import.meta.env.VITE_TELEGRAM_BOT_TOKEN}/sendMessage`;

  let bodyContent = new FormData();
  bodyContent.append("chat_id", "@assetdistcalc");
  bodyContent.append("text", bodyText);

  const { error } = useFetch(telegramBotUrl).post(bodyContent);
  if (error.value) {
    toast.error(error.value);
  } else {
    emit("showResults", { parentsPerc, spousePerc, childrenPerc });
  }
});
</script>

<template>
  <form class="space-y-6 pt-6" @submit="onSubmit">
    <FormField v-slot="{ componentField }" name="name">
      <FormItem>
        <FormLabel>Name</FormLabel>
        <FormControl>
          <Input required v-bind="componentField" />
        </FormControl>
        <FormMessage />
      </FormItem>
    </FormField>

    <FormField v-slot="{ componentField }" name="email">
      <FormItem>
        <FormLabel>Email</FormLabel>
        <FormControl>
          <Input required v-bind="componentField" />
        </FormControl>
        <FormMessage />
      </FormItem>
    </FormField>

    <FormField name="items">
      <FormItem>
        <div class="flex flex-col gap-2 md:flex-row md:gap-16">
          <FormField
            v-for="item in items"
            v-slot="{ value, handleChange }"
            :key="item.id"
            type="checkbox"
            :value="item.id"
            :unchecked-value="false"
            name="items"
          >
            <div class="space-y-3">
              <FormItem class="flex flex-row items-start space-x-3 space-y-0">
                <FormControl>
                  <Checkbox
                    :checked="value.includes(item.id)"
                    @update:checked="handleChange"
                  />
                </FormControl>
                <FormLabel class="font-normal">
                  {{ item.label }}
                </FormLabel>
                <Users v-if="item.id === 'parentsPresent'" class="h-4 w-4" />
                <HandHeart v-if="item.id === 'spousePresent'" class="h-4 w-4" />
                <Baby v-if="item.id === 'childrenPresent'" class="h-4 w-4" />
              </FormItem>
              <FormField
                v-if="
                  item.id === 'childrenPresent' &&
                  value.includes('childrenPresent')
                "
                v-slot="{ componentField }"
                name="childrenNumber"
              >
                <FormItem>
                  <FormControl>
                    <Input
                      type="number"
                      min="1"
                      max="10"
                      placeholder="How many?"
                      v-bind="componentField"
                      class="w-32"
                    />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              </FormField>
            </div>
          </FormField>
        </div>
        <FormMessage />
      </FormItem>
    </FormField>

    <FormField v-slot="{ componentField }" name="mobileNumber">
      <FormItem>
        <FormLabel>Mobile Number</FormLabel>
        <div class="flex">
          <Button class="bg-foreground" disabled>+60</Button>
          <FormControl>
            <Input type="tel" required v-bind="componentField" />
          </FormControl>
        </div>
        <FormDescription>
          Provide your WhatsApp number to receive free illustrations of your
          Asset Distribution under the 1958 Malaysian Distribution Act.
        </FormDescription>
        <FormMessage />
      </FormItem>
    </FormField>

    <Button type="submit">Submit</Button>
  </form>
</template>
