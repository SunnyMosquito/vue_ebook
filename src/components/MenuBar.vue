<template>
  <div class="menu-bar">
    <transition name="slide-up">
      <div class="menu-wrapper" :class="{'hide-box-shadow': ifSettingShow}" v-show="ifTitleAndMenuShow">
        <div class="icon-wrapper" @click="showContent">
          <span class="icon-menu icon"></span>
        </div>
        <div class="icon-wrapper" @click="toggleSetting(2)">
          <span class="icon-progress icon"></span>
        </div>
        <div class="icon-wrapper" @click="toggleSetting(1)">
          <span class="icon-bright icon"></span>
        </div>
        <div class="icon-wrapper" @click="toggleSetting(0)">
          <span class="icon-a icon">A</span>
        </div>
      </div>
    </transition>
    <transition name="slide-up">
      <div class="setting-wrapper" v-show="ifSettingShow">
        <div class="setting-font-size" v-if="tagShow === 0">
          <div class="preview" :style="{fontSize: fontSizeList[0].fontSize + 'px'}">A</div>
          <div class="select-wrapper" v-for="(item,index) in fontSizeList" :key="index" @click="setFontSize(item.fontSize)">
            <div class="line"></div>
            <div class="point-wrapper">
              <div class="point" v-show="defaultFontSize===item.fontSize"></div>
            </div>
            <div class="line"></div>
          </div>
          <div class="preview" :style="{fontSize: fontSizeList[fontSizeList.length-1].fontSize + 'px'}">A</div>
        </div>
        <div class="setting-theme" v-else-if="tagShow === 1">
          <div class="setting-theme-item" v-for="(item,index) in themeList" :key="index" @click="setTheme(index)">
            <div class="preview" :style="{background:item.style.body.background}" :class="{'border':item.style.body.background==='#ffffff'}"></div>
            <div class="text" :class="{'selected':index === defaultTheme}">{{item.name}}</div>
          </div>
        </div>
        <div class="setting-progress" v-else-if="tagShow === 2">
          <div class="progress-wrapper">
            <input class="progress" type="range" min="0" max="100" step="1" @change="onProgressChange($event.target.value)" @input="onProgressInput($event.target.value)" :value="progress" :disabled="!bookAvailable" ref="progressBar">
          </div>
          <div class="text-wrapper">
            <span>{{bookAvailable ? progress + '%' : '加载中...'}}</span>
          </div>
        </div>
      </div>
    </transition>
    <content-view :ifShowContent="ifShowContent" v-show="ifShowContent" :navigation="navigation" :currentContent="currentContent" :bookAvailable="bookAvailable" @jumpTo="jumpTo">
    </content-view>
    <transition name="fade">
      <div class="content-mask" v-show="ifShowContent" @click="hideContent"></div>
    </transition>
  </div>
</template>

<script>
import ContentView from "@/components/Content";

export default {
  data() {
    return {
      ifSettingShow: false,
      tagShow: 0,
      progress: 0,
      ifShowContent: false
    };
  },
  components: {
    ContentView
  },
  props: {
    ifTitleAndMenuShow: {
      type: Boolean,
      default: false
    },
    fontSizeList: Array,
    defaultFontSize: Number,
    defaultTheme: Number,
    bookAvailable: Boolean, //书是否解析完成
    themeList: Array, // 主题列表
    navigation: Object,
    currentContent: Object // 当前目录toc的索引
  },
  methods: {
    // 页面跳转
    jumpTo(href) {
      this.$emit("jumpTo", href);
    },
    // 隐藏目录
    hideContent() {
      this.ifShowContent = false;
    },
    // 显示目录
    showContent() {
      this.ifShowContent = true;
      this.hideSetting();
    },
    // 更改进度条背景
    onProgressInput(progress) {
      this.progress = progress;
      this.$refs.progressBar.style.backgroundSize = `${this.progress}% 100%`;
    },
    // 进度条被拖动更新页面
    onProgressChange(progress) {
      this.$emit("onProgressChange", progress);
    },
    // 显示设置栏
    showSetting() {
      this.ifSettingShow = true;
    },
    // 设置主题
    setTheme(index) {
      this.$emit("setTheme", index);
    },
    // 折叠设置栏
    toggleSetting(tagShow) {
      if (tagShow === this.tagShow) {
        this.ifSettingShow = !this.ifSettingShow;
      } else {
        this.tagShow = tagShow;
      }
    },
    // 隐藏设置栏
    hideSetting() {
      this.ifSettingShow = false;
    },
    // 设置字体大小
    setFontSize(fontSize) {
      this.$emit("setFontSize", fontSize);
    }
  }
};
</script>

<style lang='scss' scoped>
@import "../assets/styles/global";
.menu-bar {
  .menu-wrapper {
    z-index: 101;
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    height: px2rem(48);
    background: #ffffff;
    box-shadow: 0 px2rem(-8) px2rem(8) rgba(0, 0, 0, 0.15);
    display: flex;
    &.hide-box-shadow {
      box-shadow: none;
    }
    .icon-wrapper {
      flex: 1;
      @include center;
    }
  }
  .setting-wrapper {
    position: absolute;
    bottom: px2rem(48);
    left: 0;
    width: 100%;
    height: px2rem(60);
    background: #ffffff;
    box-shadow: 0 px2rem(-8) px2rem(8) rgba(0, 0, 0, 0.15);
    z-index: 100;
    .setting-theme {
      height: 100%;
      display: flex;
      .setting-theme-item {
        flex: 1;
        display: flex;
        flex-direction: column;
        padding: px2rem(5);
        box-sizing: border-box;
        .preview {
          flex: 1;
          &.border {
            border: px2rem(1) solid #000000;
          }
        }
        .text {
          flex: 0 0 px2rem(20);
          font-size: px2rem(16);
          color: #999999;
          @include center;
          &.selected {
            color: #000000;
          }
        }
      }
    }
    .setting-font-size {
      display: flex;
      height: 100%;
      .preview {
        flex: 0 0 px2rem(40);
        @include center;
      }
      .select-wrapper {
        flex: 1;
        display: flex;
        align-items: center;
        &:nth-child(2) {
          .line:first-child {
            border-top: none;
          }
        }
        &:nth-last-child(2) {
          .line:last-child {
            border-top: none;
          }
        }
        .line {
          flex: 1;
          height: 0;
          border-top: px2rem(1) solid #ccc;
        }
        .point-wrapper {
          position: relative;
          flex: 0 0 0;
          width: 0;
          height: px2rem(7);
          border-left: 1px solid #ccc;
          .point {
            position: absolute;
            top: px2rem(-8);
            left: px2rem(-8);
            width: px2rem(5);
            height: px2rem(5);
            border: px2rem(8) solid white;
            border-radius: 50%;
            box-shadow: 0 px2rem(2) px2rem(2) rgba(0, 0, 0, 0.15),
              0 px2rem(-2) px2rem(2) rgba(0, 0, 0, 0.15),
              px2rem(2) 0 px2rem(2) rgba(0, 0, 0, 0.15),
              px2rem(-2) 0 px2rem(2) rgba(0, 0, 0, 0.15);
            background: black;
          }
        }
      }
    }
    .setting-progress {
      position: relative;
      width: 100%;
      height: 100%;
      .progress-wrapper {
        width: 100%;
        height: 100%;
        @include center;
        padding: 0 px2rem(30);
        box-sizing: border-box;
        .progress {
          width: 100%;
          height: px2rem(2);
          -webkit-appearance: none;
          background: -webkit-linear-gradient(#999999, #999999) no-repeat
            #dddddd;
          background-size: 0 100%;
          &:focus {
            outline: none;
          }
          &::-webkit-slider-thumb {
            -webkit-appearance: none;
            height: px2rem(20);
            width: px2rem(20);
            border-radius: 50%;
            background: #ffffff;
            box-shadow: 0 px2rem(4) px2rem(4) rgba(0, 0, 0, 0.15);
            border: px2rem(1) solid #dddddd;
          }
        }
      }
      .text-wrapper {
        position: absolute;
        left: 0;
        bottom: 0;
        width: 100%;
        color: #333333;
        font-size: px2rem(22);
        text-align: center;
      }
    }
  }
  .content-mask {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 102;
    display: flex;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
  }
}
</style>