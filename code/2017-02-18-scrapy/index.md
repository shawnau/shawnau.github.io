# Scrapy简介(附项目:爬取实习僧实习信息)


[scrapy的官方中文文档](http://scrapy-chs.readthedocs.io/zh_CN/latest/index.html)

[源代码](https://github.com/shawnau/shixiseng)

<!--more-->

# Scrapy框架数据流

![](http://7xlen8.com1.z0.glb.clouddn.com/scrapy_work.jpeg)

上图是scrapy的数据流, 看不懂没关系, 接下来会简单介绍scrapy爬取的流程.

```bash
pip install scrapy # 安装scrapy
scrapy startproject shixiseng # 启动一个新项目,命名为shixiseng
```

接下来会在目录下看到以下结构:

```
shixiseng/
    scrapy.cfg
    shixiseng/
        __init__.py
        items.py
        pipelines.py
        settings.py
        spiders/
            __init__.py
            ...
```

我们重点关注的是以下几个文件:

`spiders`目录下的脚本负责解析和爬取网页信息 -> 将信息封装成`items.py`中定义好的格式(类似于python的dict) -> 传给`pipelines.py`处理得到的Item, 可以写入文件, 也可以传入数据库

# spider部分的构造

1. 在spiders目录下新建一个脚本, 命名为`intern_spider.py`

2. 看看实习僧主页, 列出了一堆关键词, 随便找一个进入, 发现url格式很简单, 为`http://www.shixiseng.com/interns?k=IOS&p=1`, 其中k=搜索关键词, p=显示页数. 这个结构很简单, 不涉及复杂的js/ajax功能, 例如需要下拉才能刷新出信息等. 省却了很多麻烦

![](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/Screen%20Shot%202017-02-18%20at%205.13.01%20PM.png)

3. 发现置空关键词的话, 只能显示出500页信息, 而实习信息远远不止这个数, 所以还是乖乖按关键词搜索吧. 把第一行的关键词输入一个list, 按照list一个一个爬就可以了, 见下:

 ```python
 # 爬取各种关键词
     key_words = ['软件', 'IOS', '数据库', 'C#/.NET', 
                  'Hadoop', 'Android', '算法', 'IT运维', 
                  'Python', '云计算/大数据', 'Node.js', '数据挖掘', 
                  'PHP', 'Ruby/Perl', '测试', 'Java', 
                  'C/C++', '前端']
     start_urls = ['http://www.shixiseng.com/interns?k=#&p=1'.replace('#', x) for x in key_words]
 ```

 - 其中`start_urls`就是scrapy开始爬取的链接, 我们组装出了18个关键词, 每次从搜索结果的第一页开始爬取, 一直爬到最后一页. 然而最后一页是第几页? 在不知情的情况下, 我们只能一直增加页数, 直到新的页面不能显示出实习信息为止, 像这样:

 ![](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/Screen%20Shot%202017-02-18%20at%203.14.38%20PM.png)

 - 此时使用xpath搜不到显示实习信息的div就可以停止爬取了. 所以我们的`parse()`函数, 即爬取一页信息的信息定义如下:

```python
def parse(self, response):

        # 初始化选择器和item
        sel = Selector(response)
        item = InternItem()

        # 如果超出最大页数, 就不执行抓取
        if sel.xpath(".//*[@id='load_box_item']/div").extract_first() is not None:  
            
            # 爬取一页的职业信息, 返回一个list
            title = sel.xpath(".//*[@id='load_box_item']/div[*]/div/div/a/h3/text()").extract()  
            # 爬取指向每个信息的岗位职责子页面的链接
            dec_link = sel.xpath(".//*[@id='load_box_item']/div[*]/div/div/a/@href").extract()  

            # 将爬取的unicode转换成utf-8字节码
            # 每一页所有的候选框依次清洗并封装成wrapped_dict并缓存在item_list中
            item_list = []
            for i in range(len(title)):
                # 把相对路径拼接成绝对路径
                abs_link = 'http://www.shixiseng.com' + str(dec_link[i].encode('utf-8')
                wrapped_dict = {'dec_link': abs_link),
                                'title': title[i].encode('utf-8'),
                                }
                item_list.append(wrapped_dict)

            # 按照每一个title对应的url爬取岗位职责, 同时通过meta传入打包好的item给parse_dec()
            for i in range(len(item_list)):
                yield Request(item_list[i]['dec_link'], callback=self.parse_dec, meta={'item': item_list[i]})

            # 拼接好下一页的url并爬取下一页
            next_page_index = int(response.url[-1]) + 1
            next_page = response.url[:-1] + str(next_page_index)
            yield Request(next_page, callback=self.parse)
        else:
            yield item  # 直接返回空的item
```

介绍代码之前, 首先说明一下需要爬取的信息(源代码要复杂一点, 这里是简化版): title(职位), link(打开实习信息的链接), dec(实习信息, 需要在子页面爬取). 其中链接是在存储入数据库的时候用来区分是否重复的依据.

 - `sel = Selector(response)`这行把爬取到的信息初始化为一个选择器, 可以通过调用`xpath()`(xpath选择器), `css()`(css选择器), `re()`(正则表达式)等方法提取需要的信息
 - 首先通过`xpath()`方法利用xpath提取需要的信息, 再使用`extract()`提取得到的文本list, 否则会返回一个Object而不是文本.
 - 获取title和dec_link之后, 配对封装成`wrapped_dict`, 放到`item_list`中备用
 - 提取到每个实习信息的dict, 通过`Request(item_list[i]['dec_link'], callback=self.parse_dec, meta={'item': item_list[i]})`发出请求爬取子页面
 - 解释一下这里的`Request()`函数: 
     - 首先, `Request()`返回一个`response`类, 其中包含了请求获得的网页信息, 以及自己传入的一些参数.
     - 第一个参数是需要爬取的url 
     - 第二个参数是调用相应的函数处理爬取的信息, 这里会调用自己写好的`parse_dec()`函数处理这个`Request`的`response`. 
     - 第三个参数`meta={'item': item_list[i]})`将前面爬到的信息传入这里的`response`.
 - 下一步, 在`parse_dec()`中将爬取子页面的信息和之前传入的信息合并并交给pipelines处理, 见之后介绍`parse_dec()`的部分
 - 处理结束之后, 只爬取到了一页的信息, 接下来还要去下一页, 拼接好`next_page`之后, 使用`Request(next_page, callback=self.parse)`调用自己爬取下一页, 如果下一页没有信息了, 就直接返回空的item交给pipelines.

```python
# 爬取岗位职责页面, 和之前的信息拼接成item并传给pipelines
    def parse_dec(self, response):

        sel = Selector(response)
        # # 接收之前传入的meta
        item = response.meta['item']  
        # 职位描述和截止日期
        dec_list = sel.xpath(".//*[@id='container']/div[1]/div[1]/div[3]//text()").extract()
        item['dec_content'] = [x.encode('utf-8') for x in dec_list]
        yield item  # 返回爬取一个候选框的信息给pipeline处理
```

介绍一下这段代码

 - 首先, 之前爬取的职位和链接信息都包含在了`response`的`meta`参数里面了, 通过`item = response.meta['item']`把他们传入item.
 - 其次, 通过新增`'dec_content'`键, 继续往item里添加爬取到的信息, 最后返回`item`, 给pipeline处理.

整个爬取的流程如下

```
parse()page1 --item(title1,link1)--> parse_dec()爬取子页面1 --item(title1,link1,dec1)--> pipelinesc处理
             --item(title2,link2)--> parse_dec()爬取子页面2 --item(title2,link2,dec2)--> pipelinesc处理
             ...
             --item(title9,link9)--> parse_dec()爬取子页面9 --item(title9,link9,dec9)--> pipelinesc处理

parse()进入第二页爬取

parse()page2 --item(title1,link1)--> parse_dec()爬取子页面1 --item(title1,link1,dec1)--> pipelinesc处理
             --item(title2,link2)--> parse_dec()爬取子页面2 --item(title2,link2,dec2)--> pipelinesc处理
             ...
             --item(title9,link9)--> parse_dec()爬取子页面9 --item(title9,link9,dec9)--> pipelinesc处理

...

parse()进入下一页, 找不到信息, 爬虫停止.
```

# pipelines的构造

```python
class MongoPipeline(object):
    # 这个pipeline是保存到数据库中的
    collection_name = 'interns'

    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DATABASE', 'shixiseng_interns')
        )

    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]

    def close_spider(self, spider):
        self.client.close()

    def process_item(self, item, spider):
        # 通过dec_link, 即岗位职责页面的链接来判断是否重复, 若重复就不存入数据库
        if self.db[self.collection_name].find({'dec_link': item['dec_link']}).count() == 0:
            self.db[self.collection_name].insert_one(dict(item))
        return item
```

 - 处理获取的item相对简单, 只有`process_item()`方法是必要的. 新建mongodb数据库名为`shixiseng_interns`, 新建collection名为`interns`
 - 存入数据库之前先查询一下有没有相同链接的文档已经存在了, 如果存在就视为重复, 放弃存储.
 - 通过`insert_one(dict(item)`把`item`存入数据库

# 运行与部署

进入shixiseng根目录下, 运行

```bash
sudo service mongod start
scrapy crawl shixiseng_intern_spider
```

获得大量log信息, 爬取结束之后, 进入数据库查询:

```bash
{
    "_id" : ObjectId("58a7ecbc44cefc2e49838e41"),
    "addr" : "南京",
    "title" : "产品策划（专题活动方向）实习生",
    "closing_date" : "2017-12-31",
    "money" : [
        "70",
        "90"
    ],
    "dec_link" : "http://www.shixiseng.com/intern/inn_2oycacvj6eqw",
    "dec_content" : [
        "岗位职责：",
        "1、负责与市场、运营等部门的线上专题页活动对接",
        "2、通过对公司业务梳理​和专题活动类型分析调研，挖掘需求",
        "3、负责专题页的交互设计",
        "4、组织活动需求方及研发、前端等资源确保产品按时高质交付",
        "岗位职责：",
        "1、负责与市场、运营等部门的线上专题页活动对接",
        "2、通过对公司业务梳理​和专题活动类型分析调研，挖掘需求",
        "3、负责专题页的交互设计",
        "4、组织活动需求方及研发、前端等资源确保产品按时高质交付"
    ],
    "tag" : "软件",
    "company_name" : "课窝教育",
    "job_time" : [
        "02",
        "18 "
    ]
}
```

> 未完待续

Refer:

 - [【scrapy】学习Scrapy入门](http://www.jianshu.com/p/a8aad3bf4dc4)
 - [用scrapy爬取豆瓣电影新片榜](http://aljun.me/post/4)
