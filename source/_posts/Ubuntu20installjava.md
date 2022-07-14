---
title: Ubuntu20 安装 java
date: 2022-07-14 11:28:25
tags: java
categories: 技术
---
1.官网下载

[https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)

2.解压

tar -zxvf jdk-8u111-linux-x64.tar.gz

3.配置(修改 ~/.bashrc)

export JAVA_HOME=/home/ubuntu/java/jdk1.8.0_281

export JRE_HOME=${JAVA_HOME}/jre

export CLASSPATH=.:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar

export PATH=${JAVA_HOME}/bin:$PATH

4.更新配置

source ~/.bashrc

5.验证

java -version





