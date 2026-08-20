# 快速开始

## 1. 拿到 API Key

在 [控制台](https://www.llmnex.com) 创建令牌,形如 `sk-xxxxxxxx`。

!!! tip "密钥保管"
    请放在服务端调用,不要写进前端代码或客户端 App。

## 2. 发出第一个请求

=== "生图(Gemini 协议)"

    ```bash
    curl -X POST "https://www.llmnex.com/v1beta/models/gemini-3-pro-image-1k:generateContent" \
      -H "Authorization: Bearer YOUR_API_KEY" \
      -H "Content-Type: application/json" \
      --max-time 300 \
      -d '{"contents":[{"parts":[{"text":"a red ceramic teapot"}]}]}'
    ```

=== "生图(OpenAI 协议)"

    ```bash
    curl -X POST "https://www.llmnex.com/v1/chat/completions" \
      -H "Authorization: Bearer YOUR_API_KEY" \
      -H "Content-Type: application/json" \
      --max-time 300 \
      -d '{"model":"gemini-3-pro-image-1k",
           "messages":[{"role":"user","content":"a red ceramic teapot"}],
           "stream":false}'
    ```

=== "查看可用模型"

    ```bash
    curl -H "Authorization: Bearer YOUR_API_KEY" \
      https://www.llmnex.com/v1/models
    ```

## 3. 三个必须知道的点

!!! warning "客户端超时设到 300 秒"
    生图是长耗时操作,1K 常见 20–45 秒,4K 可到 130 秒以上。默认的 30 / 60 秒超时几乎必然中途断开。

!!! warning "不要自己加激进重试"
    服务内部已有自动重试与账号轮换。你收到的错误都是重试之后仍未成功的终态结果,再重试只会加重排队。

!!! danger "451 是终态,不要重试"
    451 表示内容被安全审核拦截。同样的提示词每次都会被拦,重试只是白白消耗额度 —— 请修改提示词或参考图。

## 下一步

- [生图介绍](../image-generation/overview.md) —— 模型清单与选型
- [Gemini 接入指南](../image-generation/gemini.md) —— 完整参数、图生图、流式、错误处理
