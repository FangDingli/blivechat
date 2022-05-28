<template>
  <yt-live-chat-paid-message-renderer class="style-scope yt-live-chat-item-list-renderer" allow-animations :giftName="giftName"
  >
    <SCMessageCard
        v-if="giftName === 'superchat'"
        :bgBaseUrl="bgUrl"
        :avatarUrl="avatarUrl"
        :authorName="authorName"
        :content="content"
        :scPrimaryColor="scPrimaryColor"
        :scSecondColor="scSecondColor"
        :priceText="priceText"
    ></SCMessageCard>
    <GiftMessageCard
        v-else
        :linearColor1="price > 0 ? '#caa6ff' : '#a6c8ff'"
        :linearColor2="price > 0 ? '#f0d7ff' : '#d7e4ff'"
        :secondColor="price > 0 ? '#be95fd' : '#a6c8ff'"
        :shape1Color="price > 0 ? '#AB82FF' : '#829AFF'"
        :shape2Color="price > 0 ? '#8D5BFE' : '#5B79FE'"
        :giftName="giftName"
        :content="content"
        :avatarUrl="avatarUrl"
        :authorName="authorName"
        :bgBaseUrl="bgUrl"
    ></GiftMessageCard>
  </yt-live-chat-paid-message-renderer>
</template>

<script>
import ImgShadow from './ImgShadow'
import SCMessageCard from './SCMessageCard.vue'
import GiftMessageCard from './GiftMessageCard.vue'
import * as constants from './constants'
import * as utils from '@/utils'

export default {
  name: 'PaidMessage',
  data() {
    return {
      bgUrl: '',
      giftbgUrl: '',
      scPrimaryColor: '',
      scSecondColor: '',
    }
  },
  components: {
    ImgShadow,
    SCMessageCard,
    GiftMessageCard
  },
  props: {
    avatarUrl: String,
    authorName: String,
    giftName: String,
    price: Number, // 价格，人民币
    time: Date,
    content: String,
    isDelete: Boolean
  },
  created() {
    if (this.giftName == 'superchat') {
      const { imgFolder, primaryColor, secondColor } = constants.getPriceConfig(this.price)
      this.bgUrl = '/static/img/sc/' + imgFolder
      this.scPrimaryColor = primaryColor
      this.scSecondColor = secondColor
    } else {
      if (this.price) {
        this.bgUrl = '/static/img/gift/ff'
      } else {
        this.bgUrl = '/static/img/gift/mf'
      }
    }
  },
  computed: {
    priceRange() {
      return constants.getPriceConfig(this.price)
    },
    priceText() {
      let price_str = this.price > 0 ? ('CN¥' + utils.formatCurrency(this.price)) : '银瓜子礼物'
      return price_str
    },
    timeText() {
      return utils.getTimeTextHourMin(this.time)
    }
  }
}
</script>

<style src="@/assets/css/youtube/yt-live-chat-paid-message-renderer.css"></style>
