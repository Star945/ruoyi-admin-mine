<template>
  <div
    :class="{ fullscreen: fullscreen }"
    class="tinymce-container"
    :style="{ width: containerWidth }"
  >
    <textarea :id="tinymceId" class="tinymce-textarea" />
    <div class="editor-custom-btn-container">
      <editorImage
        color="#1890ff"
        class="editor-upload-btn"
        @successCBK="imageSuccessCBK"
      />
    </div>
  </div>
</template>

<script>
import editorImage from "./components/EditorImage"
import plugins from "./plugins"
import toolbar from "./toolbar"
import tinymce from "tinymce/tinymce"
import "tinymce/icons/default/icons";
import "tinymce/themes/silver";
import "tinymce/plugins/image";
import "tinymce/plugins/media";
import "tinymce/plugins/table";
import "tinymce/plugins/lists";
import "tinymce/plugins/contextmenu";
import "tinymce/plugins/wordcount";
import "tinymce/plugins/colorpicker";
import "tinymce/plugins/textcolor";
import "tinymce/plugins/preview";
import "tinymce/plugins/code";
import "tinymce/plugins/link";
import "tinymce/plugins/advlist";
import "tinymce/plugins/codesample";
import "tinymce/plugins/hr";
import "tinymce/plugins/fullscreen";
import "tinymce/plugins/textpattern";
import "tinymce/plugins/searchreplace";
import "tinymce/plugins/autolink";
import "tinymce/plugins/directionality";
import "tinymce/plugins/visualblocks";
import "tinymce/plugins/visualchars";
import "tinymce/plugins/template";
import "tinymce/plugins/charmap";
import "tinymce/plugins/nonbreaking";
import "tinymce/plugins/insertdatetime";
import "tinymce/plugins/imagetools";
import "tinymce/plugins/autosave";
import "tinymce/plugins/autoresize";
import "tinymce/plugins/anchor";
import "tinymce/plugins/emoticons";
import "tinymce/plugins/noneditable";
import "tinymce/plugins/paste";
import "tinymce/plugins/print";
import "tinymce/plugins/pagebreak";
import "tinymce/plugins/save";
import "tinymce/plugins/spellchecker";
import "tinymce/plugins/tabfocus";

export default {
  name: "Tinymce",
  components: { editorImage },
  props: {
    id: {
      type: String,
      default: function () {
        return (
          "vue-tinymce-" +
          +new Date() +
          ((Math.random() * 1000).toFixed(0) + "")
        )
      }
    },
    value: {
      type: String,
      default: ""
    },
    toolbar: {
      type: Array,
      required: false,
      default() {
        return []
      }
    },
    menubar: {
      type: String,
      default: "file edit insert view format table"
    },
    height: {
      type: [Number, String],
      required: false,
      default: 360
    },
    disabled: {
      type: [Boolean, String],
      default: false
    },
    width: {
      type: [Number, String],
      required: false,
      default: "auto"
    }
  },
  data() {
    return {
      hasChange: false,
      hasInit: false,
      tinymceId: this.id,
      fullscreen: false,
      baseShowHref: process.env.VUE_APP_BASE_API,
      inputChange: false,
      languageTypeList: {
        zh: "zh_CN",
      }
    }
  },
  computed: {
    containerWidth() {
      const width = this.width
      if (/^[\d]+(\.[\d]+)?$/.test(width)) {
        // matches `100`, `'100'`
        return `${width}px`
      }
      return width
    }
  },
  watch: {
    value(val, oldVal) {
      console.log('val:' + val, 'oldVal:' + oldVal)
      if (val !== "" && oldVal == "") {
        this.hasChange = false
      }
      if (val == "" && oldVal != "") {
        this.inputChange = false
        window.tinymce.get(this.tinymceId).setContent(val || "")
      }
      if (!this.hasChange && this.hasInit) {
        if (!this.inputChange) {
          this.$nextTick(() =>
            window.tinymce.get(this.tinymceId).setContent(val || "")
          )
        }
        this.dispatch("ElFormItem", "el.form.change", val || "")
      }
    }
  },
  mounted() {
    this.init()
  },
  activated() {
  },
  deactivated() {
    this.destroyTinymce()
  },
  destroyed() {
    this.destroyTinymce()
  },
  methods: {
    init() {
      this.initTinymce()
    },
    initTinymce() {
      const _this = this
      tinymce.init({
        selector: `#${this.tinymceId}`,
        language_url: "/static/tinymce/langs/zh_CN.js",
        skin_url: "/static/tinymce/skins/ui/oxide",
        content_css : '/static/tinymce/skins/content/default/content.css',
        emoticons_database_url:'/static/tinymce/plugins/emoticons/js/emojis.js',
        skin: 'oxide-light',
        language: this.languageTypeList["zh"],
        height: this.height,
        body_class: "panel-body ",
        object_resizing: false,
        toolbar: this.toolbar.length > 0 ? this.toolbar : toolbar,
        menubar: this.menubar,
        plugins: plugins,
        paste_as_text: false,
        fontsize_formats: "12px 14px 16px 18px 20px 22px 24px 26px 28px 30px 32px 34px 36px",
        end_container_on_empty_block: true,
        powerpaste_word_import: "clean",
        code_dialog_height: 350,
        code_dialog_width: 1000,
        advlist_bullet_styles: "square",
        advlist_number_styles: "default",
        imagetools_cors_hosts: ["www.tinymce.com", "codepen.io"],
        default_link_target: "_blank",
        link_title: false,
        nonbreaking_force_tab: true, // inserting nonbreaking space &nbsp; need Nonbreaking Space Plugin
        init_instance_callback: editor => {
          if (_this.value) {
            editor.setContent(_this.value)
          }
          _this.hasInit = true
          editor.on("NodeChange Change SetContent", () => {
            this.hasChange = true
            this.$emit("input", editor.getContent())
          })
          editor.on("KeyUp", () => {
            if (editor.getContent()) {
              this.inputChange = true
            }
            this.hasChange = true
            this.$emit("input", editor.getContent())
          })
        },
        setup(editor) {
          editor.on("FullscreenStateChanged", e => {
            _this.fullscreen = e.state
          })
          editor.on('init', function (e) {
            this.getBody().style.fontSize = '16px'
            this.getBody().style.fontFamily = '微软雅黑'
          })
        },
        // it will try to keep these URLs intact
        // https://www.tiny.cloud/docs-3x/reference/configuration/Configuration3x@convert_urls/
        // https://stackoverflow.com/questions/5196205/disable-tinymce-absolute-to-relative-url-conversions
        convert_urls: false
        // 整合七牛上传
        // images_dataimg_filter(img) {
        //   setTimeout(() => {
        //     const $image = $(img);
        //     $image.removeAttr('width');
        //     $image.removeAttr('height');
        //     if ($image[0].height && $image[0].width) {
        //       $image.attr('data-wscntype', 'image');
        //       $image.attr('data-wscnh', $image[0].height);
        //       $image.attr('data-wscnw', $image[0].width);
        //       $image.addClass('wscnph');
        //     }
        //   }, 0);
        //   return img
        // },
        // images_upload_handler(blobInfo, success, failure, progress) {
        //   progress(0);
        //   const token = _this.$store.getters.token;
        //   getToken(token).then(response => {
        //     const url = response.data.qiniu_url;
        //     const formData = new FormData();
        //     formData.append('token', response.data.qiniu_token);
        //     formData.append('key', response.data.qiniu_key);
        //     formData.append('file', blobInfo.blob(), url);
        //     upload(formData).then(() => {
        //       success(url);
        //       progress(100);
        //     })
        //   }).catch(err => {
        //     failure('出现未知问题，刷新页面，或者联系程序员')
        //     console.log(err);
        //   });
        // },
      })
    },
    destroyTinymce() {
      const tinymce = window.tinymce.get(this.tinymceId)
      if (this.fullscreen) {
        tinymce.execCommand("mceFullScreen")
      }

      if (tinymce) {
        tinymce.destroy()
      }
    },
    dispatch(componentName, eventName, params) {
      // 使用了elFormItem组件作为父组件，该图片上传组件的v-model绑定的数据更新时，通知elFormItem组件
      let parent = this.$parent || this.$root
      let name = parent.$options.componentName

      while (parent && (!name || name !== componentName)) {
        parent = parent.$parent

        if (parent) {
          name = parent.$options.componentName
        }
      }
      if (parent) {
        // parent.$emit(eventName, ...params)
        if (Array.isArray(params)) {
          parent.$emit.apply(parent, [eventName, ...params])
        } else {
          parent.$emit.apply(parent, [eventName, params])
        }
      }
    },
    setContent(value) {
      window.tinymce.get(this.tinymceId).setContent(value)
    },
    getContent() {
      window.tinymce.get(this.tinymceId).getContent()
    },
    imageSuccessCBK(arr) {
      const _this = this
      arr.forEach(v => {
        window.tinymce
          .get(_this.tinymceId)
          .insertContent(`<img class="wscnph" src=" ${v.url}" >`)
      })
    }
  }
};
</script>

<style scoped>
.tinymce-container {
  position: relative;
  line-height: normal;
}
.tinymce-container ::v-deep .mce-fullscreen {
  z-index: 1000000000;
}
.tinymce-textarea {
  visibility: hidden;
  z-index: -1;
}
.editor-custom-btn-container {
  position: absolute;
  right: 4px;
  top: 4px;
  z-index: 200500000;
}
.fullscreen .editor-custom-btn-container {
  z-index: 1000000000;
  position: fixed;
}
.editor-upload-btn {
  display: inline-block;
}
i.mce-i-[FONT-AWESOME-CLASSNAME]:before {
  content: "[FONT-AWESOME-CONTENT]";
  font-family: FontAwesome;
  font-style: normal;
  font-weight: normal;
  text-decoration: inherit;
  color: #000;
  font-size: 1.5em;
  padding-right: 0.5em;
  position: absolute;
  top: 15%;
  left: 0;
}
</style>
