---
title: GraphQL 的 Queries 和 Mutations
date: 2022-07-14 11:20:18
tags: graphql
categories: 技术
---
官方文档链接：[Queries and Mutations](https://links.jianshu.com/go?to=https%3A%2F%2Fgraphql.org%2Flearn%2Fqueries%2F)

 本文概述的都是最基础的查询语句，具体的建立连接和发送请求的方式可以查看 [Apollo](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.apollographql.com%2F) 相关文档

 


---


 

### Fields (字段)

 

在最简单的情况下，GraphQL 可以查询对象上的指定字段，看下下面这个简单的查询案例

 

```plain
query {
  hero {
    name
  }
}

### REULT ###

{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}
```
 
你可以很清楚地看见，返回的结果和查询的结构是一致的。

 在上面的案例中，就简单获取了 hero 的 name，但是某个查询的字段也可能是对象。下面这个例子就是字段嵌套子对象的情况。

 

```plain
query {
  hero {
    name
    # Queries can have comments!
    friends {
      name
    }
  }
}

### RESULT ###

{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
    ]
}
```
 
GraphQL 的查询语句会遍历字段中所有相近的对象，尽量让客户端在一个请求中获取大量的相关数据，而不是多次请求。

 

### Arguments (参数)

 

在实际的请求过程中，我们都需要请求符合某个条件的指定数据，所以在 GraphQL 中我们也可以将参数传递给字段，就像下面这样：

 

```plain
query {
  human(id: "1000") {
    name
    height
  }
}

### RESULT ###

{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 1.72
    }
  }
}
```
 
### Operation Name (操作名称)

 

上面的例子都是直接使用 query 这个关键字作为查询名称的（官方文档省略了前面的query）但是实际的开发过程中，肯定不希望代码描述太过模糊，所以可以给操作设定名称

 

```plain
query HeroNameAndFriends {
  hero {
    name
    friends {
      name
    }
  }
}
```
 
不光是 query 操作，后续的 mutation 和 subscription 都可以指定操作名称用于区分不同的条件语句。

 

### Variables (变量)

 

光能够指定明确的参数值肯定是不符合开发要求的，很多时候我们一个 query 语句肯定要给不同的参数值去查询的，这个时候就可以使用变量进行传参：

 

```plain
query HeroNameAndFriends($episode: Episode) {
 hero(episode: $episode) {
   name
   friends {
     name
   }
 }
}

--- variables ---
{
 "episode": "JEDI"
}

### RESULT ### 

{
 "data": {
   "hero": {
     "name": "R2-D2",
     "friends": [
       {
         "name": "Luke Skywalker"
       },
       {
         "name": "Han Solo"
       },
       {
         "name": "Leia Organa"
       }
     ]
   }
 }
}
```
 
简单概括一下上面的例子：

 

1. 使用 $variableName 代替静态的查询值
2. 将 $variableName 作为查询的形式参数
3. 传递实参进入 $variableName
 

另外和 JS 的函数传参一样，也可以给个默认的变量值

 

```plain
query HeroNameAndFriends($episode: Episode = JEDI) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```
 
### Aliases (别名)

 

上面所有例子的情况，都有一个默认的前提，那就是服务器端的字段和前端所需的字段都是吻合的。那如果要是前端所需的字段需要换个别名再去渲染那该怎么办呢？下面的例子就能满足这样的需求：

 

```plain
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}

### RESULT ###

{
  "data": {
    "empireHero": {
      "name": "Luke Skywalker"
    },
    "jediHero": {
      "name": "R2-D2"
    }
  }
}
```
 
上面的例子就是，两个 hero 字段被重新换名了然后再返回给客户端。

 

### Fragments (片段)

 

继续上面使用别名的场景，如果客户端给同一数据类型定义了两个不同的别名，然后又恰好不幸的是，查询的数据结构又很复杂，但是我们又不想再写一遍完全一样的数据结构怎么办？下面这个例子就能解决这种问题：

 

```plain
{
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  appearsIn
  friends {
    name
  }
}

### RESULT ###

{
  "data": {
    "leftComparison": {
      "name": "Luke Skywalker",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        },
        {
          "name": "C-3PO"
        },
        {
          "name": "R2-D2"
        }
      ]
    },
    "rightComparison": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```
 
你可以看到上面就是客户端需要查询两派不同的 hero 但是它们的属性基本相同，这个时候我们肯定不希望重复地去写两次类型声明，这个时候我们就可以预设好一个 Fragments 去复用。另外如果你要在 Fragments 里使用变量也是可以的：

 

```plain
query HeroComparison($first: Int = 3) {
 leftComparison: hero(episode: EMPIRE) {
   ...comparisonFields
 }
 rightComparison: hero(episode: JEDI) {
   ...comparisonFields
 }
}

fragment comparisonFields on Character {
 name
 friendsConnection(first: $first) {
   totalCount
   edges {
     node {
       name
     }
   }
 }
}
```
 
### Directives (指令)

 

上面的变量的例子，已经能帮助我们动态地获取想要的后端数据了，但有时候我们还可能需要根据变量来动态地去更改我们的结构。例如，有时候渲染某个 UI 组件需要指定的条件，不符合这个条件我们就不渲染这个 UI 组件。下面这个例子就展示了如何使用 directives 关键字去动态地更改我们需要查询的数据结构：

 

```plain
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}
--- variables ---
{
  "episode": "JEDI",
  "withFriends": false
}

### RESULT ###

{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}
```
 
简单解释一下上面这个例子，当 withFriends 的值为 false 时，该 hero 结构就不需要对应的 friends 属性了。

 

### Mutations (变更)

 

刚刚所有的讨论都是围绕着请求数据，但我们肯定也需要一种能够修改服务器端数据的方法。在 REST 风格的请求方式下，尽管 GET 也能修改服务端数据，但通常情况下我们还是使用 POST  去修改服务端数据。GraphQL 也是类似的逻辑，虽然 query 也可以实现数据的写入，但任何的写入操作都应该用 mutation 关键字。

 

就像 query 一样，mutation 操作也可以返回指定的对象类型。它可以返回更新后的最新状态。样例如下：

 

```plain
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
--- variables ---
{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}

### RESULT ###

{
  "data": {
    "createReview": {
      "stars": 5,
      "commentary": "This is a great movie!"
    }
  }
}
```
 这样我们在更新一个字段的时候，我们就可以在一次请求中更新并获取最新的值。
 另外需要注意的是：**Query 操作是并行的，而 Mutation 操作是串行的**，这就意味着如果你在一次 request 中发送两次 `CreateReviewForEpisode` ，那么第一次一定会在第二次开始之前结束，用以确保不阻塞自己。

 另外，如果你不能确定从服务端返回的是哪种类型的话，你还可以加上 `__typename` 这个字段 (两条下划线)。


 


