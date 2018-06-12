# vue
## vue+echarts-map 实现地图选择与列表联动
核心点：watch监听选中数据  echarts使用 dispatchAction 触发echarts交互事件，mapUnSelect 取消上次选择 mapSelect 选择新的数据<br>
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
