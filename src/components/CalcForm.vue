<script setup lang="ts">
import { startCase } from "lodash-es";
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
import { RadioGroup, RadioGroupItem } from "@/components/ui/radio-group";
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

function calculatePercentage(items: string[]) {
  let parentsPerc = 0;
  let spousePerc = 0;
  let childrenPerc = 0;

  if (items.includes("parentsPresent")) {
    parentsPerc =
      !items.includes("spousePresent") && !items.includes("childrenPresent")
        ? 100
        : 50;

    if (items.includes("spousePresent")) {
      spousePerc = items.includes("childrenPresent") ? 25 : 50;
      childrenPerc = items.includes("childrenPresent") ? 25 : 0;
    } else {
      spousePerc = 0;
      childrenPerc = items.includes("childrenPresent") ? 50 : 0;
    }
  } else {
    parentsPerc = 0;

    if (items.includes("spousePresent")) {
      spousePerc = items.includes("childrenPresent") ? 50 : 100;
      childrenPerc = items.includes("childrenPresent") ? 50 : 0;
    } else {
      spousePerc = 0;
      childrenPerc = items.includes("childrenPresent") ? 100 : 0;
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
    referral: z.enum(["TikTok", "Instagram", "Facebook", "Person"], {
      required_error: "You need to select a referral.",
    }),
    referralPerson: z
      .string()
      .min(3, {
        message: "Name must be at least 3 characters.",
      })
      .optional(),
  }),
);

const { values, handleSubmit } = useForm({
  validationSchema: formSchema,
  initialValues: {
    items: ["parentsPresent"],
    referral: "TikTok",
  },
});

const onSubmit = handleSubmit((values) => {
  const { parentsPerc, spousePerc, childrenPerc } = calculatePercentage(
    values.items,
  );

  let bodyText = `
    New client!
    Name: ${startCase(values.name)}
    Email: ${values.email}
    Mobile Number: +60${values.mobileNumber}
    Parents: ${parentsPerc}%
    Spouse: ${spousePerc}%
    Children: ${childrenPerc}%
    `;

  if (values.items.includes("childrenPresent")) {
    bodyText += `Number of Children: ${values.childrenNumber}\n\t`;
  }

  bodyText += `Referral: ${values.referral === "Person" ? startCase(values.referralPerson) : values.referral}`;

  let bodyContent = new FormData();
  bodyContent.append("chat_id", "@tsassetdistcalc");
  bodyContent.append("text", bodyText);

  let telegramBotUrl = `https://api.telegram.org/bot${import.meta.env.VITE_TELEGRAM_BOT_TOKEN}/sendMessage`;

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
    <div class="flex flex-col gap-6 sm:flex-row">
      <FormField v-slot="{ componentField }" name="name">
        <FormItem class="grow">
          <FormLabel>Name</FormLabel>
          <FormControl>
            <Input required v-bind="componentField" />
          </FormControl>
          <FormMessage />
        </FormItem>
      </FormField>
      <FormField v-slot="{ componentField }" name="email">
        <FormItem class="grow">
          <FormLabel>Email</FormLabel>
          <FormControl>
            <Input required v-bind="componentField" />
          </FormControl>
          <FormMessage />
        </FormItem>
      </FormField>

      <FormField name="items">
        <FormItem class="basis-1/4">
          <div class="flex flex-col gap-3">
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
                  <HandHeart
                    v-if="item.id === 'spousePresent'"
                    class="h-4 w-4"
                  />
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
    </div>

    <div class="flex flex-col gap-6 sm:flex-row">
      <FormField v-slot="{ componentField }" name="mobileNumber">
        <FormItem class="basis-3/4">
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

      <div class="space-y-3">
        <FormField v-slot="{ componentField }" type="radio" name="referral">
          <FormItem class="space-y-3">
            <FormLabel>Referral</FormLabel>
            <FormControl>
              <RadioGroup
                class="flex flex-col space-y-1"
                v-bind="componentField"
              >
                <FormItem class="flex items-center gap-x-3 space-y-0">
                  <FormControl>
                    <RadioGroupItem value="TikTok" />
                  </FormControl>
                  <FormLabel class="font-normal"> TikTok </FormLabel>
                </FormItem>
                <FormItem class="flex items-center gap-x-3 space-y-0">
                  <FormControl>
                    <RadioGroupItem value="Instagram" />
                  </FormControl>
                  <FormLabel class="font-normal"> Instagram </FormLabel>
                </FormItem>
                <FormItem class="flex items-center gap-x-3 space-y-0">
                  <FormControl>
                    <RadioGroupItem value="Facebook" />
                  </FormControl>
                  <FormLabel class="font-normal"> Facebook </FormLabel>
                </FormItem>
                <FormItem class="flex items-center gap-x-3 space-y-0">
                  <FormControl>
                    <RadioGroupItem value="Person" />
                  </FormControl>
                  <FormLabel class="font-normal"> Person </FormLabel>
                </FormItem>
              </RadioGroup>
            </FormControl>
            <FormMessage />
          </FormItem>
        </FormField>

        <FormField
          v-if="values.referral === 'Person'"
          v-slot="{ componentField }"
          name="referralPerson"
        >
          <FormItem>
            <FormControl>
              <Input
                type="text"
                placeholder="Referral Name"
                v-bind="componentField"
              />
            </FormControl>
            <FormMessage />
          </FormItem>
        </FormField>
      </div>
    </div>

    <Button type="submit">Submit</Button>
  </form>
</template>
