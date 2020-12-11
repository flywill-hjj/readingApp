<template>
  <div class="ebook-reader">
    <!-- {{ $route.params.fileName }} -->
    <div id="read"></div>
  </div>
</template>

<script>
import { ebookMixin } from "../../utils/mixin";
import Epub from "epubjs";

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
      if(this.menuVisible){
         this.setSettingVisible(-1);
      }
      this.setMenuVisible(!this.menuVisible);
    },
    hideTitleAndMenu() {
      this.setMenuVisible(false);
      this.setSettingVisible(-1);
    },
    initEpub() {
      // const url = "http://192.168.101.169:8282/" + this.fileName + ".epub";
      const url = "http://192.168.0.106:8282/" + this.fileName + ".epub";
      this.book = new Epub(url);
      this.rendition = this.book.renderTo("read", {
        width: window.innerWidth,
        height: window.innerHeight,
        method: "default",
      });

      this.rendition.display();
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
