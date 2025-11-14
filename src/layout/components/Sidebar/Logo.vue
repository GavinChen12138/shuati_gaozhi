<template>
  <div class="sidebar-logo-container" :class="{ collapse }" :style="logoStyles">
    <div class="sidebar-logo-link">
      <el-image :src="logo" :preview-src-list="[logoMax]" class="sidebar-logo" />
      <transition name="sidebarLogoFade">
        <h1 v-show="!collapse" class="sidebar-title">{{ title }}</h1>
      </transition>
    </div>
  </div>
</template>

<script>
import variables from '@/styles/variables.scss'

export default {
  name: 'SidebarLogo',
  props: {
    collapse: {
      type: Boolean,
      required: true
    }
  },
  data() {
    return {
      title: this.$store.state.settings.title,
      logo: '/favicon-64x64.ico',
      logoMax: '/favicon.ico'
    }
  },
  computed: {
    variables() {
      return variables
    },
    logoStyles() {
      return {
        background: `linear-gradient(135deg, ${this.variables.logobg} 0%, rgba(15, 23, 42, 0.92) 100%)`,
        color: this.variables.logotext
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.sidebarLogoFade-enter-active {
  transition: opacity 1.5s;
}

.sidebarLogoFade-enter,
.sidebarLogoFade-leave-to {
  opacity: 0;
}

.sidebar-logo-container {
  position: relative;
  width: 100%;
  padding: 1.25rem 1.5rem 1rem;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  border-bottom: 1px solid rgba(226, 232, 240, 0.18);

  .sidebar-logo-link {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    width: 100%;
    justify-content: center;

    .sidebar-logo {
      width: 40px;
      height: 40px;
      border-radius: 12px;
      box-shadow: 0 12px 24px rgba(15, 23, 42, 0.25);
    }

    .sidebar-title {
      margin: 0;
      font-weight: 600;
      line-height: 1.2;
      font-size: 1.05rem;
      letter-spacing: 0.04em;
      text-transform: uppercase;
    }
  }

  &.collapse {
    padding-inline: 1rem;

    .sidebar-logo-link {
      justify-content: center;
      .sidebar-logo {
        margin-right: 0;
      }
    }
  }
}
</style>
