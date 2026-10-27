---
title: "Mac切换java8和java11踩的坑"
date: 2026-10-27
description: ""
hiddenInHomeList: false
categories:
  - IT
tags:
  - IT
---

# Mac切换java8和java11踩的坑

> 记录时间：2024年04月24日
>
> 背景：以前不用java11，java环境一直都是java8，今天用的工具是必须jdk11才可使用，根据网上的文章踩了一些坑

本机机器：macOs Monterey12.2.1

shell：zsh

安装jdk11建议使用国内镜像下载（自行搜索），不建议去oracle官网下载，要注册账号的同时下载速度很慢。

配置了文件之后必须source ~/.zshrc和 source ~/.bash_profile

<sub>/.zshrc文件和</sub>/.bash_profile文件同时存在以下代码

export JAVA_HOME=$JAVA_8_HOME
export JAVA_8_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home
export JAVA_11_HOME=/Library/Java/JavaVirtualMachines/jdk-11.0.2.jdk/Contents/Home

alias jdk8="export JAVA_HOME=$JAVA_8_HOME"
alias jdk11="export JAVA_HOME=$JAVA_11_HOME"

![image-20240424125506959](assets/image-20240424125506959-20260515210026-nvt44jb.png)

java -version才会输出

java version "1.8.0_202"

Java(TM) SE Runtime Environment (build 1.8.0_202-b08)

Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)

删除~/.bash_profile的java配置，zshrc配置如下图所示，java版本输出为java11

![image-20240424130202791](assets/image-20240424130202791-20260515210027-384n25p.png)

zshrc删除了source ~/.bash_profile，java版本输出为java11

![image-20240424130501881](assets/image-20240424130501881-20260515210027-i46zwxu.png)

jdk8能切换，说明zshrc配置文件是能用的，但为什么设置的export JAVA_HOME=$JAVA_8_HOME没用呢？要~/.zshrc文件和~/.bash_profile文件同时存在export JAVA_HOME=$JAVA_8_HOME才有用？

![image-20240424130729380](assets/image-20240424130729380-20260515210027-nj6sfuh.png)

结果

**经测试~/.zshrc文件和~/.bash_profile文件同时存在以下代码**

export JAVA_HOME=$JAVA_8_HOME
export JAVA_8_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home
export JAVA_11_HOME=/Library/Java/JavaVirtualMachines/jdk-11.0.2.jdk/Contents/Home

alias jdk8="export JAVA_HOME=$JAVA_8_HOME"
alias jdk11="export JAVA_HOME=$JAVA_11_HOME"

**且~/.zshrc文件必须执行source ~/.bash_profile命令终端的java版本才可输出默认值java8**

source ~/.bash_profile
export JAVA_HOME=$JAVA_8_HOME
export JAVA_8_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home
export JAVA_11_HOME=/Library/Java/JavaVirtualMachines/jdk-11.0.2.jdk/Contents/Home

alias jdk8="export JAVA_HOME=$JAVA_8_HOME"
alias jdk11="export JAVA_HOME=$JAVA_11_HOME"

还有一些设想，是不是<sub>/.zshrc文件和</sub>/.bash_profile文件必须存在export JAVA_HOME=$JAVA_8_HOME等，java默认配置是跟java版本的高低有关吗，具体就不一个一个实验了，因为解决这个问题花费了我2-3个小时。
