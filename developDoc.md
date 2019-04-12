### 红坚果-前端开发文档
#### 项目总体架构
    
本次项目为一个基于PC WEB的事务所知识产权管理系统。开发中使用的语言包括html5，css3，javascript(es6)。使用到前端框架vue(^2.6.10)，UI框架element-ui(^2.4.11)，打包工具webpack(^3.8.1)，项目初始使用vue-cli构建。在整个开发过程中，使用了MVVM的设计模式，Model层即模型层主要表现为vue中可复用的各类组件，View层展现页面由vue编译组件后输出，ViewModel层主要由vue基于Object.defineProperty实现(该方法在IE8中不能对普通对象使用 只能作用于DOM对象 因此系统无法兼容相应浏览器)，同时页面用户事件的监听也由vue负责。该系统为单页面项目，页面路由由前端使用vue-router管理，页面公共数据由前端使用vuex管理，
#### 代码书写规范
  +  **为了统一和便于维护和合作,规范几点：**
     -  项目中组件名用大驼峰命名
     -  变量和方法函数用小驼峰命名且命名杜绝用中文拼音要用有意义可让开发人员理解的英文，提高可读性
     -  代码缩进用一个tab 或者2/4个space
     -  每一条语句结束用英文分号结束符 ';'
     -  函数名和大括号，多个形参之间，以及for if switch while等语句间都需空一格
#### 项目组件代码结构
+ ### **src**
    + **assets** -主要是放一些静态文件 如图片
    + **components** -项目的所有的单文件组件
        - **common** -公共组件
        -  **form** -表单组件（多数可复用）
        -   **page** -项目中具体某模块的父组件
             + **copyright** -版权模块组件
             + **cpc** -cpc模块组件
             + **crm** -crm模块组件
             + **dashboard**  -首页图表组件
             + **exchange** -文档交互模块
             + **finance** -部分费用模块
             + **setting** -部分设置模块
             + **trademark** -商标模块
        -   **page_extension** -项目中某模块下子孙组件 如弹窗等 
    + **const** -纯js文件
    + **fonts** -字体、图标文件
    + **formConfig** -cpc编辑器的配置文件
    + **formTemplate** -cpc编辑器配置表单布局文件（template）
    + **mixins** -组件复用的变量方法等
    +  **router** -项目路由文件
    +  **store** -全局缓存（vuex）

#### API
+ #### 模块搭建
    1.  **路由设计**
        - 找到./src/router/router.js 在该文件里定义路由 示例：
        ``` 
            import  PatentList form '@/components/page/PatentList"'    // 两种方式引入单文件组件
            const PatentList = () => import("@/components/page/PatentList")  //（建议采用这种懒加载方式，在生产环境按需加载  // 提升性能）
           const router = new Router({
                routers: [
                    ...
                    {
                       path: "/patent/list",  //  导航栏路由路径  （必须）
                       name: "patentList",  //  路由名称 （必须）
                       component: PatentList,  // 引入路由对应的组件 （必须）
                       meta: {                   // 路由元信息 ，可存放路由参数  （可选）
                            params: {
                                status: 1,
                            }
                       }
                    }
                    ...
                ]
           })
        ```
        - 然后找到./src/const/menuConst.js 在该文件里配置路由 示例：
        ```
            /**
                项目左侧导航栏配置
                以下配置是在element-ui API 下的再封装
                注： 在自定义以下路由时必须严格遵守，不然会报错或者无法渲染出来 
                      路径取名规则： 子路径应包含父路径
            **/
            ...
            const patentMenu = [{     
            type: "item",     // 一级菜单标志
            text: "专利管理",  //  名称
            path: "/patent/list",  //  路径 严格对应路由path
            icon: "iconfont el-icon-my-patent"   //  菜单图标
            },
            {
            type: "item",
            text: "立案管理",
            path: "/patent/draftbox",
            icon: "iconfont el-icon-my-new"
            },
            {
            type: "submenu",   //   二级菜单标志
            text: "通知书管理",
            path: "/patent/notice",
            icon: "iconfont el-icon-my-notice",
            children: [{   //  二级菜单
            type: "item",
            text: "未发送",
            path: "/patent/notice/sending",
            icon: "iconfont el-icon-my-send"
            },
            {
            type: "item",
            text: "已发送",
            path: "/patent/notice/sent",
            icon: "iconfont el-icon-my-task-finish"
            }]
            }];
            ...
            /**
                顶部导航栏配置
            **/
            
           menu.source = [
                ...
           
                    {
                    text: "专利",    // 顶部导航名字
                    key: "patent",  // key 类似id
                    path: "/patent",  // 路径
                    icon: "iconfont el-icon-my-patent",  // 图标
                    menu: patentMenu      // 加载对应的左侧导航栏
                    },
                ...    
           ];
           ...
           /**
            以下是配置其实是该自定义路由设计的出口，因为会通过递归遍历上面的配置成以下的格式统一输出，也可以直接配置没有联动的路由
           **/
          menu.map = [
            
            "/patent/list/detail": {
            text: "专利详情",
            icon: "",
            path: "/patent/list/detail"
            },

          ];
          
        ```
    2.  **table页配置**

        // 在你组件下引入TableComponent.vue   /src/components/common/TableComponent.vue
        
        ```

        //**************************
        // TableComponent *
        //**************************
        //<table-component
        //  :tableOption="tableOption"
        //  :data="data"
        //  :tableStyle="tableStyle"
        //  @refreshTableData="refreshTableData"
        //>
        // <div slot="headerselect"></div>
        //</table-component>
        // data 表示当前table的绑定数据
        // tableStype 可以传入自定义的表格样式
        // refreshTableData 当调用表格的refresh方法时 会自动调用refreshTableData 里面写入与后台交互的逻辑
        //  tableOption(示例)
        tableOption = {
        'name': 'proposalList',//每个表格的名字,唯一标识   type:string
        'url': URL,//一些内部发送请求的按钮的发送地址,如delete type:string
        'height': 'default',//这个地方输入表格高度,内置有'default', 'default2', 'default3', 'default4' ... type: string|number
        'search_placeholder': "搜索案号、标题、申请号、提案号", // 搜索框占位符文字 不配显示有默认值
        'highlightCurrentRow': true, //高亮显示当前选中Table 默认为false type: boolean
        'rowClick': function (row) {}, //行点击事件 type: Function, row代表当前行数据
        'is_filter': true,//是否需要快速筛选,这个需要后端返回的table数据,包含filters,才能起效 默认为false type:boolean
        'is_header': true, //是否显示表头,默认为true type: boolean
        'is_search': true, //是否显示搜索框 默认为true type: boolean (is_header将覆盖掉这个属性)
        'is_border': true, //是否显示表格中竖线,若不显示,无法调整表格宽度 默认为true type: boolean
        'is_pagination': true,//是否显示分页 默认为true type: boolean
        'is_list_filter': true, // 是否需要高级筛选,默认为false, type: boolean
        'list_type': 'patent', // 在某个组件下引入高级筛选,这个是配置文件对应 类似id 唯一 (表头筛选共用这个属性) type: string
        'treeFilter': 'patent', // 滤器筛选的标志， id 唯一, 和配置文件对应 type: string
        'is_view': true, // 是否需要表格视图, 默认为false type: boolean
        //header_btn表头按钮配置, type: Array
        'header_btn': [
        { type: 'add', click: this.add },
        //按钮字段包括type, label, icon, click
        //type样式类型大致包括: 'add', 'delete', 'export', 'export2', 'import', 'batch_upload', 'control', 'custom' 'dropdown' 'date' 'batch_update'
        // 'report'
        //其中除去'add', 'custome' 都拥有默认的点击事件
        { type: 'delete' },
        { type: 'export' }, // 直接导出
        { type: 'export2' }, // 可自定义要导出字段和调整顺序和导出文件类型
        { type: 'import' },
        { type: 'batch_upload' }, // 批量上传
        { type: 'batch_update' }, // 批量更新
        { type: 'control' }, // 字段
        { type: 'report' }, // 报表
        { type: 'dropdown' },
        { type: 'date' },
        ],
        'import_type': 'patent', // 'import',当在header_btn使用默认的导入按钮的时候,传递导入类型或者配置数据 type: string | Object
         // {
          //    action: 'getPatents',
         // title: '导入专利',
          //    url: '/patents/import',
          //    category: 1,
         // model: '/static/templates/patent_batch_template.xlsx',
         // model_name: '专利导入模板',
          // }
        'upload_type': 'patent', //''batch_upload',当在header_btn使用默认的文件上传按钮的时候,传递上传类型或者配置数据 type: string | Object
        'update_type': 'patent', // 'batch_update', 当在header_btn使用默认的文件上传按钮的时候,传递上传类型或者配置数据 type: string | Object
         // {
          //    action: 'getPatentDocuments',
          //    url: '/patents/documents',
          //    type: 'patent',
          // }
        'header_slot': [ 'headerselect' ], //自定义多个表头组件, type: Array
        //columns配置表格行 type: Array
          'columns': [
        { type: 'selection' },
        //type: 分为selection, text, array, action
        {
          type: 'text',
          label: '专利类型',//表头名
          prop: 'type',//返回数据关键字,对应后台接口
        render_simple: 'name',//当后台数据返回得是对象的时候,渲染对象下面某一个字段, {id: 1, name: '类型'}
        render_key: 'fee', // 重置prop值,需同时设置render_simple
        render_obj: 'project', // 当后台数据返回的数据是对象下又包含对象,显然下面的某一字段 {project: {serial: '4487867',}},此属性需要同时设置render_simple
        // 且此属性只能处理字符串和{id: 1, name: '案件'}对象，数组不支持,如需渲染数组请使用render函数
          render: function(h, text, row, prop) { //text代表当前需要渲染的数据,row代表行数据,
            return h('span', text);
          },//同vue的render函数用法相同,同样可使用JSX语法, 会覆盖掉render_simple
          sortable: true,//是否可排序 默认为false 用了表头筛选已弃用
          is_import: true,//是否包含在导入列表中, 默认为false
        width: '142',//宽度
        min_width: '100', //最小宽度 type: string|number
        header_align: 'left', //表头文字位置 可选择left|center|right 默认 left type: string
        show: true,// 是否显示该字段 type: boolean
        overflow: true, //是否显示tooltip 默认为true type: boolean
        className: 'patent_list', //自定义类名 type: string
        render_header: true,// 是否表头筛选, 默认为false
        expanded: true, //是否显示在行展开
        },
        {
        type: 'array',
        label: '申请人',
        prop: 'applicants',
        width: '300',
        is_import: true,
        render: _=>{
        return _.map(_=>_.name);
        }//array 类型只渲染['a','b','c']类型的数组, 如果后台返回数据不符合, 使用render进行处理
        },
        {
        type: 'text-copy', //复制
        },
        {
        type: 'text-btn', //可点击
        custom_text: '',//自定义按钮名称 type: string
        render_text_btn: (row)=>{ // 回调函数 type: function
        return row.name
        }
        },
        {
        type: 'action',
        width: '145',
        label: '操作', //默认显示为操作,可不填
        btns: [ //当前按钮数组, type: array
        {
          //默认样式类型： edit delete view download dropdown, 可以不设置类型并自定义
         type: 'delete',
          click: function(row, $event) {}
        },
        {
          btn_type: '',//使用element-ui button组件 type 关键字
          size: '',
          icon: '',
          label: '',
          click: function(row, $event) {},
        }
        ],
        },
        ]
        },

        ```
  + #### TableComponent Attributes
    |参数  |说明| 类型 | 可选值 | 默认值 |
    | --- | --- | --- | --- | --- |
    |  data | 显示的数据 | array | - | - |
    | tableOption | 表格配置 | object  |-  | - |
    | tableStyle | 样式 | object | - | - |
    | refreshProxy | aixos方法返回值 | promise | - | - |
    | refreshTableData | 请求后端数据方法 | function(option) | - | - |
  + #### TableComponent Events
    | 事件名 | 说明 | 参数 |
    | --- | --- | --- |
    | refreshTableData | 刷新后端数据（get请求） | options(主要是筛选条件) | 
  + #### TableComponent Methods
    | 方法名 | 说明 | 参数 |
    | --- | --- | --- |
    | getSelection | 表格已选择的行数据 | - |
    | getSelected|  表格已选择的行数据|  flag，默认为false ,true时则不会进行空验证|
    | update | 刷新当前页 | -|
    | refresh | 刷新回到首页 | - |
  + #### TableComponent Slot 
    | name |  说明 |
    | --- | --- |
    |bread_mark  | 面包屑 |
    | tableOption.header_slot | 表头按钮自定义 type: array |
    |row_action | 作用域插槽，table右侧按钮通过插槽自定义，btns_render （行配置里）type: string|boolean|object|function |
    
    ```
        <app-table :data="data" :columns="columns"></app-table>
    ```
   + #### AppTable Attributes
     |  参数 | 说明  |  类型 | 可选值  | 默认值 |
     | --- | --- | --- | --- | --- |
     | data | 表格数据 |  array |  - |  -|
     | columns | 表格配置 | array |  - |  -|
     | rowKey | 表格key值 (唯一值)| string |  - | id |
     | listType | 表头筛选类型 |string |  - |  -|
     | border | 表格边框 | value |  true/false |  false |
     | height | 表格高度 | string/number |  default/default2-8/...|  -|
     | highlightCurrentRow | 是否高亮当前行 | boolean |  true/false |  false|
     | defaultSort | 表格默认排序 | object |  - |  -|
     | merge | 合并的字段 | array |  - |  -|
     | showSummary | 是否显示合计 | boolean |  true/false |  false |
     | showHeader | 是否显示表头 | boolean |  true/false |  true |
     | sumFunc | 表格合计自定义方法 | function（{ columns, data }） |  - |  -|
     | tableSelected | 默认选择项 | array |  - |  -|
     | expandsCol | 展开数据列数 （必须配置expanded: true,）| number |  - |  3|
     | expandsSpan | 展开数据每列宽度（element-uirow-col布局） |number |  - |  4|
  + #### AppTable Events
    | 事件名 | 说明 | 参数 |
    | --- | --- | --- |
    | row-click |  表格行点击事件 | row, event, column |
    | cell-click |  某单元格点击事件 | row, column，cell, event |
    | header-dragend |  拖动表头宽度触发的事件 | newWidth, oldWidth, column, event  |
    | cell-mouse-enter | 当单元格 hover 进入时会触发该事件  | row, column，cell, event |
    | :table-selected.sync |  当选择项变化时触发该自定义事件 |  selection |
    | refreshHeight |  表格的高度 | height |
  + #### AppTable Methods
    | 方法名 | 说明 | 参数 |
    | --- | --- | --- |
    | getSelected|  表格已选择的行数据|  flag，默认为false ,true时则不会进行空验证|
    | - | 其他方法参照element官方文档 | - |
  + #### AppTable Slot 
    | name |  说明 |
    | --- | --- |
    |row_action | 作用域插槽，table右侧按钮通过插槽自定义，btns_render （行配置里）type: string|boolean|object|function |   
  
      ```
        <static-select v-model="form.area" type="area"></static-select>
      ```
  + #### StaticSelect Attributes
    | 参数 | 说明 | 类型 | 可选值 | 默认值 |
    | --- | --- | --- | --- | --- |
    | value/v-model | 绑定值 | boolean/string/number | - | - |
    | multiple | 是否多选 | boolean | - | false |
    | disabled | 是否禁用 | boolean | - | false |
    | size | 输入框大小 | string | medium/small/mini |  -|
    | type | 自定义数据的key | string | - | - |
    | skip | 过滤不需要的选项 | array | - | - |
    | normalFilter |  正向过滤出需要的选项 | array | - | - |
   + #### StaticSelect Events
     |  事件名| 说明 | 参数 |
     | --- | --- | --- |
     | input | input事件 | 改变的值 |
     | change | 选中时触发 | 选中的值 |
     | visible-change | 下拉框出现/隐藏触发 | 出现为true，隐藏为false，类型type |
   + #### StaticSelect Methods
     |  方法名| 说明 | 参数 |
     | --- | --- | --- |
     | getSelected | 当前选中的值（数组） | value（可自定义） |
   + #### StaticSelect Slot
      |  name| 说明 |
     | --- | --- | 
     | slot | 默认 | 
     
      ```
        <remote-select v-model="form.users" type="user" multiple></static-select>
      ```
   + #### RemoteSelect Attributes
     | 参数 | 说明 | 类型 | 可选值 | 默认值 |
     | --- | --- | --- | --- | --- |
     | value/v-model | 绑定值 | boolean/string/number | - | - |
     | multiple | 是否多选 | boolean | - | false |
     | para | 额外请求参数 | object | - | - |
     | disabled | 是否禁用 | boolean | - | false |
     | single | 是否单选 | boolean | - |  false |
     | type | 自定义数据的key | string | - | - |
     | staticMap |  可添加静态的选项 | array | - | - |
     | pageType |  页面模式 | string | add/edit | - |
     | addType |  新增入口页面类型 | string | - | - |
     | allow-create | 是否允许用户创建新条目,需在@/const/remoteConfig.js配置 | boolean | - | false |
     | PLACEHOLDER | 占位符,需在@/const/remoteConfig.js配置 | string | - | - |
    + #### RemoteSelect Events
      |  事件名| 说明 | 参数 |
      | --- | --- | --- |
      | input | input事件 | 改变的值 |
      | change | 选中时触发 | 选中的值 |
    + #### RemoteSelect Methods
       |  方法名| 说明 | 参数 |
      | --- | --- | --- |
      | getSelected | 当前选中的值（数组） | - |
      | clear | 清空选择框 | flag 默认为true时清空并请求远程方法 |
    + #### RemoteSelect Slot
      |  name| 说明 |
      | --- | --- | 
      | slot | 默认|
      
      ```  
        <app-shrink :visible.sync="visible" :title="title"></app-shrink>
      ```
    + #### AppShrink Attributes
       | 参数 | 说明 | 类型 | 可选值 | 默认值 |
      | --- | --- | --- | --- | --- |
      | visible.sync | 是否显示 | boolean | - | false |
      | modal | 是否需要遮罩 | boolean | - | false |
      | modalClick | 是否点击遮罩关闭 | boolean | - | true |
      | title | 面板的标题 | string | - | - |
      | size | 面板大小 | string | full-screen/large/middle/small | large |
      | isClose | 是否需要关闭按钮 | boolean | - | true |
    + #### AppShrink Events
      |  事件名| 说明 | 参数 |
      | --- | --- | --- |
      | close | 关闭面板 |  -|
    + #### Appshrink Methods
       |  方法名| 说明 | 参数 |
      | --- | --- | --- |
      | close | 关闭面板 | - |
    + #### Appshrink Slot
      |  name| 说明 |
      | --- | --- | 
      | info | 头部提示信息 |
      | header | 头部多加按钮等 |
      | slot | 默认 |
      ```
        <app-switch v-model="form.status" type="status"></app-switch>
      ```
    + #### AppSwitch Attributes
       | 参数 | 说明 | 类型 | 可选值 | 默认值 |
      | --- | --- | --- | --- | --- |
      | value/v-model | 绑定值 | number/boolean | - | - |
      | type | 类型 （模式）| object/string | status/is/is_boolean/自定义对象 | - |
      | simple | 是否在开关两头显示，为true时只在开关右侧显示onText和offText| boolean/string | - | false |
    + #### AppSwitch Events
      |  事件名| 说明 | 参数 |
      | --- | --- | --- |
      | input | input事件 |  改变值val|
      
      ```
        <search-input v-model="searchValue" placeholder="搜索..." clearable></search-input>
      ```
    + #### searchInput Attributes
       | 参数 | 说明 | 类型 | 可选值 | 默认值 |
      | --- | --- | --- | --- | --- |
      | value/v-model | 绑定值 | string/number | - | - |
      | placeholder | 占位符| string | - |- |
      | inputStyle | 搜索框的自定义样式 | string | - | - |
      | clearable | 是否可清空| boolean | - |false |
    + #### searchInput Events
      |  事件名| 说明 | 参数 |
      | --- | --- | --- |
      | input | input事件 |  输入值val|
      | enter | 回车事件 |  输入值val|
      | click | 点击事件 |  输入值val|
      | clear | 清空事件 |  输入值val|
      
      ```
        <upload v-model="form.attachments" :file-list="attachments"></upload>
      ```
    + #### Upload Attributes
       | 参数 | 说明 | 类型 | 可选值 | 默认值 |
      | --- | --- | --- | --- | --- |
      | value/v-model | 绑定值 | array | - | [] |
      | fileList | 上传的文件列表 | array | - | [] |
      | showFileList | 是否显示文件列表| boolean | - | true |
      | limit | 最大允许上传个数 | number | - | - |
    + #### Upload Events
      |  事件名| 说明 | 参数 |
      | --- | --- | --- |
      | uploadBefore | 上传文件前的钩子 |  file|
      | uploadSuccess | 上传成功的钩子 |  respones，file，fileList |
    + #### Upload Methods
       |  方法名| 说明 | 参数 |
      | --- | --- | --- |
      | getFileList| 获取上传文件列表 | - |
      | clearFiles| 清空上传文件列表 | - |
    + #### Upload Slot
      |  name| 说明 |
      | --- | --- | 
      | tip | 提示信息 |
      ```     
           <app-button-loading :func="save" ref="loadingBtn" text="保存"></app-button-loading>
      ```
    + #### AppButtonLoading Attributes
       | 参数 | 说明 | 类型 | 可选值 | 默认值 |
      | --- | --- | --- | --- | --- |
      | size | 按钮大小 | string | medium/small/mini | small |
      | disabled | 是否禁用| boolean | - | false |
      | text | 按钮的文字| string | - | 新建 |
      | func | 点击回调函数 | function | - | - |
      | param | add/edit模式 | string | - | add |
    
      
    