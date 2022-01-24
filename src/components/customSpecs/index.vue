<!--
 * @Author: your name
 * @Date: 2021-07-08 15:38:01
 * @LastEditTime: 2021-07-08 17:56:52
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \qingyuan-resturant-admin\src\components\customSpecs\index.vue
-->
<template>
  <div>
    <el-tag
      :key="'tag'+index"
      v-for="(tag,index) in dynamicTags"
      :size="tagSize"
      closable
      :disable-transitions="false"
      @close="handleClose(tag)"
    >
      {{ tag.specificationsName }}
    </el-tag>
    <el-input
      class="input-new-tag"
      v-if="inputVisible"
      v-model="inputValue"
      ref="saveTagInput"
      :size="buttonSize"
      @keyup.enter.native.stop.prevent="handleInputConfirm"
      @blur.prevent.stop="handleInputConfirm"
    >
    </el-input>
    <el-button v-else-if='!inputVisible&&dynamicTags.length<maxNum&&!disabled' class="button-new-tag" size="small" @click="showInput"
      >+ 新 增 </el-button
    >
  </div>
</template>

<script>
export default {
  props: {
    // 值
    value: [Array],
    // 大小限制(MB)
    maxNum:{
      type:Number,
      default:6
    },
    tagSize: {
      type: String,
      default: 'small',
    },
    disabled: { default: false, type: Boolean },

    // 文件类型, 例如['png', 'jpg', 'jpeg']
    buttonSize: {
      type: String,
      default: 'small',
    },
  },
  watch: {
    value (val, oldVal) {
      if (!val) {
        this.dynamicTags.splice(0)
        return
      }
      this.$nextTick(() => {
        if (!Array.isArray(val)) {
          this.dynamicTags = []
        } else {
          this.dynamicTags = val
        }
      })
    }
  },
  // computed: {
  //   // 列表
  //   list() {
  //     if (this.value) {
  //       console.log(this.value)
  //       // 首先将值转为数组
  //       const list = Array.isArray(this.value) ? this.value : [this.value];
  //       // 然后将数组转为对象数组
  //       return list.map((item) => {
  //         if (typeof item === "string") {
  //           item = { name: item, url: item };
  //         }
  //         return item;
  //       });
  //     } else {
  //       this.dynamicTags = [];
  //       return [];
  //     }
  //   },
  // },
  data () {
    return {
      dynamicTags: [],
      inputVisible: false,
      inputValue: ''
    }
  },
  // created() {
  //   this.dynamicTags = this.list;
  // },
  methods: {
    handleClose (tag) {
      if(this.disabled){
        return;
      }
      this.dynamicTags.splice(this.dynamicTags.indexOf(tag), 1)
      this.$emit("input",this.dynamicTags );
    },
    showInput () {
      this.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },

    handleInputConfirm () {
      let inputValue = this.inputValue
      let tagObj = {
        specificationsName:inputValue
      }
      if (inputValue) {
        this.dynamicTags.push(tagObj)
      }
      this.inputVisible = false
      this.inputValue = ''
      this.$emit("input", this.dynamicTags);
    }
  }
}
</script>

<style lang="scss" scoped>
.input-new-tag,.button-new-tag {
    width: 90px;
    margin-left: 10px;
    vertical-align: bottom
}
.el-tag+.el-tag {
    margin-left: 5px;
}
</style>