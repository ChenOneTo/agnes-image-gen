# Agnes Image Gen

> 通用 Agnes AI Image 2.0/2.1 Flash API 参考 — 文生图 · 图生图 · 多图合成，完全免费。

[![Agnes AI](https://img.shields.io/badge/Agnes_AI-Image_2.1_Flash-8b5cf6)](https://agnes-ai.com)
[![Free](https://img.shields.io/badge/Price-Free-success)](https://platform.agnes-ai.com)
[![License](https://img.shields.io/badge/License-MIT-blue)](LICENSE)
[![Languages](https://img.shields.io/badge/SDK-curl_|_Python_|_Node.js_|_Go-blue)](https://github.com/ChenOneTo/agnes-image-gen)

## 为什么用 Agnes

- 🆓 **完全免费**，无需绑定信用卡
- 🎯 两个模型覆盖：文生图（2.1） + 图生图/多图合成（2.0）
- 🔌 **OpenAI 兼容协议**，改一行 base_url 即可迁移
- 🌍 全球 Top 10 AI Lab（新加坡）

## 快速开始

```bash
# 1. 注册获取 Key → https://platform.agnes-ai.com
export AGNES_API_KEY="sk-your-api-key"

# 2. 一句命令生成图片
curl https://apihub.agnes-ai.com/v1/images/generations \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-image-2.1-flash",
    "prompt": "A serene mountain lake at golden hour, photorealistic, 8K",
    "size": "1024x768"
  }' | jq -r '.data[0].url'
```

## 模型选型

| | Image 2.0 Flash | Image 2.1 Flash |
|---|---|---|
| **定位** | 图像编辑 · 多图合成 | 文生图 |
| Text-to-Image | ✅ | ✅ **更强** |
| Image-to-Image | ✅ **专精** | ✅ |
| Multi-Image | ✅ | ❌ |
| seed 复现 | ✅ | ❌ |
| **推荐场景** | 编辑、融合、迭代 | 从零生成 |

## 代码示例

### 文生图 — Python

```python
import httpx, os

resp = httpx.post(
    "https://apihub.agnes-ai.com/v1/images/generations",
    headers={"Authorization": f"Bearer {os.environ['AGNES_API_KEY']}"},
    json={
        "model": "agnes-image-2.1-flash",
        "prompt": "A cyberpunk city at night, neon reflections on wet pavement",
        "size": "1024x768"
    },
    timeout=120
)
print(resp.json()["data"][0]["url"])
```

### 文生图 — Node.js

```js
const resp = await fetch("https://apihub.agnes-ai.com/v1/images/generations", {
  method: "POST",
  headers: {
    "Authorization": `Bearer ${process.env.AGNES_API_KEY}`,
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    model: "agnes-image-2.1-flash",
    prompt: "Futuristic city in the clouds, golden sunset, 8K",
    size: "1024x768"
  })
});
const { data } = await resp.json();
console.log(data[0].url);
```

### 图生图 — Python

```python
resp = httpx.post(
    "https://apihub.agnes-ai.com/v1/images/generations",
    headers={"Authorization": f"Bearer {os.environ['AGNES_API_KEY']}"},
    json={
        "model": "agnes-image-2.0-flash",
        "prompt": "Transform into watercolor painting style, preserve composition",
        "size": "1024x768",
        "tags": ["img2img"],
        "extra_body": {
            "image": ["https://example.com/photo.jpg"],
            "response_format": "url"
        }
    },
    timeout=120
)
```

### OpenAI SDK 兼容

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk-your-key",
    base_url="https://apihub.agnes-ai.com/v1"
)
resp = client.images.generate(
    model="agnes-image-2.1-flash",
    prompt="Cyberpunk city night scene, neon lights",
    size="1024x768"
)
print(resp.data[0].url)
```

## API 参考

| 项目 | 值 |
|------|-----|
| Base URL | `https://apihub.agnes-ai.com/v1` |
| Endpoint | `POST /images/generations` |
| Auth | `Bearer <API_KEY>` |
| Protocol | OpenAI Compatible |

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `model` | string | ✅ | `agnes-image-2.0-flash` / `agnes-image-2.1-flash` |
| `prompt` | string | ✅ | 提示词（英文效果更佳） |
| `size` | string | ❌ | `1024x768` / `1024x1024` / `768x1024` |
| `seed` | number | ❌ | 仅 2.0：固定种子复现结果 |
| `tags` | array | ❌ | 仅 2.0：`["img2img"]` 标记图生图 |
| `extra_body.image` | array | ❌ | 图生图时填输入图片 URL |
| `extra_body.response_format` | string | ❌ | `"url"` |

## Prompt 技巧

**文生图**：主体 + 场景 + 光线 + 风格 + 画质

```
A [subject] in [scene], [lighting], [style], [quality]
```

**图生图**：改什么 + 保什么

```
Transform into [new style] while preserving [original elements]
```

## 错误处理

| 状态码 | 原因 | 解决 |
|--------|------|------|
| 401 | Key 无效 | 检查 `AGNES_API_KEY` |
| 429 | 限速 | 等待 5-10s 重试 |
| 500 | 服务异常 | 等待后重试 |

## 运行测试

```bash
# 设置 API Key
export AGNES_API_KEY="sk-your-key"

# 克隆本仓库
git clone https://github.com/ChenOneTo/agnes-image-gen.git
cd agnes-image-gen

# 查看完整 API 参考
cat SKILL.md
```

## 相关链接

- [Agnes AI 官网](https://agnes-ai.com)
- [API 文档](https://agnes-ai.com/doc/overview)
- [注册平台](https://platform.agnes-ai.com)
- [完整 API 参考 (SKILL.md)](./SKILL.md)

## 许可

MIT
