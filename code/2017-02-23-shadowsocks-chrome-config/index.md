# shadowsocks+chrome代理配置速查手册


最近老是要配置ss, 顺手记下来以便存档. 为了方便0基础人士阅读极其傻瓜

<!--more-->

参考[官方文档](https://github.com/shadowsocks/shadowsocks/tree/master)

# 1. 服务端(只是使用的话可以跳过这节直接看第二节)

## 1.1 安装

```
apt-get install python-pip
pip install shadowsocks
```

 - 注意pip安装出了错千万别用sudo, 否则会有无数的(权限)火坑等着你. 好好上stackoverflow查

## 1.2 使用配置文件启动方案

配置文件建议写在`~/`下, 可以避免一些潜在的权限问题

```
cd
mkdir shadowsocks
touch ~/shadowsocks/shadowsocks.json # 创建配置文件
touch ~/shadowsocks/ss.log # 创建log(可选, 使用方案2需要)
```

配置文件格式:

```
{
    "server":"my_server_ip",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

后台运行:
```
ssserver -c /etc/shadowsocks.json -d start
ssserver -c /etc/shadowsocks.json -d stop
```

## 1.3 直接后台运行方案

```
nohup ssserver -p 443 -k password -m aes-256-cfb > ~/shadowsocks/ss.log 2>&1 &
```

之所以不使用文档里的后台运行指令是因为会出现一些权限问题. 在腾讯云上只能使用方案2, 在digitalocean上则可以完全按照文档来.

# 2. 客户端使用

## 2.1 windows

> If you want to keep a secret, you must also hide it from yourself.

^ repo上有趣的介绍 23333

### 2.1.1 下载最新版本

[目前的最新版本是3.4, 猛戳下载](https://github.com/shadowsocks/shadowsocks-windows/releases/download/3.4.3/Shadowsocks-3.4.3.zip)

[在这里查看最新版本](https://github.com/shadowsocks/shadowsocks-windows/releases)

解压后只有一个`shadowsocks.exe`, 直接双击启动就行

### 2.2.2 使用

1. 在任务栏找到`Shadowsocks`图标, 一个纸飞机
2. 在`服务器`菜单添加多个服务器. 具体填写内容见下图
3. 选择`启用系统代理`来启用系统代理, 此时如果有浏览器里的代理插件请关闭,或者使用`system proxy`, 否则会出现代理循环

![](http://www.godusevpn.cc/assets/img/wiki/windows-shadowsocks-02.png)

## 2.2 OS X

### 2.2.1 下载最新版本

[目前的最新版本是2.6.3, 猛戳下载](https://github.com/shadowsocks/shadowsocks-iOS/releases/download/2.6.3/ShadowsocksX-2.6.3.dmg)

作者自从被请喝茶之后就没更新了, 可以预见以后也不会了. 安装一路双击就行了

### 2.2.2 使用

1. 在Applications里找到shadowsocksX, 打开, 右上任务栏里会出现一个小纸飞机. 点击纸飞机
2. Servers -> Open Server Preferences -> 点左下的`+`按钮 -> 填写各种配置信息, 填错了可以用`-`按钮删除配置重新填
3. Servers -> 勾上刚刚创建的服务器配置, 如果是第一次配置的话应该已经默认勾选了
3. 勾上 Global Mode
4. Turn Shadowsocks On, 启动
5. Turn Shadowsocks Off, 关闭

# 3. chrome浏览器使用proxyOmega插件连接shadowsocks

Q: 为什么要使用proxyOmega?
A: 简单说proxyOmega可以只让你的浏览器走代理, 系统的其他部分全部使用正常连接, 配置起来方便. 否则就需要使用上一节的方法启用系统代理, 让所有流量都走代理

Q: 我是mac用户, 我用Safari还要看吗
A: 不用了, 乖乖用shadowsocksX就行

## 3.1 安装

[插件传送门](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?hl=en)

直接点击添加到chrome就ok. 国内没法上chrome应用商店怎么办? 呵呵呵翻墙工具在墙外怎么办? 我能怎么办? 我也很绝望啊

## 3.2 配置

点击浏览器右上角那个小圆圈->options->在左边profiles一栏选择proxy

Scheme | Protocol | Server | Port    
-------|----------|--------|------
(default) | SOCKS5 | 127.0.0.1 | 1080

这么填好之后点左下角绿颜色的的Apply Changes就行了

## 3.3 使用

0. 没运行shadowsocks的先运行
1. 点击浏览器右上角那个小圆圈
2. 选择proxy, 进入代理模式
3. 选择direct, 退出代理模式

