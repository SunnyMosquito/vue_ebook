<template>
  <transition name="slide-right">
    <div class="content">
      <div class="content-wrapper" v-if="bookAvailable">
        <div class="content-item" v-for="(item,index) in navigation.toc" :key="index">
          <span class="text" @click="jumpTo(item.href)" :class="{'selected-content':currentContent.tocIndex===index && currentContent.subtocIndex === null}">
            {{item.label}}
          </span>
          <div class="content-subitem" v-for="(subitem,subindex) in item.subitems" :key="subindex">
            <span class="text" @click="jumpTo(subitem.href)" :class="{'selected-content':currentContent.tocIndex===index && currentContent.subtocIndex === subindex}">
              {{subitem.label}}
            </span>
          </div>
        </div>
      </div>
      <div class="empty" v-else>加载中...</div>
    </div>
  </transition>
</template>

<script>
export default {
  props: {
    navigation: Object,
    bookAvailable: Boolean,
    currentContent: Object
  },
  methods: {
    jumpTo(href) {
      this.$emit("jumpTo", href);
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
@import "../assets/styles/global";
.content {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 103;
  width: 70%;
  height: 100%;
  background: white;
  .content-wrapper {
    width: 100%;
    height: 100%;
    overflow-y: auto;
    .content-item {
      width: 100%;
      padding: px2rem(2) px2rem(15);
      border-bottom: px2rem(1) solid #f4f4f4;
      box-sizing: border-box;
      line-height: 100%;

      .text {
        font-size: px2rem(14);
        color: #333;
      }
      .content-subitem {
        padding-left: px2rem(15);
        font-size: px2rem(12);
        width: 100%;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        span {
          font-size: px2rem(12);
        }
      }
      .selected-content {
        color: red;
      }
    }
  }
  .empty {
    width: 100%;
    height: 100%;
    @include center;
    font-size: px2rem(16);
    color: #333;
  }
}
</style>
