<template>
  <div class="upload-container">
    <div class="image-preview image-app-preview">
      <div class="image-preview-wrapper" v-for="(item, index) of imageUrl">
        <div v-if="type === 'image'">
          <img :src="fileRoot + item">
          <div class="image-preview-action">
            <i class="el-icon-delete" @click="rmImage(item)"/>
          </div>
        </div>
        <video width="320" :src="fileRoot + item.url" autobuffer controls :poster="fileRoot + item.thumbnail" v-else></video>
      </div>
    </div>
    <el-upload
      v-loading="loading"
      :multiple="false"
      :show-file-list="false"
      list-type='picture'
      :on-success="handleImageSuccess"
      :on-progress="()=> {loading = true}"
      class="image-uploader"
      drag
      :action="fileServer" v-if="isUpload">
      <i class="el-icon-upload"/>
      <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
    </el-upload>
  </div>
</template>

<script>
import { getToken } from '@/api/qiniu'
import configPath from '@/configs/path'

export default {
  name: 'SingleImageUpload3',
  props: {
    type: {
      type:String,
      default:'image'
    },
    value: {
      type: Array,
      default: []
    }
  },
  data() {
    return {
      fileServer: configPath.fileServer + (this.type === 'image' ? '/file/uploadPic' : '/file/uploadVideo'), //uploadVideo/uploadHeadImage
      fileRoot: configPath.fileRoot + '/',
      tempUrl: '',
      dataObj: { token: '', key: '' },
      loading:false
    }
  },
  computed: {
    imageUrl() {
      return this.value
    },
    isUpload() {
      return this.imageUrl[0] ? !this.imageUrl[0].thumbnail : true 
    }
  },
  methods: {
    rmImage(val) {
      let index = this.imageUrl.indexOf(val)
      if (index !== -1) {
        this.imageUrl.splice(index,1)
        this.emitInput(this.imageUrl)
      }
    },
    emitInput(val) {
      this.$emit('input', val)
    },
    handleImageSuccess(res,file,fileList) {
      if (this.type === 'image') {
        this.imageUrl.push(res.result.url)
        this.emitInput(this.imageUrl)
      } else {
        let videoInfo = {
          filename: res.result.filename,
          type: 2,
          thumbnail: res.thumbnail,
          url: res.result.url
        }
        this.imageUrl.push(videoInfo)
      }
      this.emitInput(this.imageUrl)
      this.loading = false
      // this.emitInput(configPath.fileRoot + '/' + res.result.url)
    },
    beforeUpload() {
      console.log(1)
      const _self = this
      return new Promise((resolve, reject) => {
        getToken().then(response => {
          const key = response.data.qiniu_key
          const token = response.data.qiniu_token
          _self._data.dataObj.token = token
          _self._data.dataObj.key = key
          this.tempUrl = response.data.qiniu_url
          resolve(true)
        }).catch(err => {
          console.log(err)
          reject(false)
        })
      })
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
@import "~@/styles/mixin.scss";
.upload-container {
  width: 100%;
  position: relative;
  @include clearfix;
  .image-uploader {
    width: 35%;
    float: left;
  }
  .image-preview {
    position: relative;
    float: left;
    .image-preview-wrapper {
      width: 200px;
      height: 200px;
      margin-right: 30px;
      position: relative;
      img {
        width: 100%;
        height: 100%;
      }
    }
    .image-preview-action {
      position: absolute;
      width: 100%;
      height: 100%;
      left: 0;
      top: 0;
      cursor: default;
      text-align: center;
      color: #fff;
      opacity: 0;
      font-size: 20px;
      background-color: rgba(0, 0, 0, .5);
      transition: opacity .3s;
      cursor: pointer;
      text-align: center;
      line-height: 200px;
      .el-icon-delete {
        font-size: 36px;
      }
    }
    &:hover {
      .image-preview-action {
        opacity: 1;
      }
    }
  }
  /* .image-app-preview {
    width: 320px;
    height: 180px;
    position: relative;
    border: 1px dashed #d9d9d9;
    float: left;
    margin-left: 50px;
    .app-fake-conver {
      height: 44px;
      position: absolute;
      width: 100%; // background: rgba(0, 0, 0, .1);
      text-align: center;
      line-height: 64px;
      color: #fff;
    }
  } */
}
</style>
