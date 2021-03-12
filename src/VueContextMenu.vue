<template>
  <div 
    style="display: block;" 
    ref="selfContextMenu"
    :style="style" 
    @mousedown.stop
    @contextmenu.prevent
  >
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: 'context-menu',
  data () {
    return {
      triggerShowFn: () => {},
      triggerHideFn: () => {},
      coordinate:{
        x: null,
        y: null,
      },
      // 右键菜单大小，width、height
      size:{},
      
      // style: {},

      binded: false
    }
  },
  props: {
    target: null,
    show: Boolean
  },
  computed:{
    style(){
      
      // 屏幕的大小
      let {clientWidth, clientHeight} = document.body
      // 鼠标右键坐标
      let {x, y} = this.coordinate
      // 获取右键菜单的宽、高
      let {width, height} = this.size
      
      // 判断右侧是否可以显示，如果不可以则向左显示
      let left = x + width > clientWidth && clientWidth >= 2 * width ? x - width : x
      // 判断是否下方是否可以显示，如果不可以则向上显示
      let top = y + height > clientHeight && clientHeight >= 2* height ? y - height : y

      return {
        'visibility': this.show ? 'visible' : 'hidden',
        'left': left + 'px',
        'top': top + 'px'
      }
    }
  },
  mounted () {
    this.bindEvents()

    // console.log('大小：',this.$refs.selfContextMenu.offsetHeight)
    this.size = {
      width: this.$refs.selfContextMenu.offsetWidth,
      height: this.$refs.selfContextMenu.offsetHeight
    }
  },
  watch: {
    show (v) {
      if (v) {
        this.bindHideEvents()
      } else {
        this.unbindHideEvents()
      }
    },
    target (target) {
      this.bindEvents()
    }
  },
  methods: {
    // 初始化事件
    bindEvents () {
      this.$nextTick(() => {
        if (!this.target || this.binded) return 
        this.triggerShowFn = this.contextMenuHandler.bind(this)
        // 右键事件
        this.target.addEventListener('contextmenu', this.triggerShowFn)
        this.binded = true
      })
    },

    // 取消绑定事件
    unbindEvents () {
      if (!this.target) return
      this.target.removeEventListener('contextmenu', this.triggerShowFn)
    },

    // 绑定隐藏菜单事件
    bindHideEvents () {
      this.triggerHideFn = this.clickDocumentHandler.bind(this)
      document.addEventListener('mousedown', this.triggerHideFn)
      document.addEventListener('mousewheel', this.triggerHideFn)
    },

    // 取消绑定隐藏菜单事件
    unbindHideEvents () {
      document.removeEventListener('mousedown', this.triggerHideFn)
      document.removeEventListener('mousewheel', this.triggerHideFn)
    },

    // 鼠标按压事件处理器
    clickDocumentHandler (e) {
      this.$emit('update:show', false)
    },

    // 右键事件事件处理
    contextMenuHandler (e) {
      this.coordinate = {
        x: e.clientX,
        y: e.clientY
      }
      this.$emit('update:show', true)
      this.$emit('action', e)
      e.preventDefault()
    },

    // 布局
  }
}
</script>
