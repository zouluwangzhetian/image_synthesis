<template>
  <div class="img-watermark-box">
    <div class="img-box" ref="imgBox">
      <img :src="img" alt="" class="img-watermark" ref="imgWatermark" crossOrigin="anonymous">
      <div class="watermark-box">
        <div class="item-box" v-for="(item, index) in watermark" :key="index">
          <img :ref="'img-'+index" crossOrigin="anonymous" draggable="false" :style="{width: item.width+'px',height: item.height+'px',top:item.top+'px',left:item.left+'px',opacity:item.opacity}" @mousedown="move(item, $event)" :src="item.img" alt="" class="item-watermark">
          <span class="remove-tool" @click="removeWatermark(index)" :style="{top:item.top + 'px', left:item.left + 'px'}">X</span>
          <div class="opactity-box" :style="{width: item.width + 'px',top:(item.top - 12) + 'px', left:item.left + 'px'}">
            <span class="opactity-tool reduce" @click="changeOpactity('reduce', item)">-</span>
            <span class="opactity-text">透明度</span>
            <span class="opactity-tool add" @click="changeOpactity('add',item)">+</span>
          </div>
          <span class="tool right-center" @mousedown="changeImgSize(item, index, 'right', $event)" :style="{top:(item.top + (item.height / 2) - 3) + 'px', left:(item.left + item.width) + 'px'}"></span>
          <span class="tool bottom-center" @mousedown="changeImgSize(item, index, 'bottom', $event)" :style="{top:(item.top + item.height) + 'px', left:(item.left + (item.width / 2) - 3) + 'px'}"></span>
          <span class="tool right-bottom" @mousedown="changeImgSize(item, index, 'center', $event)" :style="{top:(item.top + item.height) + 'px', left:(item.left + item.width) + 'px'}"></span>
        </div>
      </div>
    </div>
    <div class="img-watermark-tool">
      <div @click="getCanvsImg">保存</div>
      <div class="watermark-tool-box">
        <input type="file" v-show="false" ref="fileInput" multiple="multiple" @change="selectImg">
        <div class="tool-btn">
          <button class="file-btn" @click="fileBtnClick(true)">
            选择背景图
          </button>
          <button class="file-btn" @click="fileBtnClick(false)">
            选择合成图片
          </button>
        </div>
        <div class="watermark-img-box">
          <img :src="item.img" alt="" v-for="(item, index) in watermark" :key="index" class="watermark-img">
        </div>
      </div>
      <img :src="imgBase64" alt="">
    </div>
  </div>
</template>

<script>
export default {
  name: 'imgWatermark',
  data () {
    return {
      img: require('../assets/timg.jpg'), // 背景图
      imgBase64: '', // 最后储存的图片
      canvas: {}, // canvas 实例
      typeImg: false, // 判断当前添加背景图还是水印图
      watermark: [], // 需要合成图片的数组
      flag: false // 当水印大小改变时，关闭拖拽
    }
  },
  computed: {
    imgWidth () {
      return this.$refs.imgWatermark.width
    },
    imgHeight () {
      return this.$refs.imgWatermark.height
    }
  },
  created () {
    this.canvas = document.createElement('canvas')
  },
  methods: {
    // 改变透明度
    changeOpactity (way, item) {
      if (way === 'add' && item.opacity < 1) {
        item.opacity = parseFloat((item.opacity + 0.1).toFixed(1))
        return
      }
      if (way === 'reduce' && item.opacity > 0.1) {
        item.opacity = parseFloat((item.opacity - 0.1).toFixed(1))
      }
    },
    // 拖拽事件
    move (item, e) {
      console.log(e.target.width)
      console.log(this.imgWidth)
      if (this.flag) {
        return false
      }
      let x = e.clientX - e.target.offsetLeft
      let y = e.clientY - e.target.offsetTop
      let targetWidth = e.target.width
      let targetHeight = e.target.height
      document.onmousemove = (e) => {
        let left = e.clientX - x
        let top = e.clientY - y
        let chaX = this.imgWidth - left
        let chaY = this.imgHeight - top
        if (left + targetWidth >= this.imgWidth) {
          left = this.imgWidth - targetWidth
        }
        if (chaX >= this.imgWidth) {
          left = 0
        }
        if (top + targetHeight >= this.imgHeight) {
          top = this.imgHeight - targetHeight
        }
        if (chaY >= this.imgHeight) {
          top = 0
        }
        item.left = left
        item.top = top
      }
      document.onmouseup = (e) => {
        document.onmousemove = null
        document.onmouseup = null
      }
    },
    // 改变大小事件
    changeImgSize (item, index, location, e) {
      this.flag = true
      let clientX = e.clientX
      let clientY = e.clientY
      document.onmousemove = (e) => {
        console.log(e)
        let offsetLeft = this.$refs['img-' + index][0].offsetLeft
        let offsetTop = this.$refs['img-' + index][0].offsetTop
        let chaX = this.imgWidth - offsetLeft - item.width
        let chaY = this.imgHeight - offsetTop - item.height
        if (location === 'center') {
          if (clientX < e.clientX && clientY < e.clientY) {
            if (item.width < 150 && chaX > 10) {
              item.width += 1
            }
            if (item.height < 140 && chaY > 10) {
              item.height += 1
            }
          }
          if (clientX > e.clientX && clientY > e.clientY) {
            if (item.width > 40) {
              item.width -= 1
            }
            if (item.height > 30) {
              item.height -= 1
            }
          }
        }
        if (location === 'right') {
          if (clientX < e.clientX && item.width < 150) {
            if (chaX > 10) {
              item.width += 1
            }
          }
          if (clientX > e.clientX && item.width > 40) {
            item.width -= 1
          }
        }
        if (location === 'bottom') {
          if (clientY < e.clientY && item.height < 140) {
            if (chaY > 10) {
              item.height += 1
            }
          }
          if (clientY > e.clientY && item.height > 30) {
            item.height -= 1
          }
        }
        document.onmouseup = (e) => {
          document.onmousemove = null
          document.onmouseup = null
          this.flag = false
        }
      }
    },
    // 移除事件
    removeWatermark (index) {
      this.watermark.splice(index, 1)
    },
    // 选择图片上传
    fileBtnClick (type) {
      this.typeImg = type
      this.$refs.fileInput.click()
    },
    // input上传
    selectImg (e) {
      console.log(e)
      console.log(e)
      console.log(e.target.files)
      let filesArr = e.target.files || e.dataTransfer.files
      this.setSourceImg(filesArr)
    },
    setSourceImg (filesArr) {
      let filesLength = filesArr.length
      for (let i = 0; i < filesLength; i++) {
        let fr = new FileReader()
        fr.onload = () => {
          if (this.typeImg) {
            console.log(this.typeImg)
            this.img = fr.result
          } else {
            this.watermark.push({
              img: fr.result,
              top: Math.floor(Math.random() * ((this.imgHeight - 80) + 1)),
              left: Math.floor(Math.random() * ((this.imgWidth - 100) + 1)),
              width: 100,
              height: 80,
              opacity: 1
            })
          }
        }
        fr.readAsDataURL(filesArr[i])
      }
    },
    // 生成 canvas
    getCanvsImg () {
      this.canvas.width = this.imgWidth
      this.canvas.height = this.imgHeight
      var context = this.canvas.getContext('2d')
      context.drawImage(this.$refs.imgWatermark, 0, 0, this.canvas.width, this.canvas.height)
      this.watermark.map((item, index) => {
        context.globalAlpha = item.opacity
        context.drawImage(this.$refs['img-' + index][0], item.left, item.top, item.width, item.height)
      })
      console.log(context)
      this.imgBase64 = this.canvas.toDataURL('image/png')
      // console.log(this.imgBase64)
    }
  }
}
</script>

<style lang="less" scoped>
  .img-watermark-box{
    display: flex;
    .img-box{
      position: relative;
      overflow: hidden;
      .img-watermark{
        display: block;
        width: 600px;
        height: 337.5px;
        user-select:none;
      }
      .watermark-box{
        position: absolute;
        left: 0;
        top: 0;
        right: 0;
        bottom: 0;
        display: flex;
        .item-box{
          user-select:none;
          .item-watermark{
            position: absolute;
            display: block;
            cursor: move;
            user-select:none;
          }
          .remove-tool{
            position: absolute;
            display: block;
            width: 10px;
            height: 15px;
            color: rgba(60, 60, 60, .8);
            background: rgba(255,255,255,.7);
          }
          .opactity-box{
            position: absolute;
            display: flex;
            align-items: center;
            justify-content: center;
            height: 12px;
            background: rgba(0,0,0,.3);
            color: #fff;
            overflow: hidden;
            .opactity-tool{
              &.reduce{
                margin-right: 5px;
                margin-bottom: 6px;
                font-size: 34px;
              }
              &.add{
                margin-left: 5px;
                font-size: 23px;
              }
            }
          }
          .tool{
            position: absolute;
            display: block;
            width: 6px;
            height: 6px;
            user-select:none;
            border: 1px solid #3c3c3c;
            background: #fff;
          }
        }
      }
    }
    .img-watermark-tool{
      .watermark-tool-box{
        .file-btn{
          padding: 8px 15px;
          background-color: #fff;
          border: 1px solid #c1c1c1;
          border-radius: 4px;
          // box-shadow: 0 1px 1px 0 rgba(#c1c1c1, 0.65);
          outline: none;
          cursor: pointer;
          font-size: 13px;
        }
        .watermark-img-box{
          display: flex;
          align-items: center;
          .watermark-img{
            margin-right: 10px;
            width: 65px;
            height: 50px;
          }
        }
      }
    }
  }
</style>
