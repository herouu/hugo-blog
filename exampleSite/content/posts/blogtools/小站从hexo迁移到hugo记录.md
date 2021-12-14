---  
title: "小站从hexo迁移到hugo记录"
date: 2020-01-29T16:59:15+08:00  
draft: false  
categories: ["环境搭建"]  

---  

## 为什么从hexo迁移到hugo  
本站点从18年开始更新，到今年已有两年，记录的文章大大小小也有70余篇，随着文章的增加，明显感觉hexo的构建速度越来越慢。作为一个developer,对于慢是不能忍的，所以试图寻找替代品，然后就主要调研了vuepress和hugo。

vuepress和hugo同为静态网站生成器，都支持themes主题，相对而言hugo的主题支持比vuepress的好，但次于hexo。vuepress相对来说比较适合维护api文档，hugo比较适合构建个人博客或者站点。总的来说,它们的生态优劣顺序为`hexo>hugo>vuepress`，构建速度顺序`hugo>vuepress>hexo`，特别是当博客数量越来越多的时候，hugo的构建速度优势明显。

在迁移之前，站点的文章都是markdown格式的文档，使用的hexo主题是用的是hexo-yali主题，想迁移的时候，尽量做到平滑，改动较小，这个是主要原因。所以对平滑迁移的需求和站点文章数量增长不影响构建速度的考虑，最终选择使用hugo作为静态博客生成器。
<!--more-->  

## hugo主题选择  
主题的选择，第一原则自然是好看（自己改动较小），第二原则自然是方便自己拓展（开源且代码完整），本着这两个原则在hugo官方主题列表里面搜索(https://themes.gohugo.io/)，最终大概选定了4个主题。
* [hugo-theme-even](https://github.com/olOwOlo/hugo-theme-even)及其拓展主题 [hugo-theme-jane](https://github.com/xianmin/hugo-theme-jane)
* [Hermit](https://github.com/Track3/hermit)演示[demo](https://themes.gohugo.io//theme/hermit/post/)
* [Titan](https://github.com/raymondragon/titan)及演示站点(https://marsgrid.com/)
* [LoveIt](https://github.com/dillonzq/LoveIt)

jane主题是在even主题的基础上修改的，相对于后三个主题来说，样式比较单一，不好看。Hermit属于dark风格，无搜索功能，在演示站post页无文章概述,分类页,对于博客分类相当不友好，titan样式可以，但是代码不完整，主要缺少hugo主题的配合文件。相对而言loveIt就好多了，除了没有站内搜索剩下的都有，支持标签跟分类，支持双主题（light和dark），所以最终选择的主题是loveIt。
## 站点部署  
之前的静态网站使用的hexo+github page构建的博客网站，文章多了用`hexo d`命令提交到github上比较耗时，如果遇到国内网络管制，常常提交不成功。利用netlify持续集成工具部署静态站点,可以支持支持自定义域名及域名重定向，DNS解析加速网站，配置https增强网站数据安全。最主要的是代码跟部署实现解耦，只要配置正确，无需关系部署，后期只需要关心文章内容的质量。

### window下hugo安装

在window下使用window的包管理工具scoop安装。
* 安装scoop
在 PowerShell中执行，这里会将scoop默认安装到c盘，可以修改安装路径
```shell
Set-ExecutionPolicy RemoteSigned -scope CurrentUser

# 设置安装路径
$env:SCOOP='D:\scoop'
[environment]::setEnvironmentVariable('SCOOP',$env:SCOOP,'User')

Invoke-Expression (New-Object System.Net.WebClient).DownloadString('https://get.scoop.sh')
# 或
iwr -useb get.scoop.sh | iex
```
* 安装hugo
`scoop install hugo`,执行完毕后，自动将hugo命令配置到环境变量中。

在任意路径下执行
`hugo new site my-blog`

跟hexo一样，有严格的目录结构约定，目录结构说明如下

```
├───archetypes
├───content  文章路径
├───data  数据资源
├───layouts  布局文件
├───static 静态资源js、html、css
└───themes 主题
```

### fork主题后添加git submodele

在github上fork loveIt主题为自己的主题,方便以后修改。将生成的站点使用git管理，并添加submodeles为自己fork后的loveIt主题,并clone到themes文件夹下。
![](images/小站从hexo迁移到hugo记录/git-fork添加GitSubmodule.png "小站从hexo迁移到hugo记录")

### config.toml配置
将主题中的simpleSite路径下的config.toml粘贴到themes同级目录下
```
├───static 静态资源js、html、css
├───themes 主题
└───config.toml 
```
并注意以下配置

```toml

theme = "LoveIt"  #修改主题为LoveIt  
staticDir = ["../static", "../../assets/others"]    # 静态文件目录，../ 找的是主题中的static文件,若没有将simpleSite中的static文件夹复制为跟simpleSite同级
publishDir =["../public"]  

```
### 提交代码到github并部署到netlify
github站点部署到netlify主要参考 [手把手教你使用Netlify部署博客及部署自动化
](https://zhuanlan.zhihu.com/p/55252024)

除此之外，还要新增如下配置：
新建netlify.toml,跟config.toml同级并添加如下配置

```toml
[build]
  command = "hugo --gc --minify"
  

[build.environment]
  HUGO_VERSION = "0.60.1"
  HUGO_ENABLEGITINFO = "true"

[context.production.environment]
  HUGO_ENV = "production"

#[context.deploy-preview]
#  command = "hugo --gc --minify --buildFuture -b $DEPLOY_PRIME_URL"

#[context.branch-deploy]
#  command = "hugo --gc --minify -b $DEPLOY_PRIME_URL"

[[headers]]
  for = "*.webmanifest"
  [headers.values]
    Content-Type = "application/manifest+json; charset=UTF-8"

[[headers]]
  for = "index.xml"
  [headers.values]
    Content-Type = "application/rss+xml"

```
* 注意下图位置，什么也不需要填，在netlify.toml中必要项已配置
![](images/小站从hexo迁移到hugo记录/netlify_build_settings.png "小站从hexo迁移到hugo记录")


## 站内搜索  
这个对原主题改动比较多，开始使用bootstrap modal作为弹窗，但是bootstrap样式对主题侵染严重，出于少改动的原则，找modal的替代品，在
[jquery-modal](https://github.com/kylefox/jquery-modal)与[iziModal](https://github.com/marcelodolza/iziModal)之中进行尝试，因为样式侵染的问题，放弃了iziModal。

### 新建search.html
在主题路径下`themes\LoveIt\layouts\partials`新建search.html并添加如下代码
```html
<div class="aa-input-container" id="aa-input-container">
    <input type="search" id="aa-search-input" class="aa-input-search" placeholder="Search for titles or URIs..." name="search" autocomplete="off" />
    <svg class="aa-input-icon" viewBox="654 -372 1664 1664">
        <path d="M1806,332c0-123.3-43.8-228.8-131.5-316.5C1586.8-72.2,1481.3-116,1358-116s-228.8,43.8-316.5,131.5  C953.8,103.2,910,208.7,910,332s43.8,228.8,131.5,316.5C1129.2,736.2,1234.7,780,1358,780s228.8-43.8,316.5-131.5  C1762.2,560.8,1806,455.3,1806,332z M2318,1164c0,34.7-12.7,64.7-38,90s-55.3,38-90,38c-36,0-66-12.7-90-38l-343-342  c-119.3,82.7-252.3,124-399,124c-95.3,0-186.5-18.5-273.5-55.5s-162-87-225-150s-113-138-150-225S654,427.3,654,332  s18.5-186.5,55.5-273.5s87-162,150-225s138-113,225-150S1262.7-372,1358-372s186.5,18.5,273.5,55.5s162,87,225,150s113,138,150,225  S2062,236.7,2062,332c0,146.7-41.3,279.7-124,399l343,343C2305.7,1098.7,2318,1128.7,2318,1164z" />
    </svg>
</div>
```

### 添加search.css文件
在`themes\LoveIt\assets\css`路径下添加search.css样式文件
```css
@import 'https://fonts.googleapis.com/css?family=Montserrat:400,700';
.aa-input-container {
  display: inline-block;
  position: relative;
  width: 100%;
}
.aa-input-container span,.aa-input-container input {
    width: inherit;
}
.aa-input-search {
  width: 300px;
  padding: 12px 28px 12px 12px;
  border:0px;
  border: 2px solid #e4e4e4;
  border-radius: 4px;
  -webkit-transition: .2s;
  transition: .2s;
  font-family: "Montserrat", sans-serif;
  box-shadow: 4px 4px 0 rgba(241, 241, 241, 0.35);
  font-size: 11px;
  box-sizing: border-box;
  color: black;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  font-weight: bold;
}
.aa-input-search::-webkit-search-decoration, .aa-input-search::-webkit-search-cancel-button, .aa-input-search::-webkit-search-results-button, .aa-input-search::-webkit-search-results-decoration {
    display: none;
}
.aa-input-search:focus {
    outline: 0;
    border-color: #D3D3D3;
    box-shadow: 4px 4px 0 rgba(58, 150, 207, 0.1);
}
.aa-input-icon {
  height: 16px;
  width: 16px;
  position: absolute;
  top: 50%;
  right: 16px;
  -webkit-transform: translateY(-50%);
          transform: translateY(-50%);
  fill: #e4e4e4;
}
.aa-hint {
  color: yellow;
}
.aa-dropdown-menu {
  background-color: #fff;
  border: 2px solid rgba(228, 228, 228, 0.6);
  border-top-width: 1px;
  font-family: "Montserrat", sans-serif;
  width: 300px;
  margin-top: 10px;
  box-shadow: 4px 4px 0 rgba(241, 241, 241, 0.35);
  font-size: 11px;
  border-radius: 4px;
  box-sizing: border-box;
}
.aa-suggestion {
  padding: 12px;
  border-top: 1px solid gray;
  cursor: pointer;
  -webkit-transition: .2s;
  transition: .2s;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-pack: justify;
      -ms-flex-pack: justify;
          justify-content: space-between;
  -webkit-box-align: center;
      -ms-flex-align: center;
          align-items: center;
}
/* 背景色 */
.aa-suggestion:hover, .aa-suggestion.aa-cursor {
    background-color: #F5F5F5;
}
.aa-suggestion > span:first-child {
    color: #000;
}
.aa-suggestion > span:last-child {
    text-transform: uppercase;
    color: #000;
}
.aa-suggestion > span:first-child em, .aa-suggestion > span:last-child em {
  font-weight: 700;
  font-style: normal;
  background-color: rgba(0, 0, 0, 0.1);
  padding: 2px 0 2px 2px;
}

.aa-suggestion a{
  color: #000;
  font-weight:bold;
}
.aa-suggestion a:hover{
  color: #4B89DC;
}

```

### 添加search.js
在路径`themes\LoveIt\assets\js`下添加search.js
```javascript
$(function() {
  // 替换成自己的algolia信息
  var client = algoliasearch("UV3***OQZ", "7139db3dc76681*****b87c36ab191d");
  var index = client.initIndex("***");
  autocomplete(
    "#aa-search-input",
    { hint: false },
    {
      source: autocomplete.sources.hits(index, { hitsPerPage: 8 }),
      displayKey: "name",
      templates: {
        suggestion: function(suggestion) {
          console.log(suggestion);
          var search;
          if (suggestion.search) {
            search = suggestion.search;
          } else {
            search = suggestion.title;
          }
          return (
            "<span>" +
            '<a href="/' +
            search +
            '">' +
            suggestion._highlightResult.title.value +
            "</a></span>"
          );
        }
      }
    }
  );


  $(document).on("click", ".aa-suggestion", function () { 
    var aa = $(this).find("a").attr("href");
    window.location.href = aa;
  })
});

```

### 在baseof.html中添加代码
在`themes\LoveIt\layouts\_default\baseof.html`中添加代码
```html
{{ if ne .Site.Params.version "5.x" -}}
{{ errorf "\n\nThere are two possible situations that led to this error:\n  1. You haven't copied the config.toml yet. See https://github.com/dillonzq/LoveIt#installation \n  2. You have an incompatible update. See https://github.com//dillonzq/LoveIt/blob/master/CHANGELOG.md \n\n有两种可能的情况会导致这个错误发生:\n  1. 你还没有复制 config.toml 参考 https://github.com/dillonzq/LoveIt#installation \n  2. 你进行了一次不兼容的更新 参考 https://github.com//dillonzq/LoveIt/blob/master/CHANGELOG.md \n" -}}
{{ end -}}
<!DOCTYPE html>
<html lang="{{ .Site.LanguageCode }}">

<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <title>{{ block "title" . }}{{ .Site.Title }}{{ end }}</title>
    {{- partial "head.html" . }}
</head>

<body>
    <script>
        window.isDark = (window.localStorage && window.localStorage.getItem('theme')) === 'dark';
        window.isDark && document.body.classList.add('dark-theme');
    </script>
    <div class="wrapper">
        {{ partial "header.html" . -}}
        <main class="main">
            <div class="container">
                {{ block "content" . }}{{ end -}}
            </div>
        </main>
        {{ partial "footer.html" . -}}
        {{ partial "scripts.html" . -}}
    </div>
    <a href="#" class="dynamic-to-top" id="dynamic-to-top" data-scroll><span>&nbsp;</span></a>

<!-- 添加的代码  -->
    <div id="ex1" class="modal">
        {{ partial "search.html" . }}
    </div>
<!-- 添加代码结束 -->
</body>

</html>
```
### 引用css文件和js文件
* 在head.html中添加如下代码

```html
{{ $res := resources.Get "css/search.css" | resources.Minify -}}
<link rel="stylesheet" href="{{ $res.RelPermalink }}">

<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jquery-modal/0.9.1/jquery.modal.min.css" />
```
* 在scripts.html中添加如下代码

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.0.0/jquery.min.js"></script>

<!-- jQuery Modal -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-modal/0.9.1/jquery.modal.min.js"></script>

<script src="https://res.cloudinary.com/jimmysong/raw/upload/rootsongjc-hugo/algoliasearch.min.js"></script>
<script src="https://res.cloudinary.com/jimmysong/raw/upload/rootsongjc-hugo/autocomplete.min.js"></script>


 {{ $res := resources.Get "/js/search.js" }}
<script src= "{{ $res.RelPermalink }}" type="text/javascript"></script>

```

### header.html添加锚点

```html
<div class="navbar-menu">
            {{ $currentPage := . }}
            {{ range .Site.Menus.main }}
            <a class="menu-item{{ if or ($currentPage.IsMenuCurrent "main" .) ($currentPage.HasMenuCurrent "main" .) | or (eq $currentPage.RelPermalink .URL) }} active{{ end }}"
                href="{{ .URL | absLangURL }}" title="{{ .Title }}">{{ .Name | safeHTML }}</a>
            {{ end }}
            <a href="javascript:void(0);" class="theme-switch"><i class="fas fa-adjust fa-rotate-180 fa-fw"></i></a>
            <!-- 添加的代码 -->
            <a href="#ex1" rel="modal:open"><i class="fas fa-search fa-fw"></i></a>
            <!-- 添加代码结束 -->
        </div>

```
### 添加config.yaml用于algolia index的生成
在themes同级添加config.yaml文件
```yaml
---
baseURL: "https://www.alertcode.top/"
languageCode: "zh"
title: "Bob's Blog | Java Developer"
theme: "LoveIt"
description: "一个记录个人工作、生活的blog"

#下面信息填写自己的algolia信息
algolia:
  index: "bob-xxxx"
  key: "7139db3dcxxxxxxxxxxxxxxc36ab191d"
  appID: "UV3xxxxOQZ"
---

```

## 站点优化 

### 站点响应速度优化 

响应速度优化，主要是将一些js、css文件使用cdn加速，配置如下
```toml
 [params.cdn]                                       #### CSS 和 JS 文件的 CDN 设置                        # 例如 '<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@5.10.1/css/all.min.css" integrity="sha256-fdcFNFiBMrNfWL6OcAGQz6jDgNTRxnrLEd4vJYFWScE=" crossorigin="anonymous">'
    fontawesome_free_css = '<link href="https://cdn.bootcss.com/font-awesome/5.11.2/css/all.min.css" rel="stylesheet">'                           # 例如 '<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@5.10.1/css/all.min.css" integrity="sha256-fdcFNFiBMrNfWL6OcAGQz6jDgNTRxnrLEd4vJYFWScE=" crossorigin="anonymous">'
    animate_css = '<link href="https://cdn.bootcss.com/animate.css/3.7.2/animate.css" rel="stylesheet">'
    gitalk_css = '<link href="https://cdn.bootcss.com/gitalk/1.5.0/gitalk.min.css" rel="stylesheet">'
    gitalk_js = '<script src="https://cdn.bootcss.com/gitalk/1.5.0/gitalk.min.js"></script>'
    jquery_js = '<script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js"></script>'
    lazysizes_js = '<script src="https://cdn.bootcss.com/lazysizes/5.1.2/lazysizes.min.js"></script>'
    smooth_scroll_js = '<script src="https://cdn.bootcss.com/smooth-scroll/16.1.0/smooth-scroll.polyfills.min.js"></script>'
    typeit_js = '<script src="https://cdn.bootcss.com/typeit/6.0.3/typeit.min.js"></script>'

```

### 评论 
使用gittalk，麻烦的地方是github相关的配置，请参考
[静态博客配置 gitalk 评论系统](https://www.boliball.com/LlaOgQO7jBmrBkNtdAwxwV0y)
* 配置 
```toml
  [params.gitalk] # Gittalk 评论系统设置 (https://github.com/gitalk/gitalk)
    owner = "GitHub repo owner"
    repo = "'GitHub repo git项目名称" 
    clientId = "GitHub Application Client ID"
    clientSecret = "GitHub Application Client Secret"
```



### 个人简介(关于) 
使用hugo-orbit-theme主题,在原主题的基础上，通过cdn加速js、css加载，添加首次浏览，密码简单校验的功能。

![预览效果](images/小站从hexo迁移到hugo记录/简历预览.png "小站从hexo迁移到hugo记录")


### 移动端优化  
因为添加了搜索按钮、造成站点在移动端现实不美观，更改了按钮的布局样式
![](images/小站从hexo迁移到hugo记录/按钮布局.png "小站从hexo迁移到hugo记录")

### 自动构建algolia索引
#### 安装node.js
`scoop install node-lts`
#### npm init
在themes同级目录执行npm init
#### 添加构建脚本
在生成的package.json中添加脚本
```json
  "scripts": {
    "algolia": "hugo-algolia -i \"content/posts/**\" -s "
  },
```
在netlify.toml中修改build配置如下
```toml
[build]
  command = "hugo --gc --minify \n npm run algolia"
```
