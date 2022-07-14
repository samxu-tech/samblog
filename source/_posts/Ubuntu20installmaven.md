---
title: Ubuntu20 安装maven
date: 2022-07-14 11:27:13
tags: maven
categories: 技术
---
cd ~/

mkdir maven

cd maven

wget [https://ftp.yz.yamagata-u.ac.jp/pub/network/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz](https://ftp.yz.yamagata-u.ac.jp/pub/network/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz)

tar zxvf apache-maven-3.6.3-bin.tar.gz


echo "MAVEN_HOME=/home/ubuntu/maven/apache-maven-3.6.3" >> ~/.bashrc

echo "PATH=\${PATH}:\${MAVEN_HOME}/bin" >> ~/.bashrc 

echo "export PATH" >> ~/.bashrc 

source ~/.bashrc


验证

mvn -version

