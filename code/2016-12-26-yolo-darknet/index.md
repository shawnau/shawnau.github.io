# Blazing Fast! 使用YOLO+Darknet进行目标检测


YOLO做目标检测很快

<!--more-->

## 系统配置
 - 系统: Ubuntu 16.04 x64
 - 显卡: gtx960m + i5核显
 - 内存: 8G
 - CPU: i5-6300HQ

## 使用独立显卡

1. 安装驱动和切换工具
 ` sudo apt-get install nvidia-364 nvidia-prime`
 - 驱动版本可以到软件更新->其他驱动里查看
 - 安装过程中可能需要关闭图形界面, 按Ctrl + Alt + F1即可, 输入`sudo service lightdm stop` 安装完毕之后再`sudo service lightdm start`

2. 安装完毕之后重启, 启动NVIDIA X Server Settings, 可以在nvidia prime下看到切换显卡的功能

## 安装CUDA 8

1. 去[官网](https://developer.nvidia.com/cuda-downloads)下载安装包并按照指令安装, 注意使用runfile方案
2. 注意其中有一个选项是问是否需要安装自带的驱动(似乎是361), 选择no, 因为第一步已经安装过合适的驱动了
3. 安装完毕之后将目录加入环境变量. 往`~\.bashrc`下添加以下语句
 
 ```
 export PATH=/usr/local/cuda-8.0/bin${PATH:+:${PATH}}
 export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
 ```
 
4. 使用`nvidia-smi`指令测试一下, 应该能看到独立显卡的信息

## 安装opencv 3.1

> 参考[caffe的教程](https://github.com/BVLC/caffe/wiki/OpenCV-3.1-Installation-Guide-on-Ubuntu-16.04)

> 参考[官方文档](http://docs.opencv.org/3.1.0/d7/d9f/tutorial_linux_install.html)

1. 安装依赖

 ```bash
sudo apt-get install --assume-yes build-essential cmake git
sudo apt-get install --assume-yes build-essential pkg-config unzip ffmpeg qtbase5-dev python-dev python3-dev python-numpy python3-numpy
sudo apt-get install --assume-yes libopencv-dev libgtk-3-dev libdc1394-22 libdc1394-22-dev libjpeg-dev libpng12-dev libtiff5-dev libjasper-dev
sudo apt-get install --assume-yes libavcodec-dev libavformat-dev libswscale-dev libxine2-dev libgstreamer0.10-dev libgstreamer-plugins-base0.10-dev
sudo apt-get install --assume-yes libv4l-dev libtbb-dev libfaac-dev libmp3lame-dev libopencore-amrnb-dev libopencore-amrwb-dev libtheora-dev
sudo apt-get install --assume-yes libvorbis-dev libxvidcore-dev v4l-utils
 ```

2. 下载并编译opencv
 - 去[官方git repo](https://github.com/opencv/opencv)下载最新的opencv 3.1源码
 - 解压, 进入目录, 输入以下指令编译
 
 ```
 mkdir build
cd build/
cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local .. 
make -j $(($(nproc) + 1))
 ```
 - 输入以下指令安装
 
 ```
 sudo make install
sudo /bin/bash -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf'
sudo ldconfig
sudo apt-get update
 ```
 - 重启系统

3. 错误解决
 - 编译时可能会出现`/usr/bin/ld: 找不到 -lippicv(ubuntu 16.04 LTS`错误, 是因为CUDA8.0和opencv的兼容问题
 - 进入 `/home/ds/opencv-3.1.0/3rdparty/ippicv/unpack/ippicv_lnx/lib/intel64`目录, `cp liboppicv.a /usr/local/lib` 即可. 参考[这里](http://blog.csdn.net/dengshuai_super/article/details/51895120)

## 安装YOLO

1. 下载并编译darknet

 ```
git clone https://github.com/pjreddie/darknet
cd darknet
make
 ```

2. 下载预训练模型

 ```
 wget http://pjreddie.com/media/files/yolo.weights
 ```

3. 检测

 ```
 ./darknet detect cfg/yolo.cfg yolo.weights data/dog.jpg
 ```

## 使用YOLO进行实时目标检测

1. 编辑`Makefile`, 置`GPU=1`, `make`后再置`OPENCV=1`, `make`, 编译完成
2. 运行以下指令:
 ``` ./darknet detector demo cfg/coco.data cfg/yolo.cfg yolo.weights ```
 将会打开摄像头进行实时目标检测
