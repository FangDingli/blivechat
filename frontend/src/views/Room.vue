<template>
  <chat-renderer ref="renderer"
  :showGiftInfo="config.showGiftInfo"
  :danmakuAtBottom="config.danmakuAtBottom"
  :tickerAtButtom="config.tickerAtButtom"
  :randomXOffset="config.randomXOffset"
  :randomYOffset="config.randomYOffset"
  :floatTime="config.floatTime"
  :mergeSameUserDanmakuInterval="config.mergeSameUserDanmakuInterval"
  :mergeSameUserDanmaku="config.mergeSameUserDanmaku"
  :minGiftPrice="config.minGiftPrice"
  :minTickerPrice="config.minTickerPrice"
  :maxNumber="config.maxNumber"
  :fadeOutNum="config.fadeOutNum"
  :pinTime="config.pinTime"
  >
  </chat-renderer>
</template>

<script>
import * as i18n from '@/i18n'
import { mergeConfig, toBool, toInt, toFloat } from '@/utils'
import * as trie from '@/utils/trie'
import * as pronunciation from '@/utils/pronunciation'
import * as chatConfig from '@/api/chatConfig'
import ChatClientTest from '@/api/chat/ChatClientTest'
import ChatClientDirect from '@/api/chat/ChatClientDirect'
import ChatClientRelay from '@/api/chat/ChatClientRelay'
import ChatRenderer from '@/components/ChatRenderer'
import * as constants from '@/components/ChatRenderer/constants'
import axios from 'axios'

export default {
  name: 'Room',
  components: {
    ChatRenderer
  },
  props: {
    roomId: {
      type: Number,
      default: null
    },
    strConfig: {
      type: Object,
      default: () => ({})
    },
    uidColorMap: {
      type: Object,
      default: () => ({})
    },
  },
  data() {
    return {
      config: chatConfig.deepCloneDefaultConfig(),
      chatClient: null,
      pronunciationConverter: null,
      danmu_pic_json: []
    }
  },
  computed: {
    blockKeywordsTrie() {
      let blockKeywords = this.config.blockKeywords.split('\n')
      let res = new trie.Trie()
      for (let keyword of blockKeywords) {
        if (keyword !== '') {
          res.set(keyword, true)
        }
      }
      return res
    },
    blockUsersTrie() {
      let blockUsers = this.config.blockUsers.split('\n')
      let res = new trie.Trie()
      for (let user of blockUsers) {
        if (user !== '') {
          res.set(user, true)
        }
      }
      return res
    },
    blockUsersByKeywordsTrie() {
      let blockUsersByKeywords = this.config.blockUsersByKeywords.split('\n')
      let res = new trie.Trie()
      for (let user of blockUsersByKeywords) {
        if (user !== '') {
          res.set(user, true)
        }
      }
      return res
    },
    // 解析用户设置的 emoticons
    emoticonsTrie() {
      let res = new trie.Trie()
      let danmu_emoticons

      if (this.config.useLocalEmoticonSetting) {
        // 使用本地json设置
        console.log("使用本地 json 文件设置表情包")
        danmu_emoticons = this.danmu_pic_json
      } else {
        // 使用网页设置
        danmu_emoticons = this.config.emoticons
      }

      for (let emoticon of danmu_emoticons) {
        // 1个个添加 emoticon
        if (emoticon.keyword !== '' && emoticon.align !== '' && emoticon.height !== '' && emoticon.url !== '') {
          res.set(emoticon.keyword, emoticon)
        }
      }
      return res
    }
  },
  mounted() {
    this.initConfig()
    this.initChatClient()
    if (this.config.giftUsernamePronunciation !== '') {
      this.pronunciationConverter = new pronunciation.PronunciationConverter()
      this.pronunciationConverter.loadDict(this.config.giftUsernamePronunciation)
    }
    // 在页面刷新缓存时, 读取用户emoticons.json, 并建立表情包库
    axios.get('/emoticons.json')
      .then(res => {
        if (res && res.data) {
          this.danmu_pic_json = res.data
        }

      // console.log(this.danmu_pic_json)
      })

    // 提示用户已加载
    this.$message({
      message: 'Loaded',
      duration: '500'
    })
  },
  beforeDestroy() {
    if (this.chatClient) {
      this.chatClient.stop()
    }
  },
  methods: {
    initConfig() {
      let locale = this.strConfig.lang
      if (locale) {
        i18n.setLocale(locale)
      }

      //* {} 内留空的使用上次预设值
      let cfg = { ...chatConfig.getLocalConfig() }
      for (let i in this.strConfig) {
        if (this.strConfig[i] !== '') {
          cfg[i] = this.strConfig[i]
        }
      }
      //* 若上次预设值有留空，则使用默认值
      // cfg = mergeConfig(cfg, chatConfig.DEFAULT_CONFIG)
      cfg = mergeConfig(cfg, chatConfig.deepCloneDefaultConfig())

      cfg.minGiftPrice = toFloat(cfg.minGiftPrice, chatConfig.DEFAULT_CONFIG.minGiftPrice)
      cfg.minTickerPrice = toFloat(cfg.minTickerPrice, chatConfig.DEFAULT_CONFIG.minTickerPrice)

      cfg.showDanmaku = toBool(cfg.showDanmaku)
      cfg.showInteractWordEnter = toBool(cfg.showInteractWordEnter)
      cfg.showInteractWordFollow = toBool(cfg.showInteractWordFollow)
      cfg.showInteractWordShare = toBool(cfg.showInteractWordShare)
      cfg.showSuperchat = toBool(cfg.showSuperchat)
      cfg.showNewMember = toBool(cfg.showNewMember)
      cfg.showGift = toBool(cfg.showGift)
      cfg.showGiftInfo = toBool(cfg.showGiftInfo)

      cfg.mergeSameUserDanmaku = toBool(cfg.mergeSameUserDanmaku)
      cfg.mergeSameUserDanmakuInterval = toInt(cfg.mergeSameUserDanmakuInterval)
      cfg.mergeSimilarDanmaku = toBool(cfg.mergeSimilarDanmaku)
      cfg.mergeGift = toBool(cfg.mergeGift)

      cfg.danmakuAtBottom = toBool(cfg.danmakuAtBottom)
      cfg.tickerAtButtom = toBool(cfg.tickerAtButtom)

      cfg.randomXOffset = toBool(cfg.randomXOffset)
      cfg.randomXRangeMin = toInt(cfg.randomXRangeMin)
      cfg.randomXRangeMax = toInt(cfg.randomXRangeMax)
      cfg.randomYOffset = toBool(cfg.randomYOffset)
      cfg.randomYRangeMin = toInt(cfg.randomYRangeMin)
      cfg.randomYRangeMax = toInt(cfg.randomYRangeMax)
      cfg.randomXInitialOffset = toInt(cfg.randomXInitialOffset)
      cfg.randomYInitialOffset = toInt(cfg.randomYInitialOffset)
      cfg.floatDistanceXMin = toInt(cfg.floatDistanceXMin)
      cfg.floatDistanceXMax = toInt(cfg.floatDistanceXMax)
      cfg.floatDistanceYMin = toInt(cfg.floatDistanceYMin)
      cfg.floatDistanceYMax = toInt(cfg.floatDistanceYMax)
      cfg.floatDistanceThreshold = toInt(cfg.floatDistanceThreshold)
      cfg.floatTime = toInt(cfg.floatTime)
      cfg.allowTextColorSetting = toBool(cfg.allowTextColorSetting)

      cfg.blockTranslateDanmaku = toBool(cfg.blockTranslateDanmaku)
      cfg.showTranslateDanmakuOnly = toBool(cfg.showTranslateDanmakuOnly)
      cfg.translationSign = cfg.translationSign.toString()

      cfg.emoticons = this.toObjIfJson(cfg.emoticons)
      cfg.useLocalEmoticonSetting = toBool(cfg.useLocalEmoticonSetting)
      cfg.autoRenderOfficialSmallEmoji = toBool(cfg.autoRenderOfficialSmallEmoji)
      cfg.autoRenderOfficialGeneralEmoji = toBool(cfg.autoRenderOfficialGeneralEmoji)
      cfg.autoRenderStreamerEmoji = toBool(cfg.autoRenderStreamerEmoji)
      cfg.autoRenderPersonalEmoji = toBool(cfg.autoRenderPersonalEmoji)

      cfg.isGreedyMatch = toBool(cfg.isGreedyMatch)
      cfg.isSkipSameImage = toBool(cfg.isSkipSameImage)
      cfg.maxImage = toInt(cfg.maxImage, chatConfig.DEFAULT_CONFIG.maxImage)
      cfg.maxEmoji = toInt(cfg.maxEmoji, chatConfig.DEFAULT_CONFIG.maxEmoji)

      cfg.maxNumber = toInt(cfg.maxNumber, chatConfig.DEFAULT_CONFIG.maxNumber)
      cfg.fadeOutNum = toInt(cfg.fadeOutNum, chatConfig.DEFAULT_CONFIG.fadeOutNum)
      cfg.pinTime = toInt(cfg.pinTime, chatConfig.DEFAULT_CONFIG.pinTime)
      cfg.imageShowType = toInt(cfg.imageShowType, chatConfig.DEFAULT_CONFIG.imageShowType)

      cfg.blockGiftDanmaku = toBool(cfg.blockGiftDanmaku)
      cfg.blockLevel = toInt(cfg.blockLevel, chatConfig.DEFAULT_CONFIG.blockLevel)
      cfg.blockNewbie = toBool(cfg.blockNewbie)
      cfg.blockNotMobileVerified = toBool(cfg.blockNotMobileVerified)
      cfg.blockMedalLevel = toInt(cfg.blockMedalLevel, chatConfig.DEFAULT_CONFIG.blockMedalLevel)

      cfg.relayMessagesByServer = toBool(cfg.relayMessagesByServer)
      cfg.autoTranslate = toBool(cfg.autoTranslate)

      cfg.minDanmakuInterval = toInt(cfg.minDanmakuInterval, chatConfig.DEFAULT_CONFIG.minDanmakuInterval)
      cfg.maxDanmakuInterval = toInt(cfg.maxDanmakuInterval, chatConfig.DEFAULT_CONFIG.maxDanmakuInterval)

      chatConfig.sanitizeConfig(cfg)
      this.config = cfg
    },
    toObjIfJson(str) {
      if (typeof str !== 'string') {
        return str
      }
      try {
        return JSON.parse(str)
      } catch {
        return {}
      }
    },
    initChatClient() {
      if (this.roomId === null) {
        this.chatClient = new ChatClientTest(this.config.minDanmakuInterval, this.config.maxDanmakuInterval)
      } else {
        if (!this.config.relayMessagesByServer) {
          this.chatClient = new ChatClientDirect(this.roomId)
        } else {
          this.chatClient = new ChatClientRelay(this.roomId, this.config.autoTranslate)
        }
      }
      this.chatClient.onAddText = this.onAddText
      this.chatClient.onAddGift = this.onAddGift
      this.chatClient.onAddMember = this.onAddMember
      this.chatClient.onAddSuperChat = this.onAddSuperChat
      this.chatClient.onDelSuperChat = this.onDelSuperChat
      this.chatClient.onUpdateTranslation = this.onUpdateTranslation
      this.chatClient.onInteractWord = this.onInteractWord // chatClient 定义来自 frontend\src\api\chat\ChatClientDirect\index.js
      this.chatClient.start()
    },

    start() {
      this.chatClient.start()
    },
    stop() {
      this.chatClient.stop()
    },

    // TODO: 前端显示欢迎入场
    onInteractWord(data) {
      // console.log(`${data.authorName} 进入房间，data 是 ${JSON.stringify(data, null, 4)}`)

      // NOTE: 判断不同的 Interact 是否显示（进入房间、关注房间、分享房间）
      if (data.msgType === constants.INTERACT_TYPE_ENTER) {
        if (!this.config.showInteractWordEnter) {
          return
        }
      } else if (data.msgType === constants.INTERACT_TYPE_FOLLOW || data.msgType === constants.INTERACT_TYPE_SPECIAL_FOLLOW || data.msgType === constants.INTERACT_TYPE_MUTUAL_FOLLOW) {
        if (!this.config.showInteractWordFollow) {
          return
        }
      } else if (data.msgType === constants.INTERACT_TYPE_SHARE) {
        if (!this.config.showInteractWordShare) {
          return
        }
      } else {
        // 不满足指定类型的互动
        return
      }

      if (!this.filterInteractMessage(data)) {
        return
      }

      let xOffset = this.config.randomXRangeMin + Math.floor(Math.random() * (this.config.randomXRangeMax - this.config.randomXRangeMin + 1))
      let yOffset = this.config.randomYRangeMin + Math.floor(Math.random() * (this.config.randomYRangeMax - this.config.randomYRangeMin + 1))

      if (this.config.randomXOffset ^ this.config.randomYOffset) {
        if (this.config.randomXOffset) {
          yOffset = this.config.randomYInitialOffset
        } else if (this.config.randomYOffset) {
          xOffset = this.config.randomXInitialOffset
        }
      }

      let floatDistanceX = this.config.floatDistanceXMin + Math.floor(Math.random() * (this.config.floatDistanceXMax - this.config.floatDistanceXMin + 1))
      if (Math.abs(floatDistanceX) < this.config.floatDistanceThreshold) {
        if (floatDistanceX < 0) {
          floatDistanceX = -this.config.floatDistanceThreshold
        } else {
          floatDistanceX = this.config.floatDistanceThreshold
        }
      }
      let floatDistanceY = this.config.floatDistanceYMin + Math.floor(Math.random() * (this.config.floatDistanceYMax - this.config.floatDistanceYMin + 1))
      if (Math.abs(floatDistanceY) < this.config.floatDistanceThreshold) {
        if (floatDistanceY < 0) {
          floatDistanceY = -this.config.floatDistanceThreshold
        } else {
          floatDistanceY = this.config.floatDistanceThreshold
        }
      }

      let message = {
        id: data.id,
        type: constants.MESSAGE_TYPE_INTERACT,
        avatarUrl: data.avatarUrl,
        time: new Date(data.timestamp * 1000),
        msgType: data.msgType,
        authorName: data.authorName,
        authorNamePronunciation: this.getPronunciation(data.authorName),

        medalName: data.medalName,
        medalLevel: data.medalLevel,
        isFanGroup: data.isFanGroup,

        privilegeType: data.privilegeType,

        xOffset: xOffset,
        yOffset: yOffset,
        floatDistanceX: floatDistanceX,
        floatDistanceY: floatDistanceY,
      }


      this.$refs.renderer.addMessage(message)
    },
    async onAddText(data) {

      // 匹配 #Hex 的正则表达式
      let textColor = 'initial'
      if (this.config.allowTextColorSetting) {
        if (constants.UID_COLOR_MAP_REGEX.test(data.content)) {
          this.uidColorMap[data.authorName] = data.content
          textColor = data.content
          // console.log(data.authorName + ":匹配到Hex颜色" + textColor)
        } else {
          if (this.uidColorMap[data.authorName] !== undefined) {
            textColor = this.uidColorMap[data.authorName]
          }
          // console.log(data.authorName + ":没匹配到Hex颜色" + textColor)
        }
      }

      if (!this.config.showDanmaku || !this.filterTextMessage(data)) {
        return
      }
      // 合并相似弹幕
      if (await this.mergeSimilarText(data.content)) {
        return
      }
      // 合并同一用户短期内的发言
      if (await this.mergeSameUserText(data.content, this.getRichContent(data), data.authorName, data.timestamp)) {
        // console.log("收到同一个 User 发送的消息")
        // 合并消息，即插入到 Thread 的消息需要单独写平滑（拉了，和原本的平滑方案没有很好的融合）
        this.$refs.renderer.calculateHeight()
        await this.$refs.renderer.$nextTick()
        this.$refs.renderer.showNewMessages()
        return
      }

      if (this.config.showTranslateDanmakuOnly) {
        let content_str = data.content
        if (content_str.charAt(0) !== this.config.translationSign) {
          // console.log("只显示以“"+ this.config.translationSign +"”开头的翻译弹幕")
          return
        } else {
          data.content = content_str.substring(1)
        }
      }

      if (this.config.blockTranslateDanmaku) {
        let content_str = data.content
        if (content_str.charAt(0) === this.config.translationSign) {
          return
        }
      }


      // 不是同一个user的消息的话，开启新的 thread
      // 拉了，为了减少对 key 的修改
      // 直接把Thread塞到原本的 content 和 richContent
      let contentThread = []
      contentThread[0] = data.content
      let richContentThread = []
      richContentThread[0] = this.getRichContent(data)
      let translationThread = []
      translationThread[0] = data.translation
      let xOffset = this.config.randomXRangeMin + Math.floor(Math.random() * (this.config.randomXRangeMax - this.config.randomXRangeMin + 1))
      let yOffset = this.config.randomYRangeMin + Math.floor(Math.random() * (this.config.randomYRangeMax - this.config.randomYRangeMin + 1))

      if (this.config.randomXOffset ^ this.config.randomYOffset) {
        if (this.config.randomXOffset) {
          yOffset = this.config.randomYInitialOffset
        } else if (this.config.randomYOffset) {
          xOffset = this.config.randomXInitialOffset
        }
      }

      let floatDistanceX = this.config.floatDistanceXMin + Math.floor(Math.random() * (this.config.floatDistanceXMax - this.config.floatDistanceXMin + 1))
      if (Math.abs(floatDistanceX) < this.config.floatDistanceThreshold) {
        if (floatDistanceX < 0) {
          floatDistanceX = -this.config.floatDistanceThreshold
        } else {
          floatDistanceX = this.config.floatDistanceThreshold
        }
      }
      let floatDistanceY = this.config.floatDistanceYMin + Math.floor(Math.random() * (this.config.floatDistanceYMax - this.config.floatDistanceYMin + 1))
      if (Math.abs(floatDistanceY) < this.config.floatDistanceThreshold) {
        if (floatDistanceY < 0) {
          floatDistanceY = -this.config.floatDistanceThreshold
        } else {
          floatDistanceY = this.config.floatDistanceThreshold
        }
      }

      // FIXME: thread 中的翻译文本

      let message = {
        id: data.id,
        type: constants.MESSAGE_TYPE_TEXT,
        avatarUrl: data.avatarUrl,
        time: new Date(data.timestamp * 1000),
        authorName: data.authorName,
        authorType: data.authorType,
        content: data.content,
        contents: contentThread,
        // richContent: this.getRichContent(data),
        richContents: richContentThread,
        privilegeType: data.privilegeType,
        medalName: data.medalName,
        medalLevel: data.medalLevel,
        isFanGroup: data.isFanGroup,
        repeated: 1,
        repeatedThread: [1],
        threadLength: 1,
        translation: data.translation,
        xOffset: xOffset,
        yOffset: yOffset,
        floatDistanceX: floatDistanceX,
        floatDistanceY: floatDistanceY,
        textColor: textColor
      }
      this.$refs.renderer.addMessage(message)
    },
    onAddGift(data) {
      if (!this.config.showGift) {
        // console.log("收到礼物，但是否显示礼物为" + this.config.showGift)
        return
      }
      if (this.config.showTranslateDanmakuOnly) {
        // console.log("只显示以“"+ this.config.translationSign +"”开头的翻译弹幕")
        return
      }

      let price = data.coinType == 'gold' ? data.totalCoin / 1000 : 0
      if (this.mergeSimilarGift(data.authorName, price, data.giftName, data.num)) {
        return
      }
      // 银瓜子礼物不丢人
      // if (price < this.config.minGiftPrice) {
      //  return
      // }

      let xOffset = this.config.randomXRangeMin + Math.floor(Math.random() * (this.config.randomXRangeMax - this.config.randomXRangeMin + 1))
      let yOffset = this.config.randomYRangeMin + Math.floor(Math.random() * (this.config.randomYRangeMax - this.config.randomYRangeMin + 1))

      if (this.config.randomXOffset ^ this.config.randomYOffset) {
        if (this.config.randomXOffset) {
          yOffset = this.config.randomYInitialOffset
        } else if (this.config.randomYOffset) {
          xOffset = this.config.randomXInitialOffset
        }
      }

      let floatDistanceX = this.config.floatDistanceXMin + Math.floor(Math.random() * (this.config.floatDistanceXMax - this.config.floatDistanceXMin + 1))
      if (Math.abs(floatDistanceX) < this.config.floatDistanceThreshold) {
        if (floatDistanceX < 0) {
          floatDistanceX = -this.config.floatDistanceThreshold
        } else {
          floatDistanceX = this.config.floatDistanceThreshold
        }
      }
      let floatDistanceY = this.config.floatDistanceYMin + Math.floor(Math.random() * (this.config.floatDistanceYMax - this.config.floatDistanceYMin + 1))
      if (Math.abs(floatDistanceY) < this.config.floatDistanceThreshold) {
        if (floatDistanceY < 0) {
          floatDistanceY = -this.config.floatDistanceThreshold
        } else {
          floatDistanceY = this.config.floatDistanceThreshold
        }
      }

      let message = {
        id: data.id,
        type: constants.MESSAGE_TYPE_GIFT,
        avatarUrl: data.avatarUrl,
        time: new Date(data.timestamp * 1000),
        authorName: data.authorName,
        authorNamePronunciation: this.getPronunciation(data.authorName),
        price: price,
        giftName: data.giftName,
        num: data.num,
        xOffset: xOffset,
        yOffset: yOffset,
        floatDistanceX: floatDistanceX,
        floatDistanceY: floatDistanceY,
      }
      this.$refs.renderer.addMessage(message)
    },
    onAddMember(data) {
      if (!this.config.showNewMember || !this.filterNewMemberMessage(data)) {
        // console.log("收到上舰，但是否显示上舰信息为" + this.config.showNewMember)
        return
      }
      if (this.config.showTranslateDanmakuOnly) {
        // console.log("只显示以“"+ this.config.translationSign +"”开头的翻译弹幕")
        return
      }
      let price = 198
      if (data.privilegeType == 2) {
        price = 1998
      } else if (data.privilegeType == 3) {
        price = 19998
      }
      let xOffset = this.config.randomXRangeMin + Math.floor(Math.random() * (this.config.randomXRangeMax - this.config.randomXRangeMin + 1))
      let yOffset = this.config.randomYRangeMin + Math.floor(Math.random() * (this.config.randomYRangeMax - this.config.randomYRangeMin + 1))

      if (this.config.randomXOffset ^ this.config.randomYOffset) {
        if (this.config.randomXOffset) {
          yOffset = this.config.randomYInitialOffset
        } else if (this.config.randomYOffset) {
          xOffset = this.config.randomXInitialOffset
        }
      }

      let floatDistanceX = this.config.floatDistanceXMin + Math.floor(Math.random() * (this.config.floatDistanceXMax - this.config.floatDistanceXMin + 1))
      if (Math.abs(floatDistanceX) < this.config.floatDistanceThreshold) {
        if (floatDistanceX < 0) {
          floatDistanceX = -this.config.floatDistanceThreshold
        } else {
          floatDistanceX = this.config.floatDistanceThreshold
        }
      }
      let floatDistanceY = this.config.floatDistanceYMin + Math.floor(Math.random() * (this.config.floatDistanceYMax - this.config.floatDistanceYMin + 1))
      if (Math.abs(floatDistanceY) < this.config.floatDistanceThreshold) {
        if (floatDistanceY < 0) {
          floatDistanceY = -this.config.floatDistanceThreshold
        } else {
          floatDistanceY = this.config.floatDistanceThreshold
        }
      }

      let message = {
        id: data.id,
        type: constants.MESSAGE_TYPE_MEMBER,
        avatarUrl: data.avatarUrl,
        time: new Date(data.timestamp * 1000),
        authorName: data.authorName,
        authorNamePronunciation: this.getPronunciation(data.authorName),
        privilegeType: data.privilegeType,
        price: price,
        title: this.$t('chat.membershipTitle'),
        xOffset: xOffset,
        yOffset: yOffset,
        floatDistanceX: floatDistanceX,
        floatDistanceY: floatDistanceY,
      }
      this.$refs.renderer.addMessage(message)
    },
    onAddSuperChat(data) {
      if (!this.config.showSuperchat || !this.filterSuperChatMessage(data)) {
        // console.log("收到打赏(醒目留言SC)，但显示打赏(醒目留言SC)为" + this.config.showSuperchat)
        return
      }
      if (data.price < this.config.minGiftPrice) { // 丢人
        // console.log("打赏小于最低打赏金额，不以显示")
        return
      }
      if (this.config.showTranslateDanmakuOnly) {
        // console.log("只显示以“"+ this.config.translationSign +"”开头的翻译弹幕")
        return
      }

      let xOffset = this.config.randomXRangeMin + Math.floor(Math.random() * (this.config.randomXRangeMax - this.config.randomXRangeMin + 1))
      let yOffset = this.config.randomYRangeMin + Math.floor(Math.random() * (this.config.randomYRangeMax - this.config.randomYRangeMin + 1))

      if (this.config.randomXOffset ^ this.config.randomYOffset) {
        if (this.config.randomXOffset) {
          yOffset = this.config.randomYInitialOffset
        } else if (this.config.randomYOffset) {
          xOffset = this.config.randomXInitialOffset
        }
      }

      let floatDistanceX = this.config.floatDistanceXMin + Math.floor(Math.random() * (this.config.floatDistanceXMax - this.config.floatDistanceXMin + 1))
      if (Math.abs(floatDistanceX) < this.config.floatDistanceThreshold) {
        if (floatDistanceX < 0) {
          floatDistanceX = -this.config.floatDistanceThreshold
        } else {
          floatDistanceX = this.config.floatDistanceThreshold
        }
      }
      let floatDistanceY = this.config.floatDistanceYMin + Math.floor(Math.random() * (this.config.floatDistanceYMax - this.config.floatDistanceYMin + 1))
      if (Math.abs(floatDistanceY) < this.config.floatDistanceThreshold) {
        if (floatDistanceY < 0) {
          floatDistanceY = -this.config.floatDistanceThreshold
        } else {
          floatDistanceY = this.config.floatDistanceThreshold
        }
      }

      let message = {
        id: data.id,
        type: constants.MESSAGE_TYPE_SUPER_CHAT,
        avatarUrl: data.avatarUrl,
        authorName: data.authorName,
        authorNamePronunciation: this.getPronunciation(data.authorName),
        price: data.price,
        time: new Date(data.timestamp * 1000),
        content: data.content.trim(),
        translation: data.translation,
        xOffset: xOffset,
        yOffset: yOffset,
        floatDistanceX: floatDistanceX,
        floatDistanceY: floatDistanceY,
      }
      this.$refs.renderer.addMessage(message)
    },
    onDelSuperChat(data) {
      this.$refs.renderer.delMessages(data.ids)
    },
    // FIXME: 更新翻译
    onUpdateTranslation(data) {
      if (!this.config.autoTranslate) {
        return
      }
      this.$refs.renderer.updateMessage(data.id, { translation: data.translation })
    },

    filterInteractMessage(data) {
      if (this.config.blockMedalLevel > 0) {
        // 如果未佩戴当前直播间勋章，或者勋章等级低于屏蔽等级，屏蔽信息
        if (data.medalLevel < this.config.blockMedalLevel || !data.isFanGroup) {
          return false
        }
      }

      return this.filterByAuthorName(data.authorName)
    },
    filterTextMessage(data) {
      if (this.config.blockGiftDanmaku && data.isGiftDanmaku) {
        return false
      } else if (this.config.blockLevel > 0 && data.authorLevel < this.config.blockLevel) {
        return false
      } else if (this.config.blockNewbie && data.isNewbie) {
        return false
      } else if (this.config.blockNotMobileVerified && !data.isMobileVerified) {
        return false
      } else if (this.config.blockMedalLevel > 0 && data.medalLevel < this.config.blockMedalLevel) {
        return false
      }
      return this.filterByContent(data.content) && this.filterByAuthorName(data.authorName)
    },
    filterSuperChatMessage(data) {
      return this.filterByContent(data.content) && this.filterByAuthorName(data.authorName)
    },
    filterNewMemberMessage(data) {
      return this.filterByAuthorName(data.authorName)
    },
    filterByContent(content) {
      let blockKeywordsTrie = this.blockKeywordsTrie
      for (let i = 0; i < content.length; i++) {
        let remainContent = content.substring(i)
        if (blockKeywordsTrie.nonGreedyMatch(remainContent) !== null) {
          return false
        }
      }
      return true
    },
    // NOTE: 根据关键词屏蔽用户（当用户名中包含关键词，如【人气】时候，屏蔽）
    filterByAuthorNameKeywords(authorName) {
      let blockUsersByKeywordsTrie = this.blockUsersByKeywordsTrie
      for (let i = 0; i < authorName.length; i++) {
        let remainContent = authorName.substring(i)
        if (blockUsersByKeywordsTrie.nonGreedyMatch(remainContent) !== null) {
          return false
        }
      }
      return true
    },
    filterByAuthorName(authorName) {
      return !this.blockUsersTrie.has(authorName) && this.filterByAuthorNameKeywords(authorName)
    },
    mergeSameUserText(content, richContent, authorName, time) {
      if (!this.config.mergeSameUserDanmaku) {
        return false
      }
      return this.$refs.renderer.mergeSameUserText(content, richContent, authorName, time)
    },
    mergeSimilarText(content) {
      if (!this.config.mergeSimilarDanmaku) {
        return false
      }
      return this.$refs.renderer.mergeSimilarText(content)
    },
    mergeSimilarGift(authorName, price, giftName, num) {
      if (!this.config.mergeGift) {
        return false
      }
      return this.$refs.renderer.mergeSimilarGift(authorName, price, giftName, num)
    },
    getPronunciation(text) {
      if (this.pronunciationConverter === null) {
        return ''
      }
      return this.pronunciationConverter.getPronunciation(text)
    },
    generateTextData(content, textColor) {
      return {
        type: constants.CONTENT_TYPE_TEXT,
        text: content,
        textColor: textColor
      }
    },
    generateImageData(type, matchEmoticon) {
      return {
        type: type,
        text: matchEmoticon.keyword,
        align: matchEmoticon.align,
        height: matchEmoticon.height,
        level: matchEmoticon.level,
        url: matchEmoticon.url
      }
    },
    generateEmoticonData(type, text, emoticon_unique, url, height) {
      return {
        type: type,
        text: text,
        emoticon_unique: emoticon_unique,
        url: url,
        height: height
      }
    },
    // TODO: 处理不同类型表情包
    getRichContent(data) {
      let textColor = 'initial'
      if (this.config.allowTextColorSetting) {
        if (constants.UID_COLOR_MAP_REGEX.test(data.content)) {
          this.uidColorMap[data.authorName] = data.content
          textColor = data.content
        } else if (this.uidColorMap[data.authorName] !== undefined) {
          textColor = this.uidColorMap[data.authorName]
        }
      }
      let richContent = []

      if (this.config.imageShowType > 1) {
        this.config.imageShowType = 1
      }

      // 翻译弹幕，只显示文字
      if (this.config.showTranslateDanmakuOnly == true) {
        richContent.push(this.generateTextData(data.content, textColor))
        return richContent
      }
      // TODO: 处理表情相关
      let emoticonsTrie = this.emoticonsTrie

      // 存在用户自定义关键词则优先显示用户设定表情，不存在则考虑B站自带表情（通用表情、房间表情、个人购买表情）
      if (emoticonsTrie.has(data.content) === false && data.emoticonDetail) {
        // 通过 startsWith 来区分3种表情
        let emoticon_unique = data.emoticonDetail.emoticon_unique
        // B站直播间通用表情（不包括 黄豆表情 autoRenderOfficialSmallEmoji）
        if (this.config.autoRenderOfficialGeneralEmoji === true && data.emoticon !== null && emoticon_unique.startsWith('official')) {
          richContent.push(this.generateEmoticonData(constants.CONTENT_TYPE_EMOTICON, data.content, data.emoticonDetail.emoticon_unique, data.emoticonDetail.url, data.emoticonDetail.height))
          return richContent
        }

        // 主播房间表情（主播在B站网页端上传的个人表情（房间表情，房间粉丝团表情）
        if (this.config.autoRenderStreamerEmoji === true && data.emoticon !== null && emoticon_unique.startsWith('room')) {
          richContent.push(this.generateEmoticonData(constants.CONTENT_TYPE_EMOTICON, data.content, data.emoticonDetail.emoticon_unique, data.emoticonDetail.url, data.emoticonDetail.height))
          return richContent
        }

        // 个人购买表情（用户购买的）
        if (this.config.autoRenderPersonalEmoji === true && data.emoticon !== null && emoticon_unique.startsWith('upower')) {
          richContent.push(this.generateEmoticonData(constants.CONTENT_TYPE_EMOTICON, data.content, data.emoticonDetail.emoticon_unique, data.emoticonDetail.url, data.emoticonDetail.height))
          return richContent
        }
      }

      // NOTE: 上面处理了一般的表情（B站点击后显示单个图片的表情），下面开始处理可以和文字同时显示的黄豆表情和blivechat自定义表情
      let has_blivechat_emoticon = this.config.emoticons.length !== 0 || this.danmu_pic_json.length !== 0
      let has_bilibili_official_small_emoji = data.emots !== null

      // 没有blivechat自定义表情和B站官方小表情，只能是文本
      if (!has_blivechat_emoticon && !has_bilibili_official_small_emoji) {
        richContent.push(this.generateTextData(data.content, textColor))
        return richContent
      }

      // 若含有【B站官方小表情如：[dog]】需要解析，添加到 emoticonsTrie
      if (this.config.autoRenderOfficialSmallEmoji === true && has_bilibili_official_small_emoji) {
        for (let emotIndex in data.emots) {
          let emot = data.emots[emotIndex]
          if (emoticonsTrie.has(emot.descript) === false) { // 存在用户自定义关键词则优先显示用户设定表情
            let emotValue = {
              type: constants.CONTENT_TYPE_EMOTICON,
              keyword: emot.descript,
              align: "inline",
              height: emot.height,
              level: 0,
              url: emot.url
            }
            emoticonsTrie.set(emot.descript, emotValue)
          }
        }
      }

      // 开始分析弹幕文字，并按需求和表情替换
      let startPos = 0
      let pos = 0
      let emoticonCount = 0
      let imageCount = 0
      let emoticonMap = {}
      if (this.config.imageShowType === constants.IMAGE_SHOW_TYPE_ADD_AFTER) { // 在文字之后添加图片的情况，直接添加文字
        richContent.push(this.generateTextData(data.content, textColor))
      }
      while (pos < data.content.length) { // 寻找需要添加的图片
        let remainContent = data.content.substring(pos)
        let matchEmoticon
        if (this.config.isGreedyMatch) {
          matchEmoticon = emoticonsTrie.greedyMatch(remainContent)
        } else {
          matchEmoticon = emoticonsTrie.nonGreedyMatch(remainContent)
        }

        if (matchEmoticon === null) {
          pos++
          continue
        }

        // 如果是替换文字为图片，则加入之前的文本（只有【替换文字为表情包】的模式需要添加文字）
        if (pos !== startPos && this.config.imageShowType === constants.IMAGE_SHOW_TYPE_REPLACE) {
          richContent.push(this.generateTextData(data.content.slice(startPos, pos), textColor))
        }
        let emoticonLevel = toInt(matchEmoticon.level)
        let privilegeType = toInt(data.privilegeType)

        // 如果不满足使用权限，或者超过inline, block类型图片各自的上限 ———— 则此时显示文字，而不是图片
        if ((emoticonLevel > constants.PRIVILEGE_TYPE_ALL && (privilegeType > emoticonLevel || privilegeType === constants.PRIVILEGE_TYPE_ALL))
          || (matchEmoticon.align === 'inline' && emoticonCount >= this.config.maxEmoji)
          || (matchEmoticon.align === 'block' && imageCount >= this.config.maxImage)) {
          if (this.config.imageShowType === constants.IMAGE_SHOW_TYPE_REPLACE) { // 只有【替换文字为表情包】的模式需要添加文字，否则直接跳过
            richContent.push(this.generateTextData(matchEmoticon.keyword, textColor))
          }
        } else { // 如果满足使用权限
          if (this.config.isSkipSameImage === false || emoticonMap[matchEmoticon.url] === undefined) { // 且【不多次显示重复图片】或者说【当前图片没出现过】
            emoticonMap[matchEmoticon.url] = true // 将出现过的图片记录到 map
            if (matchEmoticon.align === 'inline') {
              emoticonCount++
            } else {
              imageCount++
            }
            // 添加图片到消息内容（有官方大、小表情 [dog]，和网页自定义表情中设置的表情）
            richContent.push(this.generateImageData(matchEmoticon.type ?? constants.CONTENT_TYPE_IMAGE, matchEmoticon))
          } else {
            if (this.config.imageShowType === constants.IMAGE_SHOW_TYPE_REPLACE) { // 只有【替换文字为表情包】的模式需要添加文字，否则直接跳过
              richContent.push(this.generateTextData(matchEmoticon.keyword, textColor))
            } // end if
          } // end else
        } // end else
        pos += matchEmoticon.keyword.length
        startPos = pos
      } // end while
      // 当没有出现表情时候 如果是替换文字为表情包，则加入尾部的文本
      if (pos !== startPos && this.config.imageShowType === constants.IMAGE_SHOW_TYPE_REPLACE) {
        richContent.push({
          type: constants.CONTENT_TYPE_TEXT,
          text: data.content.slice(startPos, pos),
          textColor: textColor
        })
      }
      return richContent
    }
  }
}
</script>
