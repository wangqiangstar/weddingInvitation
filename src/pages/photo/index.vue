<template>
    <div class="photo">
        <h-swiper :list="list" :isGif="isGif"></h-swiper>
    </div>
</template>

<script>
import HSwiper from 'components/swiper'
// import cloud from '@/service/cloud'
// import imgData from 'common/json/imgData.json'
export default {
  name: 'Photo',
  components: {
    HSwiper
  },
  onLoad () {
    this.isGif = !this.isGif
    if (this.list.length === 0) {
      this.getList()
    }
  },

  data () {
    return {
      list: [],
      isGif: false
    }
  },
  methods: {
    getList () {
      const that = this
      wx.showNavigationBarLoading()
      wx.cloud.callFunction({
        name: 'weddingInvite',
        data: {
          sweet: false
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
          wx.hideNavigationBarLoading()
        }
      })
    }
  }
}
</script>

<style lang="stylus" scoped>
.photo
    height 100%
</style>

