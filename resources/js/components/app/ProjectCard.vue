<script lang="ts" setup>
import { computed, ref } from 'vue';

const props = defineProps<{
  title: string;
  image: string;
  description: string;
  stack: string[];
}>();

const expanded = ref(false);

const shouldShowToggle = computed(() => props.description.length > 100);

const displayedDescription = computed(() => {
  if (expanded.value || !shouldShowToggle.value) {
    return props.description;
  }
  return props.description.slice(0, 100) + '...';
});


</script>

<template>
  <div
    class="w-full h-auto bg-sky-50 border border-cyan-200 rounded-2xl shadow-md hover:shadow-xl transition-all duration-200 overflow-hidden border border-gray-200"
  >
    <a href="#" target="_blank" rel="noopener">
      <img
        :src="image"
        alt="Project preview"
        class="w-full aspect-video object-cover"
      />
    </a>
    <div class="p-4">
      <h3 class="text-xl font-bold text-cyan-700 mb-2">{{ title }}</h3>
        <div class="flex flex-wrap gap-2 mt-2">
            <span
                v-for="tech in props.stack"
                :key="tech"
                class="px-2 py-1 text-xs bg-cyan-100 text-cyan-800 font-medium rounded-full border border-cyan-200"
            >
                {{ tech }}
            </span>
        </div>


      <p class="text-gray-700 text-sm leading-relaxed">
        {{ displayedDescription }}
        <span
          v-if="shouldShowToggle"
          @click="expanded = !expanded"
          class="text-cyan-700 underline ml-1"
        >
          {{ expanded ? 'See less' : 'See more...' }}
        </span>
      </p>
    </div>
  </div>
</template>
