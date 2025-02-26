<template>
  <div
    class="msg-item"
    v-if="item.content"
    :class="{ 'msg-current': item.userName == current.userName }"
  >
    <div class="msg-avatar-box">
      <a target="_blank" :href="`https://pwl.icu/member/${item.userName}`">
        <span class="msg-avatar avatar"><img :src="item.userAvatarURL"></span>
        </a>
    </div>
    <div :ref="`msg-${item.oId}`" :data-id="item.oId" class="msg-item-contain">
      <div class="msg-user" :title="item.userName">
        {{ item.userNickname || item.userName }}
      </div>
      <RedpacketMsg :item="item" :isCurrent="item.userName == current.userName" @click="openRedpacket(item)"/>
      <div class="msg-contain" v-if="!item.redpacket">
        <div
          class="arrow"
          v-if="item.content.replace(/\n/g, '').match(/>[^<]+?</g)"
        />
        <div
          class="msg-content md-style"
          v-html="formatContent(item.content)"
          v-if="item.content.replace(/\n/g, '').match(/>[^<]+?</g)"
        />
        <span
          class="msg-img"
          v-if="!item.content.replace(/\n/g, '').match(/>[^<]+?</g)"
          v-html="formatContent(item.content)"
        ></span>
        <span class="plus-one" @click="followMsg(item)" v-if="plusOne">+1</span>
      </div>
    </div>
  </div>
</template>

<script>
import RedpacketMsg from './RedpacketMsg.vue';
export default {
  components: { RedpacketMsg },
  name: "MessageItem",
  props: {
    current: {
      type: Object,
      default() {
        return {};
      },
    },
    item: {
      type: Object,
      require: true,
    },
    plusOne: {
      type: Boolean,
      default() {
        return false;
      },
    },
  },
  methods: {
    getRedPacket(item) {
      try {
        let data = JSON.parse(item.content);
        if (data.msgType != "redPacket") return false;
        data.empty = item.empty || data.got == data.count;
        data.readed =
          item.readed ||
          data.who.find((w) => w.userName == this.current.userName);
        return data;
      } catch (e) {
        return false;
      }
    },
    formatContent(content) {
      return content
        .replace(/(<a )/g, '$1target="_blank" data-action="open-link"')
        .replace(/<img\s+src="([^"]*?)"([^>]*?>)/g, '<a href="$1" class="image-link" target="_blank"><img src="$1" data-action="preview" $2</a>');
    },
    async followMsg(item) {
      let raw = await this.$root.request("raw", item.oId);
      this.$root.request("push", raw);
    },
    async openRedpacket(item) {
      let rsp = await this.$root.request('openRedpacket', item.oId);
      if (!rsp) return;
      this.$emit('redpacket', rsp);
    },
  },
};
</script>

<style lang="less" scoped>
.msg-item {
  display: flex;
  flex-direction: row;
  margin: 5px 0;
}

.msg-item-contain {
  display: flex;
  flex-direction: column;
  max-width: 85%;
  position: relative;
  width: 100%;
}

.msg-avatar {
  width: 35px;
  height: 35px;
  border-radius: 35px;
  margin-top: 1.5em;
  cursor: pointer;
  img {
    min-width: 100%;
    max-width: 100%;
  }
}

.msg-user {
  margin-left: 1em;
  font-size: 1em;
  margin-bottom: 2px;
  min-height: 1em;
}

.msg-contain {
  display: flex;
  flex-direction: row;
  position: relative;
  max-width: 80vw;
  .msg-img {
    padding: 10px;
    display: inline-block;
  }
  .plus-one {
    font-size: 0.8em;
    margin: auto 5px;
    font-weight: bolder;
    color: #fff;
    height: 25px;
    width: 25px;
    background: #d23f31;
    border-radius: 15px;
    text-align: center;
    cursor: pointer;
    line-height: 23px;
    font-family: mononoki, Consolas, "Liberation Mono", Menlo, Courier,
      monospace;
  }
}

.msg-content {
  background-color: var(--vscode-scrollbarSlider-background);
  border-radius: 5px;
  padding: 8px 15px;
  color: var(--vscode-button-foreground);
  word-break: break-word;
  max-width: calc(100% - 45px);
  overflow: auto;
}
.msg-current {
  flex-direction: row-reverse;
  .msg-contain {
    flex-direction: row-reverse;
  }
  .arrow {
    border-right-color: transparent;
    border-left-color: var(--vscode-button-foreground);
  }
  .msg-content {
    background-color: var(--vscode-button-foreground);
    color: var(--vscode-scrollbarSlider-hoverBackground);
  }
  .msg-user {
    text-align: right;
    margin-right: 1em;
  }
  .plus-one {
    left: -1.5em;
    right: auto;
  }
}
.msg-menu {
  position: absolute;
  background: #fff;
  box-shadow: 1px 1px 3px #515a6e;
  border-radius: 5px;
  color: #17233d;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  z-index: 50;
  .msg-menu-item {
    padding: 5px 10px;
    word-break: keep-all;
    &:hover {
      background: #dcdee2;
    }
  }
}
.hidden {
  visibility: hidden;
  position: absolute;
}
</style>
<style lang="less">
.msg-img {
  img {
    max-width: 60vw;
  }
  [alt="图片表情"] {
    max-width: 150px;
  }
}
</style>