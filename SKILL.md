---
name: agnes-image-gen
description: >-
  Agnes AI 图片生成 Skill。封装 Agnes Image 2.0/2.1 Flash API，
  支持文生图、图生图（编辑变换）、多图合成。API 完全免费，无需绑定信用卡。
  触发词：Agnes 生图、agnes 图片、用 Agnes 生成、agnes-image、agnes 画图。
agent_created: true
---

# Agnes AI Image Generation Skill

## 概述

Agnes AI 是全球 Top 10 AI Lab（新加坡），自 2025 年 6 月 1 日起**无限期免费**开放全模态 API。本 Skill 封装其图片生成能力。

### 快速开始（首次使用）

1. **注册**：访问 https://platform.agnes-ai.com 注册账号
2. **获取 API Key**：登录后进入「设置 → API 密钥」，创建新密钥并保存
3. **设置环境变量**：`export AGNES_API_KEY="你的API密钥"`

---

## 模型选型

> 两个模型使用**同一端点** `POST https://apihub.agnes-ai.com/v1/images/generations`，仅 `model` 参数不同。

| 特性 | `agnes-image-2.0-flash` | `agnes-image-2.1-flash` |
|------|--------------------------|--------------------------|
| **核心定位** | 图生图 / 图像编辑 / 多图合成 | 文生图（纯文本生成） |
| 文生图 | ✅ 支持 | ✅ **更强**（高信息密度优化） |
| 图生图 | ✅ **专精** | ✅ 支持 |
| 多图合成 | ✅ 支持 | ❌ 不支持 |
| `seed` 参数 | ✅ 支持（结果可复现） | ❌ 不支持 |
| `tags` 参数 | ✅ 需要（标记任务类型） | ❌ 不需要 |
| ELO 评分 | 1,184（Image Editing Leaderboard Top 20） | — |
| **推荐场景** | 编辑现有图片、多图融合、需要 seed 迭代 | 从零生成、复杂场景、社媒配图 |

**选择原则**：
- 从零生成图片 → 用 `2.1-flash`
- 编辑/变换已有图片 → 用 `2.0-flash` + `tags: ["img2img"]`
- 多张图片合成一张 → 用 `2.0-flash` + 多 URL 的 `image` 数组

---

## API 参考

### 基础信息

| 项目 | 详情 |
|------|------|
| **Base URL** | `https://apihub.agnes-ai.com/v1` |
| **图片端点** | `POST /v1/images/generations` |
| **认证** | `Authorization: Bearer <API_KEY>` |
| **Content-Type** | `application/json` |
| **兼容协议** | OpenAI 兼容 |
| **价格** | **免费** |

### 请求参数（通用）

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `model` | string | ✅ | `agnes-image-2.0-flash` 或 `agnes-image-2.1-flash` |
| `prompt` | string | ✅ | 文本提示词 |
| `size` | string | ❌ | `1024x768` / `1024x1024` / `768x1024` |
| `seed` | number | ❌ | （仅 2.0）随机种子，固定值可复现结果 |
| `tags` | array | ❌ | （仅 2.0）`["img2img"]` 标记图生图模式 |
| `extra_body.image` | array | ❌ | （图生图时必填）输入图片 URL 数组 |
| `extra_body.response_format` | string | ❌ | 输出格式，填 `"url"` |

### 响应格式

```json
{
  "created": 1774432125,
  "data": [
    {
      "url": "https://cdn.agnes-ai.com/generated/xxxxx.png"
    }
  ],
  "usage": {
    "generated_images": 1
  }
}
```

| 字段 | 说明 |
|------|------|
| `created` | Unix 时间戳（秒） |
| `data[].url` | 生成图片的访问 URL |
| `usage.generated_images` | 生图数量 |

---

## 调用示例

### 1. 文生图（Image 2.1-Flash）

**推荐模型**：`agnes-image-2.1-flash`

**curl**：
```bash
curl -s https://apihub.agnes-ai.com/v1/images/generations \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-image-2.1-flash",
    "prompt": "A luminous floating city above a misty canyon at sunrise, cinematic realism, 8K",
    "size": "1024x768"
  }' | jq -r '.data[0].url'
```

**Python (httpx)**：
```python
import httpx, os

resp = httpx.post(
    "https://apihub.agnes-ai.com/v1/images/generations",
    headers={"Authorization": f"Bearer {os.environ['AGNES_API_KEY']}"},
    json={
        "model": "agnes-image-2.1-flash",
        "prompt": "一只橘猫在海边散步，黄昏光线，写实风格",
        "size": "1024x768"
    },
    timeout=120
)
print(resp.json()["data"][0]["url"])
```

**Python (openai SDK)**：
```python
from openai import OpenAI

client = OpenAI(
    api_key="你的API_KEY",
    base_url="https://apihub.agnes-ai.com/v1"
)
resp = client.images.generate(
    model="agnes-image-2.1-flash",
    prompt="赛博朋克城市夜景，霓虹灯倒映在雨中的街道",
    size="1024x768"
)
print(resp.data[0].url)
```

### 2. 图生图 — 单图编辑（Image 2.0-Flash）

**推荐模型**：`agnes-image-2.0-flash` + `tags: ["img2img"]`

**Prompt 结构**：`[需要改变什么] + [需要保持什么不变]`

```bash
curl -s https://apihub.agnes-ai.com/v1/images/generations \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-image-2.0-flash",
    "prompt": "Transform into rain-soaked cyberpunk night with neon reflections, preserve original composition and main subject layout",
    "size": "1024x768",
    "seed": 42,
    "tags": ["img2img"],
    "extra_body": {
      "image": ["https://example.com/photo.jpg"],
      "response_format": "url"
    }
  }' | jq -r '.data[0].url'
```

**Python**：
```python
import httpx, os

resp = httpx.post(
    "https://apihub.agnes-ai.com/v1/images/generations",
    headers={"Authorization": f"Bearer {os.environ['AGNES_API_KEY']}"},
    json={
        "model": "agnes-image-2.0-flash",
        "prompt": "把这张照片改成水彩画风格，保持构图和主体不变",
        "size": "1024x768",
        "seed": 42,
        "tags": ["img2img"],
        "extra_body": {
            "image": ["https://example.com/photo.jpg"],
            "response_format": "url"
        }
    },
    timeout=120
)
print(resp.json()["data"][0]["url"])
```

### 3. 多图合成（Image 2.0-Flash）

**推荐模型**：`agnes-image-2.0-flash`，传入多张图片 URL

```python
import httpx, os

resp = httpx.post(
    "https://apihub.agnes-ai.com/v1/images/generations",
    headers={"Authorization": f"Bearer {os.environ['AGNES_API_KEY']}"},
    json={
        "model": "agnes-image-2.0-flash",
        "prompt": "将这两张场景无缝融合为一张全景构图",
        "size": "1024x768",
        "tags": ["img2img"],
        "extra_body": {
            "image": [
                "https://example.com/scene1.jpg",
                "https://example.com/scene2.jpg"
            ],
            "response_format": "url"
        }
    },
    timeout=120
)
print(resp.json()["data"][0]["url"])
```

---

## 执行指南

当用户请求生图时，按以下流程操作：

### Step 1：确认 API Key
- 从环境变量 `AGNES_API_KEY` 读取
- 如未设置，引导用户注册 https://platform.agnes-ai.com 并设置环境变量

### Step 2：判断任务类型 → 选模型

| 用户意图 | 模型 | 关键参数 |
|----------|------|----------|
| 从文字描述生成新图 | `agnes-image-2.1-flash` | 仅需 `prompt` + `size` |
| 编辑/变换已有图片 | `agnes-image-2.0-flash` | + `tags: ["img2img"]` + `extra_body.image` |
| 多张图合成一张 | `agnes-image-2.0-flash` | `extra_body.image` 数组传多 URL |
| 需要迭代优化（固定 seed） | `agnes-image-2.0-flash` | 加 `seed` 参数 |

### Step 3：调用 API
- 使用 Python `httpx` 发送 POST 请求
- 超时设为 120 秒
- 从响应中提取 `data[0].url`

### Step 4：展示结果
- 将生成的图片 URL 展示给用户
- 同时告知使用的模型和参数

---

## Prompt 写作建议

### 文生图 Prompt
- 使用英文效果更好（模型训练数据以英文为主）
- 包含：主体 + 场景 + 光线 + 风格 + 画质
- 示例：`A cyberpunk city street at night with neon signs, rain reflection on wet pavement, cinematic lighting, 8K, photorealistic`

### 图生图 Prompt（「改什么 + 保什么」结构）
- 明确要改变的元素（风格、色调、环境等）
- 明确要保持的元素（构图、主体、布局等）
- 使用 `while preserving...` 句式
- 示例：`Transform into oil painting style with warm tones while preserving the original composition and main subjects`

---

## 错误处理

| HTTP 状态 | 原因 | 处理 |
|-----------|------|------|
| 401 | API Key 无效或未传 | 检查 `AGNES_API_KEY` 是否正确设置 |
| 404 | 模型名称错误或端点不对 | 确认 model 参数拼写正确 |
| 429 | 速率限制 | 等待后重试（免费层有限速） |
| 500 | 服务端错误 | 稍后重试 |
| 超时 | 生图耗时较长 | 增加超时到 120s，或重试 |

---

## 补充：视频生成（Agnes Video V2.0）

本 Skill 聚焦图片生成。如需视频生成，端点如下：

- **发起任务**：`POST https://apihub.agnes-ai.com/v1/videos`
  ```json
  {
    "model": "agnes-video-v2.0",
    "prompt": "...",
    "height": 768,
    "width": 1152,
    "num_frames": 121,
    "frame_rate": 24
  }
  ```
- **查询结果**：`GET https://apihub.agnes-ai.com/v1/videos/{id}`
- 视频生成是**异步**的，需轮询 `status` 直到 `"completed"`
- `num_frames` 须满足 `8n+1`（81/121/161/241）

---

## 参考来源

- 官方平台：https://platform.agnes-ai.com
- 官方文档：https://agnes-ai.com/doc/overview
- API 地址：https://apihub.agnes-ai.com/v1
