# GPT Image 2 接入指南

通过标准 OpenAI 图像接口调用 GPT Image 2。支持文生图、图生图、三档分辨率,单次调用最高可出 **2880 × 2880**。

<div class="grid cards" markdown>

- :material-link: **Base URL** — `https://www.llmnex.com`
- :material-api: **协议** — OpenAI 兼容
- :material-image-multiple: **可用模型** — 4 个
- :material-quality-high: **最高分辨率** — 2880²

</div>

## 概览

用你现在调 OpenAI 图像接口的代码就能跑,把域名换成本站、Key 换成本站签发的即可。

| 项 | 值 |
| --- | --- |
| Base URL | `https://www.llmnex.com` |
| 文生图 | `POST /v1/images/generations` |
| 图生图 | `POST /v1/images/edits` |
| 对话式 | `POST /v1/chat/completions` |
| 单次出图 | 1 张 |

三个端点都能出图,**但返回形态不同**,见[响应结构](#响应结构)。

## 鉴权

```http
Authorization: Bearer YOUR_API_KEY
```

!!! tip "密钥保管"
    Key 形如 `sk-xxxxxxxx`。请放在服务端调用,不要写进前端代码或客户端 App。

## 快速开始

=== "curl"

    ```bash
    curl -X POST "https://www.llmnex.com/v1/images/generations" \
      -H "Authorization: Bearer YOUR_API_KEY" \
      -H "Content-Type: application/json" \
      --max-time 600 \
      -d '{
        "model": "gpt-image-2-1k",
        "prompt": "a red ceramic teapot on a wooden table, soft daylight",
        "n": 1
      }'
    ```

=== "存成图片"

    ```bash
    curl -s -X POST "https://www.llmnex.com/v1/images/generations" \
      -H "Authorization: Bearer YOUR_API_KEY" \
      -H "Content-Type: application/json" \
      --max-time 600 \
      -d '{
        "model": "gpt-image-2-1k",
        "prompt": "a red ceramic teapot on a wooden table, soft daylight",
        "n": 1
      }' \
      | python3 -c "import sys,json,base64; open('out.png','wb').write(base64.b64decode(json.load(sys.stdin)['data'][0]['b64_json'])); print('saved out.png')"
    ```

=== "OpenAI SDK"

    ```python
    from openai import OpenAI

    client = OpenAI(
        api_key="YOUR_API_KEY",
        base_url="https://www.llmnex.com/v1",
    )

    result = client.images.generate(
        model="gpt-image-2-1k",
        prompt="a red ceramic teapot on a wooden table, soft daylight",
        n=1,
    )

    import base64
    open("out.png", "wb").write(base64.b64decode(result.data[0].b64_json))
    ```

## 模型与分辨率

四个模型名对应四种档位策略,**后端是同一个 GPT Image 2**。

| 模型 | 分辨率档 | 说明 |
| --- | --- | --- |
| `gpt-image-2-1k` | 1K | 最快,适合预览与高并发场景 |
| `gpt-image-2-2k` | 2K | 均衡档 |
| `gpt-image-2-4k` | 4K | 最高档,耗时明显更长 |
| `gpt-image-2-c` | 由你指定 | 档位跟随请求里的 `quality` 字段 |

带档位后缀的三个模型**忽略请求里的 `quality`** —— 档位由模型名决定,这样计费与出图始终一致。

!!! warning "4K 档是 2880²,不是 4096²"
    GPT Image 2 的 4K 档在 1:1 下出 **2880 × 2880**。如果你需要 4096²,请改用[香蕉系列](gemini.md)。

### `gpt-image-2-c` 的 quality 取值

只有 `gpt-image-2-c` 会读 `quality`。**请使用 `1k` / `2k` / `4k` 这三个字面量**:

| `quality` | 实际档位 |
| --- | --- |
| `1k` | 1K |
| `2k` / `hd` | 2K |
| `4k` / `ultra` | 4K |

!!! danger "不要用 `low` / `medium` / `high`"
    这三个 OpenAI 习惯用词在本站的对应关系**不符合直觉**:`low` → 1K、**`medium` → 1K**、**`high` → 2K**,并且没有任何值能落到 4K。请一律使用 `1k` / `2k` / `4k`。

## 宽高比

通过标准的 `size` 参数选择,共 **5 种**:

| `size` | 宽高比 |
| --- | --- |
| `1024x1024` | 1:1(默认) |
| `1792x1024` | 16:9 |
| `1024x1792` | 9:16 |
| `1536x1024` | 3:2 |
| `1024x1536` | 2:3 |

`size` 决定的是**比例**,不是最终像素;最终像素由「档位 × 比例」共同决定。实测几何:

| 比例 | `size` | 1K | 2K | 4K |
| --- | --- | --- | --- | --- |
| 1:1 | `1024x1024` | 1024 × 1024 | 2048 × 2048 | 2880 × 2880 |
| 16:9 | `1792x1024` | 1280 × 720 | 2560 × 1440 | 3840 × 2160 |
| 9:16 | `1024x1792` | 720 × 1280 | 1440 × 2560 | 2160 × 3840 |
| 3:2 | `1536x1024` | 1248 × 832 | 2496 × 1664 | 3504 × 2336 |
| 2:3 | `1024x1536` | 832 × 1248 | 1664 × 2496 | 2336 × 3504 |

上表 15 个组合均为实测值。**不要自己按倍数推算** —— 2K 恰好是 1K 的两倍,但 4K 并非统一倍率(16:9 是 1K 的 3 倍,1:1 只有 2.81 倍)。

!!! warning "表外的 `size` 会被当成 1:1"
    传入不在上表中的尺寸(例如 `2048x2048`)不会报错,但比例会回落成 1:1。**请只使用上表中的五个值。**

!!! note "`aspect_ratio` 字段无效"
    本站网关只透传标准 OpenAI 参数,自定义的 `aspect_ratio` 字段会被丢弃。需要 21:9、4:3 等更多比例请改用[香蕉系列](gemini.md)的 `generateContent`,那里支持 14 种。

档位与比例可以自由组合:

```bash
curl -X POST "https://www.llmnex.com/v1/images/generations" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" --max-time 600 \
  -d '{
    "model": "gpt-image-2-4k",
    "prompt": "a wide cinematic landscape, golden hour",
    "size": "1792x1024",
    "n": 1
  }'
```

上面这条出 **3840 × 2160**:档位来自模型名的 `-4k`,比例来自 `size` 的 16:9。

## 图生图

`POST /v1/images/edits`,`multipart/form-data`。参数与文生图一致,额外带一个 `image` 文件。

=== "curl"

    ```bash
    curl -X POST "https://www.llmnex.com/v1/images/edits" \
      -H "Authorization: Bearer YOUR_API_KEY" \
      --max-time 600 \
      -F "model=gpt-image-2-2k" \
      -F "prompt=make the teapot blue and add steam" \
      -F "n=1" \
      -F "image=@input.png;type=image/png"
    ```

=== "OpenAI SDK"

    ```python
    from openai import OpenAI

    client = OpenAI(
        api_key="YOUR_API_KEY",
        base_url="https://www.llmnex.com/v1",
    )

    result = client.images.edit(
        model="gpt-image-2-2k",
        image=open("input.png", "rb"),
        prompt="make the teapot blue and add steam",
        n=1,
    )

    import base64
    open("out.png", "wb").write(base64.b64decode(result.data[0].b64_json))
    ```

分辨率与比例规则和文生图完全相同 —— 模型名定档位,`size` 定比例,**与输入图的尺寸无关**。

## 对话式调用

已经在用 Chat Completions 的代码可以走 `POST /v1/chat/completions`,不必改成图像接口。

```bash
curl -X POST "https://www.llmnex.com/v1/chat/completions" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" --max-time 600 \
  -d '{
    "model": "gpt-image-2-4k",
    "messages": [{"role": "user", "content": "a red ceramic teapot"}],
    "stream": false
  }'
```

档位同样由模型名决定。**但这条路径的返回形态与前两个端点不同**,见下一节。

## 响应结构

=== "generations / edits"

    图片以 base64 内嵌在 `data[0].b64_json`:

    ```json
    {
      "created": 1787250000,
      "data": [
        { "b64_json": "iVBORw0KGgoAAAANSUhEUg..." }
      ]
    }
    ```

=== "chat/completions"

    `choices[0].message.content` 是一段 Markdown,图片是**托管链接**,不是 base64:

    ```text
    ![Generated Image](https://adobe2api.llmnex.com/generated/<uuid>.png)
    ```

!!! danger "两个端点的解析方式不能互换"
    `generations` / `edits` 返回 base64,`chat/completions` 返回 URL。按前者的写法去解析后者会拿不到图。

## 超时与耗时

生图是长耗时操作。**客户端超时请设到 600 秒**,默认的 30 秒或 60 秒几乎必然中途断开。

| 档位 | 常见耗时 |
| --- | --- |
| 1K | 16 – 51 s |
| 2K | 20 – 32 s |
| 4K | 35 – 75 s |

耗时受上游负载影响明显,高峰期会整体拉长,长尾可能远超上表。服务内部已有自动重试与账号轮换,你收到的错误都是重试之后仍未成功的终态结果 —— **不需要自己加激进的重试**,那只会加重排队。

## 错误处理

| HTTP | 含义 | 怎么办 |
| --- | --- | --- |
| 400 | 提示词被内容安全拦截 | 改提示词。**不要重试**,同样的词每次都会被拦 |
| 401 | Key 无效 | 检查 `Authorization` 头 |
| 429 | 上游限流 | 已在内部重试过;稍后再试,别加激进重试 |
| 451 | 提示词或参考图被内容安全拦截 | 同 400,**不要重试** |
| 500 | 上游临时不可用 | 稍后重试 |

!!! danger "不要对 400 / 451 自动重试"
    这两个是**终态**:上游已经给出了明确判定,重试只会白白消耗额度。请当作参数错误处理。

## 完整示例

=== "Python"

    ```python
    import base64
    from openai import OpenAI

    client = OpenAI(
        api_key="YOUR_API_KEY",
        base_url="https://www.llmnex.com/v1",
        timeout=600,          # 必须给足,生图很慢
    )

    result = client.images.generate(
        model="gpt-image-2-2k",
        prompt="a wide cinematic landscape at golden hour, ultra detailed",
        size="1792x1024",     # 16:9
        n=1,
    )

    open("out.png", "wb").write(base64.b64decode(result.data[0].b64_json))
    print("saved out.png")    # 2560 x 1440
    ```

=== "Node.js"

    ```javascript
    import fs from "node:fs";

    const res = await fetch("https://www.llmnex.com/v1/images/generations", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: "Bearer YOUR_API_KEY",
      },
      body: JSON.stringify({
        model: "gpt-image-2-2k",
        prompt: "a wide cinematic landscape at golden hour, ultra detailed",
        size: "1792x1024",
        n: 1,
      }),
      signal: AbortSignal.timeout(600_000),   // 10 分钟
    });

    const data = await res.json();
    fs.writeFileSync("out.png", Buffer.from(data.data[0].b64_json, "base64"));
    console.log("saved out.png");             // 2560 x 1440
    ```

=== "图生图"

    ```python
    import base64
    from openai import OpenAI

    client = OpenAI(
        api_key="YOUR_API_KEY",
        base_url="https://www.llmnex.com/v1",
        timeout=600,
    )

    result = client.images.edit(
        model="gpt-image-2-4k",
        image=open("input.png", "rb"),
        prompt="turn it into a watercolor painting",
        n=1,
    )

    open("out.png", "wb").write(base64.b64decode(result.data[0].b64_json))
    print("saved out.png")    # 2880 x 2880
    ```
