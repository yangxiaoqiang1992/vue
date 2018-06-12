# vue
##### vue+echarts-map 实现地图选择与列表联动
### 核心点：watch监听选中数据  echarts使用 dispatchAction 触发echarts交互事件，mapUnSelect 取消上次选择 mapSelect 选择新的数据<br>
watch:{<br>
    selected:function(val,oldVal){<br>
      this.$nextTick(function(){<br>
          echarts.dispatchAction({<br>
            type:'mapUnSelect',<br>
            name:oldVal<br>
          });<br>
          echarts.dispatchAction({<br>
            type:'mapSelect',<br>
            name:val<br>
          });<br>
      }) <br>
    }<br>
  }<br>
  
###如果需要修改地图坐标，修改module下的map/province/xx.js是没有效果的，代码编译后会覆盖所做的修改，可换成json方式引入地图，在json中修改地图坐标，异步请求静态json文件即可
  $.get('static/js/beijing.json', function (beijing){
                echarts.registerMap('北京', beijing) //注册
                // this.option.series[0].map="北京";
                _this.mapCharts.setOption(_this.option);    
 }) 
