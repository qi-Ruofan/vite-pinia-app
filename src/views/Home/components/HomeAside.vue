<template>
  <el-menu :default-active="route.path" router>
    <el-sub-menu v-for="item in menus" :key="item.path" :index="item.path">
      <template #title>
        <el-icon><component :is="item.meta?.icon"></component></el-icon>
        <span>{{ item.meta?.title }}</span>
      </template>
      <el-menu-item
        v-for="itemChild in item.children"
        :key="itemChild.path"
        :index="item.path + itemChild.path"
      >
        <el-icon><component :is="itemChild.meta?.icon"></component></el-icon>
        <span>{{ itemChild.meta?.title }}</span>
      </el-menu-item>
    </el-sub-menu>
  </el-menu>
</template>

<script lang="ts" setup>
import { useRouter, useRoute } from 'vue-router'
import type { RouteRecordName } from 'vue-router'
import { useUsersStore } from '@/stores/users'
import { storeToRefs } from 'pinia'
import _ from 'lodash'

const router = useRouter()
const usersStore = useUsersStore()
const { infos } = storeToRefs(usersStore)
const route = useRoute()

const permission = infos.value.permission

// 不影响原来的数组，深拷贝一个
const menus = _.cloneDeep(router.options.routes).filter((v) => {
  v.children = v.children?.filter((v) => {
    return v.meta?.menu && (permission as (RouteRecordName | undefined)[]).includes(v.name)
  })
  return v.meta?.menu && (permission as (RouteRecordName | undefined)[]).includes(v.name)
})
</script>

<style lang="scss" scoped>
.el-menu {
  height: calc(100vh - 60px);
  border: none;
  padding-top: 30px;
}
.el-menu-item.is-active {
  background: #e6f7ff;
  border-right: 3px solid #1890ff;
}
</style>
