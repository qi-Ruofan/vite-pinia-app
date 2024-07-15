<template>
  <el-breadcrumb class="home-breadcrumb" separator="/">
    <el-breadcrumb-item v-for="item in breadcrumbs" :key="item.name">{{ item.meta?.title }}</el-breadcrumb-item>
  </el-breadcrumb>
</template>

<script lang='ts' setup>
import { useRoute } from 'vue-router';
import { computed } from 'vue'
const route = useRoute()
const breadcrumbs = computed(() => route.matched)
</script>

<style lang='scss' scoped>
.home-breadcrumb {
  background: #fff;
  height: 40px;
  display: flex;
  align-items: center;
  padding-left: 30px;
}
</style>
