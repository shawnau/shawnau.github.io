# Github Page + Hexo + next主题 blog网站的搭建


简单介绍一下本站的建立过程

<!--more-->

## 部署Hexo
系统: Ubuntu 16.04 LTS

### 0. 安装Git(略)

### 1. 安装 Node.js
最好使用nvm来安装Node.js

 - 方法一: cURL:

 ```
 $ curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | sh
 ```

 - 方法二: Wget:

 ```
 $ wget -qO- https://raw.githubusercontent.com/creationix/nvm/master/install.sh | sh
 ```

安装完毕之后重启终端, 执行:

```
$ nvm install stable
```


### 2. 安装Hexo并建站
1. 安装hexo. [官方文档](https://hexo.io/docs/)

 ```
 $ npm install -g hexo-cli
 ```
2. 初始化blog目录, 目录名为`<folder>`, 可以任选

 ```
 $ hexo init <folder>
 $ cd <folder>
 $ npm install
 ```
3. 这样建站部分就完毕了, 可以看到如下的目录结构
 
 ```
 .
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
 ```
4. 简单介绍一下主要部分的作用
 - `_config.yml`: 站点配置文件, 所有主要配置都需要在这里说明
 - `source/_posts`: 所有写好的markdown文件都会保存在这里
 - `themes`: blog主题存放地址, 一般使用git来对主题进行版本控制

## 安装next主题
 - [项目主页](http://theme-next.iissnan.com)
 - **安装**: 首先确保在blog的根目录`<folder>`下, 执行:

```
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```
 - **使用**: 打开站点配置文件`_config.yml`, 更改`theme`关键字的值为`next`, 修改后显示为: `theme: next`
 - **配置**: 在`themes/next`目录下也有一个叫`_config.yml`的文件, 为了与站点配置文件区分开来, 称之为主题配置文件. 有关主题配置的详细信息见项目主页

## 部署到github上
1. 除了部署到github, hexo还支持Heroku, Rsync, OpenShift等平台 详见[官方文档](https://hexo.io/docs/deployment.html), 本文只介绍怎样部署到github上
2. 为了将hexo生成的静态网页及时部署到github上, 需要以下配置:

 - 安装`hexo-deployer-git`: 
 - 首先确保在blog的根目录`<folder>`下, 执行:  
   `$ npm install hexo-deployer-git --save`
 - 修改站点配置文件的以下关键词:

```
deploy:
  type: git
  repo: <repository url>
  branch: [branch]
  message: [message]
```    

 - `repo`是github上的repo地址, 一般使用`<你的github用户名>.github.io`这样的格式
 - `branch`是分支, 一般选择master
 - `message `是每次commit的时候默认填写的信息,可以注释掉他, 使用默认格式

## 使用

首先确保在blog的根目录`<folder>`下

1. 写文章: `$ hexo new helloworld`, 此时会在`source/_posts/`下生成一个文件名为`helloworld.md`的文档, 直接编辑该文档即可
2. 渲染: `$ hexo g`, 此时会生成渲染好的文章, 在`public`目录下
3. 测试: `$ hexo s`, 此时打开`http://localhost:4000` 可以预览效果
4. 部署: `$ hexo d`, 会将文章部署到github上
5. 绑定域名(只需要执行一次): 在repo的setting下启用GitHub Pages即可

## Tricks
未完待续


