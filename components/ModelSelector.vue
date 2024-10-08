<template>
  <div v-if="error" class="text-red-500 mt-2">{{ error.message }}</div>
  <Listbox as="div" v-model="model" v-else>
    <ListboxLabel class="sr-only">Change model</ListboxLabel>
    <div class="relative">
      <div class="inline-flex divide-x divide-indigo-700 rounded-md shadow-sm">
        <div
            class="flex gap-x-2 rounded-md bg-indigo-600 px-3 py-2 text-sm font-semibold text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
        >
          <p class="text-sm font-semibold">Wahoo Model</p>
        </div>

      </div>

      <transition
          leave-active-class="transition ease-in duration-100"
          leave-from-class="opacity-100"
          leave-to-class="opacity-0"
      >
        <ListboxOptions
            class="absolute right-0 z-10 mt-2 w-96 origin-top-right divide-y divide-gray-200 overflow-hidden rounded-md bg-white shadow-lg ring-1 ring-black ring-opacity-5 focus:outline-none"
        >
          <ListboxOption
              as="template"
              v-for="model in models.models"
              :key="model.name"
              :value="model.name"
              v-slot="{ active, selected }"
          >
            <li
                :class="[
                active ? 'bg-indigo-600 text-white' : 'text-gray-900',
                'cursor-pointer select-none p-4 text-sm',
              ]"
            >
              <div class="flex flex-col">
                <div class="flex justify-between">
                  <p :class="selected ? 'font-semibold' : 'font-normal'">
                    {{ model.name }}
                  </p>
                  <span
                      v-if="selected"
                      :class="active ? 'text-white' : 'text-indigo-600'"
                  >
                    <CheckIcon class="h-5 w-5" aria-hidden="true"/>
                  </span>
                </div>
                <p
                    :class="[
                    active ? 'text-indigo-200' : 'text-gray-500',
                    'mt-3 space-x-1',
                  ]"
                >
                  <span class="model-label">
                    <InformationCircleIcon class="h-4 w-4"/>
                    {{ model.details.format }}
                  </span>
                  <span class="model-label">
                    <CpuChipIcon class="h-4 w-4"/>
                    {{ model.details.family }}
                  </span>
                  <span class="model-label">
                    <AdjustmentsHorizontalIcon class="h-4 w-4"/>
                    {{ model.details.parameter_size }}
                  </span>

                  <span class="model-label">
                    <CalculatorIcon class="h-4 w-4"/>
                    {{ model.details.quantization_level }}
                  </span>
                </p>
              </div>
            </li>
          </ListboxOption>
          <li>
            <div class="p-4 grid grid-cols-2 gap-1">
              <span class="model-label-description">
                <InformationCircleIcon class="h-4 w-4"/>
                Format
              </span>
              <span class="model-label-description">
                <CpuChipIcon class="h-4 w-4"/>
                Family
              </span>
              <span class="model-label-description">
                <AdjustmentsHorizontalIcon class="h-4 w-4"/>
                Parameter Size
              </span>

              <span class="model-label-description">
                <CalculatorIcon class="h-4 w-4"/>
                Quantization Level
              </span>
            </div>
          </li>
        </ListboxOptions>
      </transition>
    </div>
  </Listbox>
</template>

<script setup>
import {
  Listbox,
  ListboxLabel,
  ListboxOption,
  ListboxOptions,
} from "@headlessui/vue";
import {
  CheckIcon,
  InformationCircleIcon,
  CpuChipIcon,
  AdjustmentsHorizontalIcon,
  CalculatorIcon,
} from "@heroicons/vue/20/solid";

const {data: models, error} = await useFetch("/api/models");

const model = defineModel();
</script>
