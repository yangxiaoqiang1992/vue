# vue
vue和jquery混合开发

问题描述：一般情况，尽量避免vue+jquery混合开发，但某些情况下需要使用现有的jquery插件或自己开发插件，比如自己开发的一款日期插件，如下图所示，在jquery插件内部，操作dom,并给input赋值，虽然使用v-model双向绑定，但vue不能检测到数据的变化<br>
![blockchain](https://github.com/yangxiaoqiang1992/vue/blob/master/%E6%97%A5%E5%8E%86%E5%9B%BE%E7%89%87.jpg "日历图片")

解决方法：jquery插件内部，触发input的change事件，在change事件对input的值进行修改，这样就能获取到变化之后的数据

~~~
html部分

<div id="datePicker"> 
    <input type="text" class="date-picker" v-model="tjqMc"> 
    <input type="hidden" id="tjqId" v-model="tjq"> 
</div> 
~~~
~~~
jquery插件代码  该插件支持连续选择月份  根据是否是当前年，获取到具体的时间范围，闰年和平年均可正确获取
;(function($){
	var Calender=function(el,opts){
		this.$el = el
		this.$input = this.$el.children('.date-picker')
		this.$tjqId = this.$el.children('#tjqId');
		this.$calender=null
		this.$curDate={
			year:'',
			month:'',
			day:''
		}
		this.$result=null
		this.$yearList=null
		this.$defaults={
			theme:'blue',	
			yearRange:4
		}
		this.settings=$.extend({},this.$defaults,opts)
	}
	Calender.prototype={
	  init:function(){
		  var _this =this
		  var inputH= this.$input.height()+15
		  var date=new Date()
		  this.$curDate.year = date.getFullYear()
		  this.$curDate.month = date.getMonth() + 1
		  this.$curDate.day =date.getDate()
		  this.$el.css('position','relative').addClass('datepicker-'+this.settings.theme)
		  this.$el.append('<div class="calender theme-'+this.settings.theme+'"></div>')
		  this.$input.addClass('input-'+this.settings.theme)
		  this.$calender = this.$el.children('.calender')
		  this.$calender.css('top',inputH)
		  this.formatterResult()
		  this.renderView()
		  this.renderFooter()
		  this.updateViewByResult()
		  this.$el.on('click',function(event){
			  event.stopPropagation()
			  _this.$calender.show()
		  })
		  $(document).on('click',function(event){
			  event.stopPropagation()
			   _this.$calender.hide()
		  })
		   this.$calender.find('a.cancel').on('click',function(event){
			   event.stopPropagation()
			  _this.cancelEvent()
		  })
		  this.$calender.find('a.ok').on('click',function(event){
			  event.stopPropagation()
			  _this.okEvent()
		  })
		  tapCount=0
		  _this.$calender.find('.row').on('click','span.list span',function(event){
			 event.stopPropagation()
			if($(this).attr('disabled')){
				return 
			}else{
				var index = _this.$calender.find('.row').index($(this).parents('.row'))
				if(index == 0){
					_this.$calender.find('.row span.list span').removeClass('on hover lefthand righthand')
					$(this).addClass('on')
				}else if(index == 3){
					tapCount++
					//判断点击次数，点击两次后清空重新确定月份范围，清除绑定的鼠标悬浮事件
					var ylj = $(this).parent('span.list').find('span').not('[disabled]')
					_this.$calender.find('.row').eq(index).siblings('.row:gt(0)').find('span.list span').removeClass('on')
					
					val = $(this).data('value')
					if(tapCount%2 == 0){
						ylj.off('mouseover')
						$(this).parent('span.list').off('mouseout')
					}else{
						$(this).addClass('on').siblings().removeClass('on')
						ylj.removeClass('hover righthand lefthand')
						ylj.on('mouseover',function(){
							if(tapCount == 0){
								return
							}
							var v = $(this).data('value')
							ylj.removeClass('hover righthand lefthand')
							if(v >= val){
								ylj.filter(function(index){
									return index > (val-1) && index <= (v-1)
								}).addClass('hover')
								$(this).addClass('righthand')
							}else{
								ylj.filter(function(index){
									return index >= (v-1) && index < (val-1)
								}).addClass('hover')
								$(this).addClass('lefthand')
							}
				    	})
				    	$(this).parent('span.list').on('mouseout',function(){
				    		ylj.removeClass('hover righthand lefthand')
				    	})
					}
				}else{
					_this.$calender.find('.row:gt(0) span.list span').removeClass('on hover lefthand righthand')
					$(this).addClass('on')
				}
				_this.formatterResult(index,$(this).data('value'))
				_this.disableItemByYear()
			}
		})  
		  return this.$el
	  },
	  /**
	   * @param label:String 名称
	   * @param list:Array 生成的数组
	   */
	  renderItem:function(label,list,key){
		 var _this = this 
		 var quar =_this.getQuarterByMonth(_this.$curDate.month)
		 var temp="<div class='row' id="+key+"><span class='label'>"+label+"：</span><span class='list'>"
		 for(var i=0;i<list.length;i++){
		     // 不展示17年之前的年份
		     if(key == 'year' && list[i].value < 2017){
		         continue;
		     }
			 temp+="<span class='item' data-value="+list[i].value+">"+(list[i].name)+"</span>"
		 } 
		 temp+='</span></div>'
		 this.$calender.append(temp)
	  },
	  renderFooter:function(){
		  this.$calender.append("<div class='calender-footer'><a id='calOk' class='ok'>确定</a><a class='cancel'>取消</a></div>")
	  },
	  renderView:function(){
		  this.renderItem.call(this,'年份',this.settings.data.year,'year')
		 //生成年度
		  this.renderItem.call(this,'年度',this.settings.data.nd,'nd')
		 //生成季度
		  this.renderItem.call(this,'季度',this.settings.data.quar,'quar')
		 //月累计
		  this.renderItem.call(this,'月累计',this.settings.data.month,'monthRange')
		 //月份 
		  this.renderItem.call(this,'月份',this.settings.data.month,'month')
	  },
	  /**
	   * 根据选择结果更新视图
	   */
	  updateViewByResult:function(){
		  var _this =this
		  this.$calender.find('.row span.list span').removeClass('on hover lefthand righthand')
		  this.disableItemByYear()
		  this.$calender.find('.row').each(function(index,ele){
			var id = $(ele).attr('id')
			var val= _this.$result[id]
	        $('#'+id).find('span.list span').each(function(i,e){
	        	if(id !='monthRange' || (id =='monthRange' && val!=null && val.length == 1)){
	        		if($(this).data('value') == val){
		        		$(this).addClass('on')
		        	}
	        	}else if(val!=null){
	        		 var max=0
					 var min=12
					 $.each(_this.$result.monthRange,function(i,n){
						 max=Math.max(max,n)
						 min=Math.min(min,n)
					 })
					 if($(this).data('value') == min){
						 $(this).addClass('on')
					 }else if($(this).data('value') == max){
						 $(this).addClass('hover righthand')
					 }else{
						 $(this).filter(function(){
							 return $(this).data('value') > min && $(this).data('value') < max
						 }).addClass('hover')
					 }
	        	}else{
	        		return
	        	}
	        })
		  })
	  },
	  /**
	   * @param type 1 年度 2 季度  3 月累计 4 月份 
	   * 格式化结果数据，年度 季度转化为数字，得到具体的日期
	   * 年度:如为当前年，则  全年：当前年1月1日-当前日期  上半年 1月1日-当前日期  下半年：当前日期不超过下半年，如五月，下半年不可选  
	   *     不为当前年：全年 1/1 - 12/31  上半年1/1-6/30 下半年 7/1-12/31
	   * 季度：如为当前年，判断当前月所在季度，超过当前季度不可选
	   * 月份、月累计：当前年：不能超过当前月，超过不可选  
	   */
	  formatterResult:function(type,val){
		  var _this = this
		  switch(type){
		     case 0:getYearResult();break;
		     case 1:getNdResult();break;
		     case 2:getJdResult();break;
		     case 3:getYljResult();break;
		     case 4:getYfResult();break;
		     default:getDefaultResult();break;
		  }
		  function getYearResult(){
			  _this.emptyResult()
			  _this.$result.year =val
		  }
		  function getNdResult(){
			  _this.emptyResult()
			  _this.$result.nd =val
		  }
		  function getJdResult(){
			  _this.emptyResult()
			  _this.$result.quar = val
		  }
		  function getYljResult(){
			  $.extend(_this.$result,{nd:null,quar:null,month:null})
			  var remainder = tapCount%2
			  if(remainder!=0){
				  _this.$result.monthRange =new Array()
			  }
			  _this.$result.monthRange.push(val)
		  }
		  function getYfResult(){
			  _this.emptyResult()
			  _this.$result.month =val
		  }
		  function getDefaultResult(){
			  //默认当前年 全年
			  if(_this.settings.tjqId){
				 _this.getResultByTjqid(_this.settings.tjqId);
			  }else{
				 _this.$result={}
				 _this.$result.year = _this.$result.year? _this.$result.year:_this.$curDate.year
				 _this.$result.nd = _this.$result.nd?_this.$result.nd:0
				 _this.$result.quar = null
				 _this.$result.month =null
				 _this.$result.monthRange = null
				 _this.$result.dateRange=_this.$result.year+'/1/1-'+_this.$result.year+'/'+_this.$curDate.month+'/'+_this.$curDate.day
			  }
		  }
		  _this.getDateRangeByYear(_this.$result.year)
	  },
	  getResultByTjqid:function(tjqid){
	      this.$result={};
	      this.$result.year = tjqid.substring(0, 4);
	      this.$result.nd = null;
	      this.$result.quar = null;
	      this.$result.month = null;
	      this.$result.monthRange = null;
	      this.$result.dateRange = null;//日期区间暂时没用，先不设置了
	      
	      tjqid = parseInt(tjqid.substring(4, 6));
	      if(tjqid <= 12){
	          this.$result.month = tjqid;
	      } else if (tjqid <= 16) {
	          this.$result.quar = tjqid - 13;
	      } else if (tjqid <= 28) {
	          this.$result.monthRange = new Array();
	          this.$result.monthRange.push(1);
	          this.$result.monthRange.push(tjqid - 16);
	      } else {
	          switch(tjqid){
	              case 31:
	                  this.$result.nd = 0;
	                  break;
	              case 29:
	                  this.$result.nd = 1;
                      break;
	              case 30:
	                  this.$result.nd = 2;
                      break;
	              default:
	                  this.$result.nd = 0;
	                  break;
	          }
	      }
	  },
	  /**
	   * 根据月份获取当前所处季度
	   */
	  getQuarterByMonth:function(month){
		  return   Math.floor(month % 3 == 0 ? ( month / 3 ) : ( month / 3 + 1 )) 
	  },
	  /**
	   * 格式化日期选择具体对应时间
	   */
	  getDateRangeByYear:function(year){
		  var _this =this
		  var dateRange=null
		  if(_this.$curDate.year == year){
			  if(_this.$result.nd != null){
				  if(_this.$result.nd == 0 || (_this.$result.nd == 1 && _this.$curDate.month < 7)){
					  dateRange =  '1/1-'+_this.$curDate.month+'/'+_this.$curDate.day
				  }else if(_this.$result.nd == 1 && (_this.$curDate.month >= 7)){
					 dateRange =  '1/1-6/30'
				  }else if(_this.$result.nd == 2 && (_this.$curDate.month >= 7)){
					  dateRange='7/1-'+_this.$curDate.month+'/'+_this.$curDate.day
				  }else{
					  dateRange=null
				  }
			  }
			  if(_this.$result.quar != null){
				  if(_this.$result.quar == 1){
					  if(_this.$curDate.month <= 3){
						  dateRange =  '1/1-'+_this.$curDate.month+'/'+_this.$curDate.day
					  }else{
						  dateRange =  '1/1-3/31'
					  }
				  }else if(_this.$result.quar == 2){
					  if(_this.$curDate.month <= 6){
						  dateRange =  '4/1-'+_this.$curDate.month+'/'+_this.$curDate.day
					  }else{
						  dateRange =  '4/1-6/30'
					  }
				  }else if(_this.$result.quar == 3){
					  if(_this.$curDate.month <= 9){
						  dateRange =  '7/1-'+_this.$curDate.month+'/'+_this.$curDate.day
					  }else{
						  dateRange =  '7/1-9/30'
					  } 
				  }else {
						  dateRange =  '10/1-'+_this.$curDate.month+'/'+_this.$curDate.day
				  }
			  }
			  if(_this.$result.month != null){
				  var date =new Date(_this.$result.year,_this.$result.month,0).getDate()
				  if(_this.$result.month == _this.$curDate.month && _this.$curDate.day < date){
					  dateRange = _this.$result.month+'/1-'+_this.$result.month+'/'+_this.$curDate.day
				  }else{
					  dateRange = _this.$result.month+'/1-'+_this.$result.month+'/'+date
				  }
			  }
			  if(_this.$result.monthRange!=null){
					 //可能出现的情况 ：单个值  两个值：两个值的情况下比较大小
					 var len = _this.$result.monthRange.length
					 if(len==1){
						 if(_this.$result.monthRange[0] != _this.$curDate.month){
							 var date =new Date(_this.$result.year,_this.$result.monthRange[0],0).getDate()
							 dateRange = _this.$result.monthRange[0]+'/1/-'+_this.$result.monthRange[0]+'/'+date
						 }else{
							 dateRange = _this.$result.monthRange[0]+'/1/-'+_this.$result.monthRange[0]+'/'+_this.$curDate.day
						 }
					 }else{
						 var max=0
						 var min=12
						 $.each(_this.$result.monthRange,function(i,n){
							 max=Math.max(max,n)
							 min=Math.min(min,n)
						 })
						 if(max != _this.$curDate.month){
							 var date =new Date(_this.$result.year,max,0).getDate()
							 dateRange = min+'/1/-'+max+'/'+ date
						 }else{
							 dateRange = min+'/1/-'+max+'/'+_this.$curDate.day
						 }
					 }
				 }
		  }else{
			 _this.$calender.find('.row span.list span').removeAttr('disabled')
			 if(_this.$result.nd == 0){
				 dateRange =  '1/1-12/31'
			 } 
			 if(_this.$result.nd == 1){
				 dateRange =  '1/1-6/30'
			 } 
			 if(_this.$result.nd == 2){
				 dateRange =  '7/1-12/31'
			 } 
			 if(_this.$result.quar == 1){
				 dateRange='1/1-3/31'
			 }
			 if(_this.$result.quar == 2){
				 dateRange='4/1-6/30'
			 }
			 if(_this.$result.quar == 3){
				 dateRange='7/1-9/30'
			 }
			 if(_this.$result.quar == 4){
				 dateRange='10/1-12/31'
			 }
			 if(_this.$result.month!=null){
				 var date =new Date(_this.$result.year,_this.$result.month,0).getDate()
				 dateRange=_this.$result.month+'/1-'+_this.$result.month+'/'+date
			 }
			 if(_this.$result.monthRange!=null){
				 //可能出现的情况 ：单个值  两个值：两个值的情况下比较大小
				 var len = _this.$result.monthRange.length
				 if(len==1){
					 var date =new Date(_this.$result.year,_this.$result.monthRange[0],0).getDate()
					 dateRange = _this.$result.monthRange[0]+'/1/-'+_this.$result.monthRange[0]+'/'+date
				 }else{
					 var max=0
					 var min=12
					 $.each(_this.$result.monthRange,function(i,n){
						 max=Math.max(max,n)
						 min=Math.min(min,n)
					 })
					 var date =new Date(_this.$result.year,max,0).getDate()
					 dateRange = min+'/1/-'+max+'/'+ date
				 }
			 }
		  }
		  _this.$result.dateRange = dateRange
	  },
	  /**
	   * 判断是否当前年，禁用视图选项
	   */
	  disableItemByYear:function(){
		  var _this = this
		  var quar =_this.getQuarterByMonth(_this.$curDate.month)
		  if(_this.$result.year == _this.$curDate.year){
			  _this.$calender.find('.row').each(function(ele,index){
			        var id = $(this).attr('id')
			        if(id == 'nd' && _this.$curDate.month < 7){
			        	$(this).find('span.list span').eq(2).attr('disabled',true)
			        }
			        if(id == "quar"){
			        	$(this).find('span.list span:gt('+(quar-1)+')').attr('disabled',true)
			        }
			        if(id=="month" || id=="monthRange"){
			        	$(this).find('span.list span:gt('+(_this.$curDate.month-1)+')').attr('disabled',true)
			        }
			  })
		  }else{
			  _this.$calender.find('.row span.list span').removeAttr('disabled')
		  }
	  },
	  /**
	   * 切换时将选择结果置空
	   */
	  emptyResult:function(){
		  $.extend(this.$result,{nd:null,quar:null,month:null,monthRange:null})
		  tapCount = 0
	  },
	  /**
	   * 通过原型可取得选择的结果
	   */
	  getResult:function(){
		  return this.$result
	  },
	  /**
	   * 将选择结果转为中文
	   */
	  getNameByResult:function(key,value){
		  var name=null
		  $.each(this.settings.data[key],function(i,n){
			  if(n.value == value){
				  name = n.name
			  }
		  })
		  return name
	  },
      /**
       * 将选择结果转为统计期id
       */
      getTjqidByResult:function(key,value){
          var id=null
          $.each(this.settings.data[key],function(i,n){
              if(n.value == value){
                  id = n.id
              }
          })
          return id
      },
	  change:function(){
		  
	  },
	  /**
	   * 取消事件，将视图还原为上次选择结果
	   */
	  cancelEvent:function(){
		  this.$calender.hide()
		  this.$input.data('value')
		  $.extend(this.$result,this.$input.data('value'))
		  this.updateViewByResult()
	  },
	  /**
	   * 点击确定事件，复制视图，以便选择取消时更新视图
	   */
	  okEvent:function(){  //这里是点确定的jquery的方法
		  var _this = this
		  var key=null
		  var value=null
		  var html=_this.$result.year+'年'
		  var tjqId = _this.$result.year;
		  this.getResult.call(this)
		  for(var k in _this.$result){
			  if(k == 'nd' || k == 'quar' || k == 'monthRange' || k == 'month'){
				 if( _this.$result[k] != null){
					 key = k
					 value = _this.$result[k]
				 }
			  }
		  }
		  if(key==null){
			  this.$input.val('')
			  this.$calender.hide()
			  return 
		  }
		  if(key =='nd' || key =='quar' || key =='month'){
			  html+=this.getNameByResult(key,value)
			  tjqId+=this.getTjqidByResult(key,value)
		  }else{
		     var max=0
			 var min=12
			 $.each(_this.$result.monthRange,function(i,n){
				 max=Math.max(max,n)
				 min=Math.min(min,n)
			 })
			 html+=min+'月-'+max+'月'
             tjqId='' + tjqId + (max + 16);
		  }
		  this.$input.val(html)
		  this.$tjqId.val(tjqId);
		  this.$input.attr('value',html)
		  this.$input.data('value',$.extend({},_this.$result))
		  this.$calender.hide()
		  this.$input.trigger('change')
	  }
	}
	$.fn.calender=function(options){
		var calender =new Calender(this,options)
		return calender.init()
	}
})(jQuery)
~~~

~~~
插件调用
$('#datePicker input.date-picker').change(function(){
			 _this.tjq = $("#tjqId").attr('value')
			 _this.tjqMc = $(this).val()
		})
~~~    
