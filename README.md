# OpenAI Responses API to Chat Completions API# OpenAI响应API到聊天完成API

OpenAI Responses API → Chat Completions API 转接代理，让 Codex CLI 连接任意 Chat Completions 后端。

## 功能

- 将 Codex CLI 的 Responses API 请求转换为标准 Chat Completions 格式
- 支持 SSE 流式响应转换
- 支持 reasoning_content（DeepSeek、GLM 等推理模型）
- 支持 tool_calls 合并与格式修复
- 支持 backend_api_key 认证
- 支持多供应商动态切换

## 快速开始

### 1. 安装依赖

```bash   ”“bash
npm install
```

### 2. 配置

复制配置文件并填入你的后端信息：

```bash   ”“bash   “bash”;“bash
cp config/config.example.json config/config.jsoncp配置/ config.example。json配置/ json
```

编辑 `config/config.json`：

```json
{
  "port": 57321,
  "host": "127.0.0.1",   "host": "127.0.0.1",
  "model_map": {   "model_map": {
    "gpt-5.4-mini": "deepseek-v4-flash"
  },
  "models": [
    { "id": "deepseek-v4-flash", "owned_by": "deepseek" }
  ],
  "suppliers": [
    {
      "id": "deepseek",
      "name": "DeepSeek",
      "backend_url": "https://api.deepseek.com/v1",
      "backend_api_key": "sk-your-api-key-here",
      "enabled": true,
      "priority": 1
    }
  ]
}
```

### 3. 配置 Codex

编辑 `~/.codex/config.toml`：

```
model = "gpt-5.4-mini"
model_provider = "local-proxy"
model_reasoning_effort = "low"

[model_providers.local-proxy]
name = "local proxy"
base_url = "http://127.0.0.1:57321/v1"
experimental_bearer_token = "sk-any-value"
wire_api = "responses"
approval_policy = "never"
sandbox_mode = "danger-full-access"

```

### 4. 启动代理

```bash
node proxy.js
```

### 5. 启动 Codex

```bash
codex
```

## 配置说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `port` | number | 监听端口，默认 57321 |
| `host` | string | 监听地址，默认 127.0.0.1 |
| `model_map` | object | 模型名映射，将 Codex 模型名映射到后端模型名 |
| `models` | array | `/v1/models` 接口返回的模型列表 |
| `suppliers` | array | 供应商列表（见下方） |

### 供应商配置

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | string | 供应商唯一标识 |
| `name` | string | 供应商名称 |
| `backend_url` | string | 后端 Chat Completions API 地址 |
| `backend_api_key` | string | 后端 API Key（可选） |
| `enabled` | boolean | 是否启用，默认 true |
| `priority` | number | 优先级，数字越小越优先 |

## 工作原理

```text
Codex CLI  -->  Proxy (Responses API)  -->  Backend (Chat Completions API)
    <--               <--                        <--
```

1. Codex 发送 Responses API 格式请求
2. 代理转换为 Chat Completions 格式
3. 转发到配置的后端
4. 将后端的 SSE 流式响应转换回 Responses API 格式

## API 路由

| 路由 | 方法 | 说明 |
|------|------|------|
| `/v1/responses` | POST | 核心转换路由 |
| `/v1/chat/completions` | POST | 直接转发 |
| `/v1/models` | GET | 返回模型列表 |
| `/v1/suppliers` | GET | 列出所有供应商 |
| `/v1/suppliers` | POST | 添加供应商 |
| `/v1/suppliers/:id` | GET | 获取供应商详情 |
| `/v1/suppliers/:id` | PUT | 更新供应商 |
| `/v1/suppliers/:id` | DELETE | 删除供应商 |
| `/v1/suppliers/:id/toggle` | POST | 启用/禁用供应商 |
| `/health` | GET | 健康检查 |

## 支持的后端

- DeepSeek（deepseek-v4-flash）
- GLM（glm-4）
- 任意 OpenAI 兼容的 Chat Completions API

## 日志

日志文件位于 `logs/YYYY-MM-DD.log`，同时输出到控制台。

## License

ISC
