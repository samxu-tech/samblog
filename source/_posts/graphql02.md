---
title: 获取 graphql schema 信息
date: 2022-07-14 11:21:22
tags: graphql
categories: 技术
---
## 模块

```plain

npm install -g get-graphql-schema

get-graphql-schema GRAPHQL_URL > schema.graphql
```
## 简单使用

>使用prisma cli
```plain

prisma init appdemo

cd appdmeo

docker-compose up -d

prisma deploy
```
## 使用

```plain
get-graphql-schema http://localhost:4466 > schema.graphql
```
## 用途

```plain
我们在进行graphql-binding 模式拼接以及代码生成的时候比较有用
```
## 参考资料

[https://github.com/prismagraphql/get-graphql-schema](https://github.com/prismagraphql/get-graphql-schema) 

