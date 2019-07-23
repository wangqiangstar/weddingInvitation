<template>
    <div class="index">
        
        <div class="bg-swiper">
            <index-swiper :list="list"></index-swiper>
        </div>
        <image class="inv" src="../../static/images/inv.png"/>
        <div class="bg_music" v-if="isPlay" @tap="audioPlay">
            <image src="../../static/images/music_icon.png" class="musicImg music_icon"/>
            <image src="../../static/images/music_play.png" class="music_play pauseImg"/>
        </div>
        <div class="bg_music" v-else @tap="audioPlay">
            <image src="../../static/images/music_icon.png" class="musicImg"/>
            <image src="../../static/images/music_play.png" class="music_play playImg"/>
        </div>
        <div class="info" :animation="animationData">
            <div class="content">
                <h1>Mr.王 & Miss.贾</h1>
                <p>谨定于 2019年9月10日 （星期二）中午12:00</p>
                <p>农历 八月十二 中午十二点整 举办婚礼</p>
                <p>席设：河南省信阳市息县彭店乡王庄村何庄</p>
                <p>地址：河南省信阳市息县彭店乡王庄村何庄</p>
                <image src="../../static/images/we.png" class="img_footer"/>
            </div>
        </div>
    </div>
</template>

<script>
import IndexSwiper from 'components/indexSwiper'
import tools from 'common/js/h_tools'
import cloud from '@/service/cloud'
const audioCtx = wx.createInnerAudioContext()
export default {
  name: 'Index',
  components: {
    IndexSwiper
  },
  data () {
    return {
      isPlay: true,
      animationData: '',
      list: []
    }
  },
  onLoad () {
    // 创建动画
    let animation = wx.createAnimation({
      duration: 3600,
      timingFunction: 'ease',
      delay: 600,
      transformOrigin: '50% 50%'
    })
    animation.scale(0.9).translate(10, 10).step() // 边旋转边放大
    this.animationData = animation.export()
    if (this.list.length === 0) {
      this.getList()
    }
    this.addAbluteFn()
  },
  onShow () {
    this.isPlay = true
  },

  methods: {
    audioPlay () {
      if (this.isPlay) {
        audioCtx.pause()
        this.isPlay = false
        tools.showToast('您已暂停音乐播放~')
      } else {
        audioCtx.play()
        this.isPlay = true
        tools.showToast('背景音乐已开启~')
      }
    },

    getList () {
      const that = this
      wx.showNavigationBarLoading()
      wx.cloud.callFunction({
        name: 'weddingInvite',
        data: {
          sweet: true
        }
      }).then(res => {
        if (res.result.errMsg === 'collection.get:ok') {
          res.result.data.map((item, i) => {
            wx.cloud.downloadFile({
              fileID: item.banner[0], // 文件 ID
              success: res1 => {
                // 返回临时文件路径
                that.list.push(res1.tempFilePath)
              },
              fail: console.error
            })
          })
          let musicUrl = res.result.data[0].music
          audioCtx.src = musicUrl
          audioCtx.loop = true
          audioCtx.autoplay = true
          audioCtx.play()
          wx.hideNavigationBarLoading()
        }
      })
    },

    addList (data) {
      let _this = this
      wx.showNavigationBarLoading()
      cloud.add(data, 'weddingInvite').then((res) => {
        if (res.errMsg === 'collection.get:ok') {
          console.log('添加成功', res)
          _this.getList()
          wx.hideNavigationBarLoading()
        }
      })
    },

    addAbluteFn () {
      let triggerShake = false
      let _this = this
      // 监听加速度数据事件
      wx.onAccelerometerChange(function (res) {
        if (res.x > 0.7 && res.y > 0.7) {
          if (triggerShake) {
            return
          }
          triggerShake = true
          wx.getClipboardData({
            success (copyData) {
              if (copyData.data === 'jiayu') {
                wx.showActionSheet({
                  itemList: ['邀请函照片', '甜蜜照片'],
                  success (res) {
                    console.log(res)
                    if (res.tapIndex === 0) {
                      console.log('选择')
                      wx.chooseImage({
                        success: chooseResult => {
                          console.log('chooseImage')
                          // 将图片上传至云存储空间
                          wx.cloud.uploadFile({
                            // 指定上传到的云路径
                            cloudPath: `sweetAlbum/my-photo-${new Date().getTime()}.png`,
                            // 指定要上传的文件的小程序临时文件路径
                            filePath: chooseResult.tempFilePaths[0],
                            // 成功回调
                            success: res => {
                              console.log('上传成功', res)
                              _this.addList({
                                banner: [res.fileID],
                                music: 'https://666f-forguo-0979a1-1251886253.tcb.qcloud.la/wxapp/wedding/static/your.mp3',
                                sweet: true
                              })
                            }
                          })
                        }
                      })
                      triggerShake = false
                    } else if (res.tapIndex === 1) {
                      console.log('选择')
                      wx.chooseImage({
                        success: chooseResult => {
                          console.log('chooseImage')
                          // 将图片上传至云存储空间
                          wx.cloud.uploadFile({
                            // 指定上传到的云路径
                            cloudPath: `sweetAlbum/my-photo-${new Date().getTime()}.png`,
                            // 指定要上传的文件的小程序临时文件路径
                            filePath: chooseResult.tempFilePaths[0],
                            // 成功回调
                            success: res => {
                              console.log('上传成功', res)
                              _this.addList({
                                banner: [res.fileID],
                                music: 'https://666f-forguo-0979a1-1251886253.tcb.qcloud.la/wxapp/wedding/static/your.mp3',
                                sweet: false
                              })
                            }
                          })
                        }
                      })
                      triggerShake = false
                    }
                  },
                  fail (res) {
                    triggerShake = false
                  }
                })
              } else {
                triggerShake = false
              }
            },
            fail (res) {
              triggerShake = false
            }
          })
        }
      })
    }
  },

  onShareAppMessage: function (res) {
    return {
      title: '送上您的祝福',
      path: '/pages/index/main',
      imageUrl: '../../static/logo.jpg'
    }
  }
}
</script>

<style scoped lang="stylus">
@-webkit-keyframes musicRotate
  from
    -webkit-transformb rotate(0deg)
  to
    -webkit-transform rotate(360deg)
@-webkit-keyframes musicStop
  from
    -webkit-transform rotate(20deg)
  to
    -webkit-transform rotate(0deg)
@-webkit-keyframes musicStart
  from
    -webkit-transform rotate(0deg)
  to
    -webkit-transform rotate(20deg)
@-webkit-keyframes infoAnimation
  0%
    -webkit-transform scale(1) translate(0, 0)
  50%
    -webkit-transform scale(.9) translate(5px, 5px)
  100%
    -webkit-transform scale(1) translate(0, 0)
.index
  height 100%
  position relative
  .img
    width 100%
    height 100%
  .bg-swiper
    width 100%
    height 100%
  .inv
    position absolute
    top 20rpx
    left 89rpx
    width 572rpx
    height 69rpx
    z-index 9
  .bg_music
    position fixed
    right 0
    top 20rpx
    width 85rpx
    z-index 99
    display flex
    justify-content flex-start
    align-items flex-start
    .musicImg
      width 60rpx
      height 60rpx
    .music_icon
      animation musicRotate 3s linear infinite
    .music_play
      width 28rpx
      height 60rpx
      margin-left -10rpx
      transform-origin top
      -webkit-transform rotate(20deg)
    .playImg
      animation musicStop 1s linear forwards
    .pauseImg
      animation musicStart 1s linear forwards
  .info
    width 630rpx
    background rgba(255, 255, 255, 0.75)
    z-index 9
    position fixed
    bottom 40rpx
    left 50rpx
    padding 10rpx
    animation infoAnimation 12s linear infinite
    .content
      width 626rpx
      border 1rpx solid #000
      display flex
      flex-direction column
      justify-content flex-start
      align-items center
      position relative
      padding-bottom 30rpx
      h1
        font-size 50rpx
        height 100rpx
        line-height 100rpx
      p
        font-size 24rpx
        height 60rpx
        line-height 60rpx
      .img_footer
        position absolute
        bottom 0
        left 53rpx
        z-index 99
        height 14rpx
        width 520rpx
  .uploadAblut
    width 100vw
    height 40vh
    position absolute
    button
      width 100vw
      height 40vh
</style>
