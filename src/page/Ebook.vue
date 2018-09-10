<template>
  <div class="ebook">
    <title-bar :ifTitleAndMenuShow="ifTitleAndMenuShow"></title-bar>
    <div class="read-wrapper">
      <div id="read"></div>
      <div class="mask">
        <div class="left" @click="prevPage"></div>
        <div class="content" @click="toggleTitleAndMenu"></div>
        <div class="right" @click="nextPage"></div>
      </div>
    </div>
    <menu-bar :ifTitleAndMenuShow="ifTitleAndMenuShow" :fontSizeList="fontSizeList" :defaultFontSize="defaultFontSize" :defaultTheme="defaultTheme" :themeList="themeList" :bookAvailable="bookAvailable" :navigation="navigation" :currentContent="currentContent" @setFontSize="setFontSize" @setTheme="setTheme" @onProgressChange="onProgressChange" @jumpTo="jumpTo" ref="menuBar"></menu-bar>
  </div>
</template>

<script>
import Epub from "epubjs";
import TitleBar from "@/components/TitleBar";
import MenuBar from "@/components/MenuBar";
const DOWNLOAD_URL = "ebook.epub";

export default {
  data() {
    return {
      navigation: {},
      ifTitleAndMenuShow: false,
      fontSizeList: [
        { fontSize: 12 },
        { fontSize: 14 },
        { fontSize: 16 },
        { fontSize: 18 },
        { fontSize: 20 },
        { fontSize: 22 },
        { fontSize: 24 }
      ],
      defaultFontSize: 16,
      themeList: [
        {
          name: "default",
          style: {
            body: {
              color: "#000000",
              background: "#ffffff"
            }
          }
        },
        {
          name: "eye",
          style: {
            body: {
              color: "#ffffff",
              background: "#90ee90"
            }
          }
        },
        {
          name: "night",
          style: {
            body: {
              color: "#ffffff",
              background: "#000000"
            }
          }
        },
        {
          name: "gold",
          style: {
            body: {
              color: "#000000",
              background: "#ffd700"
            }
          }
        }
      ],
      defaultTheme: 0,
      bookAvailable: false,
      currentContent: { tocIndex: 0, subtocIndex: 0 }
    };
  },
  components: {
    TitleBar,
    MenuBar
  },
  methods: {
    jumpTo(href) {
      this.rendition.display(href).then(() => {
        this.ifTitleAndMenuShow = false;
        this.$refs.menuBar.hideSetting(); // 调用子组件的方法隐藏设置栏
        this.$refs.menuBar.hideContent(); // 调用子组件的方法隐藏目录
        this.onContentChange(); // 更新当前目录
        this.setProcess(); // 更新进度条
      });
    },
    // 进度条改变更新对应的内容
    onProgressChange(progress) {
      const percentage = progress / 100; // progress是一个0-100的数字，换成百分比
      const location =
        percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0;
      this.rendition.display(location).then(() => {
        this.onContentChange();
      });
    },
    setFontSize(fontSize) {
      this.defaultFontSize = fontSize;
      if (this.themes) {
        this.themes.fontSize(fontSize + "px");
      }
    },
    prevPage() {
      if (this.rendition && this.bookAvailable) {
        this.rendition.prev().then(() => {
          this.onContentChange();
          this.setProcess();
        });
        this.ifTitleAndMenuShow = false;
        this.$refs.menuBar.hideSetting();
      }
    },
    toggleTitleAndMenu() {
      this.ifTitleAndMenuShow = !this.ifTitleAndMenuShow;
      if (!this.ifTitleAndMenuShow) {
        this.$refs.menuBar.hideSetting();
      }
    },
    onContentChange() {
      let currentLocation = this.rendition.currentLocation();
      this.currentContent = this.navigation.tocByHref[
        currentLocation.start.href
      ];
    },
    setProcess() {
      let currentLocation = this.rendition.currentLocation();
      let currentPage = this.locations.percentageFromCfi(
        currentLocation.start.cfi
      );
      this.$refs.menuBar.progress = currentPage.toFixed(2) * 100;
      if (this.bookAvailable && this.$refs.menuBar.$refs.progressBar) {
        this.$refs.menuBar.onProgressInput(this.$refs.menuBar.progress);
      }
    },
    nextPage() {
      if (this.rendition && this.bookAvailable) {
        this.rendition.next().then(() => {
          this.setProcess();
          this.onContentChange();
        });
        this.ifTitleAndMenuShow = false;
        this.$refs.menuBar.hideSetting();
      }
    },
    showEpub() {
      this.book = new Epub(DOWNLOAD_URL);
      this.rendition = this.book.renderTo("read", {
        width: window.innerWidth,
        height: window.innerHeight
      });
      this.rendition.display();
      this.themes = this.rendition.themes;
      this.setFontSize(this.defaultFontSize);
      this.registerTheme();
      this.setTheme(this.defaultTheme);
      this.book.ready
        .then(() => {
          this.navigation = this.book.navigation;
          return this.book.locations.generate();
        })
        .then(() => {
          this.locations = this.book.locations;
          this.setProcess();
          this.onContentChange();
          this.bookAvailable = true;
        });
    },
    registerTheme() {
      this.themeList.forEach(theme => {
        this.themes.register(theme.name, theme.style);
      });
    },
    setTheme(index) {
      this.defaultTheme = index;
      this.themes.select(this.themeList[index].name);
    }
  },
  mounted() {
    this.showEpub();
  }
};
</script>

<style lang='scss' scoped>
@import "../assets/styles/global";
.ebook {
  position: relative;
  .read-wrapper {
    .mask {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 99;
      display: flex;
      width: 100%;
      height: 100%;
      .left {
        flex: 1;
      }
      .content {
        flex: 1;
      }
      .right {
        flex: 1;
      }
    }
  }
}
</style>