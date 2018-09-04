## vue+niceScroll.js插件，页面多个滚动条,位置错乱问题解决

#### ie9下，使用niceScroll.js滚动条插件，当外层和内部都有滚动条存在时，外部滚动条滚动时导致内部滚动条位置错乱
####  解决思路，外部滚动条监听scroll事件，并在scroll里调用hideRail方法，隐藏内部滚动条，内部滚动条scroll监听，再显示滚动条
```
$('outerScroll').on('scroll',function(){
   $('#innerScroll').getNiceScroll(0).hideRail()
})
$('#innerScroll').on('scroll',function(){
   $('#outerScroll').getNiceScroll(0).showRail()
})
