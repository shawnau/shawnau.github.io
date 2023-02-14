# bilibili新番/旧番弹幕抓取


主要涉及到Selenium的使用

<!--more-->

源代码:
https://github.com/shawnau/bilibili_scraper

本文中的所有技术在[上一篇文章](http://xxuan.me/2016-07-16-webscraper.html)中均有介绍, 因此本文只涉及到针对bilibili主页结构的分析, 省略了技术细节

## 0. 抓取番剧弹幕的基本步骤
![pageinfo](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/Screen%20Shot%202016-07-17%20at%207.29.41%20PM.png)
b站的每个视频页面源代码中, 负责启动播放器的script标签下保存了弹幕编号(cid)以及视频编号(aid), 例如上图中的视频, 其script内容为
```
EmbedPlayer('player', "http://static.hdslb.com/play.swf", "cid=8628660&aid=5244087&pre_ad=0");
```
找到了cid=8628660后, 前往comment.bilibili.tv下取得xml格式的弹幕, 例如本视频的弹幕地址为: http://comment.bilibili.tv/8628660.xml

幸运的是, b站看视频并不需要登录, 且cid都可以通过直接读取网页源代码并通过正则表达式匹配得到, 不幸的是分p视频的分p链接那一栏大部分需要点击展开按钮才能获得, 因此我们的selenium派上了用场.

综上所述, 我们需要实现以下功能:
 1. 获得网页上分p一栏所有视频的链接
 2. 打开每个链接, 获得cid, 标题等页面信息
 3. 利用cid下载弹幕
 
 除此之外, 爬虫需要一定的容错率, 对找不到cid/难以下载弹幕的视频, 爬虫需要在重试一定次数之后主动跳过

## 1. 旧番站点结构
![old](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/Screen%20Shot%202016-07-17%20at%207.16.38%20PM.png)
打开分p在播放器上方的视频页面, 使用xpath找到id为plist的div标签, 可以发现标签下有三部分: 

 - span标签, 代表当前页面
 - 带有超链接的a标签, 代表其他p的链接和副标题
 - 带有class名为v-part-toggle的a标签, 代表展开按钮

由此可得我们需要的操作:

```python
span = driver.find_elements_by_xpath(p.xPaths["plistButton"])
if len(span) == 0:
            raise Exception('No other page found')
```

利用plist下的span标签检查是否有分p, 如果没有则扔异常, 在异常处理中只将本页面链接添加进链接表中. 对于只有一个链接的视频也同样处理即可

```python
# 检查是否有展开按钮, 有就按一下
unfold_button = driver.find_elements_by_class_name(p.togglename)
if len(unfold_button) == 1:
    unfold_button[0].click()
# 截取展开后的所有链接, 提取源代码
plist = driver.find_element_by_xpath(p.xPaths["plist"])
list_source = plist.get_attribute('innerHTML')
```

首先有展开按钮就先点击展开, 然后再取得链接, 发送给beautifulsoup分析. (当然也可以直接用正则表达式分析)

```python
link_list = []
# 切割完整域名
cut_current_list = re.search("/video/av[0-9]+/", page_url).group(0)
# bs4识别所有链接
bsobj = BeautifulSoup(list_source, "html.parser")
a_list = bsobj.findAll("a", {"href": re.compile("/video/[a-z0-9/_.]+")})
# 保存包括本页面的所有链接
link_list.append(cut_current_list)
for a in a_list:
    link_list.append(str(a["href"]))
return link_list
```
将链接列表保存至 `link_list` , 注意匹配到的是除了本页面以外的链接, 且格式是子域名(/video/av..../), 为了保持格式一致性, 本页面的链接也应该先切割后再保存进列表.

## 2. 新番站点结构
![new](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/Screen%20Shot%202016-07-17%20at%207.52.14%20PM.png)
新番站点结构相对简单, 直接使用xpath搜索到链接表(v_bgm_list_data)并提取源代码, 用beautifulsoup分析后保存即可, 其他步骤和前面类似

## 3. 提取页面cid, 标题等信息

打开每个链接, 直接提取源代码, 使用正则表达式匹配cid和title, 注意标题需要使用utf-8编码, 否则在使用它作为文件名保存的时候会报错

```python
driver.get(page_url)
source_code = driver.page_source
# 源代码中可以找到cid和title, 对应弹幕文件和标题
title_reg = re.compile("(?<=<title>).*(?=</title>)")
cid_reg = re.compile("(?<=cid=)[0-9]+")
p.title = re.search(title_reg, source_code).group(0).encode('utf-8')
p.cid = str(re.search(cid_reg, source_code).group(0))
```

## 4. 利用cid下载弹幕并保存
获得了cid之后, 拼接出完整地址, 直接保存页面源代码为xml文件即可
```
# 拼接网址
comment_url = "http://comment.bilibili.tv/" + p.cid + ".xml"
# 保存源码, 设置30秒超时
driver.set_page_load_timeout(30)
driver.get(comment_url)
source = driver.page_source
# 保存时注意使用utf-8编码
with open("./data/" + p.title + ".xml", 'w') as fd:
    fd.write(source.encode('utf-8'))
```

## 5. 提高容错率

主程序使用对link_list的循环依次打开视频并保存.

取得页面信息和保存弹幕的函数都可以根据自身成功与否返回True或者False, 主程序根据取回情况决定是否重新执行, 执行3次依然失败的话即跳过该链接. 例子如下:
```python
for link in link_list:
    page_url = "http://www.bilibili.com" + str(link)
    # 爬取页面失败重试3次, 再失败则跳过
    cid_flag = get_page_info(driver, page_url, p)
    if not cid_flag:
        for i in range(3):
            cid_flag = get_page_info(driver, page_url, p)
            if cid_flag:
                break
            print("Get cid failed, retrying...")
        if not cid_flag:
            print("Retry times out, skip")
            continue
```

## 6. 小结

通过分析, 我们获得了提取b站弹幕的基本策略, 并使用了简单的容错机制. 还可以完善的地方有:

 - 是否可以下载视频?
 - 如何处理副标题? 
 - 如何处理视频链接? 在保存的弹幕中是否需要体现链接?
 - 是否可以并行下载以提升速度?

感谢阅读!
> Written with [StackEdit](https://stackedit.io/).
