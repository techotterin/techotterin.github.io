---
title: 雷达图中处理不同数量级的数据
date: 2018-02-18 14:50:05
author: techotter
tags: 
  - ECharts
  - 雷达图
  - 数据可视化
categories:
  - 数据可视化
---

在使用ECharts绘制雷达图时,如果每个顶点的值相差过大,数据展示中数据项很小的部分将很难区分出来。因此,需要对数据做区间处理,以每一个数量级为一个区间来展示数据。本文将介绍如何在ECharts雷达图中处理不同数量级的数据。

<!-- more -->

## 问题描述

假设我们有一组案件罚没金额的数据,包括近5年的数据,每年的数据分为5个区间:

- 1000元以下
- 1000-10000元  
- 10000-5万元
- 5万元-20万元
- 20万元以上

不同年份的数据差异较大,直接绘制雷达图,较小的数据点几乎无法展示。我们需要对数据进行预处理,将不同数量级的数据映射到一个合适的区间内,使得较小的数据点也能清晰可见。

## 解决方案

我们可以将数据按照数量级划分为不同的区间,每个区间内再做归一化处理。以1000元以下区间为例,假设`polarValue`为雷达图每个轴的最大值,`outerDataArr`为原始的案件数量数据,则可以按照以下公式将原始数据映射到`[0.8*polarValue, polarValue]`区间内:

```
subVirtureOuterDataArr.push(polarValue*((((outerDataArr[i][j]-polarValue/10)/(polarValue-polarValue/10))*0.2)+0.8));
```

类似地,对于其他区间,可以映射到`[0.6*polarValue, 0.8*polarValue]`、`[0.4*polarValue, 0.6*polarValue]`、`[0.2*polarValue, 0.4*polarValue]`和`[0, 0.2*polarValue]`区间内。特别地,对于1000元以下区间内数量为1的情况,我们直接映射到`0.1*polarValue`,而数量为0的情况则映射到0。

经过上述处理后,我们得到了一组新的虚拟数据`virtureOuterDataArr`,可以用于绘制雷达图。

## 代码实现

以下是完整的代码实现:

```javascript
//初始化函数
init : function(ec,ecConfig){
    //页面初始化加载行业案件数量
    $.ajax({
        url : getPath()+'/CaseAmountServlet.json?fillDataIntoRadar=true',
        type: 'post',
        data :{},
        async : false,
        success : function(e){
            if(e['success']==false){
                 var objs = [];
                 var len = 0;
                 var pWidth = $('#chartmain').parent().width();
                  $('#chartmain').css('width',pWidth+'px');
                  $.myChart = ec.init(document.getElementById('chartmain'));
            }else{
                var objs = e['bean'];
                var len = objs.length;
                var pWidth = $('#chartmain').parent().width();
                $('#chartmain').css('width',(pWidth*0.75)+'px');
                $('#chartmain').before('<br>');
                $.myChart = ec.init(document.getElementById('chartmain'));
            }
            var years = [];
            var tempVal = '';
            var date = new Date();
            var year = date.getFullYear();
            var month = date.getMonth()+1-1;
            for(var i=0;i<5;i++){  //for(var i=0;i<len;i++){  取固定长度，代表近5年数据，即使某一年没有数据
                //不从后台取值，直接在前台判断
                if(month==0){
                    year--;
                    month=12;
                }
                years.push((year-i)+'年');
            }
            //解析出value数组
            var outerDataArr = [];
            if(e['success']==false){  //为空
            }else{        //不为空
                for(var i=0;i<years.length;i++){
                    var innerDataArr = [];
                    var hasData = false;
                    var k = 1;      //用来计数，确保一个集合中的值有5个，k取值1-6
                    for(var j =0;j<objs.length;j++){
                        if((e['bean'][j]['CUR_TERM']+'年')==years[i]){
                            if(e['bean'][j]['SEC_RANGE'] == '01' && k>1)
                                k=1;
                            else if(e['bean'][j]['SEC_RANGE'] == '02' && k>2)
                                k=1;
                            else if(e['bean'][j]['SEC_RANGE'] == '03' && k>3)
                                k=1;
                            else if(e['bean'][j]['SEC_RANGE'] == '04' && k>4)
                                k=1;
                            else if(e['bean'][j]['SEC_RANGE'] == '05' && k>5)
                                k=1;
                            if(e['bean'][j]['SEC_RANGE']== ('0'+k))
                                innerDataArr.push(e['bean'][j]['AJ_NUM']);
                            else{
                                innerDataArr.push(0);
                                j--;
                            }
                            hasData = true;
                        }
                        k++;
                    }
                    if(hasData){
                        if(innerDataArr.length == 4)
                            innerDataArr.push(0);            //只有最后一个可能不为0
                        else if(innerDataArr.length == 3){
                            innerDataArr.push(0);
                            innerDataArr.push(0);
                        }else if(innerDataArr.length == 2){
                            innerDataArr.push(0);
                            innerDataArr.push(0);
                            innerDataArr.push(0);
                        }else if(innerDataArr.length == 1){
                            innerDataArr.push(0);
                            innerDataArr.push(0);
                            innerDataArr.push(0);
                            innerDataArr.push(0);
                        }
                        outerDataArr.push(innerDataArr);
                    }
                    else
                        outerDataArr.push([0,0,0,0,0]);
                }
                var maxValues = [];
                //获取最大值
                $.ajax({
                    url:getPath()+'/CaseAmountServlet.json?getPolarMaxValue=true',
                    type:'post',
                    async:false,
                    success : function(e2){
                        $.extend({maxValues:e2['bean']});
                    }
                });

                var polarValue = $.maxValues[0]['MAX_VALUE'];    //极值
                for(var i = 1;i<$.maxValues.length;i++){
                    if(polarValue < $.maxValues[i]['MAX_VALUE'])
                        polarValue = $.maxValues[i]['MAX_VALUE'];
                }

                polarValue = polarValue * 1.05+1;        //将这个值作为每个轴的最大值。且outerDataArr中包括了五年的所有数据，包括0值的
                var referValue = 1;
                var polarMaxValueRange = [];
                for(var i=0;i<5;i++){            //得到雷达图中的5个区间
                    var temp = polarValue/referValue;
                    polarMaxValueRange.push(temp);
                    referValue = referValue*10;
                }

                //转换虚拟值，实际值取outerDataArr
                var virtureOuterDataArr = [];        //每个数组中的值都要变化，共计执行25次
                for(var i = 0;i<5;i++){
                    var subVirtureOuterDataArr = [];
                    for(var j=0;j<5;j++){
                        if(outerDataArr[i][j]>polarValue/10){        //落点 在 第一个区间
                            subVirtureOuterDataArr.push(polarValue*((((outerDataArr[i][j]-polarValue/10)/(polarValue-polarValue/10))*0.2)+0.8));
                        }else if(outerDataArr[i][j]>polarValue/100){
                            subVirtureOuterDataArr.push(polarValue*((((outerDataArr[i][j]-polarValue/100)/(polarValue/10-polarValue/100))*0.2)+0.6));
                        }else if(outerDataArr[i][j]>polarValue/1000){
                            subVirtureOuterDataArr.push(polarValue*((((outerDataArr[i][j]-polarValue/1000)/(polarValue/100-polarValue/1000))*0.2)+0.4));
                        }else if(outerDataArr[i][j]>polarValue/10000){
                            subVirtureOuterDataArr.push(polarValue*((((outerDataArr[i][j]-polarValue/10000)/(polarValue/1000-polarValue/10000))*0.2)+0.2));
                        }else{        //最后一个区间，如果1在这个区间中，让其占有0.1的长度
                            if(outerDataArr[i][j]==1){
                                subVirtureOuterDataArr.push(polarValue*0.1);
                            }else if(outerDataArr[i][j]==0){
                                subVirtureOuterDataArr.push(0);
                            }else{
                                if(((outerDataArr[i][j]-0)/(polarValue/10000-0))*0.2>0.1)
                                    subVirtureOuterDataArr.push(polarValue*((((outerDataArr[i][j]-0)/(polarValue/10000-0))*0.2)));
                                else
                                    subVirtureOuterDataArr.push(polarValue*((((outerDataArr[i][j]-0)/(polarValue/10000-0))*0.2)+0.1));
                            }
                        }
                    }
                    virtureOuterDataArr.push(subVirtureOuterDataArr);
                }

                var optionData = [];
                for(var i=0;i<years.length;i++){
                    var obj = new Object();
                    obj.value = [virtureOuterDataArr[i][0]||0, virtureOuterDataArr[i][1]||0, virtureOuterDataArr[i][2]||0, virtureOuterDataArr[i][3]||0,virtureOuterDataArr[i][4]||0];
                    obj.name = years[i];
                    optionData.push(obj);
                }
            }

            var chartOptions = {
                    title : {
                        text: '案件罚没金额风险分析',
                        subtext: '内蒙古自治区'
                    },
                    tooltip : {
                        trigger: 'axis'
                    },
                    legend :{
                        orient : 'vertical',
                        x : 'right',
                        y : 'top',
                        data:years
                    },
                    polar : [
                             {
                                 indicator : [
                                     { text: '1000元以下', max: polarValue},
                                     { text: '1000-10000元', max: polarValue},
                                     { text: '10000-5万元', max: polarValue},
                                     { text: '5万元-20万元', max: polarValue},
                                     { text: '20万元以上', max: polarValue}
                                  ]
                              }
                          ],
                      calculable : true,
                      series : [
                          {
                              name: '案件罚没金额',
                              type: 'radar',
                              itemStyle: {
                                  normal: {
                                      areaStyle: {
                                          type: 'default'
                                      }
                                  }
                              },
                              tooltip : {
                                  trigger: 'item',
                                  formatter: function (params,ticket,callback) {
                                      var index;
                                      var returnStr = '';
                                      for(var i = 0;i<5;i++){
                                          if(years[i]==params['1'] || years[i]==params['0']){
                                              index = i;
                                              break;
                                          }
                                      }
                                      if(params['1'].indexOf('年')>=0)
                                          returnStr += params['1']+'案件数量：'+'<br>';
                                      else
                                          returnStr += params['0']+'案件数量：'+'<br>';
                                      returnStr += '1000元以下：'+outerDataArr[index][0]+'<br>';
                                      returnStr += '1000-10000元：'+outerDataArr[index][1]+'<br>';
                                      returnStr += '10000-5万元：'+outerDataArr[index][2]+'<br>';
                                      returnStr += '5万元-20万元：'+outerDataArr[index][3]+'<br>';
                                      returnStr += '20万元以上：'+outerDataArr[index][4]+'<br>';
                                      return returnStr;
                                  }
                              },
                              data : optionData
                          }
                      ]
            }
            $.myChart.setOption(chartOptions);
            if(e['success']==false){
                Ext.Msg.alert('提示','当前系统中无数据');
            }
        }
    });
},
```

## 总结

本文介绍了如何在ECharts雷达图中处理不同数量级的数据。主要思路是将不同数量级的数据映射到不同的区间,每个区间再做归一化处理,使得较小的数据点也能清晰展示。通过合理设置区间和映射方式,可以有效解决雷达图中数据相差过大导致的显示问题。
