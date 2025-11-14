
<template>
  <router-link v-if="!isExternal" :to="to" custom>
    <template #default="slotProps">
      <slot v-bind="slotProps" />
    </template>
  </router-link>
  <a v-else :href="to" target="_blank" rel="noopener">
    <slot :href="to" :navigate="noop" :is-active="false" :is-exact-active="false" />
  </a>
</template>

<script>
import { isExternal } from '@/utils/validate'

export default {
  props: {
    to: {
      type: String,
      required: true
    }
  },
  data: () => ({
    isExternal: false
  }),
  watch: {
    to: {
      handler(val) {
        this.isExternal = val && isExternal(val)
      },
      immediate: true
    }
  },
  methods: {
    noop() {}
  }
}
</script>
