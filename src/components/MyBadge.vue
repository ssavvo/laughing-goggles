<script setup lang="ts">
import { computed } from 'vue';

const {variant} = defineProps<{variant: "diet" | "tariff"}>()
const optionalClass = computed(() => {
    switch(variant) {
        case 'diet':
            return 'bg-secondary text-secondary-content'
        case 'tariff':
            return 'bg-accent text-accent-content'
    }
}
)
</script>
<template>
    <div class="px-1 rounded-badge overflow-hidden whitespace-nowrap text-ellipsis text-xs text-center" :class="optionalClass">
       <slot></slot>
    </div>
</template>