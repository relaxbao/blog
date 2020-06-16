# ElasticSearch入门之ElasticSearch是什么？有什么用？怎么学? （第一篇）



## Why

之前在区块链浏览器的开发中，用过ElasticSearch，但只是简单用了一些操作指令，并不是非常理解。因为最近要做一个项目，用ElasticSearch存储日志，所以乘机对ElasticSearch的学习做了一些学习和总结。第一篇就一些小小心得与大家分享。 我一直觉得官方文档，或者一些书，是最好的入门材料来源，所以我就不把所有的部署，入门教程的功能再抄一遍，因为已经有更好的材料了，但我会列举一些，我觉得看了之后，能够对ElasticSearch有个基本的认识，并能够立马上手实践的材料。 我的认识也还比较浅薄，入有认识不当之处，还请大家多多包涵啦~

## What

### ElasticSearch是什么？

说到ElasticSearch，大家就会说，就是搜索引擎嘛。Wikipedia的解释是：**Elasticsearch**是一个基于[Lucene](https://zh.wikipedia.org/wiki/Lucene)库的[搜索引擎](https://zh.wikipedia.org/wiki/搜索引擎)。它提供了一个分布式、支持多租户的[全文搜索](https://zh.wikipedia.org/wiki/全文檢索)引擎，具有[HTTP](https://zh.wikipedia.org/wiki/HTTP) Web接口和无模式[JSON](https://zh.wikipedia.org/wiki/JSON)文档。

光看这句话，没接触过ES的小伙伴可能会有点疑惑，ES是个搜索引擎，那ES算数据库吗？他存不存数据，和MySQL、Redis这种数据库有什么区别？

### 倒排索引

在ES中有个很重要的概念，就是[倒排索引(Inverted index)](https://zh.wikipedia.org/wiki/%E5%80%92%E6%8E%92%E7%B4%A2%E5%BC%95), 倒排索引是一种索引的方法，被用来存储在全文搜索下，某个单词在一个文档或者一组文档中的存储位置的映射。也就是说，我们可以通过某一个单词，找到一个映射（索引），通过这个映射，我们可以找到包含这个单词的一个或一组文档。举个🌰，比如我们在浏览器中，搜索一个关键词，出现很多关联的文档。这就很强大了。从这个我们也可以看到他和MySQL和Redis不一样了，他的核心在于搜索。

## How

那怎么实现ElasticSearch的入门呢，根据多感官，多渠道的原则，我们就看看视频，看看书，看看文章，再写个Demo，应该就能实现入门啦。

### 【视频】开始使用ElasticSearch

首先推荐一个ES的官方视频：[开始使用ElasticSearch](https://www.elastic.co/cn/webinars/getting-started-elasticsearch?baymax=default&elektra=docs&storm=top-video) 视频全长1小时左右，支持倍速观看，非常直观的演示了怎么部署，如何往ES中存取数据，也讲到了分词和高亮。这样就很适合对ES有一个初步的了解

### 【书】ElasticSearch：权威指南

虽然是基于ElasticSearch2.x, 但是整体介绍的很好，里面的基础入门，就可以让我们有一个基本的概念和常用的使用，要想要深入了解，也有深入介绍的部分。

[ElasticSearch:权威指南中文版](https://www.elastic.co/guide/cn/elasticsearch/guide/cn/index.html)

[ElasticSearch：权威指南英文版](https://www.elastic.co/guide/en/elasticsearch/guide/current/index.html)

在基础入门- [你知道的，为了搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/cn/intro.html) 这一章结尾就说了, 如果只是要有初步的概念，上手使用，尝试一下这部分就可以了。

> 现在大家对于通过 Elasticsearch 能够实现什么样的功能、以及上手的难易程度应该已经有了初步概念。Elasticsearch 力图通过最少的知识和配置做到开箱即用。学习 Elasticsearch 的最好方式是投入实践：尽情开始索引和搜索吧！
>
> 然而，对于 Elasticsearch 知道得越多，就越有生产效率。告诉 Elasticsearch 越多的领域知识，就越容易进行结果调优。
>
> 本书的后续内容将帮助你从新手成长为专家，每个章节不仅阐述必要的基础知识，而且包含专家建议。如果刚刚上手，这些建议可能无法立竿见影；但 Elasticsearch 有着合理的默认设置，在无需干预的情况下通常都能工作得很好。当你开始追求毫秒级的性能提升时，随时可以重温这些章节。

#### 关于部署

使用ElasticSearch的功能，最好同时安装一下kibana（kibana可以展示ES中的内容，也可以提供了方便的与ES进行交互的工具）。[Elasticsearch 最新版下载地址](https://www.elastic.co/cn/downloads/elasticsearch) 如果要下载旧版的就在[past release](https://www.elastic.co/cn/downloads/past-releases#elasticsearch) 中下载即可。

关于部署，在官方文档[Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/install-elasticsearch.html) ，有关于下载安装、配置的详细说明，选自己想要尝试的版本下载使用就可以，**注意文档也要看对应的版本。**

## How Good

看完这一些，基本就可以对ElasticSearch有一个基本的了解啦。今天就写这么多了，后续可以更新一下，自己梳理的ElasticSearch常用指令，ELK Stack的使用，使用springboot-data-ElasticSearch调用ES

