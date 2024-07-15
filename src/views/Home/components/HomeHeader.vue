<template>
  <div class="home-header">
    <span class="home-header-logo">
      <i class="iconfont icon-vue"></i>
      <i class="iconfont icon-icon-test"></i>
      <i class="iconfont icon-typescript"></i>
    </span>
    <span class="home-header-title">在线考勤系统</span>
    <el-dropdown>
      <el-badge :is-dot="isDot" class="home-header-badge">
        <el-icon :size="20"><Bell /></el-icon>
      </el-badge>
      <template #dropdown>
        <el-dropdown-menu>
          <el-dropdown-item v-if="newsInfo.applicant" @click="handleNavigate('/apply')"
            >有审批结果消息</el-dropdown-item
          >
          <el-dropdown-item v-if="newsInfo.approver" @click="handleNavigate('/check')"
            >有审批请求消息</el-dropdown-item
          >
          <el-dropdown-item v-if="!newsInfo.applicant && !newsInfo.approver"
            >暂无消息</el-dropdown-item
          >
        </el-dropdown-menu>
      </template>
    </el-dropdown>
    <el-dropdown>
      <el-space class="home-header-space"> <el-avatar :src="head" />{{ name }} </el-space>
      <template #dropdown>
        <el-dropdown-menu>
          <el-dropdown-item>个人中心</el-dropdown-item>
          <el-dropdown-item @click="handleLogout">退出</el-dropdown-item>
        </el-dropdown-menu>
      </template>
    </el-dropdown>
  </div>
</template>

<script lang="ts" setup>
import { useUsersStore } from '@/stores/users'
import { useNewsStore } from '@/stores/news'
import { storeToRefs } from 'pinia'
import { computed } from 'vue'
import { useRouter } from 'vue-router'
const usersStore = useUsersStore()
const newsStore = useNewsStore()
const { infos } = storeToRefs(usersStore)
const { info: newsInfo } = storeToRefs(newsStore)
const router = useRouter()

const head = computed(() => infos.value.head)
const name = computed(() => infos.value.name)
const isDot = computed(() => (newsInfo.value.applicant || newsInfo.value.approver) as boolean)

const handleLogout = () => {
  usersStore.clearToken()
  setTimeout(() => {
    window.location.replace('/login')
  }, 500)
}

const handleNavigate = (path: string) => {
  router.push({
    path: path
  })
}
</script>

<style lang="scss" scoped>
.home-header {
  display: flex;
  align-items: center;
  height: 100%;
  font-size: 34px;
  .home-header-logo {
    .icon-vue,
    .icon-icon-test,
    .icon-typescript {
      margin-right: 5px;
      font-size: inherit;
    }
    .icon-vue {
      color: green;
    }
    .icon-icon-test {
      color: #deb887;
    }
    .icon-typescript {
      color: blue;
    }
  }
  .home-header-title {
    margin-right: auto;
    margin-left: 30px;
    font-weight: 700;
    font-size: 18px;
  }
  .home-header-space {
    margin-left: 20px;
  }
}
</style>
