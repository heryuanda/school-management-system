<script lang="ts" setup>
import { defineProps, defineEmits } from "vue";

const props = defineProps<{
  label?: string;
  modelValue: string | number;
}>();

const { label = "" } = props;

const emit = defineEmits<{
  (event: "enter"): void;
  (event: "update:modelValue", value: string): void;
}>();

const handleEnter = () => {
  emit("enter");
};
</script>

<template>
  <div>
    <label
      v-if="label"
      class="text-sm text-neutral-700 font-medium mb-1 inline-block pl-3"
    >
      {{ label }}
    </label>

    <input
      v-bind="$attrs"
      @input="$emit('update:modelValue', $event.target.value)"
      class="px-3 py-3 focus:outline-none border border-neutral-300 w-full rounded-lg"
      :value="modelValue"
      v-on:keyup.enter="handleEnter()"
    />
  </div>
</template>

<style lang="scss" scoped>
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/* Firefox */
input[type="number"] {
  -moz-appearance: textfield;
}
</style>
