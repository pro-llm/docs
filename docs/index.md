---
hide:
  - navigation
  - toc
---

# LLMNEX 文档

一站式接入 Gemini / OpenAI 等主流图像与文本模型,**兼容官方 SDK,改个 Base URL 就能跑**。

<div class="grid cards" markdown>

-   :material-rocket-launch-outline:{ .lg .middle } **快速开始**

    ---

    拿到 Key 之后的第一个请求,三分钟跑通。

    [:octicons-arrow-right-24: 开始接入](guide/getting-started.md)

-   :material-image-multiple-outline:{ .lg .middle } **生图服务**

    ---

    两套模型:Nano Banana 最高 4096²、GPT Image 2 尺寸逐像素自选。该选哪个?

    [:octicons-arrow-right-24: 生图介绍](image-generation/overview.md)

-   :material-google:{ .lg .middle } **Gemini 接入指南**

    ---

    `generateContent` 原生协议,12 个模型的完整参数与实测数据。

    [:octicons-arrow-right-24: 查看文档](image-generation/gemini.md)

-   :material-image-edit-outline:{ .lg .middle } **GPT Image 2 接入指南**

    ---

    标准 OpenAI 图像接口,尺寸逐像素自选,文生图 / 图生图。

    [:octicons-arrow-right-24: 查看文档](image-generation/gpt-image-2.md)

-   :material-console:{ .lg .middle } **控制台**

    ---

    创建密钥、查看用量与余额。

    [:octicons-arrow-right-24: 前往控制台](https://www.llmnex.com)

</div>

## Base URL

所有接口统一使用:

```text
https://www.llmnex.com
```

## 协议兼容

| 你在用 | 直接换 Base URL 即可 |
| --- | --- |
| Google Gemini SDK / `generateContent` | ✅ |
| OpenAI SDK / `chat.completions` | ✅ |
| OpenAI `images.generations` | ✅ |
| OpenAI `images.edits` | ✅ |
