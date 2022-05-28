<template>
  <div id="card" class="style-scope yt-live-chat-paid-message-renderer">
    <!-- 为了方便后续改动，虽然SC发言框和普通发言框长得很像但还是把命名区别开 -->
    <div
      class="w100p h100p sc_border1"
      :style="{ outline: `6px solid ${scPrimaryColor}`, backgroundColor: `${scPrimaryColor}` }"
    >
      <div class="sc_left">
        <img :src="bgBaseUrl + '/circle_left.png'" alt="" />
      </div>
      <div class="split_block">
        <SplitBlock :bgc="scPrimaryColor"></SplitBlock>
      </div>
      <div class="w100p h100p sc_border2" :style="{ outline: `6px solid ${scSecondColor}` }">
        <div class="w100p h100p sc_border3" :style="{ outline: `4px dashed ${scSecondColor}` }">
          <div class="sc_avatar_wrapper">
            <div class="sc_avatar_container">
              <img class="sc_avatar" :src="avatarUrl" alt="" @error="onLoadError" />
              <div class="sc_avatar_border_container">
                <img class="sc_avatar_border" :src="bgBaseUrl + '/tx.png'" alt />
              </div>
              <div class="sc_avatar_gadget">
                <img :src="bgBaseUrl + '/butterfly.png'" alt="" />
              </div>
            </div>
          </div>

          <div class="sc_content_container">
            <div class="sc_content_top">
              <span
                class="sc_content_name"
                :style="{ borderBottom: `3px solid ${scPrimaryColor}` }"
              >
                {{ authorName }}
              </span>
              <span class="sc_money" :style="{ backgroundColor: scPrimaryColor }">
                <img :src="'/static/img/rabbit.png'" alt />
                {{ priceText }}
              </span>
            </div>
            <div class="sc_content_bottom">{{ content }}</div>
          </div>
        </div>
      </div>
      <!-- <div class="sc_right_top">
        <img :src="bgBaseUrl + '/star.png'" alt="" />
      </div>
      <div class="sc_right_bottom">
        <img :src="bgBaseUrl + '/b.png'" alt="" />
      </div> -->
    </div>
    <!-- <div class="sc_left">
      <img :src="bgBaseUrl + '/circle_left.png'" alt="" />
    </div>
    <div class="sc_avatar_wrapper">
      <div class="sc_avatar_container">
        <img class="sc_avatar" :src="avatarUrl" alt="" @error="onLoadError" />
        <div class="sc_avatar_border_container">
          <img class="sc_avatar_border" :src="bgBaseUrl + '/tx.png'" alt="" />
        </div>
      </div>
    </div>
    <div class="sc_content_container">
      <div class="sc_content_top">
        <div class="sc_content_name">
          {{ authorName }}
        </div>
        <div class="sc_money">
          <img class="sc_avatar_border" :src="bgBaseUrl + '/money.png'" alt="" />
        </div>
      </div>
      <div class="sc_content_bottom">{{ content }}</div>
    </div> -->
  </div>
</template>

<script>
import * as avatar from '@/api/chat/avatar'
import ImgShadow from './ImgShadow.vue'
import SplitBlock from './SplitBlock.vue'
export default {
  props: {
    bgBaseUrl: String,
    avatarUrl: String,
    authorName: String,
    content: String,
    scPrimaryColor: String,
    scSecondColor: String,
    priceText: String,
  },
  data() {
    return {
      showImgUrl: this.avatarUrl,
    }
  },
  methods: {
    onLoadError() {
      if (this.showImgUrl !== avatar.DEFAULT_AVATAR_URL) {
        this.showImgUrl = avatar.DEFAULT_AVATAR_URL
      }
    },
  },
  watch: {
    avatarUrl(val) {
      this.showImgUrl = val
    },
  },
  components: {
    ImgShadow,
    SplitBlock,
  },
}
</script>
