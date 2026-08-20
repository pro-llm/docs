# 生图介绍

本站提供两套生图模型,按需选用。

| 系列 | 协议 | 最高分辨率 | 宽高比 | 指南 |
| --- | --- | --- | --- | --- |
| **Nano Banana(香蕉)** | Gemini 兼容 | 4096² | 14 种 | [Gemini 接入指南](gemini.md) |
| **GPT Image 2** | OpenAI 兼容 | 3840×2160 | **尺寸自选** | [GPT Image 2 接入指南](gpt-image-2.md) |

香蕉的分辨率上限更高(4096²),但尺寸只能从 12 个模型 × 14 种比例里挑;
GPT Image 2 的尺寸是**逐像素自选**的,且用标准 OpenAI SDK 就能调。

## 香蕉系列

三个模型族 × 四个分辨率后缀,共 **12 个**可用模型。

| 模型族 | 后端 | 特点 |
| --- | --- | --- |
| `gemini-3-pro-image` | Nano Banana Pro | 质量最高,耗时最长,复杂提示词表现最好 |
| `gemini-3.1-flash-image` | Nano Banana 2 | 速度与质量均衡,**日常首选** |
| `gemini-2.5-flash-image` | Nano Banana 2 | 兼容旧命名,行为同上 |

## 分辨率档位

**分辨率由模型名后缀决定**,不需要额外传参数:

| 后缀 | 分辨率 | 实测输出(1:1) |
| --- | --- | --- |
| `-1k` | 1K | 1024 × 1024 |
| `-2k` | 2K | 2048 × 2048 |
| `-4k` | 4K | 4096 × 4096 |
| `-c` | 自选 | 由请求里的 `imageSize` 决定,不传则 1K |

例:`gemini-3-pro-image-2k` 就是 Pro 模型出 2K 图。

## 能力

| 能力 | 支持 |
| --- | --- |
| 文生图 | ✅ |
| 图生图 / 改图 | ✅ 最多 14 张参考图 |
| 宽高比 | ✅ 14 种 |
| 流式返回 | ✅ |
| 单次出图张数 | 1 张 |
| 返回形态 | base64 PNG(不返回外链 URL) |

## 接入方式

=== "推荐:Gemini 原生协议"

    `POST /v1beta/models/{model}:generateContent`

    **完整支持**宽高比、参考图、流式。详见 [Gemini 接入指南](gemini.md)。

=== "OpenAI 兼容协议"

    `POST /v1/chat/completions`

    适合已有 OpenAI SDK 的项目,但**设置不了宽高比**(`aspectRatio` 不在 OpenAI 参数集里)。
    这条路径针对香蕉系列;GPT Image 2 请用 `/v1/images/generations` 与 `/v1/images/edits`。

## GPT Image 2

**尺寸逐像素自选**,不是从固定档位里挑 —— `size` 写多大就出多大:

```json
{"model": "gpt-image-2-medium", "prompt": "...", "size": "2048x1152"}
```

约束四条:宽高都是 **16 的倍数**、最长边 **≤ 3840**、比例 **≤ 3:1**、
总像素 **655,360 ~ 8,294,400**。不满足或不传则自动用 `1024x1024`(不报错)。

模型名只决定 `quality` 的默认值,**不影响尺寸**:

| 模型 | 默认 `quality` |
| --- | --- |
| `gpt-image-2-low` | `low` |
| `gpt-image-2-medium` | `medium` |
| `gpt-image-2-high` | `high` |
| `gpt-image-2-c` / `gpt-image-2` | 由你指定,不传则 `low` |

两个端点:`/v1/images/generations`(文生图)、`/v1/images/edits`(图生图)。
细节见 [GPT Image 2 接入指南](gpt-image-2.md)。

## 耗时参考

| 档位 | 常见耗时 | 实测上限 |
| --- | --- | --- |
| 1K | 20 – 45 s | ~90 s |
| 2K | 30 – 90 s | ~290 s |
| 4K | 60 – 130 s | ~290 s |

高峰期会整体拉长。**请把客户端超时设到 300 秒。**
