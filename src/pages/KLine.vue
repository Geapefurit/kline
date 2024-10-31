<template>
  <KLine />
</template>

<script setup lang='ts'>
import { defineAsyncComponent } from 'vue'

const KLine = defineAsyncComponent(() => import('src/components/kline/KLine.vue'))

</script>
