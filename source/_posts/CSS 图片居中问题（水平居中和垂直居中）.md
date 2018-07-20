---
title: CSS 图片居中问题（水平居中和垂直居中）
date: 2018-03-13 11:29:17
categories:
- CSS
- CSS问题
tags:
- CSS问题
---
#### 1、水平居中 ####
（1）利用margin: 0 auto实现图片水平居中
利用margin: 0 auto实现图片居中就是在图片上加上css样式margin: 0 auto 如下：
```
<div style="text-align: center; width: 500px; border: green solid 1px;">
    <img alt="" src="https://www.baidu.com/img/baidu_jgylogo3.gif" 
         style="margin: 0 auto;" />
</div>
```
<!--more-->

（2）利用文本的水平居中属性text-align: center
```
<div style="text-align: center; width: 500px; border: green solid 1px;">
    <img alt="" src="https://www.baidu.com/img/baidu_jgylogo3.gif" 
         style="display: inline-block;" />
</div>
```

#### 2、垂直居中 ####
（1）利用高==行高实现图片垂直居中
这种方法是要知道高度才可以使用，代码如下：
```
<div style="text-align: center; width: 500px;height:200px; 
            line-height:200px; border: green solid 1px;">
    <img alt="" src="https://www.baidu.com/img/baidu_jgylogo3.gif" 
         style="display: inline-block; vertical-align: middle;" />
</div>
```

（2）利用table实现图片垂直居中
利用table的方法是利用了table的垂直居中属性，代码如下：
```
<div style="text-align: center; width: 500px;height:200px; 
            display: table;border: green solid 1px;">
    <span style="display: table-cell; vertical-align: middle; ">
        <img alt="" src="https://www.baidu.com/img/baidu_jgylogo3.gif" 
             style="display: inline-block;" />
    </span>
</div>
```
这里使用display: table;和display: table-cell;来模拟table，这种方法并不兼容IE6/IE7，IE67不支持display: table，如果你不需要支持IE67那就可以用
**（缺点：当你设置了display: table;可能会改变你的原有布局）**

（3）利用绝对定位实现图片垂直居中
如果已知图片的宽度和高度可以这样，代码如下：
```
<div style="width: 500px;height:200px; position: relative; border: green solid 1px;">
    <img alt="" src="https://www.baidu.com/img/baidu_jgylogo3.gif" 
         style="width: 120px; height: 40px;position: absolute; left:50%; top: 50%; 
                margin-left: -60px;margin-top: -20px;" />
</div>
```

（4）移动端可以利用flex布局实现css图片垂直居中
移动端一般浏览器版本都比较高，所以可以大胆的使用flex布局，（flex布局参考css3的flex布局用法）演示代码如下：
```
/*css代码：*/
<style type="text/css">
.ui-flex {
    display: -webkit-box !important;
    display: -webkit-flex !important;
    display: -ms-flexbox !important;
    display: flex !important;
    -webkit-flex-wrap: wrap;
    -ms-flex-wrap: wrap;
    flex-wrap: wrap
}
.ui-flex, .ui-flex *, .ui-flex :after, .ui-flex :before {
    box-sizing: border-box
}
.ui-flex.justify-center {
    -webkit-box-pack: center;
    -webkit-justify-content: center;
    -ms-flex-pack: center;
    justify-content: center
}
.ui-flex.center {
    -webkit-box-pack: center;
    -webkit-justify-content: center;
    -ms-flex-pack: center;
    justify-content: center;
    -webkit-box-align: center;
    -webkit-align-items: center;
    -ms-flex-align: center;
    align-items: center
}
</style>

/*html代码：*/
<div class="ui-flex justify-center center" 
     style="border: green solid 1px; width: 500px; height: 200px;">
    <div class="cell">
        <img alt="" src="https://www.baidu.com/img/baidu_jgylogo3.gif" style="" />
    </div>
</div>
```