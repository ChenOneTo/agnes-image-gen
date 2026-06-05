---
name: agnes-image-gen
description: Agnes AI Image Generation API Reference — Image 2.0/2.1 Flash, text-to-image & image-to-image.
agent_created: true
---

# Agnes AI · Image Generation API

> [Agnes AI](https://agnes-ai.com) 是全球 Top 10 AI Lab（新加坡），自 2025 年 6 月 1 日起**无限期免费**开放全模态 API，无需绑定信用卡。

## 目录

- [快速开始](#快速开始)
- [模型选型](#模型选型)
- [API 参考](#api-参考)
- [文生图 Text-to-Image](#文生图-text-to-image)
- [图生图 Image-to-Image](#图生图-image-to-image)
- [多图合成 Multi-Image Composition](#多图合成-multi-image-composition)
- [OpenAI SDK 兼容](#openai-sdk-兼容)
- [Prompt 写作指南](#prompt-写作指南)
- [错误处理](#错误处理)
- [速率限制](#速率限制)
- [视频生成](#视频生成)

---

## 快速开始

### 1. 获取 API Key

访问 [platform.agnes-ai.com](https://platform.agnes-ai.com) 注册，进入设置 → API 密钥 → 创建新密钥。

### 2. 配置环境变量

```bash
export AGNES_API_KEY="sk-your-api-key"
```

### 3. 发送第一个请求

```bash
curl https://apihub.agnes-ai.com/v1/images/generations \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-image-2.1-flash",
    "prompt": "A serene mountain lake at golden hour, photorealistic, 8K",
    "size": "1024x768"
  }'
```

---

## 模型选型

| | Image 2.0 Flash | Image 2.1 Flash |
|---|---|---|
| **定位** | 图像编辑 · 多图合成 | 文生图 |
| Text-to-Image | ✅ | ✅ **更强** |
| Image-to-Image | ✅ **专精** | ✅ |
| Multi-Image | ✅ | ❌ |
| `seed` 复现 | ✅ | ❌ |
| `tags` 必需 | ✅（`["img2img"]`） | ❌ |

**选择规则**：

| 场景 | 模型 | 关键参数 |
|------|------|----------|
| 从零生成 | `2.1-flash` | `prompt` + `size` |
| 编辑现有图片 | `2.0-flash` | `tags: ["img2img"]` + `image` |
| 多张图合成 | `2.0-flash` | `image` 数组 |
| 需要迭代调优 | `2.0-flash` | 固定 `seed` 值 |

---

## API 参考

### Base

```
Base URL:  https://apihub.agnes-ai.com/v1
Endpoint:  POST /images/generations
Auth:      Bearer <API_KEY>
Protocol:  OpenAI Compatible
Price:     Free
```

### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `model` | `string` | ✅ | `agnes-image-2.0-flash` / `agnes-image-2.1-flash` |
| `prompt` | `string` | ✅ | 提示词，英文效果更佳 |
| `size` | `string` | | `1024x768` / `1024x1024` / `768x1024` |
| `seed` | `number` | | 仅 2.0：固定种子使结果可复现 |
| `tags` | `string[]` | | 仅 2.0：`["img2img"]` 标记图生图 |
| `extra_body.image` | `string[]` | | 图生图时必填：输入图片 URL 数组 |
| `extra_body.response_format` | `string` | | `"url"` |

### 响应

```json
{
  "created": 1774432125,
  "data": [{ "url": "https://cdn.agnes-ai.com/generated/xxx.png" }],
  "usage": { "generated_images": 1 }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `created` | `number` | Unix 时间戳 |
| `data[].url` | `string` | 生成图片的 CDN 地址 |
| `usage.generated_images` | `number` | 本次消耗的图片数 |

---

## 文生图 Text-to-Image

**模型**：`agnes-image-2.1-flash`

### curl

```bash
curl https://apihub.agnes-ai.com/v1/images/generations \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-image-2.1-flash",
    "prompt": "A cyberpunk city street at night, neon signs, rain-slicked pavement, cinematic lighting",
    "size": "1024x768"
  }' | jq -r '.data[0].url'
```

### Python

```python
import httpx, os

resp = httpx.post(
    "https://apihub.agnes-ai.com/v1/images/generations",
    headers={"Authorization": f"Bearer {os.environ['AGNES_API_KEY']}"},
    json={
        "model": "agnes-image-2.1-flash",
        "prompt": "水墨画风格，竹林深处一位侠客独行，晨雾弥漫",
        "size": "1024x768"
    },
    timeout=120
)
image_url = resp.json()["data"][0]["url"]
```

### Node.js

```js
const API_KEY = process.env.AGNES_API_KEY;

const resp = await fetch("https://apihub.agnes-ai.com/v1/images/generations", {
  method: "POST",
  headers: {
    "Authorization": `Bearer ${API_KEY}`,
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    model: "agnes-image-2.1-flash",
    prompt: "Futuristic city floating in the clouds, golden sunset, 8K",
    size: "1024x768"
  })
});
const { data } = await resp.json();
console.log(data[0].url);
```

### Go

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
    "os"
)

func main() {
    body, _ := json.Marshal(map[string]any{
        "model":  "agnes-image-2.1-flash",
        "prompt": "A lone samurai under cherry blossoms, ukiyo-e style",
        "size":   "1024x768",
    })
    req, _ := http.NewRequest("POST",
        "https://apihub.agnes-ai.com/v1/images/generations",
        bytes.NewReader(body))
    req.Header.Set("Authorization", "Bearer "+os.Getenv("AGNES_API_KEY"))
    req.Header.Set("Content-Type", "application/json")

    resp, _ := http.DefaultClient.Do(req)
    var result map[string]any
    json.NewDecoder(resp.Body).Decode(&result)
    fmt.Println(result["data"].([]any)[0].(map[string]any)["url"])
}
```

---

## 图生图 Image-to-Image

**模型**：`agnes-image-2.0-flash` · **标签**：`tags: ["img2img"]`

> Prompt 结构：`[改变什么] + keep/preserve [保持什么]`

### curl

```bash
curl https://apihub.agnes-ai.com/v1/images/generations \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-image-2.0-flash",
    "prompt": "Transform into oil painting with warm tones, keep original composition",
    "size": "1024x768",
    "seed": 42,
    "tags": ["img2img"],
    "extra_body": {
      "image": ["https://example.com/photo.jpg"],
      "response_format": "url"
    }
  }' | jq -r '.data[0].url'
```

### Python

```python
resp = httpx.post(
    "https://apihub.agnes-ai.com/v1/images/generations",
    headers={"Authorization": f"Bearer {os.environ['AGNES_API_KEY']}"},
    json={
        "model": "agnes-image-2.0-flash",
        "prompt": "改为水彩风格，保留原始构图和人物位置",
        "size": "1024x768",
        "seed": 42,
        "tags": ["img2img"],
        "extra_body": {
            "image": ["https://example.com/input.jpg"],
            "response_format": "url"
        }
    },
    timeout=120
)
```

### Node.js

```js
const resp = await fetch("https://apihub.agnes-ai.com/v1/images/generations", {
  method: "POST",
  headers: {
    "Authorization": `Bearer ${process.env.AGNES_API_KEY}`,
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    model: "agnes-image-2.0-flash",
    prompt: "Transform into cyberpunk night scene, keep character position",
    size: "1024x768",
    seed: 42,
    tags: ["img2img"],
    extra_body: {
      image: ["https://example.com/input.jpg"],
      response_format: "url"
    }
  })
});
```

---

## 多图合成 Multi-Image Composition

**模型**：`agnes-image-2.0-flash` · 在 `extra_body.image` 传入多个 URL。

```python
resp = httpx.post(
    "https://apihub.agnes-ai.com/v1/images/generations",
    headers={"Authorization": f"Bearer {os.environ['AGNES_API_KEY']}"},
    json={
        "model": "agnes-image-2.0-flash",
        "prompt": "Fuse these two scenes into one panoramic composition",
        "size": "1024x768",
        "tags": ["img2img"],
        "extra_body": {
            "image": [
                "https://example.com/scene_left.jpg",
                "https://example.com/scene_right.jpg"
            ],
            "response_format": "url"
        }
    },
    timeout=120
)
```

---

## OpenAI SDK 兼容

Agnes Image API 兼容 OpenAI 的 `images.generate` 调用协议，可直接替换 `base_url`：

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk-your-key",
    base_url="https://apihub.agnes-ai.com/v1"
)

# 文生图
resp = client.images.generate(
    model="agnes-image-2.1-flash",
    prompt="赛博朋克城市夜景",
    size="1024x768"
)

# 图生图（需通过 extra_body 传参）
resp = client.images.generate(
    model="agnes-image-2.0-flash",
    prompt="改为水彩风格",
    size="1024x768",
    extra_body={
        "tags": ["img2img"],
        "image": ["https://example.com/input.jpg"],
        "response_format": "url"
    }
)

print(resp.data[0].url)
```

---

## Prompt 写作指南

### 文生图公式

> **主体 + 场景 + 光线 + 风格 + 画质**

```
A [subject] in/at [scene], [lighting], [style], [quality]

示例:
A majestic white tiger standing on a snowy cliff edge,
northern lights in the night sky, rim lighting,
cinematic realism, 8K ultra HD
```

- 英文效果优于中文（训练数据以英文为主）
- 越具体越好，避免模糊词汇（"beautiful"、"nice"）

### 图生图公式

> **改什么 + 保什么**

```
Transform into [new style/effect] while keeping/preserving [elements]

示例:
Transform into anime style with soft pastel colors
while preserving the original composition and character poses
```

### 尺寸选择

| 尺寸 | 适用场景 |
|------|----------|
| `1024x768` | 风景、电影感、社交横幅 |
| `768x1024` | 人物肖像、竖版海报 |
| `1024x1024` | 头像、方形社媒配图 |

---

## 错误处理

| 状态码 | 原因 | 解决方案 |
|--------|------|----------|
| `200` | 成功 | 从 `data[0].url` 取图 |
| `401` | API Key 无效 | 检查环境变量或 Key 是否正确 |
| `404` | 端点/模型不存在 | 确认 URL 和 model 名称拼写 |
| `429` | 触发速率限制 | 等待 5-10 秒后重试 |
| `500` | 服务端错误 | 等待后重试，持续失败请联系官方 |
| 超时 | 生图耗时 > timeout | 客户端设 ≥ 120s |

---

## 速率限制

免费层有每日调用量和并发限制，对个人开发、原型验证、日常使用完全足够。商业级用量请参考 [Agnes AI 官方](https://agnes-ai.com) 的付费方案。

---

## 视频生成

Agnes 同时提供视频生成模型 `agnes-video-v2.0`（异步）：

```bash
# 1. 提交生成任务
curl -X POST https://apihub.agnes-ai.com/v1/videos \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-video-v2.0",
    "prompt": "A cat walking on the beach at sunset, gentle waves, 5 seconds",
    "height": 768, "width": 1152,
    "num_frames": 121, "frame_rate": 24
  }'
# → { "id": "task_xxx", "status": "queued" }

# 2. 轮询获取结果
curl https://apihub.agnes-ai.com/v1/videos/task_xxx \
  -H "Authorization: Bearer $AGNES_API_KEY"
# → { "status": "completed", "video_url": "https://..." }
```

- `num_frames` 须为 `8n+1`：81 / 121 / 161 / 241
- `frame_rate` 推荐 24
- 请求间隔建议 ≥ 10 秒

---

## 相关链接

| 资源 | 地址 |
|------|------|
| 官网 | https://agnes-ai.com |
| 注册 | https://platform.agnes-ai.com |
| API 文档 | https://agnes-ai.com/doc/overview |
| API Base | https://apihub.agnes-ai.com/v1 |
