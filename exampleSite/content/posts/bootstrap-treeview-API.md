---
title: bootstrap-treeview API
date: 2018-07-22 22:20:50
tags: ["bootstrap"]

---
&emsp;&emsp; bootstrap-treeview API
&emsp;&emsp; github地址 https://github.com/jonmiles/bootstrap-treeview
<!--more-->
### 简要教程
bootstrap-treeview是一款效果非常酷的基于bootstrap的jQuery多级列表树插件。该jQuery插件基于Twitter Bootstrap，以简单和优雅的方式来显示一些继承树结构，如视图树、列表树等等。

bootstrap列表树示意图

### 插件依赖
Bootstrap v3.0.3
jQuery v2.0.3
以上两个外部依赖文件已经经过测试可以正常使用，其他版本的Bootstrap需要另行测试。该插件不支持bootstrap 2。

### 使用方法
首先要在页面中引入依赖文件和 bootstrap-treeview.js文件。

```html
<!-- Required Stylesheets -->
<linkhref="./css/bootstrap.css"rel="stylesheet">

<!-- Required Javascript -->
<scriptsrc="./js/jquery.js"></script>
<scriptsrc="./js/bootstrap-treeview.js"></script>
```

### HTML结构
可以使用任何HTML DOM元素来作为该列表树的容器：
```html
<div id="tree"></div>
```

### 调用插件
该列表树插件最基本的调用方法如下：

```js
function getTree() {
    // Some logic to retrieve, or generate tree structure
    returndata;
}
$('#tree').treeview({data: getTree()});  
```

### 数据结构
为了创建树的继承结构，需要为该列表树插件提供一个嵌套结构的js对象。例如：

```js
var tree = [
  {
    text:"Parent 1",
    nodes: [
      {
        text:"Child 1",
        nodes: [
          {
            text:"Grandchild 1"
          },
          {
            text:"Grandchild 2"
          }
        ]
      },
      {
        text:"Child 2"
      }
    ]
  },
  {
    text:"Parent 2"
  },
  {
    text:"Parent 3"
  },
  {
    text:"Parent 4"
  },
  {
    text:"Parent 5"
  }
];
```

最简单的树结构可以只有一个节点，使用一个带text属性的js对象来实现即可：

```js
{
    text:"Node 1"
}  
```

如果你需要自定义更多的内容，可以参考下面：

```js
{
  text:"Node 1",
  icon:"glyphicon glyphicon-stop",
  selectedIcon:"glyphicon glyphicon-stop",
  color:"#000000",
  backColor:"#FFFFFF",
  href:"#node-1",
  selectable:true,
  state: {
    checked:true,
    disabled:true,
    expanded:true,
    selected:true
  },
  tags: ['available'],
  nodes: [
    {},
    ...
  ]
}  
```

### 节点属性
下面的参数可用于树节点的属性定义，如节点的文本、颜色和标签等。

参数名称 | 参数类型 | 参数描述
- | - | -
text	| String(必选项) | 	列表树节点上的文本，通常是节点右边的小图标。
icon|	String(可选项)	|列表树节点上的图标，通常是节点左边的图标。
selectedIcon|	String(可选项)	| 当某个节点被选择后显示的图标，通常是节点左边的图标。
href|	String(可选项)	| 结合全局enableLinks选项为列表树节点指定URL。
selectable |	Boolean. Default: true|	指定列表树的节点是否可选择。设置为false将使节点展开，并且不能被选择。
state	| Object(可选项)|	一个节点的初始状态。
state.checked |	Boolean，默认值false|	指示一个节点是否处于checked状态，用一个checkbox图标表示。
state.disabled	| Boolean，默认值false	|指示一个节点是否处于disabled状态。（不是selectable，expandable或checkable）
state.expanded|	Boolean，默认值false|	指示一个节点是否处于展开状态。
state.selected|	Boolean，默认值false|	指示一个节点是否可以被选择。
color	|String. Optional|	节点的前景色，覆盖全局的前景色选项。
backColor	| String. Optional |	节点的背景色，覆盖全局的背景色选项。
tags	| Array of Strings. Optional	| 通过结合全局showTags选项来在列表树节点的右边添加额外的信息。

### 全局参数
参数可以定制treeview的默认外观和行为。参数使用一个对象来在插件初始化时传入：

```js
// Example: initializing the treeview
// expanded to 5 levels
// with a background color of green
$('#tree').treeview({
  data: data,        // data is not optional
  levels: 5,
  backColor:'green'
});
```

参数名称 |	参数类型|	默认值	|描述
-|-|-|-
data|	Array of Objects|	无	|列表树上显示的数据。
backColor	| String|	所有合法的颜色值，Default: inherits from Bootstrap.css。	|设置所有列表树节点的背景颜色。
borderColor	|String|	所有合法的颜色值，Default: inherits from Bootstrap.css。|	设置列表树容器的边框颜色，如果不想要边框可以设置showBorder属性为false。
checkedIcon	|String：class名称	|Bootstrap Glyphicons定义的 "glyphicon glyphicon-check"	|设置处于checked状态的复选框图标。
collapseIcon	|String：class名称|	Bootstrap Glyphicons定义的 "glyphicon glyphicon-minus"	| 设置列表树可收缩节点的图标。
color	|String	|所有合法的颜色值，Default: inherits from Bootstrap.css。|	设置列表树所有节点的前景色。
emptyIcon	| String：class名称|	Bootstrap Glyphicons定义的"glyphicon"。	| 设置列表树中没有子节点的节点的图标。
enableLinks	|Boolean|	false|	是否使用当前节点的文本作为超链接。超链接的href值必须在每个节点的data结构中给出。
expandIcon	| String：class名称| 	Bootstrap Glyphicons定义的 "glyphicon glyphicon-plus"	|设置列表树可展开节点的图标。
highlightSearchResults |	Boolean	|true	|是否高亮搜索结果。
highlightSelected	|Boolean|	true	|当选择节点时是否高亮显示。
onhoverColor	|String	|所有合法的颜色值， Default: '#F5F5F5'。|	设置列表树的节点在用户鼠标滑过时的背景颜色。
levels |	Integer	| Default: 2	|设置继承树默认展开的级别。
multiSelect	| Boolean	| false	| 是否可以同时选择多个节点。
nodeIcon	| String：class名称|	Bootstrap Glyphicons定义的 "glyphicon glyphicon-stop"	|设置所有列表树节点上的默认图标。
selectedIcon|	String：class名称|	Bootstrap Glyphicons定义的 "glyphicon glyphicon-stop"	|设置所有被选择的节点上的默认图标。
searchResultBackColor	|String|	所有合法的颜色值， Default: undefined, inherits。|	设置搜索结果节点的背景颜色。
searchResultColor	| String	|所有合法的颜色值， Default: '#D9534F' |	设置搜索结果节点的前景颜色。
selectedBackColor |	String|	所有合法的颜色值， Default: '#428bca'|	设置被选择节点的背景颜色。
selectedColor	| String	| 所有合法的颜色值， Default: '#FFFFFF'。|	设置列表树选择节点的背景颜色。
showBorder	| Boolean	| true|	是否在节点上显示边框。
showCheckbox	|Boolean	| false	| 是否在节点上显示checkboxes。
showIcon|	Boolean|	true	|是否显示节点图标。
showTags	| Boolean	| false	| 是否在每个节点右边显示tags标签。tag值必须在每个列表树的data结构中给出。
uncheckedIcon	| String：class名称| 	Bootstrap Glyphicons定义的 "glyphicon glyphicon-unchecked"|	设置图标为未选择状态的checkbox图标。

### 可用方法
你可以通过两种方式来调用方法：

* 1、插件包装器：插件的包装器可以作为访问底层方法的代理。
``$('#tree').treeview('methodName', args) ``
多个参数必须使用数组对象来传入。

* 2、直接使用treeview：你可以通过下面两种方法中的一种来获取treeview对象实例。

```js
//该方法返回一个treeview的对象实例
$('#tree').treeview(true).methodName(args);

//对象实例也保存在DOM元素的data中，
//可以使用'treeview'的id来访问它。
$('#tree').data('treeview')
  .methodName(args);
```
### treeview方法列表
* checkAll(options)：选择所有的节点。
``$('#tree').treeview('checkAll', { silent:true });``
触发nodeChecked事件，传入silent来阻止其它事件。
* checkNode(node | nodeId, options)：选择指定的节点，接收节点或节点ID。
``$('#tree').treeview('checkNode', [ nodeId, { silent:true } ]);``
触发nodeChecked事件，传入silent来阻止其它事件。

* clearSearch()：清空以前的搜索结果。例如清除它们的高亮状态。
``$('#tree').treeview('clearSearch');``
触发searchCleared事件。

* collapseAll(options)：折叠所有的节点，折叠整个树。
``$('#tree').treeview('collapseAll', { silent:true });``
触发nodeCollapsed事件，传入silent来阻止其它事件。

* collapseNode(node | nodeId, options)：折叠指定节点和它的子节点。如果不想折叠子节点，可以设置{ ignoreChildren: true }。
``$('#tree').treeview('collapseNode', [ nodeId, { silent:true, ignoreChildren:false } ]);``
触发nodeCollapsed事件，传入silent来阻止其它事件。

* disableAll(options)：禁用所有的节点。
``$('#tree').treeview('disableAll', { silent:true });``
触发nodeDisabled事件，传入silent来阻止其它事件。

* disableNode(node | nodeId, options)：禁用指定的节点，接收节点或节点ID。
``$('#tree').treeview('disableNode', [ nodeId, { silent:true } ]);``
触发nodeDisabled事件，传入silent来阻止其它事件。

* enableAll(options)：启用所有的节点。
``$('#tree').treeview('enableAll', { silent:true });
``触发nodeEnabled事件，传入silent来阻止其它事件。

* enableNode(node | nodeId, options)：启用指定的节点，接收节点或节点ID。
``$('#tree').treeview('enableNode', [ nodeId, { silent:true } ]);``
触发nodeEnabled事件，传入silent来阻止其它事件。

* expandAll(options)：展开所有的树节点。也可以展开任何给定级别的树节点。
``$('#tree').treeview('expandAll', { levels: 2, silent:true });``
触发nodeExpanded事件，传入silent来阻止其它事件。

* expandNode(node | nodeId, options)：展开指定的树节点，接收节点或节点ID。也可以展开任何给定级别的树节点。
``$('#tree').treeview('expandNode', [ nodeId, { levels: 2, silent:true } ]);``
触发nodeExpanded事件，传入silent来阻止其它事件。

* getCollapsed()：返回折叠节点的数组。例如state.expanded = false。
``$('#tree').treeview('getCollapsed', nodeId);``
* getDisabled()：返回被禁用节点的数组。
``$('#tree').treeview('getDisabled', nodeId);``

* getEnabled()：返回可用节点的数组。
``$('#tree').treeview('getEnabled', nodeId);``

* getExpanded()：返回所有展开节点的数组。
``$('#tree').treeview('getExpanded', nodeId);``

* getNode(nodeId)：返回给定节点ID的单一节点对象。
``$('#tree').treeview('getNode', nodeId);``

* getParent(node | nodeId)：返回给定节点的父节点，如果没有则返回undefined。
``$('#tree').treeview('getParent', node);``

* getSelected()：返回所有被选择节点的数组，例如：state.selected = true。
``$('#tree').treeview('getSelected', nodeId);``

* getSiblings(node | nodeId)：返回给定节点的兄弟节点的数组，如果没有则返回undefined。
``$('#tree').treeview('getSiblings', node);``

* getUnselected()：返回没有被选择节点的数组。例如：state.selected = false。
``$('#tree').treeview('getUnselected', nodeId);``

* remove()：移除列表树容器。移除附加的事件、附加对象和额外的html元素。
``$('#tree').treeview('remove');``

* revealNode(node | nodeId, options)：显示一个树节点，展开从这个节点开始到根节点的所有节点。
``$('#tree').treeview('revealNode', [ nodeId, { silent:true } ]);``
触发nodeExpanded事件，传入silent来阻止其它事件。

* search(pattern, options)：搜索匹配是非常的指定树节点，并高亮它们。返回配平节点的数组。
```js
$('#tree').treeview('search', ['Parent', {
  ignoreCase:true,     // case insensitive
  exactMatch:false,    // like or equals
  revealResults:true,  // reveal matching nodes
}]);
```
触发searchComplete事件。

* selectNode(node | nodeId, options)：选择一个给定的树节点，接收节点或节点ID。
``$('#tree').treeview('selectNode', [ nodeId, { silent:true } ]);``
触发nodeSelected事件，传入silent来阻止其它事件。

* toggleNodeChecked(node | nodeId, options)：切换节点的Check状态。
``$('#tree').treeview('toggleNodeChecked', [ nodeId, { silent:true } ]);``
触发nodeChecked事件或nodeUnchecked事件，传入silent来阻止其它事件。

* toggleNodeDisabled(node | nodeId, options)：切换一个节点的可用和不可用状态。
``$('#tree').treeview('toggleNodeDisabled', [ nodeId, { silent:true } ]);``
触发nodeDisabled事件或nodeEnabled事件，传入silent来阻止其它事件。

* toggleNodeExpanded(node | nodeId, options)：切换一个节点的展开和折叠状态。
``$('#tree').treeview('toggleNodeExpanded', [ nodeId, { silent:true } ]);``
触发nodeExpanded事件或nodeCollapsed事件，传入silent来阻止其它事件。

* toggleNodeSelected(node | nodeId, options)：切换一个节点的选择状态。
``$('#tree').treeview('toggleNodeSelected', [ nodeId, { silent:true } ]);``
触发nodeSelected事件或nodeUnselected事件，传入silent来阻止其它事件。

* uncheckAll(options)：Uncheck所有的节点。
`$`('#tree').treeview('uncheckAll', { silent:true });``
触发nodeUnchecked事件，传入silent来阻止其它事件。

* uncheckNode(node | nodeId, options)：Uncheck一个给定的节点，接收节点或节点ID。
``$('#tree').treeview('uncheckNode', [ nodeId, { silent:true } ]);``
触发nodeUnchecked事件，传入silent来阻止其它事件。

* unselectNode(node | nodeId, options)：不选择指定的节点，接收节点或节点ID。
``$('#tree').treeview('unselectNode', [ nodeId, { silent:true } ]);``
触发nodeUnselected事件，传入silent来阻止其它事件。

### 事件
你可以在参数中使用回调函数来绑定任何事件，或者使用标准的jQuery .on()方法来绑定事件。

在参数中调用的示例：
```js
$('#tree').treeview({
  // The naming convention for callback's is to prepend with `on`
  // and capitalize the first letter of the event name
  // e.g. nodeSelected -> onNodeSelected
  onNodeSelected:function(event, data) {
    // 事件代码...
});   
```            
使用jQuery .on方法：
```js
$('#tree').on('nodeSelected',function(event, data) {
  // 事件代码...
});
```              
### 可用事件列表
* nodeChecked (event, node)：一个节点被checked。
* nodeCollapsed (event, node)：一个节点被折叠。
* nodeDisabled (event, node)：一个节点被禁用。
* nodeEnabled (event, node)：一个节点被启用。
* nodeExpanded (event, node)：一个节点被展开。
* onNodeSelected(event, node)：一个节点被选择。
* nodeUnchecked (event, node)：一个节点被unchecked。
* nodeUnselected (event, node)：取消选择一个节点。
* searchComplete (event, results)：搜索完成之后触发。
* searchCleared (event, results)：搜索结果被清除之后触发。
