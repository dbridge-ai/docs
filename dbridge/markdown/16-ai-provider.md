# AI Provider 管理

AI Provider 是 AI 模型的服务商配置。通过管理 Provider 和模型，可以为 Agent 和 AI 功能指定使用的 AI 模型。

## 功能概述

- 管理 AI 模型服务商
- 管理每个 Provider 下的可用模型
- 设置默认模型
- 启用/禁用 Provider
- 导入/导出配置

## Provider 列表

进入「AI Provider 管理」页面，展示所有已配置的 Provider：

| 列 | 说明 |
|:---|:-----|
| 名称 | Provider 名称 |
| API 地址 | Base URL |
| 模型数 | 该 Provider 下的模型数量 |
| 默认模型 | 标记为默认的模型 |
| 状态 | 启用/禁用 |
| 操作 | 编辑、删除、展开模型列表 |

## 创建 Provider

1. 点击「新建」按钮
2. 填写 Provider 信息：

| 字段 | 说明 |
|:-----|:-----|
| 名称 | Provider 显示名称（如 "OpenAI"、"本地模型"） |
| API Key | API 密钥 |
| Base URL | API 地址（如 `https://api.openai.com/v1`） |
| 启用 | 是否启用 |

3. 保存

> DBridge Pro 兼容 OpenAI API 协议，因此任何兼容 OpenAI 接口的服务都可以接入（如 OpenAI、Azure OpenAI、Ollama、vLLM、通义千问等）。

## 管理模型

展开 Provider 行，查看和管理该 Provider 下的模型：

### 添加模型

| 字段 | 说明 |
|:-----|:-----|
| 模型名称 | API 中的模型 ID（如 `gpt-4o`、`qwen-turbo`） |
| 显示名称 | 在界面上显示的名称 |

### 设置默认模型

每个 Provider 可以设置一个默认模型。创建 Agent 时，如果未指定模型，将使用对应 Provider 的默认模型。

## 导入/导出

### 导出配置

将 Provider 和模型配置导出为 JSON 文件：

1. 点击「导出」按钮
2. 下载 JSON 文件

### 导入配置

从 JSON 文件导入 Provider 和模型配置：

1. 点击「导入」按钮
2. 选择 JSON 文件
3. 确认导入

> 导入/导出适合在不同环境间迁移 AI 配置。

## 支持的 AI 服务

由于兼容 OpenAI API 协议，以下服务均可接入：

| 服务商 | Base URL 示例 | 说明 |
|:-------|:-------------|:-----|
| OpenAI | `https://api.openai.com/v1` | GPT-4o、GPT-4 等 |
| Azure OpenAI | `https://{resource}.openai.azure.com` | Azure 部署的 OpenAI |
| 通义千问 | `https://dashscope.aliyuncs.com/compatible-mode/v1` | 阿里云大模型 |
| 智谱 AI | `https://open.bigmodel.cn/api/paas/v4` | GLM 系列 |
| 月之暗面 | `https://api.moonshot.cn/v1` | Kimi / Moonshot |
| DeepSeek | `https://api.deepseek.com/v1` | DeepSeek 系列 |
| Ollama | `http://localhost:11434/v1` | 本地部署的开源模型 |
| vLLM | `http://localhost:8000/v1` | 本地部署的推理服务 |

## 配置建议

- **API Key 安全** — API Key 加密存储，不在界面明文展示
- **多 Provider** — 可以配置多个 Provider，不同 Agent 使用不同的模型
- **本地模型** — 对数据安全要求高的场景，建议使用 Ollama 等本地部署方案
- **成本优化** — 日常查询用便宜的小模型，复杂分析用大模型
