<template>
  <yt-live-chat-membership-item-renderer show-only-header  class="style-scope yt-live-chat-item-list-renderer"
    :privilegeType="privilegeType"
    :is-deleted="isDelete"
  >
    <div id="card">
      <div class="newmenber_text">
        NEW MEMBER
      </div>
      <div class="newmember_left_gadget" :style="{ backgroundColor: `${currOpt.shape2Color}` }"></div>
      <div class="newmember_right_gadget">
        <img :src="currOpt.baseUrl + '/logo.png'" alt="" />
      </div>
      <div
          class="w100p h100p newmember_border1"
          :style="{ outline: `6px solid ${currOpt.linearColor1}`, backgroundColor: `#FFF` }"
      >

        <div
            class="w100p h100p newmember_border2"
            :style="{
          outline: `4px dashed ${currOpt.secondColor}`,
          background: `linear-gradient(${currOpt.linearColor1} 0%, 50%, ${currOpt.linearColor2} 100%)`,
        }"
        >
          <div class="newmember_avatar_wrapper">

            <div class="newmember_avatar_container">
              <img class="newmember_avatar" :src="avatarUrl" alt="" @error="onLoadError" />
              <div class="newmember_avatar_border_container">
                <img class="newmember_avatar_border" :src="currOpt.baseUrl + '/tx.png'" alt="" />
              </div>
            </div>
          </div>
          <div class="newmember_main_container">
            <div class="newmember_username">{{ authorName }}</div>
            <div class="newmember_shape">
              <div
                  class="w100p newmember_shape1"
                  :style="{
                backgroundColor: `${currOpt.shape1Color}`
              }"
              >
              </div>
              <div
                  class="newmember_shape2"
                  :style="{
                backgroundColor: `${currOpt.shape2Color}`
              }"
              >
                新大航海成员
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </yt-live-chat-membership-item-renderer>
</template>

<script>
import ImgShadow from './ImgShadow'
import AuthorChip from './AuthorChip'
import * as utils from '@/utils'

export default {
  name: 'MembershipItem',
  components: {
    ImgShadow,
    AuthorChip
  },
  props: {
    avatarUrl: String,
    authorName: String,
    privilegeType: Number,
    title: String,
    time: Date,
    isDelete: Boolean
  },
  data(){
    return{
      currOpt:this.getMemberConfig(this.privilegeType),
      showImgUrl: this.avatarUrl,
    }
  },
  methods:{
    getMemberConfig(type){
      const opt= [
        {
          type:"舰长",
          linearColor1:"#a6c8ff",
          linearColor2:"#d7e4ff",
          secondColor:"#a6c8ff",
          shape1Color:"#829AFF",
          shape2Color:"#5B79FE",
          baseUrl:"/static/img/dhh/jz"
        },
        {
          type:"总督",
          linearColor1:"#ffc892",
          linearColor2:"#ffead3",
          secondColor:"#f7b96f",
          shape1Color:"#ffa882",
          shape2Color:"#fe8245",
          baseUrl:"/static/img/dhh/zd"
        },
        {
          type:"提督",
          linearColor1:"#caa6ff",
          linearColor2:"#f0d7ff",
          secondColor:"#be95fd",
          shape1Color:"#AB82FF",
          shape2Color:"#8D5BFE",
          baseUrl:"/static/img/dhh/td"
        },
        {
          type:"舰长",
          linearColor1:"#a6c8ff",
          linearColor2:"#d7e4ff",
          secondColor:"#a6c8ff",
          shape1Color:"#829AFF",
          shape2Color:"#5B79FE",
          baseUrl:"/static/img/dhh/jz"
        }
      ]

      return opt[type]
    },
    onLoadError() {
      if (this.showImgUrl !== avatar.DEFAULT_AVATAR_URL) {
        this.showImgUrl = avatar.DEFAULT_AVATAR_URL
      }
    },
  },
  computed: {
    timeText() {
      return utils.getTimeTextHourMin(this.time)
    }
  }
}
</script>

<style src="@/assets/css/youtube/yt-live-chat-membership-item-renderer.css"></style>
