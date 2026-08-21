# GPT Image 2 接入指南

标准 OpenAI 图像接口,参数与官方一致。支持文生图、图生图,单张最高 **3840 × 2160**。

<div class="grid cards" markdown>

- :material-link: **Base URL** — `https://www.llmnex.com`
- :material-api: **协议** — OpenAI 兼容
- :material-image-size-select-large: **尺寸** — 逐像素自选
- :material-quality-high: **质量档** — low / medium / high

</div>

## 概览

用你现在调 OpenAI 图像接口的代码就能跑,把域名换成本站、Key 换成本站签发的即可。

| 项 | 值 |
| --- | --- |
| Base URL | `https://www.llmnex.com` |
| 文生图 | `POST /v1/images/generations` |
| 图生图 | `POST /v1/images/edits` |
| 返回形态 | base64 PNG(`data[0].b64_json`) |
| 单次出图 | 1 张(最多 4,见 [n 参数](#n)) |

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
        "model": "gpt-image-2-medium",
        "prompt": "a red ceramic teapot on a wooden table, soft daylight",
        "size": "2048x1152"
      }'
    ```

=== "存成图片"

    ```bash
    curl -s -X POST "https://www.llmnex.com/v1/images/generations" \
      -H "Authorization: Bearer YOUR_API_KEY" \
      -H "Content-Type: application/json" \
      --max-time 600 \
      -d '{
        "model": "gpt-image-2-medium",
        "prompt": "a red ceramic teapot on a wooden table, soft daylight",
        "size": "2048x1152"
      }' \
      | python3 -c "import sys,json,base64; open('out.png','wb').write(base64.b64decode(json.load(sys.stdin)['data'][0]['b64_json'])); print('saved out.png')"
    ```

=== "OpenAI SDK"

    ```python
    from openai import OpenAI

    client = OpenAI(
        api_key="YOUR_API_KEY",
        base_url="https://www.llmnex.com/v1",
        timeout=600,
    )

    result = client.images.generate(
        model="gpt-image-2-medium",
        prompt="a red ceramic teapot on a wooden table, soft daylight",
        size="2048x1152",
    )

    import base64
    open("out.png", "wb").write(base64.b64decode(result.data[0].b64_json))
    ```

## 模型

四个模型名,**后端是同一个 GPT Image 2**,区别只在 `quality`:

| 模型 | `quality` | 请求里传 `quality` |
| --- | --- | --- |
| `gpt-image-2-low` | 固定 `low` | 忽略 |
| `gpt-image-2-medium` | 固定 `medium` | 忽略 |
| `gpt-image-2-high` | 固定 `high` | 忽略 |
| `gpt-image-2-c` | 由你指定,不传则 `low` | **生效** |

**名字只决定 `quality`,不影响尺寸** —— 尺寸永远由 `size` 决定,四个名字一视同仁。

!!! note "带档位的三个名字会忽略请求里的 `quality`"
    选了 `gpt-image-2-medium` 就一定按 `medium` 出图,传 `quality: "high"` 也不会变。

    **需要自己控制质量档,请用 `gpt-image-2-c`。**

## quality

取值 `low` / `medium` / `high`,与官方一致。它控制**渲染精细度**,不影响输出尺寸。

**只有 `gpt-image-2-c` 会读这个字段**,不传则为 `low`。
另外三个名字固定用名字里的档,忽略此字段。

## size

**逐像素指定,不是从固定档位里挑。** 你写多大就出多大。

```bash
"size": "2048x1152"     # 就出 2048×1152
"size": "1536x1536"     # 就出 1536×1536
"size": "3840x2160"     # 就出 3840×2160
```

### 四条约束

| 约束 | 说明 |
| --- | --- |
| 宽高都是 **16 的倍数** | `1920x1080` 不合法(1080 不是 16 的倍数) |
| 最长边 **≤ 3840** | |
| 长短边之比 **≤ 3:1** | |
| 总像素在 **655,360 ~ 8,294,400** 之间 | 约 0.66 MP ~ 8.29 MP |

!!! warning "不传或不满足约束 → 自动用 1024×1024"
    本站**不会因此报错**,而是回落到 `1024x1024` 出图,优先保证你拿到结果。

    所以如果你发现出图尺寸不是你要的,**先检查 `size` 是否满足上面四条**。
    最常见的是宽或高不是 16 的倍数 —— 比如 `1920x1080`、`1366x768`。

### 常用尺寸

| 用途 | `size` | 像素量 |
| --- | --- | --- |
| 方图 | `1024x1024` | 1.05 MP |
| 方图(大) | `2048x2048` | 4.19 MP |
| 横版 16:9 | `2048x1152` | 2.36 MP |
| 横版 16:9(最大) | `3840x2160` | 8.29 MP |
| 竖版 9:16 | `1152x2048` | 2.36 MP |
| 竖版 9:16(最大) | `2160x3840` | 8.29 MP |
| 横版 3:2 | `1536x1024` | 1.57 MP |
| 竖版 2:3 | `1024x1536` | 1.57 MP |

不在表里也没关系 —— 只要满足四条约束,任意尺寸都能出。

## 图生图

`POST /v1/images/edits`,`multipart/form-data`。参数与文生图一致,额外带一个 `image` 文件。

=== "curl"

    ```bash
    curl -X POST "https://www.llmnex.com/v1/images/edits" \
      -H "Authorization: Bearer YOUR_API_KEY" \
      --max-time 600 \
      -F "model=gpt-image-2-high" \
      -F "prompt=make the teapot blue and add steam" \
      -F "size=2048x1152" \
      -F "image=@input.png;type=image/png"
    ```

=== "OpenAI SDK"

    ```python
    from openai import OpenAI

    client = OpenAI(
        api_key="YOUR_API_KEY",
        base_url="https://www.llmnex.com/v1",
        timeout=600,
    )

    result = client.images.edit(
        model="gpt-image-2-high",
        image=open("input.png", "rb"),
        prompt="make the teapot blue and add steam",
        size="2048x1152",
    )

    import base64
    open("out.png", "wb").write(base64.b64decode(result.data[0].b64_json))
    ```

输出尺寸由 `size` 决定,**与输入图的尺寸无关**。

## n 参数 { #n }

**建议传 1,默认也是 1。** 最大支持 4。

!!! danger "n > 1 会成倍变慢、成倍计费"
    多张图是**依次生成**的,不是并行。`n=4` 大约需要 **4 倍时间**,而且**按张计费**
    —— 传 4 就收 4 张图的钱。

    需要多张时,更好的做法是自己并发发 4 个 `n=1` 的请求。

## 响应结构

```json
{
  "created": 1787250000,
  "data": [
    { "b64_json": "iVBORw0KGgoAAAANSUhEUg..." }
  ]
}
```

图片是 base64 编码的 PNG,取 `data[0].b64_json` 解码即可。

## 超时与耗时

生图是长耗时操作。**客户端超时请设到 600 秒**,默认的 30 秒或 60 秒几乎必然中途断开。

| 质量档 | 常见耗时 |
| --- | --- |
| `low` | 15 – 40 s |
| `medium` | 20 – 60 s |
| `high` | 30 – 90 s |

尺寸越大也越慢。耗时受上游负载影响明显,高峰期会整体拉长,**长尾可能远超上表**。

服务内部已有自动重试与账号轮换,你收到的错误都是重试之后仍未成功的终态结果 ——
**不需要自己加激进的重试**,那只会加重排队。

## 错误处理

| HTTP | 含义 | 怎么办 |
| --- | --- | --- |
| 400 | 提示词被内容安全拦截 | 改提示词。**不要重试** |
| 401 | Key 无效 | 检查 `Authorization` 头 |
| 429 | 上游限流 | 已在内部重试过;稍后再试,别加激进重试 |
| 451 | 提示词或参考图被内容安全拦截 | 同 400,**不要重试** |
| 500 | 上游临时不可用 | 稍后重试 |
| 503 | `model_not_found` —— 模型名写错,或你的 Key 没有该模型的权限 | 见下 |

!!! danger "不要对 400 / 451 自动重试"
    这两个是**终态**:上游已经给出了明确判定,重试只会白白消耗额度。请当作参数错误处理。

!!! tip "503 `No available channel for model ...`"
    这条**不是服务故障**,而是「这个 Key 用不了这个模型」:模型名拼错,或者你的
    Key 没被授予该模型的权限。

    先确认自己能用哪些模型:

    ```bash
    curl "https://www.llmnex.com/v1/models" \
      -H "Authorization: Bearer YOUR_API_KEY"
    ```

    返回列表里没有你要调的名字,就联系我们开通;有但仍报这个错,再联系我们排查。

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
        model="gpt-image-2-high",
        prompt="a wide cinematic landscape at golden hour, ultra detailed",
        size="3840x2160",     # 逐像素指定
        n=1,
    )

    open("out.png", "wb").write(base64.b64decode(result.data[0].b64_json))
    print("saved out.png")    # 3840 x 2160
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
        model: "gpt-image-2-high",
        prompt: "a wide cinematic landscape at golden hour, ultra detailed",
        size: "3840x2160",
        n: 1,
      }),
      signal: AbortSignal.timeout(600_000),   // 10 分钟
    });

    const data = await res.json();
    fs.writeFileSync("out.png", Buffer.from(data.data[0].b64_json, "base64"));
    console.log("saved out.png");             // 3840 x 2160
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
        model="gpt-image-2-medium",
        image=open("input.png", "rb"),
        prompt="turn it into a watercolor painting",
        size="2048x2048",
    )

    open("out.png", "wb").write(base64.b64decode(result.data[0].b64_json))
    print("saved out.png")    # 2048 x 2048
    ```
