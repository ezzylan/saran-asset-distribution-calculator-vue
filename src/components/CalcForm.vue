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

function calculatePercentage(results: {
  name: string;
  email: string;
  parentsPresent?: boolean | undefined;
  spousePresent?: boolean | undefined;
  childrenPresent?: boolean | undefined;
  mobileNumber?: any;
}) {
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

  return { parentsPerc, spousePerc, childrenPerc };
}

const formSchema = toTypedSchema(
  z.object({
    name: z.string().min(3, {
      message: "Name must be at least 3 characters.",
    }),
    email: z.string().email({ message: "Invalid email address" }),
    parentsPresent: z.boolean().default(false).optional(),
    spousePresent: z.boolean().default(false).optional(),
    childrenPresent: z.boolean().default(false).optional(),
    mobileNumber: z.string().min(9).max(10).refine(isMobilePhone),
  }),
);

const { handleSubmit } = useForm({
  validationSchema: formSchema,
  initialValues: {
    parentsPresent: false,
    spousePresent: false,
    childrenPresent: false,
  },
});

const onSubmit = handleSubmit((values) => {
  const { parentsPerc, spousePerc, childrenPerc } = calculatePercentage(values);

  let telegramBotUrl = `https://api.telegram.org/bot${import.meta.env.VITE_TELEGRAM_BOT_TOKEN}/sendMessage`;

  let bodyContent = new FormData();
  bodyContent.append("chat_id", "@assetcalctest");
  bodyContent.append(
    "text",
    `
    New client!
    Name: ${values.name}
    Email: ${values.email}
    Mobile Number: +60${values.mobileNumber}
    Parents: ${parentsPerc}%
    Spouse: ${spousePerc}%
    Children: ${childrenPerc}%
    `,
  );

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

    <div class="flex flex-col gap-2 md:flex-row md:gap-16">
      <FormField
        v-slot="{ value, handleChange }"
        type="checkbox"
        name="parentsPresent"
      >
        <FormItem class="flex flex-row items-center space-x-3 space-y-0">
          <FormControl>
            <Checkbox :checked="value" @update:checked="handleChange" />
          </FormControl>
          <FormLabel>Parents Present</FormLabel>
          <Users />
          <FormMessage />
        </FormItem>
      </FormField>

      <FormField v-slot="{ value, handleChange }" name="spousePresent">
        <FormItem class="flex flex-row items-center space-x-3 space-y-0">
          <FormControl>
            <Checkbox :checked="value" @update:checked="handleChange" />
          </FormControl>
          <FormLabel>Spouse Present</FormLabel>
          <HandHeart />
          <FormMessage />
        </FormItem>
      </FormField>

      <FormField v-slot="{ value, handleChange }" name="childrenPresent">
        <FormItem class="flex flex-row items-center space-x-3 space-y-0">
          <FormControl>
            <Checkbox :checked="value" @update:checked="handleChange" />
          </FormControl>
          <FormLabel>Children Present</FormLabel>
          <Baby />
          <FormMessage />
        </FormItem>
      </FormField>
    </div>

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
