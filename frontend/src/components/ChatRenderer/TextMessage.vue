<template>
  <yt-live-chat-text-message-renderer
    :style="{'--repeated-text-color': randomColor}"
    :is-fan-group="isFanGroup"
    :medal-level="medalLevel"
    :author-type="authorTypeText"
    :privilegeType="privilegeType"
    :is-admin="authorType === 2"
    :is-owner="authorType === 3"
    :is-deleted="isDelete"
    >
    <div id="card" class="style-scope yt-live-chat-text-message-renderer">
      <div
          class="w100p h100p chat_border1"
          :style="{
          outline: `6px solid ${currOption.color1}`,
          backgroundColor: `${currOption.color1}`,
        }"
      >
        <div class="chat_left">
          <img :src="currOption.imgBaseUrl + '/circle_left.png'" alt="" />
        </div>
        <div class="chat_gadget_right_top">
          <img :src="currOption.imgBaseUrl + '/hd.png'" alt="" />
        </div>
        <div class="chat_gadget_right">
          <img :src="currOption.imgBaseUrl + '/circle_right.png'" alt="" />
        </div>
        <div class="split_block">
          <SplitBlock :bgc="currOption.color1"></SplitBlock>
        </div>
        <div
            class="w100p h100p chat_border2"
            :style="{ outline: `6px solid ${currOption.color2}` }"
        >
          <div
              class="w100p h100p chat_border3"
              :style="{ outline: `4px dashed ${currOption.color2}` }"
          >
            <div class="chat_avatar_wrapper">
              <div class="chat_avatar_container">
                <img class="chat_avatar" :src="avatarUrl" alt="" @error="onLoadError" />
                <div class="chat_avatar_border_container">
                  <img
                      class="chat_avatar_border"
                      :src="currOption.imgBaseUrl + '/avatar.png'"
                      alt
                  />
                </div>
                <div class="chat_avatar_gadget">
                  <img :src="currOption.imgBaseUrl + '/flower.png'" alt="" />
                </div>
                <div class="chat_avatar_gadget_bottom">
                  <img :src="currOption.imgBaseUrl + '/avatar_gadget.png'" alt="" />
                </div>
              </div>
            </div>

            <div class="chat_content_container">
              <div class="chat_content_top">
                <span
                    class="chat_content_name"
                    :style="{ borderBottom: `3px solid ${currOption.color1}` }"
                >
                  {{ authorName }}
                </span>
                <author-medal
                    class="chat_medal"
                    :medalLevel="medalLevel"
                    :medalName="medalName"
                    :isFanGroup="isFanGroup"
                ></author-medal>
              </div>
              <div class="chat_content_bottom">
                <template v-for="(richContent,index) in richContent">
                  <span :key="index" v-if="richContent.type === CONTENT_TYPE_TEXT">{{
                      richContent.text
                    }}</span>
                  <img
                      :key="index"
                      v-else-if="richContent.type === CONTENT_TYPE_EMOJI"
                      :src="richContent.url"
                      :alt="richContent.text"
                      :shared-tooltip-text="richContent.text"
                      :id="`emoji-${richContent.text}`"
                      :class="isOfficialEmotion(richContent.url)? 'uploader_emoticon' : 'official_emoticon'"
                  />
                  <span class="emoji_alt_text" v-if="richContent.type === CONTENT_TYPE_EMOJI">{{
                      content
                    }}</span>
                </template>
                <el-badge
                    :value="repeated"
                    :max="99"
                    v-if="repeated > 1"
                    class="style-scope yt-live-chat-text-message-renderer"
                    :style="{ '--repeated-mark-color': repeatedMarkColor }"
                ></el-badge>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </yt-live-chat-text-message-renderer>
</template>

<script>
import ImgShadow from './ImgShadow'
import AuthorMedal from './AuthorMedal'
import AuthorBadge from './AuthorBadge'
import * as constants from './constants'
import * as utils from '@/utils'
import SplitBlock from './SplitBlock.vue'

// HSL
const RANDOM_TEXT_COLOR_START = [0, 100.0, 55.0]
const RANDOM_TEXT_COLOR_END = [360, 60.0, 75.0]

const REPEATED_MARK_COLOR_START = [210, 100.0, 62.5]
const REPEATED_MARK_COLOR_END = [360, 87.3, 69.2]

export default {
  name: 'TextMessage',
  data() {
    return {
      CONTENT_TYPE_TEXT: constants.CONTENT_TYPE_TEXT,
      CONTENT_TYPE_IMAGE: constants.CONTENT_TYPE_IMAGE,
      CONTENT_TYPE_EMOJI: constants.CONTENT_TYPE_EMOJI,
      currOption: this.getAuthorOpt(this.privilegeType),
      showImgUrl: this.avatarUrl,
    }
  },
  methods:{
    isOfficialEmotion(url){
      if (url.indexOf('/bfs/garb')!=-1){
        return true
      }
      return  false
    },
    getAuthorOpt(num) {
      const opt = [
        {
          type: '观众',
          color1: '#f06363',
          color2: '#ffa4a4',
          imgBaseUrl: '/static/img/chat/gz',
        },
        {
          type: '总督',
          color1: '#a37fff',
          color2: '#c3a4fb',
          imgBaseUrl: '/static/img/chat/td',
        },
        {
          type: '提督',
          color1: '#a37fff',
          color2: '#c3a4fb',
          imgBaseUrl: '/static/img/chat/td',
        },
        {
          type: '舰长',
          color1: '#7d9de6',
          color2: '#a6c8ff',
          imgBaseUrl: '/static/img/chat/jz',
        },
      ]

      return opt[num]
    },
    onLoadError() {
      if (this.showImgUrl !== avatar.DEFAULT_AVATAR_URL) {
        this.showImgUrl = avatar.DEFAULT_AVATAR_URL
      }
    },
  },
  components: {
    ImgShadow,
    AuthorMedal,
    AuthorBadge,
    SplitBlock
  },
  props: {
    avatarUrl: String,
    time: Date,
    authorName: String,
    authorType: Number,
    medalName: String,
    medalLevel: Number,
    isFanGroup: Boolean,
    isDelete: Boolean,
    emoticon: String,
    content: String,
    richContent: Array,
    privilegeType: Number,
    repeated: Number,
    imageShowType: Number,
    maxImage: Number,
    maxEmoji: Number
  },
  computed: {
    timeText() {
      return utils.getTimeTextHourMin(this.time)
    },
    authorTypeText() {
      // 优先判断舰长
      return this.privilegeType > 0 ? 'member' : constants.AUTHOR_TYPE_TO_TEXT[this.authorType]
    },
    randomColor() {
      let color = [0, 0, 0]
      let t = Math.random()
      for (let i = 0; i < 3; i++) {
        color[i] = RANDOM_TEXT_COLOR_START[i] + ((RANDOM_TEXT_COLOR_END[i] - REPEATED_MARK_COLOR_START[i]) * t)
      }
      return `hsl(${color[0]}, ${color[1]}%, ${color[2]}%)`
    },
    repeatedMarkColor() {
      let color
      if (this.repeated <= 2) {
        color = REPEATED_MARK_COLOR_START
      } else if (this.repeated >= 10) {
        color = REPEATED_MARK_COLOR_END
      } else {
        color = [0, 0, 0]
        let t = (this.repeated - 2) / (10 - 2)
        for (let i = 0; i < 3; i++) {
          color[i] = REPEATED_MARK_COLOR_START[i] + ((REPEATED_MARK_COLOR_END[i] - REPEATED_MARK_COLOR_START[i]) * t)
        }
      }
      return `hsl(${color[0]}, ${color[1]}%, ${color[2]}%)`
    }
  }
}
</script>

<style>
yt-live-chat-text-message-renderer>#content .el-badge {
  margin-left: 10px;
}

yt-live-chat-text-message-renderer>#content .el-badge .el-badge__content {
  font-size: 12px !important;
  line-height: 18px !important;
  text-shadow: none !important;
  font-family: sans-serif !important;
  color: #FFF !important;
  background-color: var(--repeated-mark-color) !important;
  border: none;
}

</style>

<style src="@/assets/css/youtube/yt-live-chat-text-message-renderer.css"></style>
