<template>
  <div class="ebook-reader">
    <div id="read"></div>
  </div>
</template>

<script>

  import { ebookMixin } from '../../utils/mixin'
  import Epub from 'epubjs'
  import {
    getFontFamily,
    getFontSize,
    getTheme,
    saveTheme,
    saveFontFamily,
    saveFontSize,
    getLocation
  } from '../../utils/localStorage'
  import { flatten } from '../../utils/book'

  global.ePub = Epub
  export default {
    mixins: [ebookMixin],
    methods: {
      prevPage () {
        if (this.rendition) {
          this.rendition.prev().then(() => {
            this.refreshLocation()
          })
          this.hideTitleAndMenu()
        }
      },
      nextPage () {
        if (this.rendition) {
          this.rendition.next().then(() => {
            this.refreshLocation()
          })
          this.hideTitleAndMenu()
        }
      },
      toggleTitleAndMenu () {
        if (this.menuVisible) {
          this.setSettingVisible(-1)
          this.setFontFamilyVisible(false)
        }
        this.setMenuVisible(!this.menuVisible)
      },
      initFontSize () {
        let fontSize = getFontSize(this.fileName)
        if (!fontSize) {
          saveFontSize(this.fileName, this.defaultFontSize)
        } else {
          this.rendition.themes.font(fontSize)
          this.setDefaultFontSize(fontSize)
        }
      },
      initFontFamily () {
        let font = getFontFamily(this.fileName)
        if (!font) {
          saveFontFamily(this.fileName, this.defaultFontFamily)
        } else {
          this.rendition.themes.font(font)
          this.setDefaultFontFamily(font)
        }
      },
      initTheme () {
        let defaultTheme = getTheme(this.fileName)
        if (!defaultTheme) {
          defaultTheme = this.themeList[0].name
          saveTheme(this.fileName, defaultTheme)
        }
        this.setDefaultTheme(defaultTheme)
        this.themeList.forEach((theme) => {
          this.rendition.themes.register(theme.name, theme.style)
        })
        this.rendition.themes.select(defaultTheme)
      },
      initRendition () {
        this.rendition = this.book.renderTo('read', {
          width: innerWidth,
          height: innerHeight,
          method: 'default'
        })
        const location = getLocation(this.fileName)
        this.display(location, () => {
            this.initTheme()
            this.initFontSize()
            this.initFontFamily()
            this.initGlobalStyle()
        })
        this.rendition.hooks.content.register((contents) => {
          Promise.all([
            contents.addStylesheet(
              `${process.env.VUE_APP_RES_URL}/fonts/daysOne.css`
            ),
            contents.addStylesheet(
              `${process.env.VUE_APP_RES_URL}/fonts/cabin.css`
            ),
            contents.addStylesheet(
              `${process.env.VUE_APP_RES_URL}/fonts/tangerine.css`
            ),
            contents.addStylesheet(
              `${process.env.VUE_APP_RES_URL}/fonts/montserrat.css`
            )
          ]).then(() => {})
        })
      },
      initGesture () {
        this.rendition.on('touchstart', (event) => {
          this.touchStartX = event.changedTouches[0].clientX
          this.touchStartTime = event.timeStamp
        })
        this.rendition.on('touchend', (event) => {
          const offsetX = event.changedTouches[0].clientX - this.touchStartX
          const time = event.timeStamp - this.touchStartTime
          if (time < 500 && offsetX > 40) {
            this.prevPage()
          } else if (time < 500 && offsetX < -40) {
            this.nextPage()
          } else {
            this.toggleTitleAndMenu()
          }
          event.preventDefault()
          event.stopPropagation()
        })
      },
      parseBook () {
        this.book.loaded.cover.then(cover => {
          this.book.archive.createUrl(cover).then(url => {
            this.setCover(url)
          })
        })
        this.book.loaded.metadata.then(metadata => {
          this.setMetadata(metadata)
        })
        this.book.loaded.navigation.then(nav => {
          const navItem = flatten(nav.toc)
          function find (item, level = 0) {
            return (!item.parent) ? level : find(navItem.filter(parentItem => parentItem.id === item.parent)[0], ++level)
          }
          navItem.forEach(item => {
            item.level = find(item)
          })
          this.setNavigation(navItem)
        })
      },
      initEpub () {
        const url =
          process.env.VUE_APP_RES_URL + '/epub/' + this.fileName + '.epub'
        this.book = new Epub(url)
        this.setCurrentBook(this.book)
        this.initRendition()
        this.initGesture()
        this.parseBook()
        this.book.ready.then(() => {
          // book 解析后调用
          return this.book.locations.generate(
            750 * (window.innerWidth / 375) * (getFontSize(this.fileName) / 16)
          )
        }).then(locations => {
          this.setBookAvailable(true)
          this.refreshLocation()
        })
      }
    },
    mounted () {
      this.setFileName(this.$route.params.fileName.split('|').join('/')).then(
        () => {
          this.initEpub()
        }
      )
    }
  }
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
@import "../../assets/style/global.scss";
</style>
