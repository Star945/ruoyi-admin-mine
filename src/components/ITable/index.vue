<!--
 * @Author: your name
 * @Date: 2021-04-01 15:14:07
 * @LastEditTime: 2021-12-24 09:38:45
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit<!--region 封装的分页 table-->
<!--region 封装的分页 table-->
<template>
  <div class="table">
    <el-table
      id="iTable"
      :row-class-name="tableRowClassName"
      :header-cell-style="getRowClass"
      v-loading="options.loading"
      v-bind="options"
      :border="options.border"
      :data="list"
      :height="noStatic ? customHeight : height"
      :max-height="noStatic ? customHeight : height"
      ref="mutipleTable"
      @sort-change="sortChange"
      @selection-change="handleSelectionChange"
    >
      <!--region 选择框-->
      <el-table-column
        v-if="options.mutiSelect"
        type="selection"
        width="60"
        align="center"
      ></el-table-column>
      <el-table-column
        v-if="options.numbers"
        width="60"
        type="index"
        label="序号"
        align="center"
      ></el-table-column>
      <!--endregion-->
      <!--region 数据列-->
      <template v-for="(column, index) in columns">
        <!-- :render-header="headSpanFit" -->
        <el-table-column
          :render-header="headSpanFit"
          :key="index"
          :prop="column.prop"
          :label="column.label"
          :align="column.align"
          v-bind="column.el"
          :min-width="column.minWidth"
          :width="column.width"
        >
          <template slot-scope="scope">
            <template v-if="!column.render">
              <template v-if="column.formatter">
                <span v-html="column.formatter(scope.row, column)"></span>
              </template>
              <template v-else>
                <span>{{
                  scope.row[column.prop] === 0
                    ? 0
                    : scope.row[column.prop] || "--"
                }}</span>
              </template>
            </template>
            <template v-else>
              <expand-dom
                :column="column"
                :row="scope.row"
                :render="column.render"
                :index="index"
              ></expand-dom>
            </template>
          </template>
        </el-table-column>
      </template>
      <!--endregion-->
      <!--region 按钮操作组-->
      <el-table-column
        ref="fixedColumn"
        label="操作"
        align="center"
        :width="operates.width"
        :fixed="operates.fixed"
        v-if="operates.list && operates.list.length > 0"
      >
        <template slot-scope="scope">
          <div class="operate-group">
            <template v-for="(btn, key) in operates.list">
              <template v-if='btn.render'>
              <expand-dom
                :key="key+'itbtn'"
                :column="column"
                :row="scope.row"
                :render="btn.render"
                :index="key"
              ></expand-dom>
            </template>
              <div
                class="item"
                :key="key+'itbtn'"
                v-if="
                  typeof btn.show == 'function'
                    ? btn.show(scope.$index, scope.row)
                    : btn.show
                "
              >
                <el-button
                  :type="btn.type"
                  :icon="btn.icon"
                  v-if="!btn.haspermission"
                  :size="btn.size || 'mini'"
                  :style="`color:${btn.color}`"
                  :disabled="
                    typeof btn.disabled == 'function'
                      ? btn.disabled(scope.$index, scope.row)
                      : btn.disabled
                  "
                  :plain="btn.plain"
                  @click.native.prevent="btn.method(scope.$index, scope.row)"
                  >{{ btn.label }}</el-button
                >
                <el-button
                  :type="btn.type"
                  :icon="btn.icon"
                  v-else
                  v-hasPermi="btn.haspermission || []"
                  :size="btn.size || 'mini'"
                  :style="`color:${btn.color}`"
                  :disabled="
                    typeof btn.disabled == 'function'
                      ? btn.disabled(scope.$index, scope.row)
                      : btn.disabled
                  "
                  :plain="btn.plain"
                  @click.native.prevent="btn.method(scope.$index, scope.row)"
                  >{{ btn.label }}</el-button
                >
              </div>
            </template>
          </div>
        </template>
      </el-table-column>
      <!--endregion-->
    </el-table>
    <!--region 分页-->
    <div class="pagination-box">
      <el-pagination
        style="float: none; text-align: center"
        :background="true"
        v-if="pagination"
        @size-change="handleSizeChange"
        @current-change="handleIndexChange"
        :page-size.sync="pagination.pageSize"
        :page-sizes="pagination.pageSizes || pageArray"
        :current-page.sync="pagination.pageNum"
        layout="total,sizes, prev, pager, next,jumper"
        :total="total"
      ></el-pagination>
    </div>
    <!--endregion-->
  </div>
</template>
<!--endregion-->
<script>
import { debounce } from '@/utils'
const _pageArray = [10, 20, 30, 50] // 每页展示条数的控制集合
export default {
  name: 'iTable',
  props: {
    list: {
      type: Array,
      default: () => {
        return []
      } // prop:表头绑定的地段，label：表头名称，align：每列数据展示形式（left, center, right），width:列宽
    }, // 数据列表
    columns: {
      type: Array,
      default: () => {
        return []
      }  // 需要展示的列 === prop：列数据对应的属性，label：列名，align：对齐方式，width：列宽
    },
    operates: {
      type: Object,
      default: () => {
        return {}
      } // width:按钮列宽，fixed：是否固定（left,right）,按钮集合 === label: 文本，type :类型（primary / success / warning / danger / info / text），show：是否显示，icon：按钮图标，plain：是否朴素按钮，disabled：是否禁用，method：回调方法
    },
    total: {
      type: Number,
      default: 0
    }, // 总数
    pagination: {
      type: Object,
      default: null // 分页参数 === pageSize:每页展示的条数，pageNum:当前页，pageArray: 每页展示条数的控制集合，默认 _page_array
    },
    noStatic: false, // 是否计算表格高度
    customHeight: {//与noStatic一起使用
      type: Number,
      default: 320
    },
    otherHeight: {
      type: Number,
      default: 160
    }, // 计算表格的高度
    options: {
      type: Object,
      default: () => {
        return {
          stripe: false, // 是否为斑马纹 table
          loading: false, // 是否添加表格loading加载动画
          highlightCurrentRow: false, // 是否支持当前行高亮显示
          mutiSelect: false, // 是否支持列表项选中功能
          border: true
        }
      }
    } // table 表格的控制参数
  },
  components: {
    expandDom: {
      functional: true,
      props: {
        row: Object,
        render: Function,
        index: Number,
        column: {
          type: Object,
          default: null
        }
      },
      render: (h, ctx) => {
        const params = {
          row: ctx.props.row,
          index: ctx.props.index
        }
        if (ctx.props.column) params.column = ctx.props.column
        return ctx.props.render(h, params)
      }
    }
  },
  data() {
    return {
      height: 250,
      pageNum: 1,
      pageArray: _pageArray,
      multipleSelection: [] // 多行选中
    }
  },
  created() { },
  mounted() {
    this.$nextTick(function () {
      if (this.noStatic) {
        return
      }
      let staticHeight = window.innerHeight - this.$refs.mutipleTable.$el.offsetTop - this.otherHeight
      this.height = staticHeight < 250 ? 250 : staticHeight

      // 监听窗口大小变化
      let self = this
      window.onresize = function () {
        if (!self.$refs.mutipleTable) {
          return
        }
        debounce(() => {
          let staticHeight = window.innerHeight - self.$refs.mutipleTable.$el.offsetTop - this.otherHeight
          self.height = staticHeight < 250 ? 250 : staticHeight
        }, 200)()

      }
    })
  },
  computed: {
    // 计算table高度
    // height () {
    //   if(this.$refs.mutipleTable){
    //     return  window.innerHeight - this.$refs.mutipleTable.$el.offsetTop - 170;
    //   } 
    // }
  },
  methods: {
    headSpanFit(h, { column, index }) {
      let labelLong = column.label.length // 表头label长度
      let size = 20 // 根据需要定义标尺，直接使用字体大小确定就行，也可以根据需要定义
      column.minWidth = (labelLong * size < 100 ? 100 : labelLong * size)// 根据label长度计算该表头最终宽度
      return h('span', { class: 'cell-content', style: { width: '100%' } }, [column.label])
    },
    tableRowClassName({ rowIndex }) {
      if (rowIndex % 2 === 0) {
        return "warning-row"
      } else if (rowIndex % 2 === 1) {
        return "success-row"
      }
      return ""
    },
    getRowClass({ rowIndex }) {
      if (rowIndex == 0) {
        return this.options.headerCellStyle || "background-color:rgba(108,149,237,0.08);"
      } else {
        return " "
      }
    },
    // 切换每页显示的数量
    handleSizeChange(size) {
      if (this.pagination) {
        this.$emit('pagination', { ...this.pagination })
        this.$emit("handleSizeChange", { ...this.pagination })
      }
    },
    // 切换页码
    handleIndexChange(currnet) {
      if (this.pagination) {
        this.$emit('pagination', { ...this.pagination })
        this.$emit("handleIndexChange", { ...this.pagination })
      }
    },
    sortChange({ column, prop, order }) {
      this.$emit("sortChange", { column, prop, order })
    },
    // 多行选中
    handleSelectionChange(val) {
      this.multipleSelection = val
      this.$emit("handleSelectionChange", val)
    },
    // 显示 筛选弹窗
    filterDataDialog() {
      this.$emit("handleFilter")
    },
    // 显示 表格操作弹窗
    showActionTableDialog() {
      this.$emit("handleAction")
    }
  }
};
</script>

<style lang="scss" rel="stylesheet/less">
.table {
  height: 100%;
  .pagination-box {
    margin-top: 20px;
  }
  .el-table__header-wrapper,
  .el-table__fixed-header-wrapper {
    thead {
      tr {
        th {
          color: #333333;
        }
      }
    }
  }
  .el-table-column--selection .cell {
    padding: 0 !important;
    text-align: center;
  }
  .el-table__fixed-right {
    bottom: 0 !important;
    right: 6px !important;
    z-index: 1004;
  }
  .operate-group {
    display: flex;
    flex-wrap: wrap;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    .item {
      margin-right: 2px;
      display: block;
      flex: 0 1 auto;
    }
  }
  .filter-data {
    top: e("calc((100% - 100px) / 3)");
    background-color: rgba(0, 0, 0, 0.7);
  }
  .table-action {
    top: e("calc((100% - 100px) / 2)");
    background-color: rgba(0, 0, 0, 0.7);
  }
  .fix-right {
    position: absolute;
    right: 0;
    height: 100px;
    color: #ffffff;
    width: 30px;
    display: block;
    z-index: 1005;
    writing-mode: vertical-rl;
    text-align: center;
    line-height: 28px;
    border-bottom-left-radius: 6px;
    border-top-left-radius: 6px;
    cursor: pointer;
  }
}
</style>

