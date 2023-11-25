<script setup lang="ts">
const {status, dayCount} = defineProps<{
    status: 'not started' | 'active' | 'finished';
    dayCount: number;
}>()

const message = (() => {
    switch(status) {
        case 'not started': {
            const dayCountString = dayCount === 0 ? 'сегодня' : dayCount === 1 ? 'завтра' : `через ${dayCount} дней`;
            return `Начинается ${dayCountString}`
        }
        case 'active': {
            const dayCountString = dayCount === 0 ? 'сегодня' : dayCount === 1 ? 'завтра' : `через ${dayCount} дней`;
            return `Заканчивается ${dayCountString}`
        }            
        case 'finished': {
            const dayCountString = dayCount === 0 ? 'сегодня' : dayCount === 1 ? 'вчера' : `${dayCount} дней назад`;
            return `Закончилось ${dayCountString}`
        }
    }
})()
</script>
<template>
    {{ message }}
</template>