# AI Provider Management

AI Providers are the service configurations for AI models. By managing Providers and models, you can specify which AI models Agents and AI features use.

## Features

- Manage AI model service providers
- Manage available models under each Provider
- Set default models
- Enable/disable Providers
- Import/export configurations

## Provider List

The "AI Provider Management" page displays all configured Providers:

| Column | Description |
|:-------|:------------|
| Name | Provider name |
| API URL | Base URL |
| Models | Number of models under this Provider |
| Default Model | Model marked as default |
| Status | Enabled/Disabled |
| Actions | Edit, Delete, Expand model list |

## Creating a Provider

1. Click the "New" button
2. Fill in the Provider information:

| Field | Description |
|:------|:------------|
| Name | Provider display name (e.g., "OpenAI", "Local Model") |
| API Key | API key |
| Base URL | API endpoint (e.g., `https://api.openai.com/v1`) |
| Enabled | Whether to enable |

3. Save

> DBridge Pro is compatible with the OpenAI API protocol, so any service compatible with the OpenAI interface can be connected (e.g., OpenAI, Azure OpenAI, Ollama, vLLM, Tongyi Qianwen, etc.).

## Managing Models

Expand a Provider row to view and manage models under that Provider:

### Adding Models

| Field | Description |
|:------|:------------|
| Model Name | Model ID in the API (e.g., `gpt-4o`, `qwen-turbo`) |
| Display Name | Name shown in the interface |

### Setting Default Model

Each Provider can have one default model. When creating an Agent without specifying a model, the Provider's default model is used.

## Import / Export

### Exporting Configuration

Export Provider and model configurations as a JSON file:

1. Click the "Export" button
2. Download the JSON file

### Importing Configuration

Import Provider and model configurations from a JSON file:

1. Click the "Import" button
2. Select the JSON file
3. Confirm import

> Import/export is ideal for migrating AI configurations between environments.

## Supported AI Services

Since it's compatible with the OpenAI API protocol, the following services can all be connected:

| Provider | Base URL Example | Description |
|:---------|:----------------|:------------|
| OpenAI | `https://api.openai.com/v1` | GPT-4o, GPT-4, etc. |
| Azure OpenAI | `https://{resource}.openai.azure.com` | Azure-deployed OpenAI |
| Tongyi Qianwen | `https://dashscope.aliyuncs.com/compatible-mode/v1` | Alibaba Cloud large models |
| Zhipu AI | `https://open.bigmodel.cn/api/paas/v4` | GLM series |
| Moonshot | `https://api.moonshot.cn/v1` | Kimi / Moonshot |
| DeepSeek | `https://api.deepseek.com/v1` | DeepSeek series |
| Ollama | `http://localhost:11434/v1` | Locally deployed open-source models |
| vLLM | `http://localhost:8000/v1` | Locally deployed inference service |

## Configuration Recommendations

- **API Key Security** — API keys are stored encrypted and not displayed in plaintext in the interface
- **Multiple Providers** — Configure multiple Providers; different Agents can use different models
- **Local Models** — For high data security requirements, use local deployment solutions like Ollama
- **Cost Optimization** — Use cheaper smaller models for routine queries and larger models for complex analysis
