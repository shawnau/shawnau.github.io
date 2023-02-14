# nginx + uwsgi + flask部署应用


主要参考了这篇文章
[How To Serve Flask Applications with uWSGI and Nginx on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uwsgi-and-nginx-on-ubuntu-14-04)

<!--more-->

## 安装nginx + uwsgi + flask

```bash
sudo apt-get update
sudo apt-get install python-pip python-dev nginx
```

启动virtualenv, 以下操作都在virtualenv中进行

```bash
pip install uwsgi flask
```

## 下载flask应用代码

 `git clone your_flask_git_repo`


## 在repo下新建`wsgi.py`, 内容如下:

 ```python
 from visualization import app as application
 
 if __name__ == "__main__":
     application.run()
 ```
 注: 把app换成application否则可能会出错

## 在repo下新建uWSGI配置文件`visualization.ini`(文件名任意), 内容如下:

 ```
 [uwsgi]
 module = wsgi
 
 master = true
 processes = 5
 
 socket = myproject.sock
 chmod-socket = 660
 vacuum = true
 
 die-on-term = true
 ```
 注: 关键是socket, 这是和nginx沟通的unix端口

## 配置nginx

 `sudo vim /etc/nginx/sites-available/myproject`

 ```
server {
    listen 80;
    server_name server_domain_or_IP;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/home/user/myproject/myproject.sock;
    }
}
 ```
 注: 
  - `server_domain_or_IP`需要填写服务器ip
  - `unix:/home/user/myproject/myproject.sock;`段需要填写之前`visualization.ini`中的sock文件绝对地址

## 启动nginx服务器
 - 先建立软链接: `sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled`
 - 检查conf文件的正确性: `sudo nginx -t`
 - 启动: `sudo service nginx restart`
 - 此时访问服务器的80端口会看到502错误, 因为uWSGI还没启动

## 启动uWSGI服务
 ```
 uwsgi --ini <ini文件的绝对地址> --daemonize <log文件的绝对地址>
 ```
  - 在后台运行, 会将log输出到规定的文件
  - 再刷新一下地址就可以看到结果了

## 一些坑

 - falsk中的APScheduler貌似不好用, 最后自己使用丑陋的方法写了个独立的py文件处理生成的json, 再用crontab定时运行
 - 每次重启服务器没法启动uWSGI, 参考文章的配置文件似乎因为权限问题没有启动

> 待续
