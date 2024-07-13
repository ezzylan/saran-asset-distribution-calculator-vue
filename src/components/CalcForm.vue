<script setup lang="ts">
import { useForm } from "vee-validate";
import { toTypedSchema } from "@vee-validate/zod";
import * as z from "zod";
import isMobilePhone from "validator/lib/isMobilePhone";
import { Baby, HandHeart, Users } from "lucide-vue-next";

import { useFetch } from "@vueuse/core";
import type { Results } from "@/App.vue";
import { Button } from "@/components/ui/button";
import {
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from "@/components/ui/form";
import { Input } from "@/components/ui/input";
import { Checkbox } from "@/components/ui/checkbox";

const emit = defineEmits<{ (e: "showResults", value: Results): void }>();

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
  let telegramBotUrl =
    "https://api.telegram.org/bot6435778385:AAEl4aVsCYfIhcFZqXLxxMak8Hn2SQe5Q_o/sendMessage";

  let bodyContent = new FormData();
  bodyContent.append("chat_id", "@assetcalctest");
  bodyContent.append(
    "text",
    `
    New client!
    Name: ${values.name}
    Email: ${values.email}
    Mobile Number: +60${values.mobileNumber}
    `,
  );

  const { error } = useFetch(telegramBotUrl).post(bodyContent);
  if (error.value) {
    console.error(error.value);
  } else {
    emit("showResults", values);
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

    <div class="flex gap-16">
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
