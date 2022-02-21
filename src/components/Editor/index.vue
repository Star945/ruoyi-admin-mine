<template>
  <div
    class="editor-box-wrapper"
    :class="editorId"
    :style="{ height: (height || minHeight) + 66 + 'px' }"
  >
    <div class="editor" ref="editor" :id="editorId" :style="styles"></div>
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
let sizes = ['10px', '12px', false, '16px', '20px', '24px', '36px']
let Size = Quill.import("formats/size")
Size.whitelist = sizes
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
          // Authorization: "Bearer " + getToken(),
        }
      }
    },
    name: { type: String, default: 'file' },
    /* 编辑器的内容 */
    value: {
      type: String,
      default: "",
    },
    editorId: null,//一页又多个editor时 为必填不能重复
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
  data() {
    return {
      BaseHref: process.env.VUE_APP_BASE_API,
      baseShowHref: process.env.VUE_APP_BASE_SHOWHREF,
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
            // [{ size: ["small", false, "large", "huge"] }],   // 字体大小
            [{ size: sizes }],
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
    styles() {
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
      handler(val) {
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
  mounted() {
    if (!this.lazy) {
      this.init()
    }
  },
  beforeDestroy() {
    this.Quill = null
  },
  methods: {
    uploadError() {
      this.$message({
        type: "warning",
        message: err.message || "网络异常，请稍后再试",
        duration: 4000
      })
    },
    beforeUpload(file) {
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
    uploadSuccess(res) {
      var dt = res
      console.log(this.Quill)
      let quill = this.Quill
      // 如果上传成功
      if (dt.code === 200 && dt.url !== null) {
        // 获取光标所在位置
        console.log(quill)
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
    init() {
      const editor = this.$refs.editor
      let quillEditor = null
      if (this.editorId) {
        quillEditor = new Quill(`#${this.editorId}`, this.options)
      } else {
        quillEditor = new Quill(editor, this.options)
      }
      this.Quill = quillEditor
      let toolbar = this.Quill.getModule('toolbar')
      let that = this
      toolbar.addHandler('image', (value) => {
        if (value) {
          // upload点击上传事件
          if (that.editorId) {
            document.querySelector(`.${that.editorId} .avatar-uploader input`).click()
            return
          }
          document.querySelector(`.avatar-uploader input`).click()
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

/* 字号设置 */
/* 默认字号 */
/* ['10px', '12px', false, '16px', '20px', '24px', '36px'] */
.ql-snow .ql-picker.ql-size .ql-picker-label::before,
.ql-snow .ql-picker.ql-size .ql-picker-item::before {
  content: "14px";
}
.ql-snow .ql-picker.ql-size .ql-picker-label[data-value="10px"]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value="10px"]::before {
  content: "10px";
  font-size: 10px;
}

.ql-size-10px {
  font-size: 10px;
}
.ql-snow .ql-picker.ql-size .ql-picker-label[data-value="12px"]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value="12px"]::before {
  content: "12px";
  font-size: 12px;
}

.ql-size-12px {
  font-size: 12px;
}

.ql-snow .ql-picker.ql-size .ql-picker-label[data-value="16px"]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value="16px"]::before {
  content: "16px";
  font-size: 16px;
}

.ql-size-16px {
  font-size: 16px;
}

.ql-snow .ql-picker.ql-size .ql-picker-label[data-value="18px"]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value="18px"]::before {
  content: "18px";
  font-size: 18px;
}

.ql-size-18px {
  font-size: 18px;
}

.ql-snow .ql-picker.ql-size .ql-picker-label[data-value="20px"]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value="20px"]::before {
  content: "20px";
  font-size: 20px;
}

.ql-size-20px {
  font-size: 20px;
}

.ql-snow .ql-picker.ql-size .ql-picker-label[data-value="24px"]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value="24px"]::before {
  content: "24px";
  font-size: 24px;
}

.ql-size-24px {
  font-size: 24px;
}

.ql-snow .ql-picker.ql-size .ql-picker-label[data-value="26px"]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value="26px"]::before {
  content: "26px";
  font-size: 26px;
}

.ql-size-26px {
  font-size: 26px;
}

.ql-snow .ql-picker.ql-size .ql-picker-label[data-value="28px"]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value="28px"]::before {
  content: "28px";
  font-size: 28px;
}

.ql-size-28px {
  font-size: 28px;
}

.ql-snow .ql-picker.ql-size .ql-picker-label[data-value="36px"]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value="36px"]::before {
  content: "36px";
  font-size: 36px;
}

.ql-size-36px {
  font-size: 36px;
}
/* .ql-snow .ql-picker.ql-size .ql-picker-label[data-value="small"]::before,
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
.ql-snow .ql-picker.ql-size .ql-picker-label[data-value="super"]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value="super"]::before {
  content: "100px";
} */

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