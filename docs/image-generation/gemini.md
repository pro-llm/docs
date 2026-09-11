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
| 用量统计 | 非流式响应带 `usageMetadata`(token 数),见 [响应结构](#响应结构) |

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

`imageSize` 取值:`1K` / `2K` / `4K`。**只有这三档**:传 `512` 不会报错,按 `1K` 出图。

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

```json
{
  "contents": [
    { "parts": [ { "text": "a lighthouse on a cliff at dawn, cinematic" } ] }
  ],
  "generationConfig": {
    "imageConfig": { "aspectRatio": "16:9" }
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

!!! note "四个超长比例的像素比参考值大"
    `1:4` `4:1` `1:8` `8:1` 这四个比例能出图,但输出像素约为同档参考值的 **4 倍**
    (例如 `1:4` 在 1K 档出 1024 × 4128,而不是 512 × 2048),耗时也相应更长。
    `gemini-3-pro-image` 系列**不支持**这四个超长比例,会返回 400。

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

## 不生效的参数

下面这些官方参数本站会**接受但不生效**(返回 200,输出与不传时相同),请不要依赖:

| 参数 | 实际行为 |
| --- | --- |
| `generationConfig.candidateCount` | 恒只返回 1 张图,传 2 或更大也一样 |
| `generationConfig.seed` | 接受,但**不保证跨请求复现**,同 seed 同提示词也可能出不同的图 |
| `generationConfig.responseModalities` 含 `TEXT` | 只返回图片,不返回文字 |
| `tools` 里的 `googleSearch` | 不联网,响应无 `groundingMetadata` |
| `generationConfig.thinkingConfig` | 不生效,响应无思考 token 计数 |
| `contents[].parts[].fileData`(URL 参考图) | 不读取,参考图请用 `inlineData` 内嵌 |

## 流式返回(不建议使用)

!!! warning "建议用非流式的 `:generateContent`"
    **图像是一次性生成的,没有中间产物可以流式推送。**这个端点会先等整张图出完,
    再把最终结果包成**单个 SSE chunk** 发出 —— chunk 里的内容与 `generateContent`
    的响应体一致,只是**没有 `usageMetadata`**。

    也就是说,你付出了 SSE 解析的成本,却拿不到任何流式的好处:

    - **不会更早看到画面**,等待时间与非流式完全一致
    - **不能保持连接活跃** —— 出图期间服务端不发送任何字节,防不了中间层空闲超时
    - **整张图的 base64 挤在一行 `data:` 里**,常见 7–11 MB,部分 SSE 客户端和反向代理
      对单行长度有限制,反而更容易被截断
    - **出错时不走 SSE**,直接返回 JSON 错误体,客户端要额外处理两种响应格式
    - **不返回 `usageMetadata`**,需要用量统计必须用非流式

    真正防超时的做法是把客户端超时设到 300 秒,见下方「超时与耗时」。

保留该端点是为了兼容已有代码。用法是把端点换成 `:streamGenerateContent`,
返回 SSE 格式,每行以 `data: ` 开头。

```bash
curl -N -X POST "https://www.llmnex.com/v1beta/models/gemini-3-pro-image-1k:streamGenerateContent" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" --max-time 300 \
  -d '{"contents":[{"parts":[{"text":"a plain grey cube"}]}]}'
```

## 响应结构

非流式响应示例(`gemini-3-pro-image-1k` 文生图):

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
      },
      "finishReason": "STOP",
      "index": 0
    }
  ],
  "responseId": "ee2bcd924c5d",
  "upstreamJobId": "d427bc37-0f6c-458b-93c9-d40a0b5412f7",
  "usageMetadata": {
    "promptTokenCount": 4,
    "promptTokensDetails": [ { "modality": "TEXT", "tokenCount": 4 } ],
    "candidatesTokenCount": 1120,
    "candidatesTokensDetails": [ { "modality": "IMAGE", "tokenCount": 1120 } ],
    "totalTokenCount": 1124
  }
}
```

取图路径固定为 `candidates[0].content.parts[*].inlineData.data`。建议遍历 `parts` 找第一个带 `inlineData` 的元素,不要写死索引 0。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `candidates[]` | array | 恒 1 个元素 |
| `candidates[].content.role` | string | 恒 `model` |
| `candidates[].content.parts[].inlineData.mimeType` | string | 恒 `image/png` |
| `candidates[].content.parts[].inlineData.data` | string | base64 编码的 PNG |
| `candidates[].finishReason` | string | 正常出图为 `STOP` |
| `candidates[].index` | integer | 恒 `0` |
| `responseId` | string | 本站请求 ID,与响应头 `X-Oneapi-Request-Id` 相同 |
| `upstreamJobId` | string | 上游任务号,报障时连同 `responseId` 一起附上 |
| `usageMetadata` | object | 本次消耗的 token 数,见下 |

官方响应里的 `promptFeedback`、`modelVersion`、`safetyRatings`、`parts[].text` 本站不返回,请不要依赖。

### usageMetadata

`usageMetadata` 是本次请求消耗的 token 数,结构与官方一致:`promptTokenCount`(提示词与参考图)、
`candidatesTokenCount`(生成的图片)、`totalTokenCount`,以及 `*Details` 里按模态的拆分。
生成图片的 token 数只与模型和分辨率档有关。

!!! note "三点说明"
    - **流式(`:streamGenerateContent`)不返回 `usageMetadata`**。需要用量统计请用非流式。
    - `promptTokenCount` 为估算值,带参考图时尤其如此,仅供参考;`candidatesTokenCount` 按官方规则计算。
    - `usageMetadata` 是用量统计,不等于计费方式。实际扣费以账户账单为准。

!!! warning "4K 图的响应体很大"
    4K 图的 base64 约 **25–28 MB**。请确认你的 HTTP 客户端与中间层没有更小的响应体上限,
    并给传输留出时间:下方的耗时表是服务端出图时间,**不含下载**,弱网下 4K 建议把超时设到 600 秒。

## 超时与耗时

生图是长耗时操作。**客户端超时请设到 300 秒**,默认的 30 秒或 60 秒几乎必然中途断开。

| 档位 | 常见耗时(首字节) | 实测长尾 |
| --- | --- | --- |
| 1K | 15 – 45 s | ~90 s |
| 2K | 20 – 60 s | ~90 s |
| 4K | 40 – 80 s | ~120 s |

以上是服务端出图时间,图片传输另计(4K 响应约 25–28 MB)。
耗时受上游负载影响明显,高峰期会整体拉长。服务内部已有自动重试与账号轮换,你收到的错误都是重试之后仍未成功的终态结果 —— **不需要自己加激进的重试**,那只会加重排队。

## 错误处理

| HTTP | 含义 | 怎么办 |
| --- | --- | --- |
| 400 | 请求体不合法,如 `contents` 缺失或为空 | 检查 JSON 结构 |
| 401 | Key 无效或已过期 | 检查 `Authorization` 头 |
| 429 | 上游限流 | 已在内部重试过;**退避 2 – 5 秒后重试一次**通常即可成功,不要循环重试 |
| 451 | 内容被安全审核拦截 | **终态,重试无用。** 改提示词或参考图 |
| 500 | 上游临时不可用,或网关内部错误 | 稍后重试一次;持续出现请联系我们 |
| 503 | `model_not_found` —— 模型名写错,或你的 Key 没有该模型的权限 | 见下 |
| 503 | 其他消息(如 `upstream session error`) | 上游连接临时故障,退避后重试一次 |

每个响应都带 `X-Oneapi-Request-Id` 响应头。**报障时请附上这个值**,我们靠它定位到具体请求。

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

!!! tip "503 `No available channel for model ...`"
    这条**不是服务故障**,而是「这个 Key 用不了这个模型」:模型名拼错(比如漏了 `-1k` 后缀),
    或者你的 Key 没被授予该模型的权限。先用 `GET /v1/models` 确认自己能用哪些模型;
    列表里没有你要调的名字,就联系我们开通。

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
