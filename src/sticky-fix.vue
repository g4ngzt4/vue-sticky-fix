<template>
  <div ref="wrapper" :style="wrapperStyles">
    <div ref="inner" :style="innerStyles">
      <slot/>
    </div>
  </div>
</template>

<script>
export default {
  name: 'StickyFix',
  props: {
    stickyTop: {
      type: Number,
      default: 0,
    },
    zIndex: {
      type: [Number, String],
      default: 'auto',
    },
  },
  data() {
    return {
      isStuck: false,
      isSticky: false,
      wrapperStyles: {},
      innerStyles: {},
    }
  },
  computed: {
    parent() {
      return this.$el.parentNode
    },
    wrapper() {
      return this.$refs.wrapper
    },
    inner() {
      return this.$refs.inner
    },
    supportsSticky() {
      const el = document.createElement('div')
      el.style.cssText =
        'position:sticky;position:-webkit-sticky;position:-ms-sticky;'
      return el.style.position.indexOf('sticky') !== -1
    },
  },
  mounted() {
    if (!this.parent || !this.wrapper || !this.inner) return
    if (this.supportsSticky) {
      this.useNativeSticky()
      return
    }
    this.onStart()
  },
  beforeDestroy() {
    window.removeEventListener('scroll', this.onScroll)
    window.removeEventListener('resize', this.onResize)
  },
  methods: {
    onStart() {
      this.wrapperStyles.height = `${this.inner.clientHeight}px`
      this.parent.style.position = 'relative'
      this.onScroll()
      window.addEventListener('scroll', this.onScroll)
      window.addEventListener('resize', this.onResize)
    },
    onScroll() {
      const wrapperPosition = this.wrapper.getBoundingClientRect()
      const parentBottom = this.parent.getBoundingClientRect().bottom
      const isStuck =
        parentBottom - this.stickyTop - this.inner.clientHeight <= 0
      const isSticky = wrapperPosition.top <= this.stickyTop
      if (isStuck) this.makeStuck()
      else if (isSticky) this.makeSticky(wrapperPosition)
      else this.clearStyles()
    },
    onResize() {
      if (this.isStuck) this.isStuck = false
      if (this.isSticky) this.isSticky = false
      this.onScroll()
    },
    makeStuck() {
      if (this.isStuck) return
      this.innerStyles = {
        position: 'absolute',
        left: `0`,
        bottom: `0`,
        width: `100%`,
        zIndex: this.zIndex,
      }
      this.isSticky = false
      this.isStuck = true
    },
    makeSticky(position) {
      if (this.isSticky) return
      this.innerStyles = {
        position: 'fixed',
        left: `${position.left}px`,
        top: `${this.stickyTop}px`,
        width: `${position.width}px`,
        zIndex: this.zIndex,
      }
      this.isStuck = false
      this.isSticky = true
    },
    clearStyles() {
      if (!this.isStuck && !this.isSticky) return
      this.innerStyles = {}
      this.isStuck = false
      this.isSticky = false
    },
    useNativeSticky() {
      this.wrapperStyles = {
        position: 'sticky',
        top: `${this.stickyTop}px`,
        zIndex: this.zIndex,
      }
    },
  },
}
</script>
