# 在mac os 10.12 Sierra下安装caffe


RT

<!--more-->

#### 1. 配置

 - Mac OS Sierra 10.12.1

---
#### 2.  安装

 - 安装CUDA8 (略)

 ```bash
 brew install -vd snappy leveldb gflags glog szip lmdb
 brew tap homebrew/science
 brew install hdf5 opencv
 ```
 - `brew edit opencv`, 改变如下两行:
 
 ```bash
   -DPYTHON_LIBRARY=#{py_prefix}/lib/libpython2.7.dylib
   -DPYTHON_INCLUDE_DIR=#{py_prefix}/include/python2.7
 ```
 - 安装protobuf和boost-python
 
 ```bash
 brew install --build-from-source --with-python -vd protobuf
 brew install --build-from-source -vd boost boost-python
 ```
 - 下载[NVIDIA Mac drivers](http://www.nvidia.com/object/macosx-cuda-8.0.51-driver.html)

---

 - 下载caffe

 ```bash
 cd ~/Documents
git clone https://github.com/BVLC/caffe.git
 ```

 - 打开Makefile.config, 配置BLAS

 ```
BLAS_INCLUDE := /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.12.sdk/System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/Headers
BLAS_LIB := /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A
 ```
 - 去掉CPU_ONLY的注释
 - 去掉USE_LEVELDB := 0的注释

#### 注:以下这两步似乎已经解决
---

 - 下载XCode Command Line Tools for 7.3, 因为NVIDIA 暂时还不支持Xcode 8.0的. 切换到7.3的tools:
 ```bash
 sudo xcode-select --switch /Library/Developer/CommandLineTools
 ```

 - 待会还要删掉它并使用以下语句切回来, 不然你会用不了brew
 ```bash
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
 ```

---

#### 3. 编译

```bash
make all
make test
make runtest
make pycaffe
make pytest
```
除了一些warning, 似乎并没问题. 但最后一步`make pytest`出现了如下问题:
```
/bin/sh: line 1: 67818 Segmentation fault: 11  python -m unittest discover -s caffe/test
make: *** [pytest] Error 139
```

似乎是anaconda的protobuf版本问题, 这里需要一个有效hack, - (详情见这里)[https://github.com/BVLC/caffe/issues/2092#issuecomment-153986008]
 - 注释掉/python/caffe/proto/caffe_pb2.py 所有 "systax = proto " 的行, 问题解决


---

#### 4. 测试样例

```bash
# 下载测试集
cd data/ilsvrc12/
./get_ilsvrc_aux.sh
# 下载模型
cd ../../models/bvlc_reference_caffenet/
curl  http://dl.caffe.berkeleyvision.org/bvlc_reference_caffenet.caffemodel > bvlc_reference_caffenet.caffemodel
```
这200多m的文件我们还是用迅雷吧.然后运行下. 回到caffe根目录,执行

```
./build/examples/cpp_classification/classification.bin models/bvlc_reference_caffenet/deploy.prototxt models/bvlc_reference_caffenet/bvlc_reference_caffenet.caffemodel data/ilsvrc12/imagenet_mean.binaryproto data/ilsvrc12/synset_words.txt examples/images/cat.jpg
```
![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/2016-12-11)

参考资料
https://gist.github.com/doctorpangloss/f8463bddce2a91b949639522ea1dcbe4
http://caffe.berkeleyvision.org/install_osx.html
http://caffe.berkeleyvision.org/install_osx.html
https://github.com/BVLC/caffe/issues/3608

> Written with [StackEdit](https://stackedit.io/).
