---
title: 关于css的hover灵活运用。
date: 2021-07-16 
tags:
 - css
 - hover
---
## 遇到的问题
这个是我暑假无聊模仿做的英雄联盟网站的一部分,我遇到的问题就是图中画线部分如何实现鼠标在整个红色部分悬停的时候同时让背景变色和箭头的动态效果.因为只是给图片设置 hover 的话那只有悬停在图片上方才会有效果,而且整个背景也不会变色.那么如何解决呢？
![Image text](http://101.42.150.127:8081/blog/hover.png)
## 我的解决办法
话不多说先上代码
```html
<div class="a">阅读更多<a href=""><b class="b">最新咨询 -</b><div class="e"><img class="c" src="shujia/jiantou.png" alt=""></div></a></div>
```
我在图片外面添加了一个div给div设置大小和宽度等于整个划红线背景.样式如下：
```css
.e{
       position: absolute;
       top: 285px;
       width: 496px;
       height: 40px;
   }
```
注意(上面用到了绝对布局,如果想要适配不同设备的话最好在外面加一个父级div设置为相对布局,这样就能适配啦.)
然后写hover代码.
```css
.e:hover{
        animation:mymove 1s infinite;
        -moz-animation:mymove 1s infinite; /* Firefox */
        -webkit-animation:mymove 1s infinite; /* Safari and Chrome */
        -o-animation:mymove 1s infinite; /* Opera */
    }
@keyframes mymove{
        0%{
            margin-left: 2px
        }
        25%{
            margin-left: 3px;
        }
        50%{
            margin-left: 4px;
        }
        75%{
            margin-left: 5px;
        }
        100%{
            margin-left: 6px;
        }
    }
    @-moz-keyframes mymove{
        0%{
            margin-left: 2px
        }
        25%{
            margin-left: 3px;
        }
        50%{
            margin-left: 4px;
        }
        75%{
            margin-left: 5px;
        }
        100%{
            margin-left: 6px;
        }
    }
    @-webkit-keyframes mymove{
        0%{
            margin-left: 2px
        }
        25%{
            margin-left: 3px;
        }
        50%{
            margin-left: 4px;
        }
        75%{
            margin-left: 5px;
        }
        100%{
            margin-left: 6px;
        }
    }@-o-keyframes mymove{
        0%{
            margin-left: 2px
        }
        25%{
            margin-left: 3px;
        }
        50%{
            margin-left: 4px;
        }
        75%{
            margin-left: 5px;
        }
        100%{
            margin-left: 6px;
        }
    }
```
背景样式变色如下：
```css
.a:hover{
       background-color: rgb(197, 197, 202);
   }
```
问题解决,自己的一点小经验,希望大家不吝赐教.


## 参考文献

https://www.runoob.com/cssref/sel-hover.html
