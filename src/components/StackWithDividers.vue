<script setup lang="ts">
const { list } = defineProps<{
    list: {
        [key: string]: string;
    }[]
}>();
</script>

<template>
    <div class="flex flex-col justify-around max-w-[120px]">
        <div v-for="(item, i) in list">
            <slot v-bind="item"></slot>
            <div class="divider m-0" v-if="(i + 1) !== list.length"></div>
        </div>
    </div>
</template>