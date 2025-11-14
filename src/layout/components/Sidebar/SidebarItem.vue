<template>
  <div v-if="!item.hidden" class="sober-menu__item" :class="{ 'sober-menu__child': isNest }">
    <app-link
      v-if="showSingleChild && (!onlyOneChild.children || onlyOneChild.noShowingChildren) && !item.alwaysShow"
      :to="resolvePath(onlyOneChild.path)"
    >
      <template #default="{ href, navigate }">
        <a
          :href="href"
          class="sober-menu__trigger"
          :class="{ 'is-active': isActiveByPath(resolvePath(onlyOneChild.path)) }"
          @click="navigate"
        >
          <span class="sober-menu__icon">
            <SvgIcon :icon-class="resolveIcon(singleMeta, item.meta)" />
          </span>
          <span class="sober-menu__label">{{ resolveMenuLabel(singleMeta, item.meta) }}</span>
        </a>
      </template>
    </app-link>

    <div v-else class="sober-menu__branch">
      <button
        type="button"
        class="sober-menu__trigger"
        :class="{ 'is-active': isActiveByPath(resolvePath(item.path)) }"
        :aria-expanded="isOpen"
        @click="toggleOpen"
      >
        <span class="sober-menu__icon">
          <SvgIcon :icon-class="resolveIcon(item.meta)" />
        </span>
        <span class="sober-menu__label">{{ resolveMenuLabel(item.meta) }}</span>
        <span class="sober-menu__caret" :class="{ 'is-open': isOpen }">›</span>
      </button>
      <transition name="sober-menu-collapse">
        <div v-show="isOpen" class="sober-menu__children">
          <div class="sober-menu__children-inner">
            <sidebar-item
              v-for="child in item.children"
              :key="child.path"
              :is-nest="true"
              :item="child"
              :base-path="resolvePath(child.path)"
              class="sober-menu__child"
            />
          </div>
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
import SvgIcon from '@/components/SvgIcon'
import { generateTitle } from '@/utils/get-page-title'
import { isExternal } from '@/utils/validate'
import AppLink from './Link'
import FixiOSBug from './FixiOSBug'

export default {
  name: 'SidebarItem',
  components: { AppLink, SvgIcon },
  mixins: [FixiOSBug],
  props: {
    // route object
    item: {
      type: Object,
      required: true
    },
    isNest: {
      type: Boolean,
      default: false
    },
    basePath: {
      type: String,
      default: ''
    }
  },
  data() {
    this.onlyOneChild = null
    return {
      isOpen: false
    }
  },
  computed: {
    showSingleChild() {
      return this.hasOneShowingChild(this.item.children, this.item)
    },
    currentActiveMenu() {
      const { meta, path } = this.$route
      if (meta && meta.activeMenu) {
        return meta.activeMenu
      }
      return path
    },
    shouldExpandByDefault() {
      if (!this.item.children || !this.item.children.length) {
        return false
      }
      return this.containsActiveChild(this.item.children)
    },
    singleMeta() {
      return (this.onlyOneChild && this.onlyOneChild.meta) || {}
    }
  },
  watch: {
    $route() {
      if (this.item.children && this.item.children.length) {
        this.isOpen = this.shouldExpandByDefault
      }
    }
  },
  created() {
    if (this.item.children && this.item.children.length) {
      this.isOpen = this.shouldExpandByDefault
    }
  },
  methods: {
    hasOneShowingChild(children = [], parent) {
      const showingChildren = children.filter(item => {
        if (item.hidden) {
          return false
        } else {
          this.onlyOneChild = item
          return true
        }
      })
      if (showingChildren.length === 1) {
        return true
      }
      if (showingChildren.length === 0) {
        this.onlyOneChild = { ...parent, path: '', noShowingChildren: true }
        return true
      }
      return false
    },
    resolvePath(routePath) {
      if (isExternal(routePath)) {
        return routePath
      }
      if (isExternal(this.basePath)) {
        return this.basePath
      }
      const base = (this.basePath || '').replace(/\/+$/, '')
      const segment = (routePath || '').replace(/^\/+/, '')
      const merged = [base, segment].filter(Boolean).join('/') || '/'
      const normalized = merged.replace(/\/+/g, '/')
      return normalized.startsWith('/') ? normalized : `/${normalized}`
    },
    containsActiveChild(children = []) {
      return children.some(child => {
        if (child.hidden) {
          return false
        }
        if (child.children && child.children.length) {
          return this.containsActiveChild(child.children)
        }
        return this.isActiveByPath(this.resolvePath(child.path))
      })
    },
    isActiveByPath(path) {
      if (!path || isExternal(path)) {
        return false
      }
      const normalized = path.replace(/\/$/, '') || '/'
      const active = (this.currentActiveMenu || '').replace(/\/$/, '') || '/'
      const current = this.$route.path.replace(/\/$/, '') || '/'
      return active === normalized || current === normalized
    },
    toggleOpen() {
      this.isOpen = !this.isOpen
    },
    resolveMenuLabel(meta = {}, fallback = {}) {
      if (fallback && fallback.ctitle) {
        return fallback.ctitle
      }
      if (meta.ctitle) {
        return meta.ctitle
      }
      if (meta.title) {
        return generateTitle(meta)
      }
      if (fallback && fallback.title) {
        return generateTitle(fallback)
      }
      return ''
    },
    resolveIcon(meta = {}, fallback = {}) {
      if (meta && meta.icon) {
        return meta.icon
      }
      if (fallback && fallback.icon) {
        return fallback.icon
      }
      return ''
    },
    generateTitle
  }
}
</script>
