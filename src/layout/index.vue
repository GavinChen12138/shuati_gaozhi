<template>
  <div :class="[classObj]" class="app-wrapper sober-app">
    <div v-if="device === 'mobile' && sidebar.opened" class="drawer-bg" @click="handleClickOutside" />
    <sidebar class="sober-sidebar" />
    <div :class="['sober-content', { hasTagsView: needTagsView }]">
      <header :class="['sober-content__header', { 'is-fixed': fixedHeader }]">
        <navbar />
      </header>
      <main class="sober-content__body">
        <app-main />
      </main>
    </div>
  </div>
</template>

<script>
import { AppMain, Navbar, Sidebar } from './components'
import ResizeMixin from './mixin/ResizeHandler'
import { mapState } from 'vuex'

export default {
  name: 'Layout',
  components: {
    AppMain,
    Navbar,
    Sidebar,
  },
  mixins: [ResizeMixin],
  computed: {
    ...mapState({
      sidebar: (state) => state.app.sidebar,
      device: (state) => state.app.device,
      showSettings: (state) => state.settings.showSettings,
      needTagsView: (state) => state.settings.tagsView,
      fixedHeader: (state) => state.settings.fixedHeader,
    }),
    classObj() {
      return {
        hideSidebar: !this.sidebar.opened,
        openSidebar: this.sidebar.opened,
        withoutAnimation: this.sidebar.withoutAnimation,
        mobile: this.device === 'mobile',
      }
    },
  },
  methods: {
    handleClickOutside() {
      this.$store.dispatch('app/closeSideBar', { withoutAnimation: false })
    },
  },
}
</script>

<style lang="scss" scoped>
@import '~@/styles/mixin.scss';

.app-wrapper {
  @include clearfix;
  min-height: 100vh;
  width: 100%;
}

.drawer-bg {
  cursor: pointer;
}

.sober-content__header {
  display: flex;
  align-items: center;
}

.sober-content__body {
  min-height: calc(100vh - var(--sober-navbar-height));
}

.mobile.openSidebar.app-wrapper {
  position: fixed;
  inset: 0;
}
</style>
