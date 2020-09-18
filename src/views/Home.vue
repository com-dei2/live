<template>
  <div class="home">
    <div id="video-box"
         class="video_box">
      <video class="video-js vjs-big-play-centered"
             preload="auto"
             id="my-player"
             data-setup='{"autoplay": "true"}'
             controls>
        <source :src="source.url"
                :type="source.type">
      </video>
    </div>

    <div class="operation_box"
         :style="{transitionDuration: transitionDuration, opacity: settingBtnOpacity}"
         @click="openSettingDrawer">
      <Tooltip :content="isMac ? 'command + o 打开或关闭设置面板' : 'ctrl + o 打开或关闭设置面板'"
               placement="bottom-end"
               :always="showSettingTips"
               offset="8"
               transfer>
        <Icon type="ios-settings"
              size="24"
              color="#c8c8c8" />
      </Tooltip>
    </div>

    <div class="playing_info_box"
         :style="{opacity: settingDrawer.showPlayingInfo ? 1 : 0}"
         v-text="source.label || source.url"></div>

    <Drawer width="360"
            :closable="false"
            v-model="settingDrawer.shown">
      <div slot="header"
           class="setting_drawer_header">
        <div class="setting_drawer_header_title">选择播放源</div>
        <div class="setting_drawer_header_name">【{{settingDrawer.formData.label || '自定义播放源'}}】</div>
      </div>
      <div class="setting_form">
        <Input v-model="settingDrawer.formData.url"
               style="width: calc(100% - 48px);"
               placeholder="请输入数据源地址"
               @on-change="setStreamCustom">
        </Input>
        <Button type="primary"
                icon="ios-play"
                @click="playSource"></Button>
      </div>
      <div class="detail_settings">
        <div class="detail_settings_item">
          <div class="detail_settings_item_label">显示当前播放</div>
          <div class="detail_settings_item_value">
            <iSwitch v-model="settingDrawer.showPlayingInfo"
                     size="small"></iSwitch>
          </div>
        </div>
        <div class="detail_settings_item">
          <div class="detail_settings_item_label">显示音频背景</div>
          <div class="detail_settings_item_value">
            <iSwitch v-model="settingDrawer.showAudioBg"
                     size="small"></iSwitch>
          </div>
        </div>
      </div>
      <div class="stream_container">
        <div class="stream_item"
             v-for="(item, index) in allStreams"
             :key="index"
             @click="setActiveStream(index)"
             :class="[settingDrawer.activeIndex == index ? 'active' : '']">
          {{item.label}}
        </div>
      </div>
    </Drawer>
  </div>
</template>

<script>
// @ is an alias to /src
import videojs from 'video.js'
import '@videojs/http-streaming'
import { Switch, Button, Icon, Drawer, Input, Tooltip } from 'view-design'
import { STREAMS } from '../assets/streams'
require('video.js/dist/video-js.min.css')

export default {
  name: 'Home',
  components: {
    iSwitch: Switch,
    Button, Icon, Drawer, Input, Tooltip
  },
  data () {
    return {
      settingBtnOpacity: 1,
      transitionDuration: '0.2s',
      showSettingTips: false,
      settingDrawer: {
        shown: false,
        formData: {
          url: '',
          label: '',
          type: ''
        },
        showPlayingInfo: true, // 播放界面是否显示当前正在播放的内容
        showAudioBg: true, // 播放音频时，是否显示背景动画
        activeIndex: -1
      },
      allStreams: STREAMS || [],
      player: null,
      connectInterval: null,
      loaded: false,
      source: {
        // url: 'http://127.0.0.1/live/neq.m3u8',
        url: 'http://10.2.5.98/live/music.m3u8',
        // url: 'http://10.2.5.98/live/ls.m3u8',
        type: 'application/x-mpegURL'
      },
      videoTypes: {
        aac: 'audio/x-aac',
        flac: 'audio/flac',
        m4a: 'audio/mp4',
        mp3: 'audio/mpeg',
        ogg: 'audio/ogg',
        wma: 'audio/x-ms-wma',
        m3u8: 'application/x-mpegURL',
        mp4: 'video/mp4',
        avi: 'video/x-msvideo',
        ts: 'video/MP2T',
        flv: 'video/x-flv',
        wmv: 'video/x-ms-wmv',
        mov: 'video/quicktime',
        '3gp': 'video/3gpp',
        f4v: 'video/x-f4v',
        mkv: 'video/x-matroska'
      },
      isMac: /macintosh|mac os x/i.test(navigator.userAgent.toLowerCase()),
      isWindows: /windows|win32|win64|wow32|wow64/i.test(navigator.userAgent.toLowerCase()),
      musicBgs: ['http://v.bootstrapmb.com/2019/4/u0d54217', 'http://v.bootstrapmb.com/2018/12/0twha3250', 'http://v.bootstrapmb.com/2019/10/5s65c6354', 'http://v.bootstrapmb.com/2018/11/t36ed2742', 'http://v.bootstrapmb.com/2019/5/lm53h4838', 'http://v.bootstrapmb.com/2019/10/x6xcs6480', 'http://v.bootstrapmb.com/2019/4/lm8y53989', 'http://v.bootstrapmb.com/2019/3/wxmoy3693']
    }
  },
  mounted () {
    window.removeEventListener('keydown', this.keyboardEventHandler)
    window.addEventListener('keydown', this.keyboardEventHandler)
    this.$nextTick(() => {
      this.initVideo()
      this.transitionDuration = '1.5s'
      setTimeout(() => {
        this.showSettingTips = true
      }, 800)
      setTimeout(() => {
        this.settingBtnOpacity = 0.2
        setTimeout(() => {
          this.transitionDuration = '0.2s'
        }, 100)
      }, 2000)
    })
  },
  beforeDestroy: function () {
    this.player.dispose();
  },
  methods: {
    keyboardEventHandler (e) {
      if (this.settingDrawer.shown && e.keyCode == 13) {
        this.playSource()
        return
      }
      if (this.isMac && e.metaKey && e.keyCode == 79) {
        e.preventDefault()
        this.toggleSettingDrawer()
        return
      }
      if (this.isWindows && e.ctrlKey && e.keyCode == 79) {
        e.preventDefault()
        this.toggleSettingDrawer()
        return
      }
    },
    openSettingDrawer () {
      this.showSettingTips = false
      this.settingDrawer.shown = true
    },
    toggleSettingDrawer () {
      this.showSettingTips = false
      this.settingDrawer.shown = !this.settingDrawer.shown
    },
    setActiveStream (index) {
      this.settingDrawer.formData = JSON.parse(JSON.stringify(this.allStreams[index]))
      this.settingDrawer.activeIndex = Number(index)
    },
    setStreamCustom (e) {
      let _index = -1
      let item = this.allStreams.filter((item, index) => {
        if (item.url === e.target.value) {
          _index = index
          return true
        } else {
          return false
        }
      })
      this.settingDrawer.formData.label = item.length > 0 ? item[0].label : ''
      this.settingDrawer.formData.type = item.length > 0 ? item[0].type : ''
      this.settingDrawer.activeIndex = _index
    },
    playSource () {
      let _index = this.settingDrawer.activeIndex
      if (_index > -1) {
        this.source = JSON.parse(JSON.stringify(this.allStreams[_index]))
      } else {
        this.source = {
          url: this.settingDrawer.formData.url,
          label: '',
          type: this.videoTypes[this.settingDrawer.formData.url.split('.').pop()] || 'application/x-mpegURL'
        }
      }
      setTimeout(() => {
        this.initVideo()
      }, 100)
    },
    reloadVideo () {
    },
    initVideo () {
      var options = {
        bigPlayButton: true,
        textTrackDisplay: false,
        posterImage: true,
        errorDisplay: false,
        techOrder: ['html5'],
        notSupportedMessage: '此视频暂无法播放，请稍后再试',
        preload: 'auto',
        language: 'zh-CN',
        loop: true
      }

      this.player = videojs('my-player')
      if (this.player) {
        try {
          this.player.pause()
          this.player.dispose()
        } catch (err) { }
      }

      let videoBox = document.querySelector('#video-box')
      videoBox.innerHTML = ''
      let videoStr = `<video class="video-js vjs-big-play-centered"
             controls
             preload="auto"
             id="my-player"
             data-setup='{"autoplay": "true"}'
             style="width: 100%; height: 100%; outline: none;">
        <source id="source"
                src="${this.source.url}"
                type="${this.source.type}">
        </source>
      </video>`
      videoBox.innerHTML = videoStr

      const that = this
      this.player = videojs('my-player', options, function () {
        document.querySelector('.video-js .vjs-control-bar').style.zIndex = 3
        if (!!that.source.type.match(/^audio/)) {
          let musciBgs = that.musicBgs
          let ran = Math.floor(Math.random() * musciBgs.length)
          that.addMusicBox(musciBgs[ran] || 'http://v.bootstrapmb.com/2019/3/wxmoy3693')
        }
        // this.play()
        this.on('play', () => {
          that.showSettingTips = false
        })
      })
    },
    addMusicBox (src) {
      // vjs-tech
      const that = this
      let playlistEle = document.createElement('div')
      !playlistEle.classList.contains('music_container') && playlistEle.classList.add('music_container')
      let musicBox = document.createElement('iframe')
      if (!that.settingDrawer.showAudioBg) {
        playlistEle.style.opacity = 0
      } else {
        playlistEle.style.opacity = 1
      }
      musicBox.src = src || 'http://v.bootstrapmb.com/2019/4/u0d54217/'
      playlistEle.appendChild(musicBox)
      let volumeEle = document.querySelector('.vjs-tech')
      volumeEle.parentNode.insertBefore(playlistEle, volumeEle)
      // document.querySelector('.video-js').innerHTML += '<div class="music_container"></div>'
    }
  },
  watch: {
    'settingDrawer.showAudioBg' (val) {
      let playBg = document.querySelector('.music_container')
      if (!playBg) return
      if (val) {
        playBg.style.opacity = 1
      } else {
        playBg.style.opacity = 0
      }
    }
  }
}
</script>
<style lang="less" scoped>
.home {
  position: relative;
  height: 100%;
  .video_box {
    width: 100%;
    height: 100%;
    .video-js {
      outline: none !important;
      width: 100%;
      height: 100%;
      video {
        outline: none !important;
      }
      &:active {
        box-shadow: none !important;
      }
    }
  }

  .operation_box {
    position: absolute;
    width: 50px;
    height: 50px;
    right: 30px;
    top: 30px;
    z-index: 99;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    // opacity: 0.2;
    // transition: opacity 0.2s ease-in-out;
    transition-property: opacity;
    transition-timing-function: ease-in-out;
    &:hover {
      opacity: 1 !important;
    }
  }

  .playing_info_box {
    position: absolute;
    left: 30px;
    top: 30px;
    z-index: 99;
    padding: 4px 8px;
    box-sizing: border-box;
    transition: opacity 0.2s ease-in-out;
    color: #c8c8c8;
    text-shadow: 0 0 #000;
    max-width: 80%;
    white-space: nowrap;
    text-overflow: ellipsis;
    overflow: hidden;
    pointer-events: none;
  }
}
.setting_form {
  width: 100%;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-around;
}
.stream_container {
  width: 100%;
  height: calc(~"100% - 140px");
  margin-top: 18px;
  padding: 10px 0;
  box-sizing: border-box;
  overflow-y: auto;
  background-color: #fafafa;
  .stream_item {
    width: 100%;
    height: 32px;
    padding: 0 15px;
    box-sizing: border-box;
    display: flex;
    flex-direction: row;
    align-items: center;
    cursor: pointer;
    transition: background-color 0.1s ease-in-out;
    color: #333;
    &:hover {
      background-color: #eee;
    }
    &.active {
      color: #fff !important;
      background-color: #66c18c !important;
      &:hover {
        background-color: #66c18c !important;
      }
    }
  }
}
.setting_drawer_header {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  &_title {
    // width: 100%;
    height: 20px;
    line-height: 20px;
    font-size: 16px;
    color: #17233d;
    font-weight: 500;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  &_name {
    font-size: 13px;
    color: darken(#66c18c, 25);
  }
}
.detail_settings {
  padding: 15px 10px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  &_item {
    width: 100%;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: flex-start;
    transition: background-color 0.2s ease;
    &:hover {
      background-color: #f8f8f8;
    }
    &_label {
      font-size: 14px;
      color: #121212;
      margin-right: 8px;
    }
  }
}
</style>