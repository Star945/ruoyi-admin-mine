<template>
  <div class="upload-container">
    <el-button  icon="el-icon-upload" size="mini" type="primary" @click=" dialogVisible=true">
      上传
    </el-button>
    <el-dialog :visible.sync="dialogVisible" :append-to-body="true"  width="30%"  @close='closeDialog'>
      <el-upload
        :multiple="true"
        :file-list="fileList"
        :show-file-list="true"
        :on-remove="handleRemove"
        :on-success="handleSuccess"
        :before-upload="beforeUpload"
        class="editor-slide-upload"
        :action="baseUrl+uploadImgUrl"
        list-type="picture-card"
        accept="image/*"
      >
        <el-button size="small" type="primary">
          点击上传
        </el-button>
      </el-upload>
      <el-button @click="dialogVisible = false">
        关闭
      </el-button>
      <el-button type="primary" @click="handleSubmit">
        确定
      </el-button>
    </el-dialog>
  </div>
</template>

<script>
// import { getToken } from 'api/qiniu'

export default {
  name: 'EditorSlideUpload',
  props: {
    color: {
      type: String,
      default: '#1abc9c'
    }
  },
  data() {
    return {
      uploadImgUrl:'/picture/saveFile',
      baseUrl:window.g.api,
      dialogVisible: false,
      listObj: {},
      fileList: []
    }
  },
  methods: {
    //关闭弹窗
    closeDialog(){
      // this.uploadImgUrl= "";
    },
    checkAllSuccess() {
      return Object.keys(this.listObj).every(item => this.listObj[item].hasSuccess)
    },
    handleSubmit() {
      const arr = Object.keys(this.listObj).map(v => this.listObj[v])
      // if (!this.checkAllSuccess()) {
      //   this.$message('Please wait for all images to be uploaded successfully. If there is a network problem, please refresh the page and upload again!')
      //   return
      // }
      this.$emit('successCBK', arr)
      this.listObj = {}
      this.fileList = []
      this.dialogVisible = false
    },
    handleSuccess(response, file) {
      const uid = file.uid
      const objKeyArr = Object.keys(this.listObj)
      for (let i = 0, len = objKeyArr.length; i < len; i++) {
        if (this.listObj[objKeyArr[i]].uid === uid) {
          this.listObj[objKeyArr[i]].url = window.g.baseUrl + response.data
          this.listObj[objKeyArr[i]].hasSuccess = true
          return
        }
      }
    },
    handleRemove(file) {
      const uid = file.uid
      const objKeyArr = Object.keys(this.listObj)
      console.log(objKeyArr,"objKeyArr")
      for (let i = 0, len = objKeyArr.length; i < len; i++) {
        if (this.listObj[objKeyArr[i]].uid === uid) {
          delete this.listObj[objKeyArr[i]]
          return
        }
      }
    },
    beforeUpload(file) {
      let type = file.type;
        if (type != "image/jpeg" && type != "image/png") {
         // 若不符合图片类型，则让当前上传的文件去除掉即可，即从上传对列删除本次上传
          // let fileLength =this.fileList.length
          // this.fileList.splice(fileLength - 1, 1);
          console.log(file,222)
          this.handleRemove(file);
          console.log(this.listObj,2)
          this.$message.error('只允许上传图片');
        }else{
          const _self = this
          const _URL = window.URL || window.webkitURL
          const fileName = file.uid
          this.listObj[fileName] = {}
          return new Promise((resolve, reject) => {
            const img = new Image()
            img.src = _URL.createObjectURL(file)
            img.onload = function() {
              _self.listObj[fileName] = { hasSuccess: false, uid: file.uid, width: this.width, height: this.height }
            }
            resolve(true)
          })
        }
    }
  }
}
</script>

<style lang="scss" scoped>
.editor-slide-upload {
  margin-bottom: 20px;
  /deep/ .el-upload--picture-card {
    width: 100%;
  }
}
</style>
