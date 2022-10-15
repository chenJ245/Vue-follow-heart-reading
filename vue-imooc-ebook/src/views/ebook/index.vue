<template>
  <div class="ebook" ref="ebook">
    <!-- 阅读器页眉组件 -->
    <ebook-header></ebook-header>
    <!-- 阅读器标题组件 -->
    <ebook-title></ebook-title>
     <!-- 阅读器组件 -->
    <ebook-reader></ebook-reader>
    <!-- 阅读器菜单组件 -->
    <ebook-menu></ebook-menu>
    <!-- 阅读器书签组件 -->
    <ebook-bookmark></ebook-bookmark>
    <!-- 阅读器页脚组件 -->
    <ebook-footer></ebook-footer>

  </div>
</template>

<script>
  import EbookReader from '../../components/ebook/EbookReader.vue'
  import EbookTitle from '../../components/ebook/EbookTitle.vue'
  import EbookMenu from '../../components/ebook/EbookMenu.vue'
  import EbookBookmark from '../../components/ebook/EbookBookmark.vue'
  import EbookHeader from '../../components/ebook/EbookHeader.vue'
  import EbookFooter from '../../components/ebook/EbookFooter.vue'
  import { getReadTime, saveReadTime } from '../../utils/localStorage'
  import { ebookMixin } from '../../utils/mixin'

  export default {
    mixins: [ebookMixin],
    components: {
    EbookReader,
    EbookTitle,
    EbookMenu,
    EbookBookmark,
    EbookHeader,
    EbookFooter
},
    watch: {
      offsetY (v) {
        if (!this.menuVisible && this.bookAvailable) {
          if (v > 0) { // 上拉就是 相反
          this.move(v)
          } else if (v === 0) {
            this.restore()
          }
        }
      }
    },
    methods: {
      restore () {
        this.$refs.ebook.style.top = 0
        this.$refs.ebook.style.transition = 'all .2s linear'
        setTimeout(() => {
          this.$refs.ebook.style.transition = ''
        }, 200)
      },
      move (v) {
        this.$refs.ebook.style.top = v + 'px'
      },
      startLoopReadTime () {
        let readTime = getReadTime(this.fileName)
        if (!readTime) {
          readTime = 0
        }
        this.task = setInterval(() => {
          readTime++
          if (readTime % 30 === 0) {
            saveReadTime(this.fileName, readTime)
          }
        }, 1000)
      }
    },
    mounted () {
      this.startLoopReadTime()
    },
    beforeDestroy () {
      if (this.task) {
        clearInterval(this.task)
      }
    }
  }
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
  @import "../../assets/style/global.scss";
  .ebook {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
  }
</style>
