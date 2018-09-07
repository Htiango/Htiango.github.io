---
title: Maven的配置(Mac)
date: 2017-05-28 22:20:21
tags: [maven,Java,配置,Mac]
categories: [工欲善其事]
---

最近在公司负责一款避孕智能咨询的聊天机器人，是嵌入手淘中的。上周刚刚基于Lucene为机器人添加了搜索引擎，在项目进行的过程中深深领略到了maven的过人之处。
<!-- more -->

maven作为一款Java项目的管理工具，在项目依赖管理上的优点简直完美！只需在pom中添加一小段描述以及配置信息，就可以在项目里调用外包中的函数，岂不美哉，下面简要介绍下Maven的配置和使用。

首先从官网下载最新的maven的binary并进行解压。然后我们需要确定在Mac上已经设置了JAVA_HOME并指向了你的jdk安装位置，没有的话在Terminal中进行如下操作：

```
$ vi ~/.bash_profile
```

添加：

```
export JAVA_8_HOME=your_java8_home
export JAVA_7_HOME=your_java7_home
export JAVA_HOME=$JAVA_8_HOME
export PATH=${PATH}:${JAVA_HOME}/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

此时我们已经添加了JAVA_HOME，现在需要安装maven，同样也是在该文件下进行添加

```
export MAVEN_HOME=your_maven_download_dir
export PATH=${PATH}:${MAVEN_HOME}/bin
```

退出后更新

```
$ source ~/.bash_profile
```

这时输入mvn -v应该可以看到Maven的相关信息

安装完成！

至于maven的使用，墙裂推荐使用JetBrains系列下的IDEA，反正对教育邮箱都是免费的，很赞。我接下来会通过一个简单的项目介绍Lucene，届时也会进一步深入maven的使用。

