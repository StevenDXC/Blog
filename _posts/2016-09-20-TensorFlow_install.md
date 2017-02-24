---
layout: post
title: "TensorFlow安装"
subtitle: ""
date: 2016-09-12 14:00:00
author: "steven"
catalog: true
tags:
    - AI
---

在Mac上安装TensorFlow有以下种方式：

* virtualenv
* pip “native”
* Docker
* 从源码安装
* Anaconda

其中 virtualenv和Anaconda其实都是使用Pip来下载安装的。

## 从源码安装
---

* 由于Windows上无法编译Tensorflow，所以暂时无法在Windows上通过源码安装。
   
1.clone源码   

```shell
git clone https://github.com/tensorflow/tensorflow 
```

clone完成之后选择版本

```shell
cd tensorflow
git checkout Branch  #选择版本，如r1.0
```

2.准备编译环境

需要安装以下编译工具：

* bazel
* TensorFlow Python dependencies
* NVIDIA packages to support TensorFlow for GPU(如要运行支持GPU版本的Tensorflow)

bazel：

安装Homebrew

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

使用Homebrew安装bazel

```shell
brew install bazel
```

TensorFlow Python dependencies：

依赖的python工具有：

* six
* numpy
* wheel

```shell
sudo pip install six numpy wheel
```

若使用GPU版本，需要安装coreutils：

```shell
brew install coreutils
```

3.配置安装环境

TensorFlow根文件下有个configure文件，这个脚本配置编译时依赖工具的路径和其他环境配置。
例子：

```shell
$ cd tensorflow  # cd to the top-level directory created
$ ./configure
Please specify the location of python. [Default is /usr/bin/python]: /usr/bin/python2.7
Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native]:
Do you wish to use jemalloc as the malloc implementation? [Y/n]
jemalloc enabled
Do you wish to build TensorFlow with Google Cloud Platform support? [y/N]
No Google Cloud Platform support will be enabled for TensorFlow
Do you wish to build TensorFlow with Hadoop File System support? [y/N]
No Hadoop File System support will be enabled for TensorFlow
Do you wish to build TensorFlow with the XLA just-in-time compiler (experimental)? [y/N]
No XLA JIT support will be enabled for TensorFlow
Found possible Python library paths:
  /usr/local/lib/python2.7/dist-packages
  /usr/lib/python2.7/dist-packages
Please input the desired Python library path to use.  Default is [/usr/local/lib/python2.7/dist-packages]
Using python library path: /usr/local/lib/python2.7/dist-packages
Do you wish to build TensorFlow with OpenCL support? [y/N] N
No OpenCL support will be enabled for TensorFlow
Do you wish to build TensorFlow with CUDA support? [y/N] Y
CUDA support will be enabled for TensorFlow
Please specify which gcc should be used by nvcc as the host compiler. [Default is /usr/bin/gcc]:
Please specify the Cuda SDK version you want to use, e.g. 7.0. [Leave empty to use system default]: 8.0
Please specify the location where CUDA 8.0 toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]:
Please specify the cuDNN version you want to use. [Leave empty to use system default]: 5
Please specify the location where cuDNN 5 library is installed. Refer to README.md for more details. [Default is /usr/local/cuda]:
Please specify a list of comma-separated Cuda compute capabilities you want to build with.
You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus.
Please note that each additional compute capability significantly increases your build time and binary size.
[Default is: "3.5,5.2"]: 3.0
Setting up Cuda include
Setting up Cuda lib
Setting up Cuda bin
Setting up Cuda nvvm
Setting up CUPTI include
Setting up CUPTI lib64
Configuration finished
```

4.编译

编译CPU版本：

```shell
bazel build --config=opt //tensorflow/tools/pip_package:build_pip_package
```

GPU版本：

```shell
bazel build --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package 
```

安装pip：

```shell
sudo pip install /tmp/tensorflow_pkg/tensorflow-1.0.0-py2-none-any.whl
```


## 使用Docker安装

目前只能在mac上通过Docker安装使用CPU的TensorFlow,暂不支持安装GPU版本：

* 先去官网下载安装Docker应用 https://www.docker.com/products/docker#/mac
* 安装完成之后运行docker,启动虚拟镜像
* 运行终端输入命令：

```shell
docker run -it -p 8888:8888 gcr.io/tensorflow/tensorflow
```

docker会自动下载tensorflow镜像并安装执行。国内需要翻墙

## 使用virtualenv安装

1.需要先安装pip(Pip是一个python的软件包管理系统，可以安装，卸载和管理软件包)

```shell
sudo easy_install pip                     #安装Pip
sudo pip install --upgrade virtualenv     #安装virtualenv
```

2.创建virtualenv运行环境：

```shell
virtualenv --system-site-packages targetDirectory
```

targetDirectory：指定项目的目录，一般是 ~/tensorflow

3.激活virtualenv环境

```shell
source ~/tensorflow/bin/activate
source ~/tensorflow/bin/activate.csh   #使用csh或tcsh时
```

若Pip版本>= 8.1，激活的时候下面的命令可能会出行错误：

```shell
pip install --upgrade tensorflow      # for Python 2.7
pip3 install --upgrade tensorflow     # for Python 3.n
pip install --upgrade tensorflow-gpu  # for Python 2.7 and GPU
pip3 install --upgrade tensorflow-gpu # for Python 3.n and GPU
```

若出现错误需要先执行以下命令:

```shell
pip install --upgrade TF_BINARY_URL   # Python 2.7
pip3 install --upgrade TF_BINARY_URL  # Python 3.N
```

TF_BINARY_URL：定义了TensorFlow python安装包的URL，通过系统的版本，python的版本和是否支持GPU来找到正确的URL，如在Python 3.4环境下安装cpu版本的命令为：
```shell
pip3 install --upgrade \
 https://storage.googleapis.com/tensorflow/mac/cpu/tensorflow-1.0.0-py3-none-any.whl
```

卸载TensorFlow:

```shell
rm -r ~/tensorflow
```

## 使用native pip安装

也需要先安装Pip，如上，不再赘述。

安装和使用virtualenv的步骤类似：

```shell
pip install --upgrade tensorflow      # for Python 2.7
pip3 install --upgrade tensorflow     # for Python 3.n
pip install --upgrade tensorflow-gpu  # for Python 2.7 and GPU
pip3 install --upgrade tensorflow-gpu # for Python 3.n and GPU
```

卸载：

```shell
pip uninstall tensorflow
pip3 uninstall tensorflow 
```

## 使用Anaconda安装

Anaconda是一个用于科学计算的Python发行版，提供了包管理与环境管理的功能，可以很方便地解决多版本python并存、切换以及各种第三方包安装问题。

1.到官网下载Anaconda(https://www.continuum.io/downloads),下载Command Line Installer版本.

2.打开终端输入命令安装：
```shell
bash Anaconda2-4.3.0-MacOSX-x86_64.sh   #python2.7
bash Anaconda3-4.3.0-MacOSX-x86_64.sh   #python 3.x 
```
3.创建执行环境
```shell
 conda create -n tensorflow
```
4.激活TensorFlow
```shell
source activate tensorflow
(tensorflow)$  # Your prompt should change
```
执行时出行错误的时候使用下面的命令安装：

```shell
pip install --ignore-installed --upgrade $TF_PYTHON_URL 
```

TF_PYTHON_URL：参考使用virtualenv安装步骤中的说明


## 验证安装

1.打开终端输入

```shell
python      #若同时安装了python2和python3，请确认当前的Python版本
```

2.输入以下文本
```shell
>>> import tensorflow as tf
>>> hello = tf.constant('Hello, TensorFlow!')
>>> sess = tf.Session()
>>> print(sess.run(hello))
```

3.执行之后输出以下消息即安装成功：
```shell
Hello, TensorFlow!
```
