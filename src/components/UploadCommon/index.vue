<template>
  <div class="attach-box">
    <template v-if="listType === 'piclist' && fileList.length > 0">
      <div v-for="(item, key) in fileList" :key="key" class="img-list">
        <template v-if="item.status === 'success' || item.status === undefined">
          <img :src="baseShowHref + item[urlName]" />
          <div v-if="!disabled" class="demo-upload-list-cover">
            <i class="el-icon-view" @click="handlePreview(item)" />
            <i class="el-icon-delete" @click="handleRemove(item, key)" />
          </div>
        </template>
        <template v-else>
          <el-progress
            :key="key"
            :width="60"
            type="circle"
            :percentage="item.percentage"
            color="#8e71c7"
          />
        </template>
      </div>
    </template>
    <template v-if="listType === 'coverImg' && fileList.length > 0">
      <div
        v-for="(item, index) in fileList"
        :key="index"
        class="cover-box-list"
      >
        <template v-if="item.status === 'success' || item.status === undefined">
          <div class="img-box">
            <img :src="baseShowHref + item[urlName]" alt />
            <div v-show="!disabled" class="cover-handle">
              <i
                class="el-icon-delete"
                title="删除"
                alt="删除"
                @click="handleRemove(item, index)"
              />
            </div>
          </div>
        </template>
        <template v-else>
          <el-progress
            :width="100"
            style="margin: 0 auto"
            type="circle"
            :percentage="
              item.percentage === undefined
                ? 100
                : Math.floor(item.percentage * 100) / 100
            "
            color="#8e71c7"
          />
        </template>
      </div>
    </template>

    <div
      v-if="listType === 'identity' && fileList.length > 0"
      class="identtiy-box"
    >
      <img :src="baseShowHref + fileList[0].url" />
      <div v-show="!disabled" class="cover-handle">
        <i
          class="el-icon-delete"
          title="删除"
          alt="删除"
          @click="handleRemove(fileList[0], 0)"
        />
      </div>
    </div>
    <template
      v-else-if="
        (!listType || listType === undefined || listType === 'picture') &&
        fileList.length > 0
      "
    >
      <div class="demo-upload-list">
        <template v-if="fileList[0].status === 'success'">
          <img :src="baseShowHref + fileList[0].url" />
          <div v-if="!disabled" class="demo-upload-list-cover">
            <i class="el-icon-view" @click="handlePreview(fileList[0])" />
            <i class="el-icon-delete" @click="handleRemove(fileList[0], 0)" />
          </div>
        </template>
        <template v-else>
          <el-progress
            :width="60"
            type="circle"
            :percentage="fileList[0].percentage"
            color="#8e71c7"
          />
        </template>
      </div>
    </template>
    <el-upload
      v-show="fileList.length < (limit || 1)"
      :disabled="disabled"
      ref="uploadfile"
      class="upload-demo"
      :headers="headers"
      :action="BaseHref + action"
      :name="name"
      :on-preview="handlePreview"
      :on-error="handleError"
      :multiple="multiple"
      :data="data"
      :on-success="handleSuccess"
      :on-exceed="handleExceed"
      :accept="accept"
      :file-list="fileList"
      :show-file-list="showFileList"
      :before-upload="handleBeforeUpload"
      :on-change="handleChange"
      :limit="limit"
      :list-type="listType"
      :drag="!multiple"
      style="display: inline-block"
    >
      <template v-if="listType === 'text'">
        <el-button style="float: left" type="primary" round>
          {{ fileList.length > 0 ? "继续" + upLoadBtnTxt : upLoadBtnTxt }}
        </el-button>
        <div class="tips">
          支持{{
            accept === "image/*"
              ? ".jpg,.jpeg,.png,.gif,.bmp,.doc,.docx,.ppt,.pptx,.xls,.xlsx,.pdf"
              : accept
          }}文件（大小不超过{{ maxSize }}MB）
        </div>
        <!-- <el-button size="small" type="primary">{{upLoadBtnTxt}}</el-button> -->
      </template>
      <template v-else-if="listType === 'coverImg'">
        <div class="cover-box">
          <i class="el-icon-circle-plus" />
          <div>
            将{{ coverTxt }}拖到此处，或
            <em>点击上传</em>
          </div>
          <div class="tips">
            支持{{
              accept === "image/*" ? ".jpg,.jpeg,.png,.gif,.bmp" : accept
            }}文件
            <br />
            （大小不超过{{ maxSize }}MB）
          </div>
        </div>
      </template>
      <template v-else-if="listType === 'identity'">
        <div class="identtiy-box">
          <img v-if="innerIcon" width="80" :src="innerIcon" />
          <i v-else style="font-size: 28px" class="el-icon-picture" />
        </div>
      </template>
      <div v-else class="picture-box">
        <el-button
          style="
            width: 100px;
            height: 100px;
            text-align: center;
            line-height: 100px;
          "
        >
          <img v-if="innerIcon" width="28" :src="innerIcon" />
          <i v-else style="font-size: 28px" class="el-icon-picture" />
        </el-button>
      </div>
    </el-upload>
    <div v-if="listType === 'text' && fileList.length > 0" class="file-box">
      <div v-for="(item, key) in fileList" :key="key" class="">
        <!-- 根据文件类型显示图标 -->
        <template v-if="item.status === 'success' || item.status === undefined">
          <div class="file-list-item">
            <div class="top-box">
              <img
                :src="getAttachType( item.name || item.fileName || item.url)"
                alt
              />
              <!-- <img src="../../assets/images/file-icon.png" alt="" /> -->
              <div v-if="!disabled" class="demo-upload-list-cover">
                <i class="el-icon-delete" @click="handleRemove(item, key)" />
              </div>
            </div>
            <p :title="item.name || item.fileName || item.url">
              {{ item.name || item.fileName || item.url }}
            </p>
          </div>
          <!-- <div class="attach-icon-box">
            <img :src="getAttachType(item.name || item.fileCaption)" alt />
          </div>
          <div class="attach-name-box" :title="item.name || item.fileCaption">
            {{ item.name || item.fileCaption }}
          </div>
          <div v-show="!disabled" class="attach-operate-box">
            <span class="check-box" @click="changeShow(item, key)">
              <i
                v-show="item.show === undefined ? 0 : !item.show"
                class="el-icon-circle-close"
                style="color: #ff4949"
              />
              <i
                v-show="item.show === undefined ? 1 : item.show"
                class="el-icon-circle-check"
              />
              <label style="disply: inline-block">对外展示</label>
            </span>
            <span class="delete-text" @click="handleRemove(item, key)"
              >删除</span
            >
          </div> -->
        </template>
        <template
          v-else-if="item.status !== 'success' && item.status !== undefined"
        >
          <el-progress :percentage="item.percentage" color="#409eff" />
        </template>
      </div>
    </div>
    <!-- <p class="upload-tips" v-if="uploadTips" v-html="uploadTips"></p> -->
    <p v-if="listType === 'identity'" style="text-align: center">
      {{
        tname === "identityFront"
          ? "证件正面"
          : tname === "identityBack"
          ? "证件背面"
          : "手持证件照"
      }}
    </p>
    <el-dialog :visible.sync="dialogVisible" :append-to-body="true">
      <img width="100%" :src="baseShowHref + dialogImageUrl" alt />
    </el-dialog>
  </div>
</template>

<script>
import { getToken } from "@/utils/auth";
export default {
  name: "uploadAttach",
  data() {
    return {
      fileList: [],
      tnameList: ["identityFront", "identityBack", "handIdentityPhoto"],
      showFileArr: [1, 1, ...[1]],
      dialogVisible: false,
      delVisible: false,
      responseNum: 0,
      dialogImageUrl: "",
      imgData: {
        aep: require("@/assets/images/exicon/aep.png"),
        ai: require("@/assets/images/exicon/ai.png"),
        avi: require("@/assets/images/exicon/avi.png"),
        cdr: require("@/assets/images/exicon/cdr.png"),
        css: require("@/assets/images/exicon/css.png"),
        doc: require("@/assets/images/exicon/doc.png"),
        eps: require("@/assets/images/exicon/eps.png"),
        gif: require("@/assets/images/exicon/gif.png"),
        html: require("@/assets/images/exicon/html.png"),
        jpeg: require("@/assets/images/exicon/jpeg.png"),
        jpg: require("@/assets/images/exicon/jpeg.png"),
        mov: require("@/assets/images/exicon/mov.png"),
        mp3: require("@/assets/images/exicon/mp3.png"),
        new: require("@/assets/images/exicon/new.png"),
        pdf: require("@/assets/images/exicon/pdf.png"),
        php: require("@/assets/images/exicon/php.png"),
        png: require("@/assets/images/exicon/png.png"),
        ppt: require("@/assets/images/exicon/ppt.png"),
        psd: require("@/assets/images/exicon/psd.png"),
        rar: require("@/assets/images/exicon/rar.png"),
        ttf: require("@/assets/images/exicon/ttf.png"),
        txt: require("@/assets/images/exicon/txt.png"),
        xls: require("@/assets/images/exicon/xls.png"),
        zip: require("@/assets/images/exicon/zip.png"),
        unknown: require("@/assets/images/exicon/file.png"),
        // unknown: require('@/assets/images/attachicon/unknown.png')
      },

      showFileList: false, // 是否显示默认列表样式----->不显示
      // 默认配置项
      BaseHref: process.env.VUE_APP_BASE_API,
      baseShowHref: process.env.VUE_APP_BASE_API,
    };
  },
  props: {
    action: {
      type: String,
      default: "/common/upload", // 请求链接
    },
    headers: {
      type: Object,
      default: () => {
        return {
          Accept: "application/json; charset=UTF-8",
          Authorization: "Bearer " + getToken(),
        };
      },
    },

    data: {
      type: Object,
      default: () => {
        return {};
      },
    },
    uploadTips: { type: String, default: "只能上传格式为.jpg,.png的图片" },
    coverTxt: { type: String, default: "封面" }, // 封面文本

    name: { type: String, default: "file" }, // 上传文件字段名

    tname: { type: String, default: "coverImg" },
    // 上传文件字段名

    multiple: {
      type: Boolean,
      default: true,
    },
    // 是否多附件上传

    accept: { type: String, default: "image/*" },
    // 接受上传格式参考链接：https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/file#Unique_file_type_specifiers900

    drag: { type: Boolean, default: true },
    // 是否允许拖拽上传

    listType: { type: String, default: "picture" },
    // 文件列表的类型 <text/picture/identity/coverImg/piclist>

    autoUpload: { type: Boolean, default: true },
    // 是否选择文件后立即上传

    disabled: { default: false, type: Boolean },
    // 是否禁用

    limit: { type: Number, default: 1 },
    // 最大允许上传个数默认只能上传一个附件

    maxSize: { default: 2, type: Number },
    // 单个文件大小最大值

    downloadAttach: { type: Boolean, default: true },
    // 是否允许下载

    viewAttach: { type: Boolean, default: true },
    // 是否允许预览

    deleteAttach: { type: Boolean, default: true },
    // 是否允许删除

    uploadUrlType: { type: String, default: "project" },
    // 根据上传文件Url判断删除逻辑

    upLoadBtnTxt: { type: String, default: "添加附件" },

    innerIcon: { type: String, default: "" },

    dataPid: {
      default: () => {
        return {};
      },
    }, // 操作回传值
    // 返回的Url 名称
    urlName: {
      type: String,
      default: "fileUrl",
    },
    value: {
      type: [Object, Array, String],
      default: () => {
        return "";
      },
    },
  },
  watch: {
    value(val, oldVal) {
      console.log(val)
      if (this.limit > 1 && val.length === undefined) {
        return;
      }
      if (
        !val ||
        (this.limit > 1 && val.length === 0) ||
        (this.limit <= 1 && val === "")
      ) {
        this.fileList.splice(0);
        return;
      }
      this.$nextTick(() => {
        if (!Array.isArray(val)) {
          this.fileList = [];
          // if(this.listType=='text'){
          //   this.fileList.push(val);
          //   return
          // }
          let params = {};
          params.status = "success";
          params.url = val;
          this.fileList.push(params);
        } else {
          this.fileList = val.slice(0);
        }
      });
    },
  },
  methods: {
    changeShow(file, key) {
      this.$set(
        this.fileList[key],
        "show",
        this.fileList[key].show === 0 ? 1 : 0
      );
      this.fileList = this.fileList.slice(0);
    },

    handlePreview(file) {
      this.dialogVisible = true;
      this.dialogImageUrl = file.url || file[this.urlName];
    },

    dispatch(componentName, eventName, params) {
      // 使用了elFormItem组件作为父组件，该图片上传组件的v-model绑定的数据更新时，通知elFormItem组件
      let parent = this.$parent || this.$root;
      let name = parent.$options.componentName;

      while (parent && (!name || name !== componentName)) {
        parent = parent.$parent;

        if (parent) {
          name = parent.$options.componentName;
        }
      }
      if (parent) {
        // parent.$emit(eventName, ...params)
        if (Array.isArray(params)) {
          parent.$emit.apply(parent, [eventName, ...params]);
        } else {
          parent.$emit.apply(parent, [eventName, params]);
        }
      }
    },
    // 文件下载方法
    handleDownload(item) {
      console.log(item);
    },

    // 文件超出个数限制时的钩子
    handleExceed(file, fileList) {
      this.$message.warning(
        `当前限制选择 ${this.limit}  个文件，本次选择了 ${
          file.length
        } 个文件，共选择了 ${file.length + fileList.length} 个文件`
      );
    },

    // 获取附件名称
    getAttachType(url) {
      if(!url){
        return 
      }
      const nowUrl = url;
      let pos = nowUrl.lastIndexOf("/");
      let commonTypeList = ["xlsx","pptx","docx"];
      if (pos === -1) {
        pos = url.lastIndexOf("\\");
      }
      const fileName = url.substr(pos + 1);
      let extension = fileName.split(".").pop();
      if (commonTypeList.indexOf(extension) >= 0) {
        extension = extension.slice(0, extension.length - 1);
      }
      if (extension && this.imgData[extension]) {
        return this.imgData[extension];
      } else return this.imgData.unknown;
    },

    // 删除附件方法
    handleRemove(file, index) {
      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.handledeleteAttach(file, index);
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },

    // 删除请求
    handledeleteAttach(file, index) {
      this.fileList.splice(index, 1);
      const fileList = this.fileList.slice(0);
      this.$emit("deleteHeader", "1");
      this.responseNum = this.fileList.length;
      if (this.limit === 1&&this.listType != "text") {
        this.dispatchUplaodedList({});
      } else {
        this.dispatchUplaodedList(fileList);
      }
    },

    handleChange(file, fileList) {
      if (file.status === "success") {
        if (file.response && file.response.code != 200) {
          const index = fileList.indexOf(file);
          this.fileList.splice(index, 1);
          return;
        }
        this.fileList = fileList.slice(0);
        if (this.limit === 1&&this.listType != "text") {
          this.dispatchUplaodedList(file);
        } else {
          const newArr = this.fileList.slice(0);
          for (let i = 0; i < newArr.length; i++) {
            newArr[i].response = "";
          }
          this.dispatchUplaodedList(newArr);
        }
      }
    },

    dispatchUplaodedList(obj) {
      if (this.limit == 1&&this.listType!='text') {
        this.$emit("input", obj.url);
      } else {
        this.$emit("input", obj);
      }
      this.$nextTick(() => {
        this.dispatch("ElFormItem", "el.form.change", obj);
      });
    },

    handleError(err, file, fileList) {
      console.log(err);
      this.$message({
        type: "warning",
        message: err.message || "网络异常，请稍后再试",
        duration: 4000,
      });
      for (let i = 0; i < this.fileList.length; i++) {
        if (this.fileList[i].uid === file.uid) {
          this.fileList.splice(i, 1);
          if (this.limit === 1&&this.listType !="text") {
            this.dispatchUplaodedList({});
          } else {
            this.dispatchUplaodedList(fileList);
          }
          break;
        }
      }
    },

    handleSuccess(response, file, fileList) {
      if (response.code == "401") {
        this.$message({
          type: "warning",
          message: response.msg,
          duration: 4000,
        });
        // this.removeUserName('userName')
        this.$router.push({ name: "login" });
        return;
      } else if (response.code != 200) {
        this.$message({
          type: "warning",
          message: response.msg,
        });
        return;
      } else {
        this.responseNum++;
        if (this.responseNum == fileList.length) {
          this.$message({
            type: "success",
            message: "上传成功",
            duration: 4000,
          });
        }
      }
      const { fileName } = response;
      this.$set(file, "url", fileName);
      this.$set(file, this.urlName, fileName);
      this.$set(file, "fileName", file.name);
    },

    // 上传前的钩子用于判断文件的类型和文件的大小
    handleBeforeUpload(file) {
      const imgAccept = ".jpg,.png,"; // 接受的图片格式类型
      let attachAccept =
        ".jpg,.jpeg,.png,.gif,.bmp,.doc,.docx,.ppt,.pptx,.xls,.xlsx,.pdf"; // 接受的附件格式类型
      const isIMAGE =
        [
          "image/jpeg",
          "image/gif",
          "image/png",
          "image/bmp",
          "image/jpg",
        ].indexOf(file.type) >= 0; // 验证图片类型
      if (this.accept && this.accept !== "image/*") {
        // 有传值则用传值做验证
        attachAccept = this.accept;
      }

      const fileFormat = file.name.split(".").pop().toLocaleLowerCase(); // 获取文件扩展名
      const checked = attachAccept
        .split(",")
        .some((item) => item.toLocaleLowerCase() === "." + fileFormat); // 验证是否存在改扩展名的格式
      const isLimit = file.size / 1024 / 1024 < this.maxSize; // 验证文件大小是否超过限制
      if (this.listType !== "text" && !isIMAGE) {
        this.$message({
          type: "warning",
          message: "上传文件只能是图片(" + imgAccept + ")格式!",
          duration: 2000,
        });
        return false;
      } else if (this.listType === "text" && !checked) {
        this.$message({
          type: "warning",
          message: "上传文件只能是(" + attachAccept + ")格式!",
          duration: 2000,
        });
        return false;
      }
      if (!isLimit) {
        this.$message({
          type: "warning",
          message: "上传文件大小不能超过 " + this.maxSize + "MB!",
          duration: 2000,
        });
      }
      return (checked || isIMAGE) && isLimit;
    },
  },

  // methods
  /**
   * 附件预览方法
   * 接收文件对象值
   */
};
</script>

<style lang='scss' scoped>
.file-box {
  display: flex;
  padding-top: 20px;
  flex-direction: row;
  .file-list-item {
    .top-box {
      &:hover {
        .demo-upload-list-cover {
          display: flex;
        }
      }
      height: 48px;
    }
    cursor: pointer;
    display: flex;
    max-width: 150px;
    flex-direction: column;
    align-items: center;
    margin-right: 20px;
    position: relative;
    .demo-upload-list-cover {
      align-items: center;
      justify-content: center;
    }
    p {
      font-size: 14px;
      width: 100%;
      font-weight: 500;
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
      color: #333333;
    }
    img {
      width: 47px;
    }
  }
}
.attach-box {
  .upload-tips {
    font-size: 8px;
    color: #ff4949;
    margin: 0;
    line-height: 20px;
  }
  display: inline-block;
  width: 100%;
  min-width: 140px;
  ::v-deep .el-upload-dragger {
    background-color: #fff;
    border: none;
    border-radius: 0px;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    width: 100%;
    height: 100%;
    text-align: center;
    position: relative;
    overflow: hidden;
    &.is-dragover {
      .cover-box {
        border: 1px dashed #409eff;
        // border-color: #409EFF;
      }
    }
  }
  .el-upload--text {
    border: none;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    width: 100%;
    height: 36px;
    line-height: 36px;
    text-align: left;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .img-list {
    display: inline-block;
    vertical-align: middle;
    width: 100px;
    height: 100px;
    text-align: center;
    line-height: 100px;
    border: 1px solid transparent;
    border-radius: 4px;
    overflow: hidden;
    background: #fff;
    position: relative;
    box-shadow: 0 1px 1px rgba(0, 0, 0, 0.2);
    margin-right: 4px;
  }
  .img-list img {
    width: 100%;
    height: 100%;
  }
  .user-btn {
    cursor: pointer;
    padding: 5px 0px;
    display: block;
    color: #10a44a;
    border: 1px solid #10a44a;
    font-size: 16px;
    transition: all 0.5s;
    height: 36px;
    line-height: 24px;
    &:hover {
      color: #fff;
      background: #99b2f2;
      border: 1px solid #99b2f2;
    }
  }
  .demo-upload-list-cover {
    display: none;
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    background: rgba(0, 0, 0, 0.6);
  }
  .img-list:hover .demo-upload-list-cover {
    display: block;
  }
  .demo-upload-list-cover i {
    color: #fff;
    font-size: 20px;
    cursor: pointer;
    margin: 0 2px;
  }
  .inner-icon {
    width: 70px;
    height: 70px;
    text-align: center;
  }
  .attach-list-box {
    width: 100%;
    min-width: 400px;
    text-align: left;
    font-size: 12px;
    display: inline-block;
    .attach-list {
      max-width: 490px;
      height: 48px;
      margin: 5px 0;
    }
    .attach-list:hover {
      background: #f3f3f3;
    }
    .attach-list:hover .attach-operate-box {
      display: block;
    }
    .attach-icon-box,
    .attach-name-box,
    .attach-operate-box {
      display: inline-block;
      color: #6ea5ff;
      font-size: 14px;
      height: 48px;
      line-height: 48px;
      vertical-align: middle;
      > span {
        float: left;
        cursor: pointer;
        padding-left: 5px;
      }
      .check-box {
        position: relative;
        width: 110px;
        display: flex;
        height: 48px;
        padding: 13px 0;
        font-size: 14px;
        flex-direction: row;
        flex-wrap: nowrap;
        align-items: center;
        &::after {
          content: "";
          position: absolute;
          width: 1px;
          height: 16px;
          top: 16px;
          right: 12px;
          background: #6ea5ff;
        }
      }
    }
    .attach-operate-box i {
      cursor: pointer;
      display: block;
      width: 24px;
      font-size: 22px;
    }
    .attach-icon-box {
      width: 48px;
      height: 48px;
      padding: 4px;
      float: left;
    }
    .attach-icon-box img {
      width: 100%;
      height: 100%;
    }
    .attach-name-box {
      width: 220px;
      height: 48px;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      float: left;
      padding-left: 5px;
    }
    .attach-operate-box {
      width: 180px;
      font-size: 14px;
      float: right;
    }
  }
  .cover-box-list {
    width: 300px;
    height: 150px;
    border: 1px solid #10a44a;
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    .img-box {
      width: 100%;
      height: 100%;
      img {
        width: 100%;
        height: 100%;
      }
    }

    &:hover .cover-handle {
      display: flex;
      flex-direction: column;
      justify-content: center;
    }
    .cover-handle {
      width: 100%;
      height: 100%;
      display: none;
      position: absolute;
      top: 0;
      bottom: 0;
      left: 0;
      right: 0;
      background: rgba(0, 0, 0, 0.6);
      text-align: center;
      i {
        font-size: 20px;
        color: #fff;
        cursor: pointer;
      }
    }
  }
  .demo-upload-list {
    display: inline-block;
    width: 100px;
    height: 100px;
    text-align: center;
    line-height: 100px;
    border: 1px solid transparent;
    border-radius: 4px;
    overflow: hidden;
    background: #fff;
    position: relative;
    box-shadow: 0 1px 1px rgba(0, 0, 0, 0.2);
    margin-right: 4px;
    vertical-align: middle;
  }
  .demo-upload-list img {
    width: 100%;
    height: 100%;
  }

  .demo-upload-list:hover .demo-upload-list-cover {
    display: block;
  }

  .identtiy-box {
    width: 140px;
    height: 90px;
    text-align: center;
    line-height: 90px;
    border: 1px solid #10a44a;
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    &:hover .cover-handle {
      display: block;
    }
    .cover-handle {
      display: none;
      width: 100%;
      height: 100%;
      display: none;
      position: absolute;
      top: 0;
      bottom: 0;
      left: 0;
      right: 0;
      background: rgba(0, 0, 0, 0.6);
      text-align: center;
      i {
        font-size: 20px;
        color: #fff;
        margin-top: 35px;
        cursor: pointer;
      }
    }
    img {
      width: 100%;
      height: 100%;
    }
  }
  .cover-box {
    width: 300px;
    height: 150px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    border: 1px solid #10a44a;
    &:hover {
      border: 1px dashed #409eff;
      // border-color: #409EFF;
    }
    div {
      height: 24px;
      line-height: 24px;
      font-weight: bold;
      &.tips {
        font-weight: normal;
        color: #99b2f2;
      }
    }
    i {
      color: #99b2f2;
      font-size: 40px;
    }
  }
  .tips {
    font-weight: normal;
    color: #999999;
    font-size: 14px;
    float: left;
    display: flex;
    padding-left: 10px;
    flex-direction: row;
    align-items: center;
    height: 38px;
    // clear: both;
  }
  .upload-demo {
    vertical-align: middle;
    line-height: 1;
  }
}
</style>
