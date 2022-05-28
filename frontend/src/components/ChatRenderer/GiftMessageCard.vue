<template>
  <div id="card" class="style-scope yt-live-chat-paid-message-renderer">
    <div class="gift_left_gadget" :style="{ backgroundColor: `${shape2Color}` }"></div>
    <div class="gift_right_gadget">
      <img :src="bgBaseUrl + '/star.png'" alt="" />
    </div>
    <div
      class="w100p h100p gift_border1"
      :style="{ outline: `6px solid ${linearColor1}`, backgroundColor: `#FFF` }"
    >
      <div
        class="gift_border2"
        :style="{
          outline: `4px dashed ${secondColor}`,
          background: `linear-gradient(${linearColor1} 0%, 50%, ${linearColor2} 100%)`,
        }"
      >
        <div class="gift_avatar_wrapper">

          <div class="gift_avatar_container">
            <img class="gift_avatar" :src="avatarUrl" alt="" @error="onLoadError" />
            <div class="gift_avatar_border_container">
              <img class="gift_avatar_border" :src="bgBaseUrl + '/avatar.png'" alt="" />
            </div>
            <div class="gift_avatar_gadget">
              <img :src="bgBaseUrl + '/butterfly.png'" alt="" />
            </div>
          </div>
        </div>
        <div class="gift_main_container">
          <div class="gift_username">{{ authorName }}</div>
          <div class="gift_shape">
            <div
              class="w100p gift_shape1"
              :style="{
                backgroundColor: `${shape1Color}`,
                textShadow: `0 1px ${linearColor1}, 1px 0 ${linearColor1}, -1px 0 ${linearColor1}, 0 -1px ${linearColor1}`,
              }"
            >
              <span>赠送了</span>
            </div>
            <div
              class="gift_shape2"
              :style="{
                backgroundColor: `${shape2Color}`,
                textShadow: `0 1px ${linearColor1}, 1px 0 ${linearColor1}, -1px 0 ${linearColor1}, 0 -1px ${linearColor1}`,
              }"
            >
              {{ content }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as avatar from '@/api/chat/avatar'
export default {
  props: {
    linearColor1: String,
    linearColor2: String,
    secondColor: String,
    giftName: String,
    content: String,
    avatarUrl: String,
    authorName: String,
    shape1Color: String,
    shape2Color: String,
    bgBaseUrl: String,
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
}
</script>
