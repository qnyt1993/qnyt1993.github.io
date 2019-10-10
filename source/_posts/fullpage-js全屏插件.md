---
title: fullpage.js全屏插件
categories: [前端,插件]
tags: [javascript] 
date: 2019-10-10 20:18:51
---

# fullpage 全屏插件

>全屏滚动效果，原生js也很好实现，主要是用 mousewheel  鼠标滚轮滚动事件， 来判断上滚动还是下滚动，之后设置每次滚动的高度为屏幕的高度即可。但是，虽然效果简单，但是兼容性很差，要做很多兼容处理及比较麻烦啦！

`fullPage.js` 是一个基于 `jQuery` 的插件，它能够帮你很方便、很轻松的制作出全屏网站。

`github` 官网     `https://github.com/alvarotrigo/fullPage.js`  

中文演示地址   `http://www.dowebok.com/demo/2014/77/` 

## 1. 主要功能有：
  
      支持鼠标滚动
      
      支持前进后退和键盘控制
    
      多个回调函数
      
      支持手机、平板触摸事件
      
      支持 CSS3 动画
      
      支持窗口缩放
      
      窗口缩放时自动调整
      
      可设置滚动宽度、背景颜色、滚动速度、循环选项、回调、文本对齐方式等等
      
## 2. 使用步骤

### 2.1 引入资源文件

        <link rel="stylesheet" href="css/jquery.fullPage.css">
        <!--需要引入jquery-->
        <script src="js/jquery.min.js"></script>
        <!-- jquery.easings.min.js 是必须的，用于 easing 参数，也可以使用完整的 jQuery UI 代替 -->
        <script src="js/jquery.easing.1.3.js"></script>
        <script src="js/jquery.fullPage.min.js"></script>
        <!--引入初始化js-->
        <script src="js/myPage.js"></script>
    
### 2.2 编写HTML 结构

     <div id="fullpage">
         <div class="section">第一屏</div>
         <div class="section">第二屏</div>
         <div class="section">
             <div class="slide">第三屏的第一屏</div>
             <div class="slide">第三屏的第二屏</div>
             <div class="slide">第三屏的第三屏</div>
             <div class="slide">第三屏的第四屏</div>
         </div>
         <div class="section">第四屏</div>
     </div> 
  
  结构大致如下图所示
  
  ![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/24/full.png)
   
### 2.3 初始化
myPage.js

      $(document).ready(function() {
          $('#fullpage').fullpage({
              //options here
              autoScrolling:true,
              scrollHorizontally: true
          });
      
      }); 
      
## 3. 演示效果

<iframe src="//player.bilibili.com/player.html?aid=70789760&cid=122649501&page=1" scrolling="no" 
 width=100% 
  height=600 border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

## 4. 参数补充

### 4.1 详细参数

所有选项设置的更复杂的初始化如下所示：

     var myFullpage = new fullpage('#fullpage', {
        //导航
        menu: '#menu',
        lockAnchors: false,
        anchors:['firstPage', 'secondPage'],
        navigation: false,
        navigationPosition: 'right',
        navigationTooltips: ['firstSlide', 'secondSlide'],
        showActiveTooltip: false,
        slidesNavigation: false,
        slidesNavPosition: 'bottom',
     
        //滚动
        css3: true,
        scrollingSpeed: 700,
        autoScrolling: true,
        fitToSection: true,
        fitToSectionDelay: 1000,
        scrollBar: false,
        easing: 'easeInOutCubic',
        easingcss3: 'ease',
        loopBottom: false,
        loopTop: false,
        loopHorizontal: true,
        continuousVertical: false,
        continuousHorizontal: false,
        scrollHorizontally: false,
        interlockedSlides: false,
        dragAndMove: false,
        offsetSections: false,
        resetSliders: false,
        fadingEffect: false,
        normalScrollElements: '#element1, .element2',
        scrollOverflow: false,
        scrollOverflowReset: false,
        scrollOverflowOptions: null,
        touchSensitivity: 15,
        bigSectionsDestination: null,
     
        //可访问
        keyboardScrolling: true,
        animateAnchor: true,
        recordHistory: true,
     
        //设计
        controlArrows: true,
        verticalCentered: true,
        sectionsColor : ['#ccc', '#fff'],
        paddingTop: '3em',
        paddingBottom: '10px',
        fixedElements: '#header, .footer',
        responsiveWidth: 0,
        responsiveHeight: 0,
        responsiveSlides: false,
        parallax: false,
        parallaxOptions: {type: 'reveal', percentage: 62, property: 'translate'},
        cards: false,
        cardsOptions: {perspective: 100, fadeContent: true, fadeBackground: true},
     
     
        //自定义选择器
        sectionSelector: '.section',
        slideSelector: '.slide',
     
        lazyLoading: true,
     
        //事件
        onLeave: function(origin, destination, direction){},
        afterLoad: function(origin, destination, direction){},
        afterRender: function(){},
        afterResize: function(width, height){},
        afterReBuild: function(){},
        afterResponsive: function(isResponsive){},
        afterSlideLoad: function(section, origin, destination, direction){},
        onSlideLeave: function(section, origin, destination, direction){}
     });     
     
### 4.2 详细参数说明

     
| 选项                                | 类型   | 默认值         | 说明                                   |
| --------------------------------- | ---- | ----------- | ------------------------------------ |
|                                   |      |             |                                      |
| verticalCentered                  | 字符串  | true        | 内容是否垂直居中                             |
| resize                            | 布尔值  | false       | 字体是否随着窗口缩放而缩放                        |
| sectionColor                      | 函数   | 无           | 设置背景颜色                               |
| anchors                           | 数组   | 无           | 定义锚链接                                |
| scrollingSpeed                    | 整数   | 700         | 滚动速度，单位为毫秒                           |
| easing                            | 字符串  | easeInQuart | 滚动动画方式                               |
| menu                              | 布尔值  | false       | 绑定菜单，设定的相关属性与 anchors 的值对应后，菜单可以控制滚动 |
| navigation                        | 布尔值  | false       | 是否显示项目导航                             |
| navigationPosition                | 字符串  | right       | 项目导航的位置，可选 left 或 right              |
|                                   |      |             |                                      |
| navigationTooltips                | 数组   | 空           | 项目导航的 tip                            |
| slidesNavigation                  | 布尔值  | false       | 是否显示左右滑块的项目导航                        |
| slidesNavPosition                 | 字符串  | bottom      | 左右滑块的项目导航的位置，可选 top 或 bottom         |
| controlArrowColor                 | 字符串  | #fff        | 左右滑块的箭头的背景颜色                         |
| loopBottom                        | 布尔值  | false       | 滚动到最底部后是否滚回顶部                        |
| loopTop                           | 布尔值  | false       | 滚动到最顶部后是否滚底部                         |
| loopHorizontal                    | 布尔值  | true        | 左右滑块是否循环滑动                           |
| autoScrolling                     | 布尔值  | true        | 是否使用插件的滚动方式，如果选择 false，则会出现浏览器自带的滚动条 |
| scrollOverflow                    | 布尔值  | false       | 内容超过满屏后是否显示滚动条                       |
| css3                              | 布尔值  | false       | 是否使用 CSS3 transforms 滚动              |
| paddingTop                        | 字符串  | 0           | 与顶部的距离                               |
| paddingBottom                     | 字符串  | 0           | 与底部距离                                |
| fixedElements                     | 字符串  | 无           |                                      |
| normalScrollElements              |      | 无           |                                      |
| keyboardScrolling                 | 布尔值  | true        | 是否使用键盘方向键导航                          |
| touchSensitivity                  | 整数   | 5           |                                      |
| continuousVertical                | 布尔值  | false       | 是否循环滚动，与 loopTop 及 loopBottom 不兼容    |
| animateAnchor                     | 布尔值  | true        |                                      |
| normalScrollElementTouchThreshold | 整数   | 5           |                                      |     


### 4.3 fullPage.js 方法
         
注意方法的使用时需要添加：
         `$.fn.fullpage`   比如
         
         
         $.fn.fullpage.moveTo(1);
    
         
| 名称                     | 说明                      |
| ---------------------- | ----------------------- |
| moveSectionUp()        | 向上滚动                    |
| moveSectionDown()      | 向下滚动                    |
| moveTo(section, slide) | 滚动到                     |
| moveSlideRight()       | slide 向右滚动              |
| moveSlideLeft()        | slide 向左滚动              |
| setAutoScrolling()     | 设置页面滚动方式，设置为 true 时自动滚动 |
| setAllowScrolling()    | 添加或删除鼠标滚轮/触控板控制         |
| setKeyboardScrolling() | 添加或删除键盘方向键控制            |
| setScrollingSpeed()    | 定义以毫秒为单位的滚动速度           |
         
         
         
### 4.4 回调函数
         
| 名称             | 说明                                       |
| -------------- | ---------------------------------------- |
| afterLoad      | 滚动到某一屏后的回调函数，接收 anchorLink 和 index 两个参数，anchorLink 是锚链接的名称，index 是序号，从1开始计算 |
| onLeave        | 滚动前的回调函数，接收 index、nextIndex 和 direction 3个参数：index 是离开的“页面”的序号，从1开始计算；nextIndex 是滚动到的“页面”的序号，从1开始计算；direction 判断往上滚动还是往下滚动，值是 up 或 down。 |
 | afterRender    | 页面结构生成后的回调函数，或者说页面初始化完成后的回调函数            |
| afterSlideLoad | 滚动到某一水平滑块后的回调函数，与 afterLoad 类似，接收 anchorLink、index、slideIndex、direction 4个参数 |
| onSlideLeave   | 某一水平滑块滚动前的回调函数，与 onLeave 类似，接收 anchorLink、index、slideIndex、direction 4个参数 |
