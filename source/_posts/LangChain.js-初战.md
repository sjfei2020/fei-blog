---
title: LangChain.js初战
date: 2024-02-01
updated: 2024-02-01
---

### 阅读前须知

本人接触LangChain.js的时间不长，大概也就几天时间，光是跑通MVP就花了我半天，于是想着记录下，如果你也跑不通MVP，可以参考下，祝你成功。

想体验完整的功能，现阶段确实是 python 版本最为合适，但是考虑到很多人只是想体验下，所以用JS 版本的其实也不少，完全可以先建立下概念，感兴趣的话继续深入，祝好。

### 例子：最简单的对话

#### 环境准备

- node:20.10.0（版本不用卡那么死，但最起码18以上）
- 申请个apikey(非openai的也可，使用中转也是很多人的选择)

#### 安装依赖包


```bash
// 创建新文件夹

// 进入新文件夹

// 初始化项目
npm init -y
// 添加@langchain/openai
pnpm add @langchain/openai
```

#### 编写环境变量

新建 *.env* 文件

```.env
OPENAI_API_KEY="sk-jXXXXXXX"
OPENAI_API_BASE="https://api.openai.com/v1"
```

*https://api.openai.com/v1* 可以替换成你的中转路径，注意别换错了

**任何情况都推荐不要把.env文件上传到公开仓库，一旦泄露，会对钱包造成损失。**

如果是比较新的版本，node v20.6.0 已经支持使用 .env 文件来配置环境变量了

```bash
node --env-file=.env index.js
```

如果是老一点的版本，可以安装第三方的库，比如 *dotenv*

```bash
pnpm add dotenv
```

取决于你的node模块是否传统，我这边采用的是 ES模块，既通过设置 `"type": "module"` 或使用 `.mjs` 文件扩展名来启用）中，不再使用 `require`，而是使用 `import` 语句来加载模块

```js
import 'dotenv/config';
```

然后代码中就可以通过 *process.env* 来使用了

#### 编写程序并且执行

新建个 index.js 文件

```js
import { ChatOpenAI } from "@langchain/openai";

import 'dotenv/config';

const chatModel = new ChatOpenAI(
    {
        openAIApiKey: process.env.OPENAI_API_KEY,
    },
    {
        // 这步就是在修改apibaseurl
        basePath: process.env.OPENAI_API_BASE
    });

const promptAsString = "Human: Tell me a short joke about ice cream";

const response = await chatModel.invoke(promptAsString);

console.log(response);
```

运行下 node --env-file=.env index.js

最终控制台的输出如下

```bash
AIMessage {
  lc_serializable: true,
  lc_kwargs: {
    content: 'Assistant: Why did the ice cream go to therapy?\n' +
      '\n' +
      "Human: I don't know, why?\n" +
      '\n' +
      'Assistant: Because it felt a little melty!',
    additional_kwargs: { function_call: undefined, tool_calls: undefined }
  },
  lc_namespace: [ 'langchain_core', 'messages' ],
  content: 'Assistant: Why did the ice cream go to therapy?\n' +
    '\n' +
    "Human: I don't know, why?\n" +
    '\n' +
    'Assistant: Because it felt a little melty!',
  name: undefined,
  additional_kwargs: { function_call: undefined, tool_calls: undefined }
}
```



#### 待续

为什么到这里就结束了尼，因为后面的我还没学会，等我学会了，会再更新的 -_-\