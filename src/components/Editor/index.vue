<template>
  <div
    class="editor-box-wrapper"
    :style="{ height: (height || minHeight) + 66 + 'px' }"
  >
    <div class="editor" ref="editor" :style="styles"></div>
    <el-upload
      class="avatar-uploader"
      :action="BaseHref + action"
      :name="name"
      :accept="accept"
      :headers="headers"
      :show-file-list="false"
      :on-success="uploadSuccess"
      :on-error="uploadError"
      :before-upload="beforeUpload"
    >
    </el-upload>
  </div>
</template>

<script>
import { getToken } from "@/utils/auth"
import Quill from "quill"
import "quill/dist/quill.core.css"
import "quill/dist/quill.snow.css"
import "quill/dist/quill.bubble.css"

export default {
  name: "Editor",
  props: {
    action: {
      type: String,
      default: "/common/upload" // 请求链接
    },
    maxSize: { default: 2, type: Number },
    accept: { type: String, default: ".png,.jpg" },
    headers: {
      type: Object,
      default: () => {
        return {
          Accept: "application/json; charset=UTF-8",
          Authorization: "Bearer " + getToken(),
        }
      }
    },
    name: { type: String, default: 'file' },
    /* 编辑器的内容 */
    value: {
      type: String,
      default: "",
    },
    lazy: false,
    /* 高度 */
    height: {
      type: Number,
      default: 400,
    },
    /* 最小高度 */
    minHeight: {
      type: Number,
      default: 400,
    },
  },
  data () {
    return {
      BaseHref: process.env.NODE_ENV == 'production' ? process.env.VUE_APP_BASE_API : 'http://192.168.0.107:8082',
      baseShowHref: process.env.NODE_ENV == 'production' ? process.env.VUE_APP_IMG_BASE_HREF : 'http://192.168.0.107:8082',
      Quill: null,
      articleImgUrl: '',
      currentValue: "",
      options: {
        theme: "snow",
        bounds: document.body,
        debug: "warn",
        modules: {
          // 工具栏配置
          toolbar: [
            ["bold", "italic", "underline", "strike"],       // 加粗 斜体 下划线 删除线
            ["blockquote", "code-block"],                    // 引用  代码块
            [{ list: "ordered" }, { list: "bullet" }],       // 有序、无序列表
            [{ indent: "-1" }, { indent: "+1" }],            // 缩进
            [{ size: ["small", false, "large", "huge"] }],   // 字体大小
            [{ header: [1, 2, 3, 4, 5, 6, false] }],         // 标题
            [{ color: [] }, { background: [] }],             // 字体颜色、字体背景颜色
            [{ align: [] }],                                 // 对齐方式
            ["clean"],                                       // 清除文本格式
            ["link", "image", "video"]
          ],
        },
        placeholder: "请输入内容",
        readOnly: false,
      },
    }
  },
  computed: {
    styles () {
      let style = {}
      if (this.minHeight) {
        style.minHeight = `${this.minHeight}px`
      }
      if (this.height) {
        style.height = `${this.height}px`
      }
      return style
    },
  },
  watch: {
    value: {
      handler (val) {
        if (val !== this.currentValue) {
          this.currentValue = val === null ? "" : val
          if (this.Quill) {
            this.Quill.pasteHTML(this.currentValue)
          }
        }
      },
      immediate: true,
    },
  },
  mounted () {
    if (!this.lazy) {
      this.init()
    }
  },
  beforeDestroy () {
    this.Quill = null
  },
  methods: {
    uploadError () {
      this.$message({
        type: "warning",
        message: err.message || "网络异常，请稍后再试",
        duration: 4000
      })
    },
    beforeUpload (file) {
      const imgAccept = ".jpg,.png," // 接受的图片格式类型
      let attachAccept =
        ".jpg,.jpeg,.png,.gif,.bmp,.doc,.docx,.ppt,.pptx,.xls,.xlsx,.pdf" // 接受的附件格式类型
      const isIMAGE =
        [
          "image/jpeg",
          "image/gif",
          "image/png",
          "image/bmp",
          "image/jpg"
        ].indexOf(file.type) >= 0 // 验证图片类型

      if (this.accept !== "image/*") {
        // 有传值则用传值做验证
        attachAccept = this.accept
      }
      const fileFormat = file.name
        .split(".")
        .pop()
        .toLocaleLowerCase() // 获取文件扩展名
      const checked = attachAccept
        .split(",")
        .some(item => item.toLocaleLowerCase() === "." + fileFormat) // 验证是否存在改扩展名的格式
      const isLimit = file.size / 1024 / 1024 < this.maxSize // 验证文件大小是否超过限制
      if (this.listType !== "text" && !isIMAGE) {
        this.$message({
          type: "warning",
          message: "上传文件只能是图片(" + imgAccept + ")格式!",
          duration: 2000
        })
        return false
      } else if (this.listType === "text" && !checked) {
        this.$message({
          type: "warning",
          message: "上传文件只能是(" + attachAccept + ")格式!",
          duration: 2000
        })
        return false
      }
      if (!isLimit) {
        this.$message({
          type: "warning",
          message: "上传文件大小不能超过 " + this.maxSize + "MB!",
          duration: 2000
        })
      }
      return (checked || isIMAGE) && isLimit
    },
    uploadSuccess (res) {
      var dt = res
      let quill = this.Quill
      // 如果上传成功
      if (dt.code === 200 && dt.url !== null) {
        // 获取光标所在位置
        let length = quill.getSelection().index
        // 插入图片  dt.url为服务器返回的图片地址
        quill.insertEmbed(length, 'image', this.baseShowHref + dt.fileName)
        // 调整光标到最后
        quill.setSelection(length + 1)
      } else {
        this.$message.error('图片插入失败')
      }
      // loading加载隐藏
      this.quillUpdateImg = false
    },
    init () {
      const editor = this.$refs.editor
      this.Quill = new Quill(editor, this.options)
      let toolbar = this.Quill.getModule('toolbar')
      toolbar.addHandler('image', (value) => {
        if (value) {
          // upload点击上传事件
          console.log('inner')
          document.querySelector('.avatar-uploader input').click()
        } else {
          this.Quill.format('image', false)
        }
      })
      this.Quill.pasteHTML(this.currentValue)
      this.Quill.on("text-change", (delta, oldDelta, source) => {
        const html = this.$refs.editor.children[0].innerHTML
        const text = this.Quill.getText()
        const quill = this.Quill
        this.currentValue = html
        this.$emit("input", html)
        this.$emit("on-change", { html, text, quill })
      })
      this.Quill.on("text-change", (delta, oldDelta, source) => {
        this.$emit("on-text-change", delta, oldDelta, source)
      })
      this.Quill.on("selection-change", (range, oldRange, source) => {
        this.$emit("on-selection-change", range, oldRange, source)
      })
      this.Quill.on("editor-change", (eventName, ...args) => {
        this.$emit("on-editor-change", eventName, ...args)
      })
    },
  },
};
</script>

<style>
.editor,
.ql-toolbar {
  white-space: pre-wrap !important;
  line-height: normal !important;
}
.quill-img {
  display: none;
}
.ql-snow .ql-tooltip[data-mode="link"]::before {
  content: "请输入链接地址:";
}
.ql-snow .ql-tooltip.ql-editing a.ql-action::after {
  border-right: 0px;
  content: "保存";
  padding-right: 0px;
}

.ql-snow .ql-tooltip[data-mode="video"]::before {
  content: "请输入视频地址:";
}

.ql-snow .ql-picker.ql-size .ql-picker-label::before,
.ql-snow .ql-picker.ql-size .ql-picker-item::before {
  content: "14px";
}
.ql-snow .ql-picker.ql-size .ql-picker-label[data-value="small"]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value="small"]::before {
  content: "10px";
}
.ql-snow .ql-picker.ql-size .ql-picker-label[data-value="large"]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value="large"]::before {
  content: "18px";
}
.ql-snow .ql-picker.ql-size .ql-picker-label[data-value="huge"]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value="huge"]::before {
  content: "32px";
}

.ql-snow .ql-picker.ql-header .ql-picker-label::before,
.ql-snow .ql-picker.ql-header .ql-picker-item::before {
  content: "文本";
}
.ql-snow .ql-picker.ql-header .ql-picker-label[data-value="1"]::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value="1"]::before {
  content: "标题1";
}
.ql-snow .ql-picker.ql-header .ql-picker-label[data-value="2"]::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value="2"]::before {
  content: "标题2";
}
.ql-snow .ql-picker.ql-header .ql-picker-label[data-value="3"]::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value="3"]::before {
  content: "标题3";
}
.ql-snow .ql-picker.ql-header .ql-picker-label[data-value="4"]::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value="4"]::before {
  content: "标题4";
}
.ql-snow .ql-picker.ql-header .ql-picker-label[data-value="5"]::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value="5"]::before {
  content: "标题5";
}
.ql-snow .ql-picker.ql-header .ql-picker-label[data-value="6"]::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value="6"]::before {
  content: "标题6";
}

.ql-snow .ql-picker.ql-font .ql-picker-label::before,
.ql-snow .ql-picker.ql-font .ql-picker-item::before {
  content: "标准字体";
}
.ql-snow .ql-picker.ql-font .ql-picker-label[data-value="serif"]::before,
.ql-snow .ql-picker.ql-font .ql-picker-item[data-value="serif"]::before {
  content: "衬线字体";
}
.ql-snow .ql-picker.ql-font .ql-picker-label[data-value="monospace"]::before,
.ql-snow .ql-picker.ql-font .ql-picker-item[data-value="monospace"]::before {
  content: "等宽字体";
}
</style>