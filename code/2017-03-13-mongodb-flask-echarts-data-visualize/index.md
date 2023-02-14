# scrapy + mongodb + flask + echarts 数据可视化


[源码](https://github.com/shawnau/intern_visualization)
[项目地址](http://54.169.117.35:8080)

<!--more-->

## 项目框架

1. scrapy负责定时抓取数据到mongodb中
2. 每小时定时生成echarts需要的数据, 以json格式保存
3. flask读取json数据之后用jinja2渲染到echarts里作图, 使用的都是修改过的(毫无美感的)官方例子

整个过程大致如图:

```
             spider       pipelines.py
shixiseng.com------>scrapy----------->mongodb
                                       |
                                       | model.py
       template.html        view.py    v
echart<-------------flask<----------.json
```


## Prerequisite:

1. 定时工具: crontab(调度scrapy), apscheduler(调度flask)
2. 数据库: mongodb + pymongo
3. 后端: flask + flask-pymongo(负责flask和mongodb的对接, 其实也可以不用)
4. 绘图: echarts


## 安装 (Ubuntu 14.04/16.04)

1. apscheduler
[apscheduler官方文档](http://apscheduler.readthedocs.io/en/3.3.1/userguide.html)

 ```bash
$ pip install apscheduler
```

2. mongodb & pymongo

 [mongodb官方文档](https://docs.mongodb.com/getting-started/shell/tutorial/install-mongodb-on-ubuntu/), [pymongo官方文档](https://api.mongodb.com/python/current/installation.html)

 ```bash
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
# Ubuntu 14.04
$ echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
# Ubuntu 16.04
$ echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list

$ sudo apt-get update
$ sudo apt-get install -y mongodb-org

# pymongo
$ python -m pip install pymongo
```

3. flask & flask-pymongo
[flask中文文档](http://docs.jinkan.org/docs/flask/), [flask-pymongo官方文档](http://www.pythondoc.com/flask-pymongo/)

 ```bash
$ sudo pip install Flask # 推荐使用virtualenv, 这里懒得用了
$ pip install Flask-PyMongo
```

## 测试

`cd`到git目录下

```bash
$ sudo service mongod start # 启动mongod (其实不用, 因为已经预加载了json)
$ python visualization/view.py
```

进入`127.0.0.1:5000`下预览吧

(2023/02/14更新: 以下是一个 echarts 图标源数据, 需要安装插件渲染, 但因为年久失修没有维护, 我已经禁用了它)

```text
{% echarts 400 '100%' %}
        var dataAxis = ["北京", "上海", "广州", "杭州", "成都", "深圳", "南京", "武汉", "大连", "苏州", "西安", "厦门", "珠海", "重庆", "天津"];
        var data = [720, 420, 171, 128, 109, 103, 66, 53, 31, 18, 18, 17, 14, 14, 14];
        var yMax = 500;
        var dataShadow = [];
        
        for (var i = 0; i < data.length; i++) {
            dataShadow.push(yMax);
        }
        
        option = {
            backgroundColor: '#fff',
            title: {
                text: '实习数量-城市分布排名',
                subtext: 'data from 实习僧',
                sublink: 'http://www.shixiseng.com',
            },
            tooltip : {
                trigger: 'axis',
                axisPointer : {        // 坐标轴指示器，坐标轴触发有效
                type : 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
                }
            },
            xAxis: {
                data: dataAxis,
                axisLabel: {
                    inside: true,
                    textStyle: {
                        color: '#000'
                    }
                },
                axisTick: {
                    show: false
                },
                axisLine: {
                    show: false
                },
                z: 10
            },
            yAxis: {
                axisLine: {
                    show: false
                },
                axisTick: {
                    show: false
                },
                axisLabel: {
                    textStyle: {
                        color: '#999'
                    }
                }
            },
            dataZoom: [
                {
                    type: 'inside'
                }
            ],
            series: [
                {
                    type: 'bar',
                    name: '数量',
                    itemStyle: {
                        normal: {
                            color: new echarts.graphic.LinearGradient(
                                0, 0, 0, 1,
                                [
                                    {offset: 0, color: '#83bff6'},
                                    {offset: 0.5, color: '#188df0'},
                                    {offset: 1, color: '#188df0'}
                                ]
                            )
                        },
                        emphasis: {
                            color: new echarts.graphic.LinearGradient(
                                0, 0, 0, 1,
                                [
                                    {offset: 0, color: '#2378f7'},
                                    {offset: 0.7, color: '#2378f7'},
                                    {offset: 1, color: '#83bff6'}
                                ]
                            )
                        }
                    },
                    data: data
                }
            ]
        };
{% endecharts %}


{% echarts 400 '100%' %}
        option = {
            backgroundColor: '#fff',
            title: {
                text: '实习标签/公司-数量分布',
                left: 'left',
                textStyle: {
                    color: '#000'
                }
            },
            tooltip: {
                trigger: 'item',
                formatter: "{a} <br/>{b}: {c} ({d}%)"
            },
            legend: {
                orient: 'vertical',
                x: 'left',
            },
            series: [
                {
                    name:'标签',
                    type:'pie',
                    radius: ['50%', '70%'],
                    avoidLabelOverlap: true,
                    label: {
                        normal: {
                            show: true,
                            position: 'outside'
                        },
                        emphasis: {
                            show: true,
                            textStyle: {
                                fontSize: '30',
                                fontWeight: 'bold'
                            }
                        }
                    },
                    labelLine: {
                        normal: {
                            show: true
                        }
                    },
                    data: [{"name": "软件", "value": 422}, {"name": "IT运维", "value": 185}, {"name": "Java", "value": 147}, {"name": "前端", "value": 144}, {"name": "算法", "value": 138}, {"name": "云计算/大数据", "value": 129}, {"name": "C/C++", "value": 124}, {"name": "测试", "value": 124}, {"name": "数据挖掘", "value": 108}, {"name": "数据库", "value": 101}, {"name": "PHP", "value": 95}, {"name": "Android", "value": 77}, {"name": "Python", "value": 63}, {"name": "IOS", "value": 63}, {"name": "Node.js", "value": 11}]
                },

                {
                    name:'公司',
                    type:'pie',
                    radius: ['10%', '40%'],
                    avoidLabelOverlap: true,
                    label: {
                        normal: {
                            show: false,
                            position: 'inside'
                        },
                        emphasis: {
                            show: true,
                            textStyle: {
                                color: '#000',
                                fontSize: '20',
                                fontWeight: 'bold'
                            }
                        }
                    },
                    labelLine: {
                        normal: {
                            show: true
                        }
                    },
                    data: [{"name": "好未来", "value": 21}, {"name": "百度", "value": 19}, {"name": "爱奇艺", "value": 18}, {"name": "链家网", "value": 17}, {"name": "金山云", "value": 16}, {"name": "睿琪软件", "value": 16}, {"name": "讯猫", "value": 15}, {"name": "搜狐", "value": 14}, {"name": "UISEE", "value": 12}, {"name": "非白三维", "value": 12}, {"name": "星环科技", "value": 12}, {"name": "美团点评", "value": 11}, {"name": "今日头条", "value": 11}, {"name": "北纬通信", "value": 11}, {"name": "大连东软", "value": 10}]
                }
            ]
        };
{% endecharts %}

{% echarts 400 '100%' %}
            option = {
                backgroundColor: '#fff',
                title: {
                    text: '堆叠区域图',
                    left: 'left'
                },
                tooltip : {
                    trigger: 'axis'
                },
                legend: {
                    data:["软件", "IT运维", "Java", "前端", "算法", "云计算/大数据", "C/C++", "测试", "数据挖掘", "数据库", "PHP", "Android", "Python", "IOS", "Node.js"]
                },
                toolbox: {
                    feature: {
                        saveAsImage: {}
                    }
                },
                grid: {
                    left: '3%',
                    right: '4%',
                    bottom: '3%',
                    containLabel: true
                },
                xAxis : [
                    {
                        type : 'category',
                        boundaryGap : false,
                        data : ['2017/01/05', '2017/01/10', '2017/01/11', '2017/01/12', '2017/01/18', '2017/01/19', '2017/02/03', '2017/02/04', '2017/02/06', '2017/02/07', '2017/02/08', '2017/02/09', '2017/02/10', '2017/02/11', '2017/02/12', '2017/02/13', '2017/02/14', '2017/02/15', '2017/02/16', '2017/02/17', '2017/02/19', '2017/02/20', '2017/02/21', '2017/02/22', '2017/02/23', '2017/02/24', '2017/02/25', '2017/02/26', '2017/02/27', '2017/02/28', '2017/03/01', '2017/03/02', '2017/03/03', '2017/03/04', '2017/03/05', '2017/03/06', '2017/03/07', '2017/03/08', '2017/03/09', '2017/03/10', '2017/03/11', '2017/03/12', '2017/03/13', '2017/03/14']
                    }
                ],
                yAxis : [
                    {
                        type : 'value'
                    }
                ],
                series : [
                    
                    {
                        name: '软件',
                        type: 'line',
                        stack: '总量',
                        areaStyle: {normal: {}},
                        data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 2, 0, 1, 0, 1, 0, 2, 1, 2, 0, 1, 3, 4, 3, 4, 26, 5, 6, 38, 2, 9, 22, 16, 3, 3, 84, 168]
                    },
                    
                    {
                        name: 'IT运维',
                        type: 'line',
                        stack: '总量',
                        areaStyle: {normal: {}},
                        data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 3, 1, 1, 4, 2, 0, 0, 7, 4, 8, 0, 4, 1, 0, 5, 13, 9, 9, 19, 0, 0, 62, 29]
                    },
                    
                    {
                        name: 'Java',
                        type: 'line',
                        stack: '总量',
                        areaStyle: {normal: {}},
                        data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 2, 0, 1, 0, 0, 6, 16, 25, 2, 3, 63, 27]
                    },
                    
                    {
                        name: '前端',
                        type: 'line',
                        stack: '总量',
                        areaStyle: {normal: {}},
                        data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 4, 16, 10, 16, 6, 0, 53, 37]
                    },
                    
                    {
                        name: '算法',
                        type: 'line',
                        stack: '总量',
                        areaStyle: {normal: {}},
                        data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 16, 1, 15, 20, 13, 4, 4, 51, 10]
                    },
                    
                    {
                        name: '云计算/大数据',
                        type: 'line',
                        stack: '总量',
                        areaStyle: {normal: {}},
                        data: [0, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 2, 1, 1, 2, 2, 2, 4, 1, 0, 0, 4, 4, 4, 0, 0, 0, 5, 1, 3, 2, 1, 1, 0, 5, 5, 5, 15, 8, 2, 0, 29, 9]
                    },
                    
                    {
                        name: 'C/C++',
                        type: 'line',
                        stack: '总量',
                        areaStyle: {normal: {}},
                        data: [0, 0, 0, 0, 0, 0, 0, 0, 2, 1, 0, 0, 0, 0, 0, 0, 0, 2, 4, 2, 0, 5, 1, 0, 0, 0, 0, 0, 2, 4, 5, 3, 5, 0, 0, 4, 7, 12, 7, 9, 1, 3, 30, 8]
                    },
                    
                    {
                        name: '测试',
                        type: 'line',
                        stack: '总量',
                        areaStyle: {normal: {}},
                        data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 4, 0, 0, 1, 0, 0, 0, 0, 0, 0, 82, 32]
                    },
                    
                    {
                        name: '数据挖掘',
                        type: 'line',
                        stack: '总量',
                        areaStyle: {normal: {}},
                        data: [0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 3, 0, 0, 2, 0, 2, 1, 0, 0, 1, 3, 5, 2, 2, 5, 0, 0, 9, 9, 5, 9, 8, 1, 0, 20, 15]
                    },
                    
                    {
                        name: '数据库',
                        type: 'line',
                        stack: '总量',
                        areaStyle: {normal: {}},
                        data: [0, 0, 0, 0, 0, 0, 0, 0, 2, 0, 2, 0, 1, 0, 1, 1, 2, 1, 0, 0, 0, 0, 1, 1, 1, 2, 0, 0, 2, 2, 4, 1, 5, 0, 0, 8, 4, 12, 14, 8, 1, 0, 13, 6]
                    },
                    
                    {
                        name: 'PHP',
                        type: 'line',
                        stack: '总量',
                        areaStyle: {normal: {}},
                        data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 4, 0, 3, 0, 0, 0, 2, 1, 2, 1, 0, 1, 0, 2, 1, 3, 1, 2, 0, 0, 3, 4, 7, 7, 6, 2, 1, 22, 12]
                    },
                    
                    {
                        name: 'Android',
                        type: 'line',
                        stack: '总量',
                        areaStyle: {normal: {}},
                        data: [0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1, 2, 0, 0, 0, 0, 1, 1, 1, 1, 0, 0, 1, 0, 0, 0, 0, 0, 3, 1, 1, 0, 0, 1, 2, 1, 7, 5, 6, 5, 0, 27, 7]
                    },
                    
                    {
                        name: 'Python',
                        type: 'line',
                        stack: '总量',
                        areaStyle: {normal: {}},
                        data: [0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 2, 2, 1, 1, 0, 1, 0, 1, 5, 2, 0, 0, 1, 1, 4, 4, 6, 0, 1, 12, 14]
                    },
                    
                    {
                        name: 'IOS',
                        type: 'line',
                        stack: '总量',
                        areaStyle: {normal: {}},
                        data: [0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 1, 1, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 1, 1, 3, 2, 1, 0, 1, 2, 0, 4, 8, 3, 3, 0, 15, 6]
                    },
                    
                    {
                        name: 'Node.js',
                        type: 'line',
                        stack: '总量',
                        areaStyle: {normal: {}},
                        data: [0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 1, 2, 0, 1, 2, 0]
                    },
                    
                ]
            };
{% endecharts %}
```

## 一些坑

1. 这篇blog中插入了echarts图表, 参见[这篇文章](http://kchen.cc/2016/11/05/echarts-in-hexo/), 并做了一些魔改以适应复杂的图表
2. falsk中的APScheduler貌似不好用, 最后自己使用丑陋的方法写了个独立的py文件处理生成的json, 再用crontab定时运行

> 待续
