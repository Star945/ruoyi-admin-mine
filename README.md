## 开发

```bash
# 克隆项目
git clone https://gitee.com/y_project/RuoYi-Vue

# 进入项目目录
cd ruoyi-ui

# 安装依赖
npm install

# 建议不要直接使用 cnpm 安装依赖，会有各种诡异的 bug。可以通过如下操作解决 npm 下载速度慢的问题
npm install --registry=https://registry.npm.taobao.org

# 启动服务
npm run dev
```

浏览器访问 http://localhost:80

## 发布

```bash
# 构建测试环境
npm run build:stage

# 构建生产环境
npm run build:prod

本地测试时需批量替换本地连接全局查找：http://192.168.x.x自行解决
history 模式部署时 需再router 'src\router\index.js' 下添加 base:'你的目录' vueconfig.js中publicPath 同你的path 后台需配置webInfo文件
```
## 日期格式化
```javascript

借助[moment](http://momentjs.cn/)插件实现日期
moment().format('MMMM Do YYYY, h:mm:ss a'); // 七月 5日 2021, 4:39:45 下午
moment().format('dddd');                    // 星期一
moment().format("MMM Do YY");               // 7月 5日 21
moment().format('YYYY [escaped] YYYY');     // 2021 escaped 2021
moment().format();                          // 2021-07-05T16:39:45+08:00
##相对时间
moment("20111031", "YYYYMMDD").fromNow(); // 10 年前
moment("20120620", "YYYYMMDD").fromNow(); // 9 年前
moment().startOf('day').fromNow();        // 17 小时前
moment().endOf('day').fromNow();          // 7 小时内
moment().startOf('hour').fromNow();       // 40 分钟前
##日历时间
moment().subtract(10, 'days').calendar(); // 2021/06/25
moment().subtract(6, 'days').calendar();  // 上星期二16:39
moment().subtract(3, 'days').calendar();  // 上星期五16:39
moment().subtract(1, 'days').calendar();  // 昨天16:39
moment().calendar();                      // 今天16:39
moment().add(1, 'days').calendar();       // 明天16:39
moment().add(3, 'days').calendar();       // 下星期四16:39
moment().add(10, 'days').calendar();      // 2021/07/15
##多语言支持
moment.locale();         // zh-cn
moment().format('LT');   // 16:39
moment().format('LTS');  // 16:39:45
moment().format('L');    // 2021/07/05
moment().format('l');    // 2021/7/5
moment().format('LL');   // 2021年7月5日
moment().format('ll');   // 2021年7月5日
moment().format('LLL');  // 2021年7月5日下午4点39分
moment().format('lll');  // 2021年7月5日 16:39
moment().format('LLLL'); // 2021年7月5日星期一下午4点39分
moment().format('llll'); // 2021年7月5日星期一 16:39
```
## 表格封装
*表格封装适合大部分场景，但未实现表格行编辑，以及表格的自定义表头，避免过度封装导致 灵活性变差*
```javascript
//食用方法（此框架以全局引入改组件--不想全局引入可以去掉main.js中的引用单独页面引用）
//父组件中写法
      <i-table
        :list="list"
        :total="total"
        :otherHeight="otherHeight"
        :options="options"
        :pagination="pagination"
        :columns="columns"
        @handleSizeChange="handleSizeChange"
        @handleIndexChange="handleIndexChange"
        @handleSelectionChange="handleSelectionChange"
      ></i-table>
      //父组件中单元格数据
      columns: [
        {
          prop: "yogaClubName",
          label: "场馆名称",
          align: "center",
        },
        {
          prop: "yogaClubCurriculumName",
          label: "课程名称",
          align: "center",
        },
        {
          prop: "yogaClubCurriculumType",
          label: "课程类型",
          align: "center",
          formatter: (row) => {
            return row.yogaClubCurriculumType == 1 ? '团课' : row.yogaClubCurriculumType == 2 ? '私教' : '--'
          }
        },
        {
          prop: "whetherRecommend",
          label: "私教页推荐",
          align: "center",
          render: (h, params) => {//渲染自定义组件（用于图片，切换switch等组件的渲染）
            if(params.row.yogaClubCurriculumType == 1){
              return h('span','--',)
            }

            return h("el-switch", {
              props: {
                value: params.row.whetherRecommend,
                inactiveValue: 2,
                activeValue: 1
              },
              on: {
                change: e => {
                  this.editId = params.row.idYogaClubCurriculum
                  this.updateState(params.row)
                }
              }
            })
          }
        },
        {
          prop: "createdTime",
          label: "添加时间",
          align: "center",
          formatter: (row) => {//格式化方法
            return //参考moment格式化时间
          }
        },
      ], 
      // 操作按钮组
      operates: {
        width: 150,
        list: [
          {
            label: "编辑",//按钮名
            color: "#409EFF",//按钮颜色
            type: "text",
            haspermission:['settings:course:edit'],//结合ruoyi权限的 权限
            show: (index, row) => { //按钮显示逻辑 支持boolen 和方法
              return true
            },
            icon: "",
            plain: false,
            disabled: false,
            method: (index, row) => {//按钮操作方法
              this.handleEdit(index, row)
            }
          },
          {
            label: "删除",
            haspermission:['settings:course:del'],
            color: "#dd6161",
            type: "text",
            icon: "",
            show: true,
            plain: false,
            disabled: (index, row) => {
              return false
            },
            method: (index, row) => {
              this.handleDel(index, row)
            }
          },
        ]
      }, 
     //父组件中表格默认参数 
      options: {
        border: true,
        fit: true,
        headerCellStyle: 'background: "#dfe6ec"',
        stripe: true, // 是否为斑马纹 table
        loading: false, // 是否添加表格loading加载动画
        highlightCurrentRow: true, // 是否支持当前行高亮显示
        mutiSelect: false, // 是否支持列表项选中功能
      }, // table 的参数
```
配置化表格[参考地址](https://juejin.cn/post/6844903555598401544)
| 参数名 | 说明 | 类型 | 默认值 |
| :-----| ----: | :----: |:----: |
| list | 表格的数据 | array | [] |
| total | 数据总条数 | number |0 |
| otherHeight | 除去表格部分外的视窗高度| number | 170 |
| options | 表格相关配置(参见elementtable) | Object | --|
| columns | 表头配置 | array |[] |
| handleSizeChange | 表格Size更改时的回调 | function |返回paginationd对象{pageNum: pageNume,pageSize: size} |
| handleIndexChange | 表格页数更改时的回调 | function |返回paginationd对象{pageNum: pageNume,pageSize: size} |
| handleSelectionChange |表格勾选的回调 | function |见element-table [row] |
| operates |操作按钮组 | array |参照上方注释 |

## 配置化表单
配置化表单开源[项目地址](https://github.com/FEMessage/el-form-renderer)
配置化表单开源[项目文档](https://github.com/FEMessage/el-form-renderer/blob/dev/README-zh.md)
[项目文档](https://femessage.github.io/el-form-renderer/)
感谢作者的付出
文档中已经写得非常清楚，这里简述几个坑：
+ 无法动态更改label标签，（一般也不会有这种需求）
+ 级联选择器无法动态设置options（可以循环遍历表单配置对其options进行$set操作，也可使用slot）
+ slot时如果是表单项提交是要单独处理slot表单参数，因其不再配置项里固需要自己进行合并，请注意多场景处理（搜索编辑等）
+ 如果你的表单特别复杂建议纯手撸，不然插件可能会影响你的情绪莫名其妙的bug会很多
+ 如果你已知晓上述情况，还解决了部分需求，请教教我，我太懒了
```javascript
//表单配置化~找到了网上的解决方案~原谅我是一个菜鸡
```
# ERP项目就不要尝试了，这个封装配不上它
