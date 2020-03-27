<style lang="less" scoped>
    @import "../../assets/style/mixin";

    .img-container {
        width: 100%;
    }

    .upload-img {
        position: relative;
        .bgImgMixin(@s: contain);
        width: 80px;
        height: 80px;
        margin-right: 10px;
        margin-bottom: 10px;
        float: left;

        .footer {
            cursor: pointer;
            display: none;
            position: absolute;
            left: 0;
            bottom: 0;
            width: 100%;
            height: 88px;
            line-height: 22px;
            padding: 0 8px;
            // background-color:#eee;
            color: rgba(0, 0, 0, 0.7);
            font-size: 12px;

            .icon-arrow_left {
                padding-right: 10px;
            }

            .icon-delete {

            }

            .icon {
                font-size: 10px;
            }

            &.circle {
                background: transparent;
                color: red;
            }

            .icon-tiaoxu-px.rt {
                transform: rotate(180deg);
                margin-left: 5px;
                position: relative;
                top: 1px;
            }
        }

        &:hover .footer {
            display: block;
        }

        &.circle {
            .bgImgMixin(@s: cover);
            border-radius: 50%;
        }
    }

    .upload-icon {
        width: 80px;
        height: 80px;
        line-height: 80px;
        font-size: 24px;
        text-align: center;
        background-color: rgb(248, 248, 248);
        color: rgb(225, 225, 225);
    }
</style>

<template lang="html">
    <div class="content clearfix">
        <div
            v-loading="item && item.loading"
            class="upload-img"
            v-for="(item, index) in imageList"
            v-if="item"
            @click.stop.prevent
            :style="{
        width: width,
        height: height,
        backgroundImage: `url(${ API_ROOT+item.url })`
      }">
            <div class="footer clearfix">
                <i class="icon iconfont icon-tiaoxu-px pull-left" v-if="imageList.length>1&&index>0"
                   @click="moveToLeft(index)"></i>
                <i class="icon iconfont icon-tiaoxu-px pull-left rt"
                   v-if="imageList.length>1&&index!==imageList.length-1" @click="moveToRight(index)"></i>
                <i class="icon iconfont icon-delete el-icon-circle-close pull-right" @click="deleteImg(index)"></i>
            </div>
        </div>
        <el-upload
            name="filename"
            :disabled="disabled"
            :class="uploadClass"
            :multiple="multiple"
            :data="{type: params}"
            :action="uploadUrl"
            :show-file-list="false"
            :on-success="uploadSucceed"
            :http-request="ctrlUpload"
            :sort="sort"
            :before-upload="beforeUpload">
            <div v-if="isOnce" class="img-container">
                <div
                    @click="submit"
                    class="upload-img"
                    :class="{circle: type === 'circle'}"
                    v-show="imageUrl"
                    v-loading="loading"
                    :style="{
            width: width,
            height: height,
            backgroundImage: `url(${ imageUrl })`
          }">
                    <div class="footer clearfix" v-if="type !== 'circle'">
                        <i class="icon iconfont icon-delete pull-right" @click="deleteImg()"></i>
                    </div>
                </div>
                <slot v-if="!imageUrl" name="header-img">
                    <div
                        v-loading="loading"
                        class="upload-icon"
                        :style="{ width: width, height: height, lineHeight: height }">
                        +
                    </div>
                </slot>
            </div>
            <div
                v-else
                class="upload-icon"
                :style="{ width: width, height: height }">
                +
            </div>
        </el-upload>
    </div>
</template>

<script>

export default {
  name: 'upload-img',
  data () {
    return {
      API_ROOT:'http://fdfdsfds.com', //随意
      loading: false,
      uploadUrl: API_ROOT + '/imageFileUpload',
      imageUrl: '',
      imageList: []
    }
  },
  props: {
    limit: {
      type: Number,
      default: 5
    },
    sort: {
      type: Boolean,
      default: false
    },
    isOnce: {
      type: Boolean,
      default: true
    },
    disabled: {
      type: Boolean,
      default: false
    },
    multiple: {
      type: Boolean,
      default: false
    },
    width: {
      type: String,
      default: '80px'
    },
    height: {
      type: String,
      default: '80px'
    },
    image: {
      type: String,
      default: ''
    },
    type: {
      type: String,
      default: 'square' // circle
    },
    params: {
      type: [String, Object],
      default: null
    },
    imgList: {
      type: Array,
      default () {
        return []
      }
    },
    uploadClass: {
      type: String,
      default () {
        return ''
      }
    },
    resizeW: {
      type: Number,
      default () {
        return 0
      }
    }
  },
  methods: {
    async ctrlUpload (e) {
      const self = this
      const canvas = document.createElement('canvas')
      const img = new Image()
      const windowURL = window.URL || window.webkitURL
      const dataURL = windowURL.createObjectURL(e.file)
      img.src = dataURL

      img.onload = function () {
        const w = img.width
        const h = img.height
        const scale = parseFloat(w / h)
        if (self.resizeW && self.resizeW < w) {
          const realW = self.resizeW
          const realH = parseInt(realW / scale)
          canvas.width = realW
          canvas.height = realH
          const ctx = canvas.getContext('2d')
          ctx.drawImage(img,
            0, // sourceX,
            0, // sourceY,
            realW, // sourceWidth,
            realH // sourceHeight,
            // 0, // destX,
            // 0 // destY,
            // w, // destWidth,
            // h// destHeight
          )
          const base64str = canvas.toDataURL('image/png', 0.99)
          const file = self.dataURLtoBlob(base64str)
          self.handleUpload(file, e.data)
        } else {
          self.handleUpload(e.file, e.data)
        }
      }
    },

    async handleUpload (file, params) {
      if (this.sort) {
        const res = await uploadSortImage(file, params)
        if (res.errorCode === 0) {
          const i = res.data[0].fileName.split('.')[0]
          if (!(i - 0)) {
            return this.$notify.warning({
              title: '警告',
              message: '文件命名错误,按照顺序填写数字'
            })
          }
          if (this.isOnce) {
            this.imageUrl = res.data[0].url
            this.$emit('success', this.imageUrl)
          } else {
            // const index = this.imageList.findIndex(item => item.loading)
            this.imageList.splice(i - 1, 1, {
              loading: false,
              url: res.data[0].url
            })
            this.$emit('success', this.imageList)
          }
          this.loading = false
        }
      } else {

          /*请求*/

        
        // setTimeout(async () => {
        //   const res = await uploadImageFile(file, params)
        //   if (res.errorCode === 0) {
        //     if (this.isOnce) {
        //       this.imageUrl = res.data
        //       this.$emit('success', this.imageUrl)
        //     } else {
        //       const index = this.imageList.findIndex(item => item.loading)
        //       this.imageList.splice(index, 1, {
        //         loading: false,
        //         url: res.data.fpicurl
        //       })
        //       this.$emit('success', this.imageList)
        //     }
        //     this.loading = false
        //   }
        // }, Math.random() * 100)
      }
    },

    dataURLtoBlob (dataurl) {
      var arr = dataurl.split(',')
      var mime = arr[0].match(/:(.*?);/)[1]
      var bstr = atob(arr[1])
      var n = bstr.length
      var u8arr = new Uint8Array(n)
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n)
      }
      return new Blob([u8arr], { type: mime })
    },

    // 手动上传
    submit (ev) {
      if (this.type !== 'circle') {
        ev.stopPropagation()
      }
    },

    // 左移
    moveToLeft (index) {
      const prev = Object.assign({}, this.imageList[index - 1])
      this.imageList.splice(index - 1, 1, this.imageList[index])
      this.imageList.splice(index, 1, prev)
      this.$emit('success', this.imageList)
    },

    // 右移
    moveToRight (index) {
      const next = Object.assign({}, this.imageList[index + 1])
      this.imageList.splice(index + 1, 1, this.imageList[index])
      this.imageList.splice(index, 1, next)
      this.$emit('success', this.imageList)
    },

    // 删除图片
    deleteImg (index) {
      if (this.isOnce) {
        this.imageUrl = ''
        this.$emit('delete', this.imageUrl)
      } else {
        this.imageList.splice(index, 1)
        this.$emit('delete', this.imageList)
      }
    },

    // 上传成功
    uploadSucceed (res, file) {
      // if (res.errorCode === 0) {
      //   if (this.isOnce) {
      //     this.imageUrl = res.data
      //     this.$emit('success', this.imageUrl)
      //   } else {
      //     const index = this.imageList.findIndex(item => item.loading)
      //     this.imageList.splice(index, 1, {
      //       loading: false,
      //       url: res.data
      //     })

      //     this.$emit('success', this.imageList)
      //   }
      //   this.loading = false
      // }
    },

    // 上传之前
    beforeUpload (file) {
      const isJPG = file.type === 'image/jpeg' || file.type === 'image/png'
      const isLt5M = file.size / 1024 / 1024 < 5
      const isErrName = file.name.includes(',')
      const isLimit = this.imageList.length < this.limit

      if (!isLimit) {
        this.$message.error(`上传图片不能超过${this.limit}张`)
      }
      if (!isJPG) {
        this.$message.error('上传的图片文件只能是 JPG/PNG 格式!')
      }
      if (!isLt5M) {
        this.$message.error('上传的图片文件大小不能超过 5MB!')
      }
      if (isErrName) {
        this.$message.error('上传的图片名称不允许有逗号等特殊字符!')
      }
      if (this.isOnce && isJPG && isLt5M) {
        this.loading = true
      }
      if (
        this.isOnce == false &&
                    isJPG &&
                    isLt5M &&
                    isErrName == false &&
                    isLimit
      ) {
        if (this.sort) {
          let i = file.name.split('.')[0]
          if (!(i - 0)) {
            return this.$notify.warning({
              title: '警告',
              message: '文件命名错误,按照顺序填写数字'
            })
          } else {
            this.imageList[i - 1] = {
              url: '',
              loading: true
            }
          }
        } else {
          this.imageList.push({
            url: '',
            loading: true
          })
        }
      }
      return isJPG && isLt5M && isErrName == false
    }
  },
  created () {
    if (this.isOnce) {
      this.imageUrl = this.image
    } else {
      this.imageList = this.imgList || []
    }
  },
  watch: {
    imgList: {
      handler (newVal) {
        this.imageList = newVal || []
      },
      deep: true
    },
    image (newVal) {
      this.imageUrl = newVal
    }
  }
}
</script>
