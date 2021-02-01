---
title: hexo折腾日记
date: 2018-06-01 18:16:40

---
&emsp;&emsp;hexo作为静态博客还是不错的，主要记录了个人的hexo 搭建及改主题遇到的一些坑。
<!--more-->

### 2018-02-21 hexo-next 主题改进
**1. 增加图片功能**
* 把主页配置文件_config.yml 里的`post_asset_folder`:这个选项设置为`true`
*  在你的hexo目录下执行这样一句话`npm install hexo-asset-image --save`，这是下载安装一个可以上传本地图片的插件
* 等待一小段时间后，再运行`hexo n "xxxx"`来生成md博文时，/source/_posts文件夹内除了xxxx.md文件还有一个同名的文件夹

**2. 网页在线联系功能**

&emsp;&emsp;主要参照了[asdfv1929][id]的博客,添加之后效果如下：
[id]: https://asdfv1929.github.io/2018/01/21/daovoice/#more
![image](hexo添加在线聊天功能.png)   

**3. 修改分享功能**
 &emsp;&emsp;之前的博客分享功能是采用主题配置文件下的`needmoreshare2`,这个插件的微信分享功能不好用，替换成`jiathis`,需要将_config.yml中的jiathis设置为true：
```
jiathis: true
```
**4. 增加卜算子统计**
&emsp;&emsp;主题配置文件中busuanzi_count的 `enable: false` 变更为`true`
**5. 文章修改时间变更**
&emsp;&emsp;主题配置文件更改post_meta下，update_at属性
```
post_meta:
  item_text: true
  created_at: true
  updated_at: false 变更为 true
  categories: true

```
### 2018-03-08 hexo-next 计划添加
** 1. 一个时间插件 **
** 2. 鼠标拖动特效 **
** 3. 添加链接GitHub图标 **

### 2018-03-09 hexo-next 添加时间插件
** 1. 在next主题下添加配置time_clock: true**
** 2. 在source/js/目录下添加`moment.min.js`,`jquery-16.11.3.min.js`,`clock.js` ,在source/css/目录下添加`clock03.css`**
        moment.js 地址 https://momentjs.com/
        jquery.js 地址 https://jquery.com/
        clock.js 代码：
```javascript
$(function() {

                var week = $("#clock").find(".week");
                var alarm = $("#clock").find(".alarm");
                var time = $("#clock").find(".time");
                var alarm = $("#clock").find(".ampm");
                //定义时间字符串
                var time_name = "zero one two three four five six seven eight nine".split(" ");
                //定义日期字符串       
                var week_name = "MON TUE WED THU FRI SAT SUN".split(" ");
                var position = "h1 h2 : m1 m2 : s1 s2".split(" ");
                var arr = {};
                $.each(position, function() {
                    if (this == ":") {
                        time.append("<div class='dig'></div>");
                    } else {
                        var pos = $("<div>");
                        for (var i = 1; i < 8; i++) {
                            pos.append("<span class='d" + i + "'></span>")
                            arr[this] = pos;
                        }
                        time.append(pos);
                    }
                });
                (function time_update() {
                    var now = moment().format("HHmmssdA");
                    arr.h1.attr('class', time_name[now[0]]); //h1位置的时间，对应time_name数组的名字，下面同理
                    arr.h2.attr('class', time_name[now[1]]);
                    arr.m1.attr('class', time_name[now[2]]);
                    arr.m2.attr('class', time_name[now[3]]);
                    arr.s1.attr('class', time_name[now[4]]);
                    arr.s2.attr('class', time_name[now[5]]);
                    var dom = now[6];
                    dom--;
                    if (dom < 0) {
                        dom = 6;
                    }
                    var ampm = now[7] + now[8];
                    setTimeout(time_update, 1000);
                })();
            });
```
    clock03.css 代码
```css
    * {
                padding: 0;
                margin: 0;
            }

            html {
                overflow: scroll;
            }

            body {
                font: 15px/1.3 Arial, sans-serif;
                color: #4f4f4f;
                z-index: 1;
            }




            #clock .time {
                padding: 10px 5px 10px 10px;
                border-radius: 6px;
                height: 50px;
            }
            /*添加light样式 */

            #clock.light {
                background-color: #f3f3f3;
            }

            #clock.light:after {
                box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
            }
            #clock.light .time{
                background-color:#DDDDDD ;
                box-shadow: 0 1px 1px rgba(0,0,0,0.15) inset,0 1px 1px rgba(0,0,0,0.15);
            }


#clock .time div{
    text-align:left;
    position:relative;
    width: 28px;
    height:50px;
    display:inline-block;
    margin:0 2px;
}

#clock .time div span{
    opacity:0;
    position:absolute;
    border-color: #272e38;
    background-color: #272e38;
    -webkit-transition:0.25s;
    -moz-transition:0.25s;
    transition:0.25s;
}

#clock.light .time div.dig:before,
#clock.light .time div.dig:after{
    background-color:#272e38;
}


#clock .time div span:before,
#clock .time div span:after{
    content:'';
    position:absolute;
    width:0;
    height:0;
    border:5px solid transparent;
}

#clock .time .d1{           height:5px;width:16px;top:0;left:6px;}
#clock .time .d1:before{    border-width:0 5px 5px 0;border-right-color:inherit;left:-5px;}
#clock .time .d1:after{ border-width:0 0 5px 5px;border-left-color:inherit;right:-5px;}

#clock .time .d2{           height:5px;width:16px;top:24px;left:6px;}
#clock .time .d2:before{    border-width:3px 4px 2px;border-right-color:inherit;left:-8px;}
#clock .time .d2:after{ border-width:3px 4px 2px;border-left-color:inherit;right:-8px;}

#clock .time .d3{           height:5px;width:16px;top:48px;left:6px;}
#clock .time .d3:before{    border-width:5px 5px 0 0;border-right-color:inherit;left:-5px;}
#clock .time .d3:after{ border-width:5px 0 0 5px;border-left-color:inherit;right:-5px;}

#clock .time .d4{           width:5px;height:14px;top:7px;left:0;}
#clock .time .d4:before{    border-width:0 5px 5px 0;border-bottom-color:inherit;top:-5px;}
#clock .time .d4:after{ border-width:0 0 5px 5px;border-left-color:inherit;bottom:-5px;}

#clock .time .d5{           width:5px;height:14px;top:7px;right:0;}
#clock .time .d5:before{    border-width:0 0 5px 5px;border-bottom-color:inherit;top:-5px;}
#clock .time .d5:after{ border-width:5px 0 0 5px;border-top-color:inherit;bottom:-5px;}

#clock .time .d6{           width:5px;height:14px;top:32px;left:0;}
#clock .time .d6:before{    border-width:0 5px 5px 0;border-bottom-color:inherit;top:-5px;}
#clock .time .d6:after{ border-width:0 0 5px 5px;border-left-color:inherit;bottom:-5px;}

#clock .time .d7{           width:5px;height:14px;top:32px;right:0;}
#clock .time .d7:before{    border-width:0 0 5px 5px;border-bottom-color:inherit;top:-5px;}
#clock .time .d7:after{ border-width:5px 0 0 5px;border-top-color:inherit;bottom:-5px;}

#clock .time div.one .d5,
#clock .time div.one .d7{
    opacity:1;
}

/* 2 */

#clock .time div.two .d1,
#clock .time div.two .d5,
#clock .time div.two .d2,
#clock .time div.two .d6,
#clock .time div.two .d3{
    opacity:1;
}

/* 3 */

#clock .time div.three .d1,
#clock .time div.three .d5,
#clock .time div.three .d2,
#clock .time div.three .d7,
#clock .time div.three .d3{
    opacity:1;
}

/* 4 */

#clock .time div.four .d5,
#clock .time div.four .d2,
#clock .time div.four .d4,
#clock .time div.four .d7{
    opacity:1;
}

/* 5 */

#clock .time div.five .d1,
#clock .time div.five .d2,
#clock .time div.five .d4,
#clock .time div.five .d3,
#clock .time div.five .d7{
    opacity:1;
}

/* 6 */

#clock .time div.six .d1,
#clock .time div.six .d2,
#clock .time div.six .d4,
#clock .time div.six .d3,
#clock .time div.six .d6,
#clock .time div.six .d7{
    opacity:1;
}


/* 7 */

#clock .time div.seven .d1,
#clock .time div.seven .d5,
#clock .time div.seven .d7{
    opacity:1;
}

/* 8 */

#clock .time div.eight .d1,
#clock .time div.eight .d2,
#clock .time div.eight .d3,
#clock .time div.eight .d4,
#clock .time div.eight .d5,
#clock .time div.eight .d6,
#clock .time div.eight .d7{
    opacity:1;
}

/* 9 */

#clock .time div.nine .d1,
#clock .time div.nine .d2,
#clock .time div.nine .d3,
#clock .time div.nine .d4,
#clock .time div.nine .d5,
#clock .time div.nine .d7{
    opacity:1;
}

/* 0 */

#clock .time div.zero .d1,
#clock .time div.zero .d3,
#clock .time div.zero .d4,
#clock .time div.zero .d5,
#clock .time div.zero .d6,
#clock .time div.zero .d7{
    opacity:1;
}
#clock .time div.dig{
    width:5px;
}

#clock .time div.dig:before,
#clock .time div.dig:after{
    width:5px;
    height:5px;
    content:'';
    position:absolute;
    left:0;
    top:14px;
}

#clock .time div.dig:after{
    top:34px;
}
#wh {
position:fixed; top:10px; left:10px;
}
```
** 3. 在layout/_partials/head.swig中添加下面这段代码 **
```javascript
 {% if theme.time_clock %}
<link rel="stylesheet" type="text/css" href="/css/clock03.css"/>    

        <script type="text/javascript" src="/js/src/jquery-1.11.3.min.js">
        </script>
        <script type="text/javascript" src="/js/src/moment.min.js"></script>
        <script type="text/javascript" src="/js/src/clock.js"></script>
        <div id="wh">
            <div id="clock" class="light">
                <div class="week"></div>
                <div class="alarm"></div>
                <div class="time"></div>
                <div class="ampm"></div>
            </div>
        </div>
{% endif %}
```

到此，时间插件完成，效果如网站的左上角所示。个人的next主题上传至：
https://github.com/alertcode/alertcode-blog-themes-next.git
#### 发现的问题
上述代码会导致hexo-next主题的网站图标找不到，后期修复解决。
待改善的功能： hexo 缺少文章置顶功能或者根据文章的更新时间进行排序，待之后参考资料解决。

### 2018-03-10 hexo-next 添加鼠标跟随特效
资源下载地址：
https://github.com/alertcode/alertcode-blog-themes-next/files/1799167/jquery-mouse-star-animation.zip

1. js位置 ：资源中的js文件放入\next\source\js\src下
2. 图片位置: 将资源中的images\mouse 文件夹放进\next\source\images 下
3. 主题配置文件添加属性mouse_star: true
4. themes\next\layout\_layout.swig文件末尾添加代码
```javascript
{% if theme.mouse_star %}
 <script type="text/javascript" src="/js/src/mymouse.js" id="mymouse"></script>
{% endif %}
```

5. 更改mymouse.js中的这段代码,如下所示,图片的路径
```javascript
for (i = 0; i < 1; i++) {
                    var j = "/images/mouse/subball" + (i + 1) + ".png";
                    giffy_bp_0013.ballImageResource[i] = new Image;
                    giffy_bp_0013.ballImageResource[i].src = j
                }
                for (i = 0; i < 8; i++) j = "/images/mouse/subball" + (i + 1) + ".png", giffy_bp_0013.subBallImageResource[i] =
                    new Image, giffy_bp_0013.subBallImageResource[i].src = j;
```

### 2018-06-30 hexo-next 按照文章更新时间排序
主配置文件修改如下：
```
index_generator:
   path: ''
   per_page: 10
   order_by: -updated
```
