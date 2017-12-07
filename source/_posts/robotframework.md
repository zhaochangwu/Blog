---
title: Robot Framework 安装
date: 2017-09-15 16:27:13
categories:
- robotframework
tags:
- test
- python
- wxPython
- robotframework
---

# Robot Framework

## 简介
- [官网](http://robotframework.org/)
- [github](https://github.com/robotframework)

## 安装
### 基础环境安装
- 首先需要python2的环境
- 使用`pip`去安装时官方的推荐方式，如果使用的时`python2.7`的版本，应该自带了`pip`，如果没有`pip`，自行百度
### robot framework 安装
#### 通过工具安装

```bash
# 安装robot framework
pip install robotframework
# 安装依赖库
pip install -U requests
pip install -U robotframework-requests
```
> 还有一些安装工具也可以用，`easy_install`等等啊，各位看官自己搞定啊，我就会这一个

#### 从源码安装
鉴于公司的网络问题，说一下怎么从源码安装
1 先将源码下载并解压
2 进入到解压出文件夹内, 确认目录下存在`setup.py`
3 运行安装命令
```bash
python setup.py install
```
> ps: 因为网络问题肯能存在运行命令的时候安装的依赖库下载不下来的问题，所以推荐可以先将依赖库安装了

#### 运行
以下两种方式效果相同
```bash

robot --help
python -m robot --help

```

### RIDE 安装
[`RIDE`](https://github.com/robotframework/RIDE/wiki)是wxpython开发的图形界面，可以方便我们写case, 个人觉得不是很用，同样功能的工具还有，RED(eclipes-java的扩展)，atom、sublime插件等，详细自己查阅[官网](http://robotframework.org/#tools)

这个工具不太推荐 mac 使用，因为依赖`wxpython2.8.12.1`，需要32bit的环境，`mac`装起来比较麻烦

`win`的话，可以直接使用`*.exe`安装, [下载地址](https://sourceforge.net/projects/wxpython/files/wxPython/2.8.12.1/), 我们需要的是`win32 unicoding`版本。

`wxPython`安装好了的话，剩下的就不麻烦了，直接通过源码安装或者使用`pip`安装
```bash
# 安装
pip install robotframework-ride
# 运行
python ride.py
```

## 依赖库
- [requests](https://github.com/requests/requests/)
- [robotframework-requests](https://github.com/bulkan/robotframework-requests)
- [Selenium2Library](https://github.com/robotframework/Selenium2Library)
- [DatabaseLibrary](https://github.com/franz-see/Robotframework-Database-Library)

## 其他问题解决
- [日志打印的中文编码问题](自行百度吧)

## 其他说明

### 背景
这次公司有需求让我做一个 demo，最终环境需要我配置到一台 linux 服务器上

### 遇到问题
1 给我分配的用户没有权限，无法系统内置的 python 目录下的写入权限，所以无法直接通过 `python setup.py install`，`sudo` 也不行
2 公司对网络有限制，及时可以使用 `python setup.py install`， 相关的依赖库也会安装失败

### 解决方案
在自己的用户目录下面安装了一个新的 `python`, 使之与系统的 `python` 共存又独立
[参考](https://github.com/csuldw/EasyNotes/blob/master/chapter2_1-environment.md)

1 先去下载 `python` 版本库 [https://www.python.org/ftp/python/](https://www.python.org/ftp/python/)
2 解压`tar -xzf `，`cd` 进去
3 在你想要安装的路径下建文目录, `mkdir -p ~/python27`
4 `./configure --prefix="~/python27"`
5 `make`
6 `make install`, 等待安装完了其实就可以用了, `~/python27/bin/python setup.py install`
7 设置别名，alias mypython='~/python27/bin/python'，`mypython setup.py install`
8 后面你需要手动安装 python 库的时候就使用 `mypython`,  比方说
```bash
# 运行 ride
$ mypython ride.py
# 运行 robot 命令
$ mypython -m robot --help
```
