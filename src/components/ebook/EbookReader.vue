<template>
  <div class="ebook-reader">
    <!-- {{ $route.params.fileName }} -->
    <div id="read"></div>
  </div>
</template>

<script>
import { ebookMixin } from "../../utils/mixin";
import Epub from "epubjs";
import {
  getFontFamily,
  getFontSize,
  getTheme,
  saveFontFamily,
  saveFontSize,
  saveTheme,
} from "../../utils/localStorage";
import { addCss } from "../../utils/book";

global.ePub = Epub;

export default {
  mixins: [ebookMixin],
  methods: {
    prevPage() {
      if (this.rendition) {
        this.rendition.prev();
        this.hideTitleAndMenu();
      }
    },
    nextPage() {
      if (this.rendition) {
        this.rendition.next();
        this.hideTitleAndMenu();
      }
    },
    toogleTitleAndMenu() {
      if (this.menuVisible) {
        this.setSettingVisible(-1);
        this.setFontFamilyVisible(false);
      }
      this.setMenuVisible(!this.menuVisible);
    },
    hideTitleAndMenu() {
      this.setMenuVisible(false);
      this.setSettingVisible(-1);
      this.setFontFamilyVisible(false);
    },
    initFontSize() {
      let fontSize = getFontSize(this.fileName);
      if (!fontSize) {
        saveFontSize(this.fileName, this.defaultFontSize);
      } else {
        //* 如果能够获取到
        this.rendition.themes.fontSize(fontSize);
        this.setDefaultFontSize(fontSize);
      }
    },
    initFontFamily() {
      let font = getFontFamily(this.fileName);
      if (!font) {
        saveFontFamily(this.fileName, this.defaultFontFamily);
      } else {
        //* 如果能够获取到
        this.rendition.themes.font(font);
        this.setDefaultFontFamily(font);
      }
    },
    initTheme() {
      let defaultTheme = getTheme(this.fileName);
      if (!defaultTheme) {
        defaultTheme = this.themeList[0].name;
        saveTheme(this.fileName, defaultTheme);
      }
      this.setDefaultTheme(defaultTheme);

      this.themeList.forEach((theme) => {
        this.rendition.themes.register(theme.name, theme.style);
      });
      this.rendition.themes.select(defaultTheme);
    },
    
    initEpub() {
      // const url = "http://192.168.101.169:8282/" + this.fileName + ".epub";
      const url = `${process.env.VUE_APP_RES_URL}/` + this.fileName + ".epub";
      this.book = new Epub(url);
      this.setCurrentBook(this.book);
      this.rendition = this.book.renderTo("read", {
        width: window.innerWidth,
        height: window.innerHeight,
        method: "default",
      });

      this.rendition.display().then(() => {
        this.initTheme();
        this.initFontSize();
        this.initFontFamily();
        this.initGlobalStyle();
      });
      this.rendition.on("touchstart", (event) => {
        this.touchStartX = event.changedTouches[0].clientX;
        this.touchStartTime = event.timeStamp;
      });
      this.rendition.on("touchend", (event) => {
        const offsetX = event.changedTouches[0].clientX - this.touchStartX;
        const time = event.timeStamp - this.touchStartTime;
        if (time < 500 && offsetX > 40) {
          this.prevPage();
        } else if (time < 500 && offsetX < -40) {
          this.nextPage();
        } else {
          this.toogleTitleAndMenu();
        }
        event.preventDefault();
        event.stopPropagation();
      });
      this.rendition.hooks.content.register((contents) => {
        Promise.all([
          contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/daysOne.css`),
          contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/montserrat.css`),
          contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/cabin.css`),
          contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/tangerine.css`),
        ]).then(() => {
          console.log("字体全部加载完毕....");
        });
      });
    },
  },
  mounted() {
    this.setFileName(this.$route.params.fileName.split("|").join("/")).then(() => {
      this.initEpub();
    });
  },
};
</script>

<style lang="scss" scoped></style>
