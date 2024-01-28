---
title: "写在做大模型应用的一年以后"
date: "2024-01-28T21:20:50+08:00"
author: Spike
layout: post
categories:
  - 人工智能
tags:
  - 人工智能
  - 个人思考
---

几个月前，前同事出差来北京，一起吃了顿饭，聊到我最近一年在做大模型应用，前同事打趣说我真是每次都走在风口上。最初是教育，然后是 web3，再到大模型，是的，每次走在风口上，从来都没有像猪一样飞起来，反而每次都是好巧不巧在盛极一时的时候加入，然后又在一片落寞中离开。但是这次，相比于之前加入教育和 web3 行业，大模型倒是真切地点燃了我的兴趣。

## 我怎么看大模型

### 什么是大模型

我是一个做工程出身的 RD，我没有读研究生，对人工智能基本算上是一个门外汉。第一次接触 gpt3.5 也就是 openAI 发布的时候。包括在这之后一段时间我对大模型的认知还是不足的，印象大概就是他能懂很多知识，能够回答问题，写文章。我相信大部分还没接触或者非软件相关的人也是跟我一样的看法。在我阅读了《AI 3.0》，《这就是 chatgpt》等一些书籍以后，我才算是有了一些基础的认识。

大模型只是一个序列生成模型。抛开 encoder, decoder 之类的专业词汇，大模型其实就是一个序列生成模型，序列生成包含文本生成，音频生成以及视频生成等等。在人类眼中大模型懂知识，懂代码，其实在大模型看来只是不停地完成生成任务而已。所以，从这一点来说，大模型是通过生成而学会了世界的压缩表示。它根据已给出的序列以及经验不断推测下一个 token 的概率，从而完成整个生成任务。所以，生成的内容所含有的意义都是人赋予的，幻觉也是，在大模型看来，只是完成了一次计算任务来生成序列。

大模型就是一个巨大的数学公式。上面我们提到，大模型是通过不断完成生成任务学会了世界上的知识，而这些训练学习过程的结果就会内化到大模型的参数上。大模型的组成单元是神经元，一个神经元包含权重和偏置两个参数，可以表达为 wx + θ，大模型包含了上亿个这样的神经元，这些上亿的神经元按层排列，输入信息通过不同的层逐级处理，向前传递，最终获得输出结果。大家可以类比我们初高中学习的视觉神经成像的原理，整个视网膜从感知光源到大脑成像也经过感知神经元，中枢神经等类似的链路。而这种由单个神经元组成，通过多层处理的方式可以表示为一个复合函数，类似 F(X) = w1x1 + w2x2 + ... + wnxn + b。 这其实就是一个拟合的过程，而大规模的参数对复杂任务能够达到更好的拟合效果。

### 对我们的意义是什么

大模型刚问世的这一年，到处都是惊喜，狂热和焦虑。新技术的出现，特别是是另一个强大的智能体，因为未知感而产生的失业焦虑，以及外部环境的不景气，一种无所适从的感觉自然而然压在每个脑力工作者的头上，甚至已经有人提前感受到了大模型的威胁。而我作为相关从业者也在不断了解的过程中也不断转变着心态，从开始的焦虑，逐渐地惊喜，狂热再到如今的平淡，仿佛现在大模型已经是日常生活的一部分。

生产力变革带来的就业改变是无法避免的事情。有天在徒步的时候，有位律师问我大模型会让律师失业吗，我说汽车肯定会替代马车作为主要交通工具，但是也会创造司机这个工作岗位。是的，汽车的出现，让大部分马夫失业是无法避免的事情，而骑马也会逐渐变成一种爱好或者是偏远地方的交通方式。因为，人工智能生产力的提升带来的生产成本降低，必然导致人工成本失去相对竞争力，而最终投票的，也是你我以一样的消费者。但是，脑力劳动者的被大规模替代并不会在短时间内快速发生，这一方面是当前的大模型能力仍然是受限的，这表现模型的幻觉，稳定性等问题上。同时，不同行业的大模型的渗透速度也是不均匀的，与之类似的一个有意思的现象是日本政府仍然在使用软盘作为存储介质，这在云时代，对很多数字化企业来说都很少见。受限于不同行业对于信息的安全性，准确性要求以及距离科技行业的距离，个人认为相对来说游戏行业和娱乐行业会率先成为 AI 应用大规模落地和推广的行业。

AI 会深度改变交互形式，个人设备会更加个性化和定制化。过去几年看的电影里，《她》真的让我眼前一亮，不仅在于萨曼莎这种先进人工智能给生活带来的便捷性，更在于她基本是为个体完全定制的人工智能，甚至能够和交互主体之间产生感情。我想，在这大模型出现以后都显得触手可及了。首先在交互形式上，我们目前所有使用的应用软件都是程序化的，都是制作者提前设计的功能，不同的应用软件之间有着不同的交互方式和交互入口，而大模型有望统一所有的交互方式和交互入口，即自然语言的交互方式。同时，基于大模型的交互方式也更能够定制化和个性化，智能体具备理解和处理信息，执行动作的能力，现在的 tiktok 等流媒体应用已经让人们感受到了推荐算法的魅力，而这仅仅是单应用领域的推荐，倘若今后你只跟某个人工智能一直交互，那么他会更加了解你的喜好以及习惯。

## 做了哪些东西

过去几个月我一直在做 RAG 和对话机器人相关的领域，这两个领域也是过去一段时间非常火的两个方向。我们做的第一个产品是检索领域信息的文心一言插件，像文心一言这种预训练大模型，会有信息过时，幻觉等问题，而解决这个问题，一种通用的方式即检索增强生成（Retrieval Augmented Generation），本质在于大模型生成内容之前提供给大模型准确的信息。比如，在不借助于 RAG 工具的情况下，chatgpt 和文心一言是无法回答“今天是多少号”，“北京今天多少度”类似的问题的，而借助搜索工具，大模型就能够就能够回答这类需要精确且实时信息的问题。

关于 RAG 插件，我们先是尝试了基于意图识别+查询模板的方案，而这种方案的弊端在于模板开发的成本，无法做到真正的定制化。我们基于数据的种类分成了很多种不同的意图，通过将用户的问句使用 NER 切分成主体词+意图词，根据不同意图对应的数据获取模板来获取到不同的数据，最终交由大模型做内容生成。

在受限于模板的限制性以及意图识别的准确性问题，我们调研和参考了行业内更通用的解决方案--知识图谱。知识图谱这个概念在大模型出现之前就一直存在，其本质是基于人工智能和图数据库而实现的一种高度个性化的数据检索方式。在数据存储上，图数据库相较于关系数据库以及其他 NoSQL 数据库更适合推理且灵活。知识图谱的使用包含知识抽取，知识存储，知识查询等，而查询可按照基于模板，基于语义结构等方式将自然语言转换为图查询语言。在大模型出现之前，无论使用哪种方式将自然语言转换为图查询语言都是需要训练自己的小模型的，而在大模型出现以后，自然语言到图查询语言的转换通过一个 prompt 就能够获得还可以的效果。同时，因为我们领域的数据本身就是高度结构化的数据，所以我们不需要做大量的知识挖掘的工作。

最终，我们搭建了一个涵盖 15 亿点数据，7 亿边数据的知识图谱。我们通过已有的 NER 和检索系统完成查询语句主体消歧的过程，然后基于向量化检索相似查询语句+图查询语句来提高大模型编写图查询语句的准确性。基于知识图谱的查询方案解决了通过属性检索主体以及多跳查询等模板方案无法解决的问题。在查询准确性上也上升到了 90%+。之后我们也打算继续通过基于大模型的 bad case 离线分析系统以及向量化检索策略的优化来提高查询场景和准确率的覆盖率上。

## 收获了哪些认知

大模型这种突然涌现的新技术，特别是像我这种开始不具备人工智能相关经验的 RD 来说，在从 0 到 1 不断摸索的过程走弯路是免不了的，难得的是走弯路的过程中也收获到了一些认知。

### 在实践中获得认知，丰富认知指导实践

过去几个月，各种各样大模型相关的新概念和新方法都在层出不穷地涌现，像 Agent，Plugin，RAG，MoE 之类的新的事物每隔一段时间就会出现，而每一种新的概念或者实践方法都可能意味着更好的效果，相比于以前工程开发，更深切地感受到技术视野对做大模型领域相关研发的巨大价值。我在推上 follow 了几个大模型的大 V，开始收听 Data skeptic，Practical AI 类似的播客以及跟跟同行做一些交流，这些都或多或少在我对这个领域的方向和认知上提供了帮助。而这些信息指导了我知识图谱的转型，研发范式的转换以及 prompt 的调优等等。比如在我了解到大模型是个序列生成模型以及前馈传播的本质之前，我无法了解大模型生成不稳定以及无法执行复杂计算的原因，而在了解大模型的本质和工作原理之后，这让我开始学会使用技巧来控制大模型输出。

然而光收获到认知也是不够的，特别是没经过实践总结和内化的，我过去做工程研发我认为最大的问题就是纸上学来终觉浅，以至于我学习到的认知并没有运用到实践过程中，面试聊的都是分布式一致性各种各样没实践过的方案。大模型作为很新的技术，在很多领域的探索还属于萌芽阶段，在很多方面都没有形成最佳实践的体系，这就需要研发者不停的探索和实践。比如 prompt 的编写和优化上，你很难用比较工程化和系统化的方式去编写 prompt， 目前只有各种各样的技巧，你必须要实践这些技巧。相比于过去，现在我则更善于用一些 demo 和 poc 去快速验证并总结经验，过去几个月我搭过知识图谱，大模型信息抽取等等 demo, 通过 demo 的效果我展示了更有说服力的效果并且内化了认知。

### 重视系统化和机制化

我过去的研发经验中，程序本身的行为是确定性的，这意味着测试只相对较小的测试集和人力投入。但是基于大模型开发的应用和功能继承了大模型概率性的天性，这意味着小样本的测试集很难系统化的评估整体效果，而大规模的测试意味着投入更多的人力。这就要求我们需要改变已有的研发范式，使用更加系统化的评估方式和更加敏捷的研发流程。

系统化意味着有明确的评估标准，更全面且自动化的评估手段以及标准化的评估流程。就我们来说，目前我们一共有 20 多个 prompt，一共有 4，5 个人会优化和管理这些 prompt。在最开始的一段时间，我们的优化和评估流程都是很粗糙的，我们在收到 case 反馈后优化 prompt 都是基于少量 case 进行人工测试的，这耗费了我们大量的时间而且并不能阻止 case 还是不停地出现，最终我们的 prompt 越来越长，充满了各种各样的限制条件。这一部分原因也是评估和优化流程问题导致的。在意识到这个问题后，一方面我协同测试研发落地自动化测试工具以减少端到端测试人力投入，一方面调研和落地了 prompt 相关的评估和管理工具。有了各种各样的工具，更重要的是流程上的标准和规范化，毕竟工具只是减小成本，流程才是对质量的兜底，当前我们还存在流程上的问题，未来预计会引入盲评，准出标准等机制来达到流程规范化。

### 把问题简单化的能力

从 0 到 1 是不容易的，特别是做好 MVP 上，在平衡产品的最终设计与 MVP 上我们算是走了弯路。在开始搭建对话机器人的项目时，我们过多地考虑了长尾需求，导致在产品设计以及技术架构上埋了很多雷，以至于为这种设计不断地打补丁。我后来才意识到，如果在最初我们就只关注于核心功能，那么我们就能取得更好的效果。能够专注于重点，把问题简单化也是一种重要的能力。

### 小心路径依赖，不要一条路走到黑

在这种探索性和不确定性很强的工作中，路径依赖有时候会带来更多的负面效应。在做 RAG 插件的时候，我们最初的意图识别方还是走以前的小模型方案，我们花了大量的时间去准备语料，去标注数据以及训模型，这不仅效率不高而且意图识别的效果短时间也不好。包括我们的对话机器人，我们在一段时间内也没有去探索新的方向和办法，我们不断地在屎山里加上令人生厌的补丁。我想，当一次又一次证明现有方案效果不好时，那就意味着要主动去推翻现在的方案并去探索新的方案了。