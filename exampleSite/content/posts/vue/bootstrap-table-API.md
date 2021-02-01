---
title: bootstrap-table API
date: 2018-07-26 22:20:50
tags: ["bootstrap"]

---
文档原址：http://bootstrap-table.wenzhixin.net.cn/zh-cn/documentation/

<!--more-->
### 表的各项(Table options )
* 定义在 jQuery.fn.bootstrapTable.defaults 文件内

名称 | 标签 | 类型 | 默认 | 描述
-|-|-|-|-
- |	data-toggle |	String |	'table'	| 不用写 JavaScript 直接启用表格。
classes |	data-classes |	String	| 'table table-hover'| 	表格的类名称。默认情况下，表格是有边框的，你可以添加 'table-no-bordered' 来删除表格的边框样式。
sortClass	| data-sort-class |	String |	undefined	|被排序的td元素的类名。
height	| data-height |	Number |	undefined |	定义表格的高度。
undefinedText	| data-undefined-text	| String |	'-' | 当数据为 undefined 时显示的字符。
striped	| data-striped |	Boolean	| false	| 设置为 true 会有隔行变色效果。
sortName |	data-sort-name |	String	| undefined	|定义排序列，通过url方式获取数据填写字段名，否则填写下标。
sortOrder	| data-sort-order	| String	| 'asc'	|定义排序方式，'asc' 或者 'desc'。
sortStable |	data-sort-stable |	Boolean |	false	| 设置为 true 将获得稳定的排序，我们会添加\_position属性到 row 数据中。
iconsPrefix	| data-icons-prefix	| String |	'glyphicon'	| 定义字体库 ('Glyphicon' or 'fa' for FontAwesome)，使用"fa"时需引用 FontAwesome，并且配合 icons 属性实现效果。 Glyphicon 集成于Bootstrap可免费使用，参考： http://glyphicons.com/ </br>FontAwesome 参考： http://fortawesome.github.io/
icons	| data-icons |	Object	| ``{paginationSwitchDown: 'glyphicon-collapse-down icon-chevron-down',paginationSwitchUp: 'glyphicon-collapse-up icon-chevron-up',refresh: 'glyphicon-refresh icon-refresh',toggle: 'glyphicon-list-alt icon-list-alt',columns: 'glyphicon-th icon-th',detailOpen: 'glyphicon-plus icon-plus',detailClose: 'glyphicon-minus icon-minus'}``	| 自定义图标
columns	| - |	Array |	[]	| 列配置项，详情请查看 列参数 表格.
data	| -	| Array	| [] |	加载json格式的数据。
ajax	| data-ajax	| Function | 	undefined	| 自定义 AJAX 方法，须实现 jQuery AJAX API。
method	| data-method	| String	| 'get'	| 服务器数据的请求方式 'get' 或 'post'。
url	| data-url |	String	| undefined	| 服务器数据的加载地址。
cache	| data-cache |	Boolean	| true	| 设置为 false 禁用 AJAX 数据缓存。
contentType	| data-content-type	| String	| 'application/json' |	发送到服务器的数据编码类型。
dataType	| data-data-type	| String	| 'json' | 	服务器返回的数据类型。
ajaxOptions	| data-ajax-options	| Object	| {}	| 提交ajax请求时的附加参数，可用参数列请查看http://api.jquery.com/jQuery.ajax.
queryParams	| data-query-params	| Function	| ``function(params) {return params;}	`` |请求服务器数据时，你可以通过重写参数的方式添加一些额外的参数，例如 toolbar 中的参数 如果 queryParamsType = 'limit' ,返回参数必须包含limit, offset, search, sort, order 否则, 需要包含:pageSize, pageNumber, searchText, sortName, sortOrder.返回false将会终止请求。
queryParamsType |	data-query-params-type |	String |	'limit'	| 设置为 'limit' 则会发送符合 RESTFul 格式的参数。
responseHandler	| data-response-handler	| Function	| function(res) {return res;}	| 加载服务器数据之前的处理程序，可以用来格式化数据。参数：res为从服务器请求到的数据。
pagination	| data-pagination	| Boolean	| false	| 设置为 true 会在表格底部显示分页条。
paginationLoop |	data-pagination-loop	| Boolean	| true	| 设置为 true 启用分页条无限循环的功能。
onlyInfoPagination |	data-only-info-pagination	| Boolean |	false	|设置为 true 只显示总数据数，而不显示分页按钮。需要设置 pagination='true'。
sidePagination	| data-side-pagination	| String	| 'client'	|设置在哪里进行分页，可选值为 'client' 或者 'server'。设置 'server'时，必须设置服务器数据地址（url）或者重写ajax方法。
pageNumber	| data-page-number |	Number	| 1	|如果设置了分页，首页页码。
pageSize |	data-page-size	| Number	| 10	| 如果设置了分页，页面数据条数。
pageList	| data-page-list	| Array	| [10, 25, 50, 100, All]	| 如果设置了分页，设置可供选择的页面数据条数。设置为 All 或者 Unlimited，则显示所有记录。
selectItemName	| data-select-item-name |	String |	'btSelectItem'	| radio 或者 checkbox 的字段 name 名。
smartDisplay |	data-smart-display	| Boolean	| true |	设置为 true 是程序自动判断显示分页信息和 card 视图。
escape |	data-escape	| Boolean	| false	| 转义HTML字符串，替换 &, <, >, ", \`, 和 ' 字符。
search	| data-search	| Boolean	| false	| 是否启用搜索框。
searchOnEnterKey |	data-search-on-enter-key |	Boolean |	false	| 设置为 true时，按回车触发搜索方法，否则自动触发搜索方法。
strictSearch	| data-strict-search | 	Boolean	| false	| 设置为 true启用全匹配搜索，否则为模糊搜索。
searchText	| data-search-text	| String	| '' |	初始化搜索文字。
searchTimeOut	| data-search-time-out |	Number |	500 |	设置搜索超时时间。
trimOnSearch	| data-trim-on-search	| Boolean	| true	| 设置为 true 将自动去掉搜索字符的前后空格。
showHeader	| data-show-header |	Boolean	| true	| 是否显示列头。
showFooter	| data-show-footer |	Boolean	| false	| 是否显示列脚。
showColumns | data-show-columns	| Boolean |	false	| 是否显示内容列下拉框。
showRefresh	| data-show-refresh	| Boolean	| false	| 是否显示刷新按钮。
showToggle	| data-show-toggle	| Boolean	| false |	是否显示切换视图（table/card）按钮。
showPaginationSwitch	| data-show-pagination-switch |	Boolean |	false	|是否显示切换分页按钮。
showFullscreen |	data-show-fullscreen |	Boolean	 | false	| 是否显示全屏按钮。
minimumCountColumns	| data-minimum-count-columns	| Number |	1	|最小隐藏列的数量。
idField	| data-id-field	| String |	undefined	| 指定主键列。
uniqueId	| data-unique-id	| String | 	undefined	| 对每一行指定唯一标识符。
cardView	| data-card-view |	Boolean	| false	| 设置为 true将显示card视图，适用于移动设备。否则为table试图，适用于pc端。
detailView	| data-detail-view	| Boolean	| false	| 设置为 true 可以显示详细页面模式。
detailFormatter	| data-detail-formatter	| Function |	``function(index, row) {return '';} ``|	格式化详细页面模式的视图。
searchAlign	| data-search-align	| String	| 'right'	| 指定 搜索框 水平方向的位置。'left' 或 'right'。
buttonsAlign	| data-buttons-align	| String	| 'right'	| 指定 按钮栏 水平方向的位置。'left' 或 'right'。
toolbarAlign |	data-toolbar-align |	String |	'left'	| 指定 toolbar 水平方向的位置。'left' 或 'right'。
paginationVAlign	| data-pagination-v-align	| String |	'bottom'	| 指定 分页条 在垂直方向的位置。'top'，'bottom' 或 'both'。
paginationHAlign	| data-pagination-h-align	| String |	'right'	| 指定 分页条 在水平方向的位置。'left' 或 'right'。
paginationDetailHAlign	| data-pagination-detail-h-align |	String	| 'left' |	指定 分页详细信息 在水平方向的位置。'left' 或 'right'。
paginationPreText	| data-pagination-pre-text	| String |	'<'	| 指定分页条中上一页按钮的图标或文字。
paginationNextText |	data-pagination-next-text	| String	| '>'	| 指定分页条中下一页按钮的图标或文字。
clickToSelect	| data-click-to-select	| Boolean	| false |	设置 true 将在点击行时，自动选择 rediobox 和 checkbox。
ignoreClickToSelectOn	| data-ignore-click-to-select-on |	Function	| ``{ return $.inArray(element.tagName, ['A', 'BUTTON']); }	`` |包含一个参数：element: 点击的元素。返回 true 是点击事件会被忽略，返回 false 将会自动选中。该选项只有在 clickToSelect 为 true 时才生效。
singleSelect	| data-single-select |	Boolean |	false	| 设置 true 将禁止多选。
toolbar	| data-toolbar	| String	| undefined	| 一个jQuery 选择器，指明自定义的 toolbar。例如:#toolbar, .toolbar.
buttonsToolbar	| data-buttons-toolbar |	String | Node 	undefined	|一个jQuery 选择器，指明自定义的 buttons toolbar。例如:#buttons-toolbar, .buttons-toolbar 或 DOM 节点。
checkboxHeader |	data-checkbox-header |	Boolean	| true	| 设置 false 将在列头隐藏全选复选框。
maintainSelected |	data-maintain-selected |	Boolean	| false	| 设置为 true 在点击分页按钮或搜索按钮时，将记住checkbox的选择项。
sortable	| data-sortable	| Boolean	| true	| 设置为false 将禁止所有列的排序。
silentSort	| data-silent-sort |	Boolean	| true	| 设置为 false 将在点击分页按钮时，自动记住排序项。仅在 sidePagination设置为 server时生效。
rowStyle	| data-row-style |	Function |	``function(row,index) {return class;}	`` | 自定义行样式 参数为：row: 行数据index: 行下标返回值可以为class或者css
rowAttributes	| data-row-attributes	| Function |	``function(row,index) {return attributes;}``|	自定义行属性 参数为：row: 行数据 index: 行下标 返回值可以为class或者css 支持所有自定义属性
customSearch	| data-custom-search |	Function |	$.noop	| 自定义搜索方法来替代内置的搜索功能，它包含一个参数：text：搜索文字。用法示例：``function customSearch(text) {//Search logic here.//You must use `this.data` array in order to filter the data. NO use `this.options.data`.}``
customSort	| data-custom-sort	| Function	| $.noop |	自定义排序方法来替代内置的搜索功能，它包含一个参数 sortName: 排序名。sortOrder: 排序顺序。用法示例：function customSort(sortName, sortOrder) {//Sort logic here.//You must use `this.data` array in order to sort the data. NO use `this.options.data`.}

### 列的各项（Column options ）

The column options is defined in jQuery.fn.bootstrapTable.columnDefaults.

名称	| 属性	| 类型	| 默认值	| 作用描述
- | - | - | - | -
radio	| data-radio |	Boolean	| false	|  默认false不显示radio（单选按钮），设为true则显示，radio宽度是固定的
checkbox	| data-checkbox	| Boolean |	false	| 默认false不显示checkbox（复选框），设为true则显示，checkbox的每列宽度已固定
field	| data-field	| String |	undefined	| 是每列的字段名，不是表头所显示的名字，通过这个字段名可以给其赋值，相当于key，表内唯一
title	| data-title	| String	| undefined	| 这个是表头所显示的名字，不唯一，如果你喜欢，可以把所有表头都设为相同的名字
titleTooltip	| data-title-tooltip	| String	| undefined	| 当悬浮在某控件上，出现提示 - 参考 Bootstrap 提示工具（Tooltip）插件
class	| class/data-class |	String	| undefined	| 没什么好说的，就是class
rowspan	| rowspan/data-rowspan |	Number	| undefined	| 每格所占的行数
colspan	| colspan/data-colspan	| Number	| undefined	|每格所占的列数
align |	data-align	| String |	undefined	| 每格内数据的对齐方式，有：left（靠左）、right（靠右）、center（居中）
halign	| data-halign	| String	| undefined	| table header（表头）的对齐方式，有：left（靠左）、right（靠右）、center（居中）
falign	| data-falign	| String	| undefined	| table footer（表脚，就这样译，任性，其实随便译啦，知道就好）的对齐方式，有：left（靠左）、right（靠右）、center（居中）
valign	| data-valign	| String	| undefined	| 每格数据的对齐方式，有：top（靠上）、middle（居中）、bottom（靠下）
width	| data-width |	Number（单位：px或%）|	undefined	| 每列的宽度。 - 如果没有自定义宽度，那么宽度会根据内容的宽度自适应。- 如果表是左适应（left responsive）或者设置的宽度小于内容的宽度，那么，宽度还是会自适应（可以在class或其他的属性中使用 min-width 或 max-width）。- 你也可以使用”%”作为单位，这样的话，bootstapTable将按百分比划分，如果你想使用”pixels（像素值）”，你可以只写数字（只要不加”%”，单位默认”px”，不嫌麻烦，或者想更清晰，你也可以加上”px”）
sortable	| data-sortable	| Boolean	| false	| 默认false就默认显示，设为true则会被排序
order	| data-order |	String |	asc |	默认的排序方式为”asc（升序）”，也可以设为”desc（降序）”。- 与上面的结合使用，不然都不让排序了，你还考虑什么升降。
visible	| data-visible |	Boolean	| true |	默认为true显示该列，设为false则隐藏该列。 - 还是很有用的，例如隐藏自定义index列，按某属性排序后会变乱，你可以读取某行的index来进行赋值
cardVisible	| data-card-visible	| Boolean	| true	| 默认为true显示该列，设为false则隐藏。
switchable	| data-switchable	| Boolean	| true |	默认为true显示该列，设为false则禁用列项目的选项卡。
clickToSelect	| data-click-to-select	| Boolean |	true	| 默认true不响应，设为false则当点击此行的某处时，不会自动选中此行的checkbox（复选框）或radiobox（单选按钮）
formatter	| data-formatter	| Function |	undefined	|需要此列的对象。 某格的数据转换函数，需要三个参数： -value： field（字段名） -row：行的数据 -index：行的（索引）index
footerFormatter | data-footer-formatter |	Function	| undefined	| 需要此列的对象。 某格的数据转换函数，需要一个参数： -data： 所有行数据的数组 函数需要返回（return）footer某格内所要显示的字符串的格式，还要包括内容
events	| data-events	| Object	| undefined	| 当某格使用formatter函数时，事件监听会响应，需要四个参数： -event：jQuery事件（参考Events）。 - value：字段名 - row：行数据 - index：此行的index 举个例子： ``<th .. data-events=”operateEvent”> var operateEvents = {‘click .like’: function (e, value, row, index) {}};``
sorter	| data-sorter	| Function	| undefined	| 自定义的排序函数，实现本地排序，需要两个参数： - a：第一个字段名 - b：第二个字段名
sortName |	data-sort-name	| String	| undefined	| 除了表头默认的sort-name或列的字段名，还可以使用自定义的sort-name 对特殊情况说明： - 一个列显示的内容或许是”html”代码，如：``<b><span style=”color:red”>abc</span></b>，``但是被排列的是html代码中的内容（content）：abc
cellStyle	| data-cell-style |	Function |	undefined	| 对某格中显示样式（style）进行改变，需要三个函数： - value：字段名 - row：行数据 - index：此行的index - field：行的字段名 支持classes和css，用法如下： ``function cellStyle(value, row, index, field) { return { classes: ‘text-nowrap another-class’, css: {“color”: “blue”, “font-size”: “50px”} }; }``
searchable	| data-searchable	| Boolean	| true |	默认true，表示此列数据可被查询
searchFormatter |	data-search-formatter	| Boolean	| true	| 默认true，可使用格式化的数据查询
escape |	data-escape	| Boolean	| false	| 跳过插入HTML中的字符串，替换掉特殊字符（后面符号没有逗号）：&，<，>，"，`，'

### 事件（Events）
```js
定义事件的格式：
$(‘#table’).bootstrapTable({
onEventName: function (arg1, arg2, …) {
// …
} });

$(‘#table’).on(‘event-name.bs.table’, function (e, arg1, arg2, …) {
// …
});
```

Option事件 | jQuery 事件 | 参数 | 描述
-| -| -| -
onAll	 | all.bs.table	| name, args	| 所有的事件都会触发该事件，参数包括：name：事件名，args：事件的参数。
onClickRow | 	click-row.bs.table	| row, $element	| 当用户点击某一行的时候触发，参数包括：row：点击行的数据，$element：tr 元素，field：点击列的 field 名称。
onDblClickRow	| dbl-click-row.bs.table	| row, $element	| 当用户双击某一行的时候触发，参数包括：row：点击行的数据，$element：tr 元素，field：点击列的 field 名称。
onClickCell	| click-cell.bs.table	| field, value, row, $element	| 当用户点击某一列的时候触发，参数包括：field：点击列的 field 名称，value：点击列的 value 值，row：点击列的整行数据，$element：td 元素。
onDblClickCell	| dbl-click-cell.bs.table	| field, value, row, $element	| 当用户双击某一列的时候触发，参数包括：field：点击列的 field 名称，value：点击列的 value 值，row：点击列的整行数据，$element：td 元素。
onSort	| sort.bs.table	| name, order	| 当用户对某列进行排序时触发，参数包括：name：排序列的 filed 名称，order：排序顺序。
onCheck	| check.bs.table	| row	| 当用户选择某一行时触发，参数包含：row：与点击行对应的记录，$element：选择的DOM元素。
onUncheck	| uncheck.bs.table |	row	| 当用户反选某一行时触发，参数包含：row：与点击行对应的记录，$element：选择的DOM元素。
onCheckAll	| check-all.bs.table	| rows	| 当用户全选所有的行时触发，参数包含：rows：最新选择的所有行的数组。
onUncheckAll	| uncheck-all.bs.table	| rows	| 当用户反选所有的行时触发，参数包含：rows：最新选择的所有行的数组。
onCheckSome |	check-some.bs.table	| rows |	当用户选择某些行时触发，参数包含：rows：相对于之前选择的行的数组。
onUncheckSome	| uncheck-some.bs.table	| rows	| 当用户反选某些行时触发，参数包含：rows：相对于之前选择的行的数组。
onLoadSuccess	| load-success.bs.table	| data	| 远程数据加载成功时触发成功。
onLoadError	| load-error.bs.table	| status	| 远程数据加载失败时触发成功。
onColumnSwitch	| column-switch.bs.table	| field, checked	| 当切换列的时候触发。
onColumnSearch	| column-search.bs.table	| field, text	| 当搜索列时触发。
onPageChange	| page-change.bs.table	| number, size |	当页面更改页码或页面大小时触发。
onSearch	| search.bs.table	| text |	当搜索表格时触发。
onToggle	| toggle.bs.table	| cardView |	切换表格视图时触发。
onPreBody	| pre-body.bs.table |	data	| 在表格 body 渲染之前触发。
onPostBody	| post-body.bs.table	| none |	在表格 body 渲染完成后触发。
onPostHeader	| post-header.bs.table	| none	| 在表格 header 渲染完成后触发。
onExpandRow	| expand-row.bs.table	| index, row, $detail	| 当点击详细图标展开详细页面的时候触发。
onCollapseRow	| collapse-row.bs.table	| index, row	| 当点击详细图片收起详细页面的时候触发。
onRefreshOptions |	refresh-options.bs.table	| options	| 刷新选项之后并在销毁和初始化表之前触发。
onRefresh	| refresh.bs.table	| params	| 点击刷新按钮后触发。
onScrollBody	| scroll-body.bs.table | |表格 body 滚动时触发。



### 方法（Methods）
* 使用方法的语法：$('#table').bootstrapTable('method', parameter);。

方法名	| 参数	| 描述	| 举例
- | - | - | -
getOptions	| none	| 返回表格的 Options。|
getSelections	| none	| 返回所选的行，当没有选择任何行的时候返回一个空数组。|
getAllSelections|	none	|返回所有选择的行，包括搜索过滤前的，当没有选择任何行的时候返回一个空数组。	|
getData	|useCurrentPage	|或者当前加载的数据。假如设置 useCurrentPage 为 true，则返回当前页的数据。|
getRowByUniqueId	|id	|根据 uniqueId 获取行数据。	|
load	|data	|加载数据到表格中，旧数据会被替换。	|
showAllColumns	|none|	显示所有列。|
hideAllColumns	|none	|隐藏所有列。|
append	|data	|添加数据到表格在现有数据之后。	|
prepend	|data	|插入数据到表格在现有数据之前。	|
remove	|params	|从表格中删除数据，包括两个参数： field: 需要删除的行的 field 名称，values: 需要删除的行的值，类型为数组。
removeAll	|-	|删除表格所有数据。|
removeByUniqueId	|id	|根据 uniqueId 删除指定的行。	|
insertRow	|params|	插入新行，参数包括：index: 要插入的行的 index，row: 行的数据，Object 对象。	|
updateRow	|params|	更新指定的行，参数包括：index: 要更新的行的 index，row: 行的数据，Object 对象。
showRow	|params	|显示指定的行，参数包括：index: 要更新的行的 index 或者 uniqueId，isIdField: 指定 index 是否为 uniqueid。
hideRow	|params	|显示指定的行，参数包括：index: 要更新的行的 index，uniqueId: 或者要更新的行的 uniqueid。
getHiddenRows	|show	|获取所有隐藏的行，如果show参数为true，行将再次显示，否则，只返回隐藏的行。|
mergeCells	| options	| 将某些单元格合并到一个单元格，选项包含以下属性： index: 行索引，field: 字段名称，rowspan: 要合并的rowspan数量，colspan: 要合并的colspan数量。
updateCell	| params	|更新一个单元格，params包含以下属性：index: 行索引。field: 字段名称。value: 新字段值。
refresh	| params	|刷新远程服务器数据，可以设置{silent: true}以静默方式刷新数据，并设置{url: newUrl}更改URL。 要提供特定于此请求的查询参数，请设置{query: {foo: 'bar'}}。|
refreshOptions	| options	| 刷新选项。|
resetSearch	| text	| 设置搜索文本。|
showLoading	| none | 显示加载状态。|
hideLoading	| none	| 隐藏加载状态。|
checkAll	| none	| 选中当前页面所有行。|
uncheckAll	| none |	取消选中当前页面所有行。|
check	| index	| 选中某一行，行索引从0开始。|
uncheck	| index	| 取消选中某一行，行索引从0开始。|
checkBy	| params	| 按值或数组选中某行，参数包含：field: 用于查找记录的字段的名称，values: 要检查的行的值数组。例子:``$("#table").bootstrapTable("checkBy", {field:"field_name", values:["value1","value2","value3"]})`` |
uncheckBy	| params	| 按值数组取消选中某行，参数包含：field: 用于查找记录的字段的名称，values: 要检查的行的值数组。例子:``$("#table").bootstrapTable("uncheckBy", {field:"field_name", values:["value1","value2","value3"]})``|
resetView	| params | 重置引导表视图，例如重置表高度。|
resetWidth | none	| 调整页眉和页脚的大小以适合当前列宽度。|
destroy	| none | 销毁表。|
showColumn | field | 显示指定的列。|
hideColumn | field | 隐藏指定的列。|
getHiddenColumns | - |获取隐藏的列。|
getVisibleColumns	| -	| 获取可见列。|
scrollTo | value | 滚动到指定位置，单位为 px，设置 'bottom' 表示跳到最后。|
getScrollPosition	| none | 获取当前滚动条的位置，单位为 px。|
filterBy | params	| （只能用于 client 端）过滤表格数据， 你可以通过过滤{age: 10}来显示 age 等于 10 的数据。|
selectPage |	page | 跳到指定的页。|
prevPage | none |	跳到上一页。|
nextPage	| none |	跳到下一页。|
togglePagination |	none |	切换分页选项。|
toggleView | none |	切换 card/table 视图 |
expandRow	| index	| 如果详细视图选项设置为True，可展开索引为 index 的行。|
collapseRow	| index	| 如果详细视图选项设置为True，可收起索引为 index 的行。|
expandAllRows	| none | 如果详细视图选项设置为True，可展开所有行。|
collapseAllRows	| none	| 如果详细视图选项设置为True，可收起开所有行。|

### 多语言
 名称 | 参数 |	默认	| 说明
  - | - | -
formatLoadingMessage |	-	|‘Loading, please wait…’	| “加载中，请等待……”
formatRecordsPerPage	| pageNumber	| ‘%s records per page’	|“每页显示 15 条记录”
formatShowingRows	| pageFrom, pageTo, totalRows	| ‘Showing %s to %s of %s rows’	|“显示第 1 到第 15 条记录”
formatDetailPagination |	totalRows |	‘Showing %s rows’	| “总共 15 条记录”
formatSearch |	-	| ‘Search’ |	“搜索”（占位符）
formatNoMatches	| -	|‘No matching records found’ |	“没有找到匹配的记录”
formatRefresh	| -	| ‘Refresh’	| “刷新”（鼠标悬浮显示的文字，下同）
formatToggle	| -	| ‘Toggle’ |	“切换”
formatColumns	| -	| ‘Columns’ |	“列”

两种方式引入：
```js
<script src="bootstrap-table-en-US.js"></script>
<script src="bootstrap-table-zh-CN.js"></script>
...
```
```js
$.extend($.fn.bootstrapTable.defaults, $.fn.bootstrapTable.locales['en-US']);
// $.extend($.fn.bootstrapTable.defaults, $.fn.bootstrapTable.locales['zh-CN']);
// ...
```
