---
title: jQuery Easing.js 插件
categories: [前端,插件]
tags: [javascript] 
date: 2019-10-14 09:47:58
---

>介绍：easing是jquery的一个插件，使用它可以创建更加绚丽的动画效果。

>环境：因为easing是jQuery的插件，所以必须是在引入jquery之后再引入它

如果只想要简单的写法就用 

    $(".car").animate({"left": "150%"},  4000, "easeInElastic", function() {});


    easing:格式为json,{duration:持续时间,easing:过渡效果,complete:成功后的回调函数}

示例

    $(element).animate({ 
        height:500, 
        width:600 
        },{ 
        easing: 'easeInOutQuad', 
        duration: 500, 
        complete: function(){} 
    }); 




1. linear
2. swing
3. easeInQuad
4. easeOutQuad
5. easeInOutQuad
6. easeInCubic
7. easeOutCubic
8. easeInOutCubic
9. easeInQuart
10. easeOutQuart
11. easeInOutQuart
12. easeInQuint
13. easeOutQuint
14. easeInOutQuint
15. easeInExpo
16. easeOutExpo
17. easeInOutExpo
18. easeInSine
19. easeOutSine
20. easeInOutSine
21. easeInCirc
22. easeOutCirc
23. easeInOutCirc
24. easeInElastic
25. easeOutElastic
26. easeInOutElastic
27. easeInBack
28. easeOutBack
29. easeInOutBack
30. easeInBounce
31. easeOutBounce
32. easeInOutBounce

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/24/esse.png)
