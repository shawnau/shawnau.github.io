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

(2023/02/14更新: 以下本来是一个 echarts 图标源数据, 需要安装插件渲染, 但因为年久失修没有维护, 我已经禁用了它, 请参考文末的链接)

## 一些坑

1. 这篇blog中插入了echarts图表, 参见[这篇文章](http://kchen.cc/2016/11/05/echarts-in-hexo/), 并做了一些魔改以适应复杂的图表
2. falsk中的APScheduler貌似不好用, 最后自己使用丑陋的方法写了个独立的py文件处理生成的json, 再用crontab定时运行

> 待续
