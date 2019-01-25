---
title: GraphQL简介
date: 2019-01-26T16:00:00+08:00
slug: graphql-introduction
tags: ["GraphQL", "API", "技术分享"]
categories: ["技术分享"]
---

# API的问题

随着前/后端的分离，前端与后端关于API的争吵也越来越多。前端希望一次请求可以取到页面上所有的数据，后端坚持REST原则，每个接口只负责某个模型的相关数据。前端有时只需要一些用户的昵称列表，但是后端返回都是标准的全量数据。前端经常抱怨展示一个页面数据，需要多次请求，还需要处理网络问题导致其中一个请求失败情况。

为了解决这个问题，后端不得不出一些组合接口，比如可以使用API Gateway等工具。但发现每个端的需要的接口数据又不同，Web端与APP有时产品逻辑也不太一样，所以又不得不开发两套。

互联网的产品迭代速度又很快，APP与Web端页面变化也很多，这时后端需要经常修改API，添加字段或者出新版本API。

# GraphQL的出现与历史

GraphQL刚好很完美的解决了上面提到的问题，之前有个做过Facebook应用开发的朋友给我说过，Facebook的API非常好用，可以定制自己需要的字段，需要什么返回什么，有一个可以实时调试API的工具，那会Facebook还没有把GraphQL开源，我还仿照着简单实现了一部分功能。比如可以指定返回一个接口的部分字段，通过哪个字段排序等。但是没有实现自定义组合接口，比如想一次取得一篇文章的内容，作者信息，最近评论人的昵称头像信息。

Facebook 的移动应用从 2012 年就开始使用 GraphQL。GraphQL 规范于 2015 年开源，现已经支持多种主流的开发语言，GitHub、Shopify、Twitter、Yelp等大厂商已用于生产环境。

![GraphQL Customs](https://magnet-file.qn.cichang.net/yuanping/blog/graphql-users.png)

GraphQL的出现不仅仅是针对开发人员的，Facebook在2012年开始在其native mobile apps中使用GraphQL。但有趣的是GraphQL大部分都是在web技术的背景下使用的，并且在native mobile 领域中只得到很少的支持。 Facebook第一次公开谈论GraphQL是在宣布开源计划后不久的2015年React峰会的时候。GraphQL开源之后有了一个快速增长的社区 ,事实上GraphQL是一种技术，可以在客户端与API通信的任何地方使用。

# 什么是GraphQL

GraphQL是一个API的标准，一种用于 API 的查询语言。

GraphQL 既是一种用于 API 的查询语言也是一个满足你运行时的数据查询。 GraphQL 对你的 API 中的数据提供了一套易于理解的完整描述，使得客户端能够准确地获得它需要的数据，而且没有任何冗余，也让 API 更容易地随着时间推移而演进，还能用于构建强大的开发者工具。

# RESTful vs GraphQL

GraphQL目的在于构建使Client更加易用的服务，可以说GraphQL是更好的RESTful设计，很多人称是下一代的REST。在过去的十多年中，REST已经成为设计web API的标准（虽然只是一个模糊的标准），并且非常流行。它提供了一些很棒的想法，比如无状态服务器和结构化的资源访问。然而REST表现得过于僵化，无法跟上访问它们的客户的快速变化的需求。 GraphQL的开发是为了应付更多的灵活性和效率，它解决了与REST交互时开发人员所经历的许多缺点和低效之处。

为了说明在从API获取数据时REST和GraphQL之间的主要区别，让我们考虑一个简单的示例场景：在blog应用程序中，应用程序需要显示特定用户的文章的标题。同一屏幕还显示该用户最后3个关注者的名称。REST和GraphQL如何解决这种情况?

使用REST API来现实时，我们通常可以通过访问多次请求来收集数据。比如在这个示例中，我们可以通过下面的三步来实现：

1. 通过 /user/:id 获取初始用户数据

2. 通过/user/:id/posts 返回用户的所有帖子

3. 请求/user/:id/followers，返回每个用户的关注者列表

![RESTful Timeline](https://magnet-file.qn.cichang.net/yuanping/blog/graphql-rest.png)

如果用GraphQL的话，我们只需要一次请求就可以完成上述的需求

![GraphQL Timeline](https://magnet-file.qn.cichang.net/yuanping/blog/graphql-example.png)

可以看到，使用GraphQL，我们可以取到刚刚好的数据，不多也不少。

# GraphQL前/后端可同时开发

GraphQL使用强大的Type System来定义API的功能。所有在API中公开的类型都是使用GraphQL schema Definition Language (SDL)在模式中编写的。该模式充当客户端和服务器之间的契约，以定义客户机如何访问数据。 一旦定义了模式，在前端和后端工作的团队就可以同时进行开发。 前端团队可以通过mock所需的数据结构来轻松测试他们的应用程序。一旦后端API实现完成，就可以对客户端应用程序进行切换来调用实际的API获取数据，这也可以使得我们实现更好的客户端和服务端的分离。