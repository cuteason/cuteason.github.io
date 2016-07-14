---
layout: post
title: ECharts制作旅游地图
date: 2015-05-24 14:30
location: 北京
category: tech
tags: [echarts, javascript]
---
心血来潮，想到一个点子：把之前和女盆友异地的时候来回奔波过的地方描绘出来。想起来之前看到的百度春运迁徙图，就尝试做一个这样的地图。使用了百度开源的一个纯JavaScript的图标库[ECharts](http://echarts.baidu.com/)，可以很方便地实现这个想法。

<div id="chart" style="height:670px"></div>

准备按照出行的时间顺序，搭配一个时间轴，依次显示路线和旅行城市。

To be continued.
<script src="{{ site.baseurl }}/js/echarts.js"></script>
<script type="text/javascript">
    require.config({
        paths: {
            echarts: '{{ site.baseurl }}/js'
        }
    });
    require(
        [
            'echarts',
            'echarts/chart/map'
        ],
        function (ec) {
            var myChart = ec.init(document.getElementById('chart')); 
            var option = {
                // 开始option
                backgroundColor: '#1b1b1b',
                color: ['gold', 'aqua', 'lime', 'coral', 'lightblue', 'lightsalmon', 'orchid', 'orangered'],
                title : {
                    text: '那些年我们异地的日子',
                    subtext: 'For Memory',
                    x: 'center',
                    textStyle : {
                        color: '#fff'
                    }
                },
                tooltip : {
                    trigger: 'item',
                    formatter: '{b}'
                },
                legend: {
                    orient: 'vertical',
                    x:'left',
                    data:['北京', '洛阳', '九江', '黄冈', '西安', '贵阳', '成都', '遵义'],
                    selectedMode: 'multiple',
                    textStyle : {
                        color: '#fff'
                    }
                },
                toolbox: {
                    show : false
                },
                dataRange: {
                    min : 0,
                    max : 100,
                    calculable : true,
                    color: ['#ff3333', 'orange', 'yellow','lime','aqua'],
                    textStyle:{
                        color:'#fff'
                    }
                },
                animationDurationUpdate: 2000, // for update animation, like legend selected.
                series : [
                    {
                        name: '北京',
                        type: 'map',
                        roam: true,
                        hoverable: false,
                        mapType: 'china',
                        itemStyle:{
                            normal:{
                                borderColor:'rgba(100,149,237,1)',
                                borderWidth:0.5,
                                areaStyle:{
                                    color: '#1b1b1b'
                                },
                                label : {
                                    show: false
                                }
                            },
                            emphasis:{
                                label:{
                                    show:true
                                }
                            }
                        },
                        data:[],
                        geoCoord: {
                            '北京': [116.4551,40.2539],
                            '遵义': [106.9,27.7],
                            '西安': [109.1162,34.2004],
                            '贵阳': [106.6992,26.7682],
                            '成都': [103.9526,30.7617],
                            '九江': [115.97,29.71],
                            '洛阳': [112.44,34.7],
                            '黄冈': [114.87373352050781,30.42832342058197]
                        },
                        markLine : {
                            smooth:true,
                            effect : {
                                show: true,
                                scaleSize: 1,
                                period: 30,
                                color: '#fff',
                                shadowBlur: 10
                            },
                            itemStyle : {
                                normal: {
                                    borderWidth:1,
                                    label: {
                                        show: false
                                    },
                                    lineStyle: {
                                        type: 'solid',
                                        shadowBlur: 10
                                    }
                                }
                            },
                            data : [
                                [{name:'北京'}, {name:'贵阳',value:95}],
                                [{name:'北京'}, {name:'洛阳',value:60}],
                                [{name:'北京'}, {name:'九江',value:50}],
                                [{name:'北京'}, {name:'成都',value:20}]
                            ]
                        },
                        markPoint : {
                            symbol:'emptyCircle',
                            symbolSize : function (v){
                                return 10 + v/10
                            },
                            effect : {
                                show: true,
                                shadowBlur : 0
                            },
                            itemStyle:{
                                normal:{
                                    label:{show:false}
                                },
                                emphasis: {
                                    label:{position:'top'}
                                }
                            },
                            data : [
                                {name:'贵阳',value:95},
                                {name:'成都',value:20},
                                {name:'九江',value:50},
                                {name:'洛阳',value:60}
                            ]
                        }
                    },
                    {
                        name: '贵阳',
                        type: 'map',
                        mapType: 'china',
                        data:[],
                        markLine : {
                            smooth:true,
                            effect : {
                                show: true,
                                scaleSize: 1,
                                period: 30,
                                color: '#fff',
                                shadowBlur: 10
                            },
                            itemStyle : {
                                normal: {
                                    borderWidth:1,
                                    label: {
                                        show: false
                                    },
                                    lineStyle: {
                                        type: 'solid',
                                        shadowBlur: 10
                                    }
                                }
                            },
                            data : [
                                [{name:'贵阳'}, {name:'北京',value:50}]
                            ]
                        }
                    },
                    {
                        name: '成都',
                        type: 'map',
                        mapType: 'china',
                        data:[],
                        markLine : {
                            smooth:true,
                            effect : {
                                show: true,
                                scaleSize: 1,
                                period: 30,
                                color: '#fff',
                                shadowBlur: 10
                            },
                            itemStyle : {
                                normal: {
                                    borderWidth:1,
                                    label: {
                                        show: false
                                    },
                                    lineStyle: {
                                        type: 'solid',
                                        shadowBlur: 10
                                    }
                                }
                            },
                            data : [
                                [{name:'成都'}, {name:'北京',value:20}]
                            ]
                        }
                    },
                    {
                        name: '九江',
                        type: 'map',
                        mapType: 'china',
                        data:[],
                        markLine : {
                            smooth:true,
                            effect : {
                                show: true,
                                scaleSize: 1,
                                period: 30,
                                color: '#fff',
                                shadowBlur: 10
                            },
                            itemStyle : {
                                normal: {
                                    borderWidth:1,
                                    label: {
                                        show: false
                                    },
                                    lineStyle: {
                                        type: 'solid',
                                        shadowBlur: 10
                                    }
                                }
                            },
                            data : [
                                [{name:'九江'}, {name:'洛阳',value:80}],
                                [{name:'九江'}, {name:'北京',value:60}]
                            ]
                        }
                    },
                    {
                        name: '黄冈',
                        type: 'map',
                        mapType: 'china',
                        data:[],
                        markLine : {
                            smooth:true,
                            effect : {
                                show: true,
                                scaleSize: 1,
                                period: 30,
                                color: '#fff',
                                shadowBlur: 10
                            },
                            itemStyle : {
                                normal: {
                                    borderWidth:1,
                                    label: {
                                        show: false
                                    },
                                    lineStyle: {
                                        type: 'solid',
                                        shadowBlur: 10
                                    }
                                }
                            },
                            data : [
                                [{name:'黄冈'}, {name:'九江',value:70}]
                            ]
                        }
                    },
                    {
                        name: '西安',
                        type: 'map',
                        mapType: 'china',
                        data:[],
                        markLine : {
                            smooth:true,
                            effect : {
                                show: true,
                                scaleSize: 1,
                                period: 30,
                                color: '#fff',
                                shadowBlur: 10
                            },
                            itemStyle : {
                                normal: {
                                    borderWidth:1,
                                    label: {
                                        show: false
                                    },
                                    lineStyle: {
                                        type: 'solid',
                                        shadowBlur: 10
                                    }
                                }
                            },
                            data : [
                                [{name:'西安'}, {name:'洛阳',value:15}]
                            ]
                        }
                    },
                    {
                        name: '遵义',
                        type: 'map',
                        mapType: 'china',
                        data:[],
                        markLine : {
                            smooth:true,
                            effect : {
                                show: true,
                                scaleSize: 1,
                                period: 30,
                                color: '#fff',
                                shadowBlur: 10
                            },
                            itemStyle : {
                                normal: {
                                    borderWidth:1,
                                    label: {
                                        show: false
                                    },
                                    lineStyle: {
                                        type: 'solid',
                                        shadowBlur: 10
                                    }
                                }
                            },
                            data : [
                                [{name:'遵义'}, {name:'贵阳',value:10}]
                            ]
                        }
                    },
                    {
                        name: '洛阳',
                        type: 'map',
                        mapType: 'china',
                        data:[],
                        markLine : {
                            smooth:true,
                            effect : {
                                show: true,
                                scaleSize: 1,
                                period: 30,
                                color: '#fff',
                                shadowBlur: 10
                            },
                            itemStyle : {
                                normal: {
                                    borderWidth:1,
                                    label: {
                                        show: false
                                    },
                                    lineStyle: {
                                        type: 'solid',
                                        shadowBlur: 10
                                    }
                                }
                            },
                            data : [
                                [{name:'洛阳'}, {name:'北京',value:55}],
                                [{name:'洛阳'}, {name:'西安',value:15}]
                            ]
                        }
                    }
                ]
                // -- 结束option
            };
    
            // 为echarts对象加载数据 
            myChart.setOption(option); 
        }
    );
</script>
