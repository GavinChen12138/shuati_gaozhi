<template>
  <section v-if="isAlive" class="app-main">
    <transition name="fade-transform">
      <keep-alive :include="cachedViews">
        <router-view :key="key" />
      </keep-alive>
    </transition>
  </section>
</template>

<script>
export default {
  name: 'AppMain',
  data: () => ({
    isAlive: true
  }),
  computed: {
    cachedViews() {
      return this.$store.state.tagsView.cachedViews
    },
    key() {
      return this.$route.fullPath
    }
  },
  methods: {
    reload() {
      this.isAlive = false
      this.$nextTick(() => {
        this.isAlive = true
        this.$store.dispatch('user/initUserInfo')
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.app-main {
  width: 100%;
  position: relative;
  overflow: hidden;
  flex: 1;
  min-height: calc(100vh - var(--sober-navbar-height));
}

.hasTagsView {
  .app-main {
    min-height: calc(100vh - var(--sober-navbar-height));
  }
}
</style>

