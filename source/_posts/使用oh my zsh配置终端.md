---
title: 使用「oh my zsh」配置MAC终端
date: 2016-07-10 16:27:13
thumbnail: /images/OhMyZsh/OhMyZsh.png
categories:
- MAC
tags:
- MAC
- 终端
- Shell
---
# 使用「oh my zsh」配置MAC终端
首先什么是oh-my-zsh, 说实话, 我也不是特别明白, 我过我看同事的MAC终端都用这个配置了, 我就回来学习配置了一下.

![ohmyzsh](/images/OhMyZsh/OhMyZsh.png)

最初我的目的很单纯. 只是为了我的mac有个好看的主题(大家请不要嘲笑我). 结果学习下发现, 除了主题S, 这个东西还有很多牛逼的功能(不管你怎么认为, 反正我觉得很牛逼).

## 功能简介
好了, 废话有点多, 下面简单介绍下「oh my zsh」能干啥
- 更换终端主题
- 给一些终端命令设置别名
- 提供了丰富的插件, 具体插件的功能需要大家自行研究

## 安装
#### 自动安装
自动安装很简单, 只要在终端中执行下面两个命令中的任意一句就行
```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

```
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```
#### 手动安装
1. 首先使用下面的命令, 从`github`将代码先clone下来
```
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
```

2. 这一步是可选的, 备份你存在的`~/.zshrc`文件. 这个文件是配置主题, 插件, 和命令别名的文件, 还有其他一些设置
```
cp ~/.zshrc ~/.zshrc.orig
```

3. 创建一个新的`~/.zshrc`文件
```
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```

4. 将默认的`shell`切换过来
```
chsh -s /bin/zsh
```
5. 完成上面的步骤之后, 重启终端就可以看到效果了

我是用的自动安装, 手动安装有什么的问题的话自行解决啊:grimacing:

## 配置
安装完成后就是根据自己的喜好进行配置, 下面说的配置都是修改`~/.zshrc`文件来完成的, 关于**自定义主题插件**等如何使用这里先**不**做介绍
#### 主题
- 找到`ZSH_THEME`(我的是在第8行), 默认的是`ZSH_THEME="robbyrussell"`, 只要把引号里换成你自己喜欢的主题就好了
- 如果你不想任何主题生效, 只要写成`ZSH_THEME=""`就行了
- 主题可以在[GitHub上的themes文件夹](https://github.com/robbyrussell/oh-my-zsh/tree/master/themes)看到
- 也可以在本地的`~/.oh-my-zsh/themes`文件夹中看到

#### 插件
- 找到`ZSH_THEME`(我的是在第52行), `plugins=()`在括号中填写你需要的插件就可以了.
- 有多个插件可以用`plugins=(rails git textmate ruby lighthouse)`的形式
- 插件可以在[GitHub上的plugins文件夹](https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins)看到
- 也可以在本地的`~/.oh-my-zsh/plugins`文件夹中看到
- 这里有官方给出的[插件简介](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins-Overview)

> 提示: 加太多的插件是会变慢的

#### 设置别名
```
alias zshconfig="mate ~/.zshrc"
alias ohmyzsh="mate ~/.oh-my-zsh"
```

#### 其他
还有很多其他的功能和配置, 大家自行查找:grimacing:

## 参考文档
- 这篇博客介绍了什么是Zsh, 以及「oh my zsh」的使用配置
[终极 Shell——ZSH](https://zhuanlan.zhihu.com/p/19556676)
- 「oh my zsh」的 [官网](http://ohmyz.sh/)
- 「oh my zsh」的 [GitHub地址](https://github.com/robbyrussell/oh-my-zsh)
- 「oh my zsh」的 [wiki](https://github.com/robbyrussell/oh-my-zsh/wiki)
