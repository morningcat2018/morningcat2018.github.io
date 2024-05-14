---
layout:     post
title:      搭建本地大预言模型
subtitle:   使用ollama
date:       2024-05-14
author:     BY morningcat
header-img: png/llm/llm.jpeg
catalog: true
tags:
    - LLM
---


## 什么是大预言模型

大语言模型（英语：large language model，LLM）是一种语言模型，由具有许多参数（通常数十亿个权重或更多）的人工神经网络组成，使用自监督学习或半监督学习对大量未标记文本进行训练。大型语言模型在2018年左右出现，并在各种任务中表现出色。

尽管这个术语没有正式的定义，但它通常指的是参数数量在数十亿或更多数量级的深度学习模型。大型语言模型是通用的模型，在广泛的任务中表现出色，而不是针对一项特定任务（例如情感分析、命名实体识别或数学推理）进行训练。

尽管在预测句子中的下一个单词等简单任务上接受过训练，但发现具有足够训练和参数计数的神经语言模型可以捕获人类语言的大部分句法和语义。 此外大型语言模型展示了相当多的关于世界的常识，并且能够在训练期间“记住”大量事实。

虽然 ChatGPT 为代表的LLM在生成类人文本方面表现出了卓越的能力，但它们很容易继承和放大训练数据中存在的偏差。这可能表现为对不同人口统计数据的歪曲表述或不公平待遇，例如基于种族、性别、语言和文化群体的不同观点与态度。

-- [维基百科](https://zh.wikipedia.org/wiki/%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B)

## 什么是ollama

Ollama 是一个便于本地部署和运行大型语言模型（Large Language Models, LLMs）的工具。使用通俗的语言来说，如果你想在自己的电脑上运行如 GPT-3 这样的大型人工智能模型，而不是通过互联网连接到它们，那么 Ollama 是一个实现这一目标的工具。

- 本地运行大型语言模型：Ollama 允许用户在自己的设备上直接运行各种大型语言模型，包括 Llama 2、Mistral、Dolphin Phi 等多种模型。这样用户就可以在没有网络连接的情况下也能使用这些先进的人工智能模型。
- 跨平台支持：Ollama 支持 macOS、Windows（预览版）、Linux 以及 Docker，这使得几乎所有主流操作系统的用户都可以利用这个工具。
- 语言库和第三方库支持：它提供了一个模型库，用户可以从中下载并运行各种模型。此外，也支持通过 ollama-python 和 ollama-js 等库与其他软件集成。
- 快速启动和易于定制：用户只需简单的命令就可以运行模型。对于想要自定义模型的用户，Ollama 也提供了如从 GGUF 导入模型、调整参数和系统消息以及创建自定义提示（prompt）的功能。

网站
- [ollama 官网](https://ollama.com/)
- [ollama Github](https://github.com/ollama/ollama)

## 支持哪些大预言模型

- llama3
    - Meta Llama 3: The most capable openly available LLM to date
- phi3
    - Phi-3 Mini is a 3.8B parameters, lightweight, state-of-the-art open model by Microsoft.
- gemma
    - Gemma is a family of lightweight, state-of-the-art open models built by Google DeepMind. Updated to version 1.1
- [更多](https://ollama.com/library)[若前面网址无法打开可以查看离线文档](pdf/llm/ollama library.pdf)

## 如何下载

- [官网下载](https://ollama.com/download)
- [github 下载](https://github.com/ollama/ollama/releases)

下载正常双击安装即可

测试是否安装成功,命令行输入: ollama

```
Usage:
  ollama [flags]
  ollama [command]

Available Commands:
  serve       Start ollama
  create      Create a model from a Modelfile
  show        Show information for a model
  run         Run a model
  pull        Pull a model from a registry
  push        Push a model to a registry
  list        List models
  cp          Copy a model
  rm          Remove a model
  help        Help about any command

Flags:
  -h, --help      help for ollama
  -v, --version   Show version information

Use "ollama [command] --help" for more information about a command.
```

输出一些如何使用的信息即安装成功

## 什么是 llama3

Meta Llama 3, a family of models developed by Meta Inc. are new state-of-the-art , available in both 8B and 70B parameter sizes (pre-trained or instruction-tuned).

Llama 3 instruction-tuned models are fine-tuned and optimized for dialogue/chat use cases and outperform many of the available open-source chat models on common benchmarks.

[https://ai.meta.com/blog/meta-llama-3/](https://ai.meta.com/blog/meta-llama-3/)

## 以 llama3 为例进行实践

执行命令

> ollama run llama3

首次运行会自动进行下载相关文件

```
➜  ~ ollama run llama3
pulling manifest 
pulling 00e1317cbf74... 100% ▕██████████████████████████████████████████████████████████████████████████████████████████▏ 4.7 GB                         
pulling 4fa551d4f938... 100% ▕██████████████████████████████████████████████████████████████████████████████████████████▏  12 KB                         
pulling 8ab4849b038c... 100% ▕██████████████████████████████████████████████████████████████████████████████████████████▏  254 B                         
pulling 577073ffcc6c... 100% ▕██████████████████████████████████████████████████████████████████████████████████████████▏  110 B                         
pulling ad1518640c43... 100% ▕██████████████████████████████████████████████████████████████████████████████████████████▏  483 B                         
verifying sha256 digest 
writing manifest 
removing any unused layers 
success 
```

当出现 `>>>` 即可与其进行对话了,默认情况下都会以英文的形式进行输出,可以在问题的最后加上`中文回复`输出就支持中文交流了.

```
>>> 你是谁 中文回复
😊我是一个人工智能 chatbot，我的目的是帮助用户解答问题、提供信息和进行交流。我可以通过互联网与用户沟通，回答各种问题，从新闻到娱乐，从技术到生活等多个方面。我是一种机器，但我旨在为人类服务，提高我们的交流效率和质量。😊

>>> /?
Available Commands:
  /set            Set session variables
  /show           Show model information
  /load <model>   Load a session or model
  /save <model>   Save your current session
  /clear          Clear session context
  /bye            Exit
  /?, /help       Help for a command
  /? shortcuts    Help for keyboard shortcuts

Use """ to begin a multi-line message.
```

使用 `/bye` 退出交流

## llama3 还有[其他的模型文件](https://ollama.com/library/llama3/tags)

> ollama run llama3:70b

> ollama run llama3:instruct

> ollama run llama3:text

> ollama run llama3:70b-instruct

> ollama run llama3:70b-text

> ollama run llama3:8b-text

[等等](pdf/llm/llama3 tags.pdf)

## 使用 API 与 llama3 进行对话

```
curl -X POST http://localhost:11434/api/generate -d '{
  "model": "llama3",
  "prompt":"who are you?"
 }'
```

默认情况下是流式输出

```
{"model":"llama3","created_at":"2024-05-14T18:20:32.8543Z","response":"I","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:33.017673Z","response":" am","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:33.194024Z","response":" L","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:33.37874Z","response":"La","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:33.537033Z","response":"MA","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:33.71959Z","response":",","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:33.888875Z","response":" a","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:34.050666Z","response":" large","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:34.209947Z","response":" language","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:34.373938Z","response":" model","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:34.549135Z","response":" trained","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:34.722514Z","response":" by","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:34.888269Z","response":" a","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:35.066635Z","response":" team","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:35.23378Z","response":" of","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:35.40503Z","response":" researcher","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:35.584319Z","response":" at","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:35.765454Z","response":" Meta","done":false}
{"model":"llama3","created_at":"2024-05-14T18:20:35.998841Z","response":" AI","done":false}
...
{"model":"llama3","created_at":"2024-05-14T18:21:07.238123Z","response":"","done":true,"done_reason":"stop","context":[128006,882,128007,198,198,14965,527,499,30,128009,128006,78191,128007,198,198,40,1097,445,8921,4940,11,264,3544,4221,1646,16572,555,264,2128,315,32185,520,16197,15592,13,358,6,76,264,955,315,15592,6319,311,7068,3823,12,4908,1495,3196,389,279,1988,358,5371,13,3092,6156,734,374,311,3619,323,6013,311,5933,4221,11374,304,264,1648,430,6,82,11190,323,23387,13,198,198,40,6,76,15320,6975,323,18899,856,14847,3196,389,279,21633,358,617,449,3932,1093,499,13,358,649,7068,1495,389,264,7029,2134,315,13650,11,505,8198,323,3925,311,16924,323,7829,13,358,649,1524,1893,4113,7493,11,45319,11,477,7402,1157,0,198,198,4599,499,2610,757,4860,477,3493,1988,11,358,1005,430,2038,311,7068,264,2077,430,6,82,6319,311,387,11190,11,39319,11,323,7170,1524,30311,13,3092,5915,374,311,3493,13687,323,9959,11503,1418,1101,1694,23387,323,2523,311,3137,311,13,198,198,4516,11,1148,1053,499,1093,311,6369,922,30,128009],"total_duration":35277839676,"load_duration":4294667,"prompt_eval_count":9,"prompt_eval_duration":887864000,"eval_count":173,"eval_duration":34382909000}
```

若要一次性输出可以添加参数 `"stream": false`

```
curl -X POST http://localhost:11434/api/generate -d '{
  "model": "llama3",
  "prompt":"who are you?",
  "stream": false
 }'
```

不过要等一段时间,个人觉得交互不友好

```
{"model":"llama3","created_at":"2024-05-14T18:25:40.489548Z","response":"I am LLaMA, an AI assistant developed by Meta AI that can understand and respond to human input in a conversational manner. I'm not a human, but a computer program designed to simulate conversation and answer questions to the best of my ability based on my training data.\n\nI'm a large language model trained on a massive corpus of text data, which enables me to generate human-like responses to a wide range of topics and questions. My training data includes a vast amount of text from various sources, including books, articles, research papers, and more.\n\nMy primary function is to provide helpful and accurate information in response to user input, while also being engaging and fun to interact with. I can answer questions, tell jokes, offer advice, and even generate creative content like stories or poems.","done":true,"done_reason":"stop","context":[128006,882,128007,198,198,14965,527,499,30,128009,128006,78191,128007,198,198,40,1097,445,8921,4940,11,459,15592,18328,8040,555,16197,15592,430,649,3619,323,6013,311,3823,1988,304,264,7669,1697,11827,13,358,6,76,539,264,3823,11,719,264,6500,2068,6319,311,38553,10652,323,4320,4860,311,279,1888,315,856,5845,3196,389,856,4967,828,13,198,198,40,6,76,264,3544,4221,1646,16572,389,264,11191,43194,315,1495,828,11,902,20682,757,311,7068,3823,12,4908,14847,311,264,7029,2134,315,13650,323,4860,13,3092,4967,828,5764,264,13057,3392,315,1495,505,5370,8336,11,2737,6603,11,9908,11,3495,16064,11,323,810,13,198,198,5159,6156,734,374,311,3493,11190,323,13687,2038,304,2077,311,1217,1988,11,1418,1101,1694,23387,323,2523,311,16681,449,13,358,649,4320,4860,11,3371,32520,11,3085,9650,11,323,1524,7068,11782,2262,1093,7493,477,45319,13,128009],"total_duration":27654480508,"load_duration":4008600,"prompt_eval_duration":172797000,"eval_count":160,"eval_duration":27476636000}% 
```

[更多API文档资料](https://github.com/ollama/ollama/blob/main/docs/api.md)


## 更多资料

- https://blog.csdn.net/sinat_29950703/article/details/136194337
- https://sspai.com/post/85193
- https://zhuanlan.zhihu.com/p/695040359




