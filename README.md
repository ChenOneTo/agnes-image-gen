# Agnes Image Gen — WorkBuddy Skill

> 封装 Agnes AI Image 2.0/2.1 Flash API，在 WorkBuddy 对话中一键生图。

[![Agnes AI](https://img.shields.io/badge/Agnes_AI-Image_2.1_Flash-8b5cf6?logo=data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48cmVjdCB3aWR0aD0iMjQiIGhlaWdodD0iMjQiIHJ4PSI2IiBmaWxsPSIjOGI1Y2Y2Ii8+PHRleHQgeD0iMTIiIHk9IjE2IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBmaWxsPSJ3aGl0ZSIgZm9udC1zaXplPSIxMiIgZm9udC13ZWlnaHQ9ImJvbGQiPkFHPC90ZXh0Pjwvc3ZnPg==)](https://agnes-ai.com)
[![Free](https://img.shields.io/badge/Price-Free-success)](https://platform.agnes-ai.com)
[![License](https://img.shields.io/badge/License-MIT-blue)](LICENSE)

## 功能

- **文生图**：从文字描述生成高质量图片（推荐 `agnes-image-2.1-flash`）
- **图生图**：编辑/变换已有图片（推荐 `agnes-image-2.0-flash`）
- **多图合成**：多张图片融合为一张

## 快速开始

### 1. 获取 API Key

访问 [platform.agnes-ai.com](https://platform.agnes-ai.com) 注册，在设置中创建 API 密钥。

### 2. 安装 Skill

```bash
# 复制到 WorkBuddy skills 目录
cp -r agnes-image-gen ~/.workbuddy/skills/
```

### 3. 配置环境变量

```bash
# Linux / macOS
export AGNES_API_KEY="sk-your-api-key"

# Windows PowerShell
[System.Environment]::SetEnvironmentVariable('AGNES_API_KEY', 'sk-your-api-key', 'User')
```

### 4. 使用

在 WorkBuddy 对话中直接说：

```
用 Agnes 生成一张图片：一只橘猫在海边散步，黄昏光线，写实风格
```

触发词：**Agnes 生图**、**agnes 画图**、**用 Agnes 生成**

## 模型对比

| | Image 2.0 Flash | Image 2.1 Flash |
|---|---|---|
| **定位** | 图生图 / 图像编辑专精 | 文生图专精 |
| 文生图 | ✅ | ✅ **更强** |
| 图生图 | ✅ **专精** | ✅ |
| 多图合成 | ✅ | ❌ |
| seed 复现 | ✅ | ❌ |
| **推荐场景** | 编辑、融合、迭代 | 从零生成 |

## API 端点

```
POST https://apihub.agnes-ai.com/v1/images/generations
Authorization: Bearer <API_KEY>
```

### 文生图示例

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

### 图生图示例

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

## 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `model` | string | ✅ | `agnes-image-2.0-flash` 或 `agnes-image-2.1-flash` |
| `prompt` | string | ✅ | 提示词（英文效果更佳） |
| `size` | string | ❌ | `1024x768` / `1024x1024` / `768x1024` |
| `seed` | number | ❌ | 随机种子（仅 2.0） |
| `tags` | array | ❌ | `["img2img"]`（仅 2.0 图生图时） |
| `extra_body.image` | array | ❌ | 输入图片 URL 数组 |
| `extra_body.response_format` | string | ❌ | 填 `"url"` |

## Prompt 写作技巧

**文生图公式**：主体 + 场景 + 光线 + 风格 + 画质

```
A [subject] in [scene] with [lighting], [style], [quality]
```

**图生图公式**：改什么 + 保什么

```
Transform into [new style] while preserving [original elements]
```

## 常见错误

| 状态码 | 原因 | 解决 |
|--------|------|------|
| 401 | Key 无效 | 检查 `AGNES_API_KEY` |
| 429 | 限速 | 稍后重试 |
| 500 | 服务异常 | 重试 |

## 相关链接

- [Agnes AI 官网](https://agnes-ai.com)
- [API 文档](https://agnes-ai.com/doc/overview)
- [注册平台](https://platform.agnes-ai.com)

## 许可

MIT © 2025
