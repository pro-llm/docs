# 香蕉生图指南

通过 Gemini 兼容接口调用 Nano Banana 系列图像模型。支持文生图、图生图、14 种宽高比与三档分辨率,单次调用最高可出 **4096 × 4096**。

<div class="grid cards" markdown>

- :material-link: **Base URL** — `https://www.llmnex.com`
- :material-api: **协议** — Gemini `generateContent`
- :material-image-multiple: **可用模型** — 12 个
- :material-quality-high: **最高分辨率** — 4096²

</div>

## 概览

你用调用 Gemini 官方 API 的方式请求,只需把域名换成本站、把 API Key 换成本站签发的 Key。

| 项 | 值 |
| --- | --- |
| Base URL | `https://www.llmnex.com` |
| 生图端点 | `POST /v1beta/models/{model}:generateContent` |
| 流式端点 | `POST /v1beta/models/{model}:streamGenerateContent` |
| 返回形态 | base64 PNG(`inlineData`),不返回外链 URL |
| 单次出图 | 1 张 |

能力:**文生图**、**图生图**(最多 14 张参考图)、**14 种宽高比**、**1K / 2K / 4K 三档分辨率**、**流式返回**。

## 鉴权

两种方式任选其一,效果相同。用 Google 官方 SDK 时选第二种,改个 base URL 就能直接跑。

```http
Authorization: Bearer YOUR_API_KEY
```

```http
x-goog-api-key: YOUR_API_KEY
```

!!! tip "密钥保管"
    Key 形如 `sk-xxxxxxxx`。请放在服务端调用,不要写进前端代码或客户端 App。

## 快速开始

=== "curl"

    ```bash
    curl -X POST "https://www.llmnex.com/v1beta/models/gemini-3-pro-image-1k:generateContent" \
      -H "Authorization: Bearer YOUR_API_KEY" \
      -H "Content-Type: application/json" \
      --max-time 300 \
      -d '{
        "contents": [
          { "parts": [ { "text": "a red ceramic teapot on a wooden table, soft daylight" } ] }
        ]
      }'
    ```

=== "存成图片"

    ```bash
    curl -s -X POST "https://www.llmnex.com/v1beta/models/gemini-3-pro-image-1k:generateContent" \
      -H "Authorization: Bearer YOUR_API_KEY" \
      -H "Content-Type: application/json" --max-time 300 \
      -d '{"contents":[{"parts":[{"text":"a red ceramic teapot"}]}]}' \
      | python3 -c "import sys,json,base64;
    d=json.load(sys.stdin);
    b=d['candidates'][0]['content']['parts'][0]['inlineData']['data'];
    open('out.png','wb').write(base64.b64decode(b));
    print('saved out.png')"
    ```

## 模型与分辨率

**分辨率由模型名的后缀决定**,不需要额外传参数。三个模型族的画风与速度不同,分辨率行为完全一致。

| 模型族 | 后端 | 特点 |
| --- | --- | --- |
| `gemini-3-pro-image` | Nano Banana Pro | 质量最高,耗时最长,复杂提示词表现最好 |
| `gemini-3.1-flash-image` | Nano Banana 2 | 速度与质量均衡,日常首选 |
| `gemini-2.5-flash-image` | Nano Banana 2 | 兼容旧命名,行为同上 |

每族四个后缀,共 12 个可用模型:

| 后缀 | 分辨率 | 实测输出(1:1) | 说明 |
| --- | --- | --- | --- |
| `-1k` | 1K | 1024 × 1024 | 默认档,最快 |
| `-2k` | 2K | 2048 × 2048 | 四倍像素 |
| `-4k` | 4K | 4096 × 4096 | 最高档,耗时明显更长 |
| `-c` | 自选 | 由 `imageSize` 决定 | 见下 |

### 后缀 `-c`:分辨率交给调用方

`-c` 结尾的模型不锁定分辨率,由你在请求里用 `generationConfig.imageConfig.imageSize` 指定。**不传则为 1K。**

```json
{
  "contents": [ { "parts": [ { "text": "a plain grey cube" } ] } ],
  "generationConfig": {
    "imageConfig": { "imageSize": "4K" }
  }
}
```

`imageSize` 取值:`1K` / `2K` / `4K`。

!!! warning "别在固定档位的模型上传 imageSize"
    在 `-1k` / `-2k` / `-4k` 模型上传 `imageSize` **不会生效** —— 分辨率已由模型名锁定。需要动态切换分辨率,请用 `-c` 系列。

!!! note "关于 4K"
    4K 档在 1:1 下稳定输出 4096 × 4096,但上游偶发会给出**等面积的其他尺寸**(实测出现过一次 5632 × 3072)。请读取返回图的实际尺寸,不要在代码里硬编码宽高。

## 请求体结构

与 Gemini 官方格式一致。最简形态只需要 `contents`。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `contents[].parts[].text` | string | 提示词。多个 text part 会用空格拼接 |
| `contents[].parts[].inlineData` | object | 参考图,见「图生图」。最多 14 张 |
| `generationConfig.imageConfig.aspectRatio` | string | 宽高比,如 `"16:9"`。默认 `1:1` |
| `generationConfig.imageConfig.imageSize` | string | `1K`/`2K`/`4K`,仅 `-c` 模型有效 |
| `generationConfig.seed` | integer | 随机种子,同参数下可复现 |

```json
{
  "contents": [
    { "parts": [ { "text": "a lighthouse on a cliff at dawn, cinematic" } ] }
  ],
  "generationConfig": {
    "imageConfig": { "aspectRatio": "16:9" },
    "seed": 42
  }
}
```

## 宽高比

支持 14 种比例:`1:1` `16:9` `9:16` `4:3` `3:4` `3:2` `2:3` `5:4` `4:5` `21:9` `1:4` `4:1` `1:8` `8:1`

非 1:1 的比例下,输出像素会在保持总面积的前提下按比例分配。1K 档的实测结果:

| aspectRatio | 1K 实测输出 | 典型用途 |
| --- | --- | --- |
| `1:1` | 1024 × 1024 | 头像、商品图 |
| `16:9` | 1376 × 768 | 横版封面、视频首帧 |
| `9:16` | 768 × 1376 | 竖屏短视频、手机壁纸 |
| `4:3` | 1200 × 896 | 传统照片比例 |
| `21:9` | 1584 × 672 | 超宽横幅 |
| `4:5` | 928 × 1152 | 社交媒体竖图 |

!!! tip
    2K / 4K 档的输出像素按同样比例等倍放大。传入未支持的比例会自动回落到 `1:1`,不会报错。

## 图生图

在 `parts` 里加入 `inlineData` 即为图生图。文字描述你想要的改动,参考图提供内容或风格来源。

```json
{
  "contents": [
    {
      "parts": [
        { "text": "把这张照片改成水彩画风格,保留构图" },
        {
          "inlineData": {
            "mimeType": "image/png",
            "data": "iVBORw0KGgoAAAANSUhEUg..."
          }
        }
      ]
    }
  ]
}
```

| 约束 | 值 |
| --- | --- |
| 参考图数量 | 最多 14 张 |
| 格式 | PNG / JPEG / WebP,由 `mimeType` 声明 |
| 编码 | 纯 base64,**不要**带 `data:image/png;base64,` 前缀 |
| 输出分辨率 | 仍由模型后缀决定,与输入图尺寸无关 |

## 流式返回

把端点换成 `:streamGenerateContent` 即可。返回 SSE 格式,每行以 `data: ` 开头。

```bash
curl -N -X POST "https://www.llmnex.com/v1beta/models/gemini-3-pro-image-1k:streamGenerateContent" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" --max-time 300 \
  -d '{"contents":[{"parts":[{"text":"a plain grey cube"}]}]}'
```

!!! info "流式不会逐步渲染画面"
    图像是一次性生成的,流式只是把最终结果以 SSE 包一层送出。它的价值在于保持连接活跃、避免中间层超时断开。

## 响应结构

```json
{
  "candidates": [
    {
      "content": {
        "role": "model",
        "parts": [
          {
            "inlineData": {
              "mimeType": "image/png",
              "data": "iVBORw0KGgoAAAANSUhEUgAABAAA..."
            }
          }
        ]
      }
    }
  ]
}
```

取图路径固定为 `candidates[0].content.parts[*].inlineData.data`。建议遍历 `parts` 找第一个带 `inlineData` 的元素,不要写死索引 0。

!!! warning "4K 图的响应体很大"
    4K 图的 base64 约 **20–25 MB**。请确认你的 HTTP 客户端与中间层没有更小的响应体上限。

## 超时与耗时

生图是长耗时操作。**客户端超时请设到 300 秒**,默认的 30 秒或 60 秒几乎必然中途断开。

| 档位 | 常见耗时 | 实测上限 |
| --- | --- | --- |
| 1K | 20 – 45 s | ~90 s |
| 2K | 30 – 90 s | ~290 s |
| 4K | 60 – 130 s | ~290 s |

耗时受上游负载影响明显,高峰期会整体拉长。服务内部已有自动重试与账号轮换,你收到的错误都是重试之后仍未成功的终态结果 —— **不需要自己加激进的重试**,那只会加重排队。

## 错误处理

| HTTP | 含义 | 怎么办 |
| --- | --- | --- |
| 400 | 请求体不合法,如 `contents` 缺失或为空 | 检查 JSON 结构 |
| 401 | Key 无效或已过期 | 检查 `Authorization` 头 |
| 429 | 触发限流 | 退避后重试,建议指数退避起步 2 s |
| 451 | 内容被安全审核拦截 | **终态,重试无用。** 改提示词或参考图 |
| 500 | 模型不可用,或网关内部错误 | 确认模型名拼写;持续出现请联系我们 |
| 503 | 上游暂时不可用 | 退避后重试 |

```json title="451 内容拦截的典型响应"
{
  "error": {
    "message": "image_unsafe: The generated images appear to be unsafe.",
    "type": "invalid_request_error"
  }
}
```

!!! danger "不要对 451 自动重试"
    同样的提示词每次都会被拦,只会白白消耗额度。把它当作参数错误处理。

## OpenAI 兼容路径

如果你的代码已经在用 OpenAI SDK,也可以走 `/v1/chat/completions`,不必改成 Gemini 格式。

```bash
curl -X POST "https://www.llmnex.com/v1/chat/completions" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" --max-time 300 \
  -d '{
    "model": "gemini-3-pro-image-1k",
    "messages": [{"role":"user","content":"a red ceramic teapot"}],
    "stream": false
  }'
```

返回的 `choices[0].message.content` 是一段 Markdown,图片以 **base64 data URI** 内嵌:

```text
![image](data:image/png;base64,iVBORw0KGgoAAAANSUhEUg...)
```

!!! warning "这条路径设置不了宽高比"
    `aspectRatio` 不在 OpenAI 参数集里,会被丢弃。**需要控制比例请用 `generateContent`。**

## 完整示例

=== "Python"

    ```python
    import base64, requests

    API_KEY = "YOUR_API_KEY"
    BASE    = "https://www.llmnex.com"
    MODEL   = "gemini-3-pro-image-2k"

    resp = requests.post(
        f"{BASE}/v1beta/models/{MODEL}:generateContent",
        headers={"Authorization": f"Bearer {API_KEY}"},
        json={
            "contents": [{"parts": [{"text": "a lighthouse on a cliff at dawn"}]}],
            "generationConfig": {"imageConfig": {"aspectRatio": "16:9"}},
        },
        timeout=300,          # 必须给足,生图很慢
    )
    resp.raise_for_status()

    parts = resp.json()["candidates"][0]["content"]["parts"]
    b64 = next(p["inlineData"]["data"] for p in parts if "inlineData" in p)
    with open("out.png", "wb") as f:
        f.write(base64.b64decode(b64))
    print("saved out.png")
    ```

=== "Node.js"

    ```javascript
    import { writeFile } from "node:fs/promises";

    const API_KEY = "YOUR_API_KEY";
    const MODEL   = "gemini-3-pro-image-2k";

    const res = await fetch(
      `https://www.llmnex.com/v1beta/models/${MODEL}:generateContent`,
      {
        method: "POST",
        headers: {
          "Authorization": `Bearer ${API_KEY}`,
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          contents: [{ parts: [{ text: "a lighthouse on a cliff at dawn" }] }],
          generationConfig: { imageConfig: { aspectRatio: "16:9" } },
        }),
        signal: AbortSignal.timeout(300_000),   // 5 分钟
      }
    );
    if (!res.ok) throw new Error(`HTTP ${res.status}: ${await res.text()}`);

    const parts = (await res.json()).candidates[0].content.parts;
    const b64 = parts.find((p) => p.inlineData)?.inlineData.data;
    await writeFile("out.png", Buffer.from(b64, "base64"));
    console.log("saved out.png");
    ```

=== "图生图"

    ```python
    import base64, requests

    src = base64.b64encode(open("input.png", "rb").read()).decode()

    resp = requests.post(
        "https://www.llmnex.com/v1beta/models/gemini-3-pro-image-1k:generateContent",
        headers={"Authorization": "Bearer YOUR_API_KEY"},
        json={"contents": [{"parts": [
            {"text": "把这张照片改成水彩画风格,保留构图"},
            {"inlineData": {"mimeType": "image/png", "data": src}},
        ]}]},
        timeout=300,
    )
    parts = resp.json()["candidates"][0]["content"]["parts"]
    b64 = next(p["inlineData"]["data"] for p in parts if "inlineData" in p)
    open("out.png", "wb").write(base64.b64decode(b64))
    ```
