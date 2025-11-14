<template>
  <div class="sober-navbar">
    <div class="sober-navbar__left">
      <hamburger
        id="hamburger-container"
        :is-active="sidebar.opened"
        class="sober-navbar__hamburger"
        @toggleClick="toggleSideBar"
      />
      <breadcrumb id="breadcrumb-container" class="sober-navbar__breadcrumb" />
    </div>

    <div v-loading="loading" class="sober-navbar__right">
      <template v-if="device !== 'mobile'">
        <search id="header-search" class="sober-navbar__action" />
        <error-log class="sober-navbar__action" />
        <screenfull id="screenfull" class="sober-navbar__action" />
      </template>
      <UserPannel ref="userPannel" :user-card-show.sync="userCardShow" />

      <div v-if="currentUser.data" class="sober-navbar__user-meta">
        <span class="sober-navbar__user-name">{{ currentUser.name }}</span>
        <span class="sober-navbar__user-role">{{ currentUser.data.dutiesName }}</span>
      </div>

      <div v-if="!hasLogin" class="sober-navbar__auth">
        <button type="button" class="sober-link" @click="$refs.userPannel.userCardShowing(true)">登录</button>
        <button type="button" class="sober-link" @click="$refs.userPannel.handleReg(true)">注册</button>
      </div>

      <div v-else class="sober-navbar__actions">
        <el-popover trigger="hover" popper-class="sober-navbar__popover">
          <BBSMessageBox />
          <button slot="reference" type="button" class="sober-link">消息</button>
        </el-popover>
        <el-popover trigger="hover" popper-class="sober-navbar__popover">
          <Loading />
          <button slot="reference" type="button" class="sober-link">收藏</button>
        </el-popover>
      </div>

      <div v-if="device !== 'mobile' && currentTime" class="sober-navbar__clock">
        <span>{{ currentTime.split('\n')[0] }}</span>
        <span>{{ currentTime.split('\n')[1] }}</span>
      </div>
    </div>
  </div>
</template>

<script>
import { datedifference, parseTime } from '@/utils'
export default {
  components: {
    Breadcrumb: () => import('@/components/Breadcrumb'),
    Hamburger: () => import('@/components/Hamburger'),
    ErrorLog: () => import('@/components/ErrorLog'),
    Screenfull: () => import('@/components/Screenfull'),
    Search: () => import('@/components/HeaderSearch'),
    Loading: () => import('@/views/Loading'),
    BBSMessageBox: () => import('@/views/BBSMessage/BBSMessageBox'),
    UserPannel: () => import('./UserPannel')
  },
  data: () => ({
    checker: null,
    lastUpdateShow: new Date(),
    isToShowPasswordModefier: false,
    loading: false,
    check: {
      check_sync_time: 0,
      check_user_login: 0
    },
    currentTime: null,
    userCardShow: false
  }),
  computed: {
    currentUser() {
      return this.$store.state.user
    },
    hasLogin() {
      return this.currentUser.userid
    },
    device() {
      return this.$store.state.app.device
    },
    sidebar() {
      return this.$store.state.app.sidebar
    }
  },
  mounted() {
    this.checker = setInterval(() => {
      this.check_sync_time()
      this.check_user_login()
      const s = this.$store.state.settings.currentTime_left_status
      if (s) {
        this.currentTime = `时间同步\n${s}`
      } else {
        const d = new Date(
          new Date() - 0 + this.$store.state.settings.currentTimeDelta_left
        )
        this.currentTime = parseTime(d, '{y}年{m}月{d}日\n{h}时{i}分{s}秒')
      }
    }, 1000)
    this.check_sync_time(true)
  },
  destroyed() {
    if (this.checker) clearInterval(this.checker)
  },
  methods: {
    check_sync_time(direct_load = false) {
      const d = datedifference(new Date(), this.check.check_sync_time, 'minute')
      if (d < 30 && !direct_load) return // 30分钟同步一次
      // console.log('check sync time after ' + d + ' minute(s)')
      this.check.check_sync_time = new Date()
      this.$store.dispatch('settings/sync_time')
    },
    check_user_login() {
      const d = datedifference(
        new Date(),
        this.check.check_user_login,
        'minute'
      )
      if (d < 10) return
      // console.log('check login status')
      this.check.check_user_login = new Date()
      this.$store.dispatch('user/check_login')
    },
    toggleSideBar() {
      this.$store.dispatch('app/toggleSideBar')
    }
  }
}
</script>

<style lang="scss" scoped>
@import './menu-divider.scss';
@import './nav-bar.scss';
</style>
