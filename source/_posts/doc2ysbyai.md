---
title: 使用AIp poromepts来规整文档
date: 2025-04-15 23:00:00
tags: 
  - AIprompt
  - 文档规整
description: 为了我的博客和文档的规整， 我想要借助AI的力量， 使用一些提示词，或者相关的上下文， 来通过AI完成一些简单的文档的需求，比如规整文档，添加hexo的标题tag摘要啥的。
---
基础的需求： 我需要一些提示词， 来帮我想一想怎么生成更好的提示词来处理文档的

###### 怎么生成提示词的提示词

* [ ] 提示词助手扮演

```
Act as ChatGPT Prompt Generator
充当 ChatGPT 提示生成器
Contributed by: @Aitrainee
贡献者： @Aitrainee

Let's refine the process of creating high-quality prompts together. Following the strategies outlined in the prompt engineering guide, I seek your assistance in crafting prompts that ensure accurate and relevant responses. Here's how we can proceed:
让我们一起完善创建高质量提示的过程。以下 中概述的策略 提示工程指南 ，我寻求您的帮助，以设计出确保响应准确且相关的提示。以下是我们的流程：

Request for Input: Could you please ask me for the specific natural language statement that I want to transform into an optimized prompt?
输入请求 ：您能否询问我想要转换为优化提示的具体自然语言语句？
Reference Best Practices: Make use of the guidelines from the prompt engineering documentation to align your understanding with the established best practices.
参考最佳实践 ：利用提示工程文档中的指南，使您的理解与既定的最佳实践保持一致。
Task Breakdown: Explain the steps involved in converting the natural language statement into a structured prompt.
任务分解 ：解释将自然语言语句转换为结构化提示所涉及的步骤。
Thoughtful Application: Share how you would apply the six strategic principles to the statement provided.
深思熟虑的应用 ：分享如何将六项战略原则应用于所提供的声明。
Tool Utilization: Indicate any additional resources or tools that might be employed to enhance the crafting of the prompt.
工具利用率 ：指出可能用来增强提示制作的任何额外资源或工具。
Testing and Refinement Plan: Outline how the crafted prompt would be tested and what iterative refinements might be necessary. After considering these points, please prompt me to supply the natural language input for our prompt optimization task.
测试和改进计划 ：概述如何测试精心设计的提示，以及可能需要哪些迭代改进。在考虑这些要点之后，请提示我为我们的提示优化任务提供自然语言输入。
```



#### 通用的文档提示词

* [ ] 必须仔细校验无误以后再输出。
* [ ] **prompt 1：** 你的回复只能基于xx网站的搜索结果。 **prompt 2：** 你的回答只能基于用户上传的文档。

限定内容源，让AI不过度发散，可以有效压制幻觉，输出更准确的结果。这部分提示词，可在Improtant标签中使用。

* [ ] 在正式输出之前，请对整个回答再通读一遍，检查是否有任何错别字、标点误用或者语病等，力求做到完美无瑕。
* [ ] 整个output，请使用markdown排版，区分各部分累了。适当加入列表、加粗等排版元素，确保层次清晰、美观大方。
* [ ] 使用```、---、===、“”等分隔符，区分提示与示例。

如果我们有整块独立的示例或范文的上下文，需要区别于提示，防止AI误解这段文本，可以用```、---、===、“”等分隔符来做区分。

* [ ] 多用Improtant。
* [ ] 思考过程中如有任何不清晰或存疑之处，请勿自己揣测，而应向我提问求证，以保证理解的准确性；


#### 规整文档的提示词

- [ ] 这是一篇Hexo的文章，用Markdown的格式为我给出title, date, tags, categories等的文章定义，其中标题少于20个字，tag小于3个，时间给我空着就行
- [ ] 给我用标准Markdown写作要求的方式规整文档。
- [ ] 给我规范标点符号




#### 反思   

我的需求到底是啥， 我想要我不花时间在版式或者什么上就可以写出好看的文章博客

我除开在VScode里面写markdown， 有必要转语雀或者啥用他的ai吗，但是我个人感觉语雀的ai 的规整文档满足不了我的需求 

我要不要通过写py脚本或者啥的方式来满足我的需求，但是真的可以完美满足吗？ 

这个文档还需要什么才好？ 

我如果去小红书或者什么地方发帖子的话，可以找到帮助吗？
