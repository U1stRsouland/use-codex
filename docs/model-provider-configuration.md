# Codex 接入不同模型提供商和模型

## 1. 配置关系

Codex 使用顶部两个字段确定当前请求：

```toml
model = "gpt-5.6-luna"
model_provider = "opencodego_responses"
```

然后按 `model_provider` 查找对应的 provider 段：

```toml
[model_providers.opencodego_responses]
name = "OpenCode Go (Responses)"
base_url = "https://opencode.ai/zen/go/v1"
wire_api = "responses"
env_key = "OPENCODE_GO_API_KEY"
```

字段含义：

- `model`：发送给服务商的模型 ID，必须使用服务商认可的值。
- `model_provider`：provider 段的内部名称。
- `name`：界面或日志中的显示名称。
- `base_url`：API 基础 URL。
- `wire_api`：请求协议。`responses`、`chat` 等值不能混用。
- `env_key`：从环境变量读取认证密钥。

## 2. 安全保存 API Key

推荐在 Windows 用户环境变量中保存密钥，而不是写入 TOML：

```powershell
[Environment]::SetEnvironmentVariable(
    "OPENCODE_GO_API_KEY",
    "你的真实 API Key",
    "User"
)
```

检查变量是否存在时不要打印完整密钥：

```powershell
$key = [Environment]::GetEnvironmentVariable(
    "OPENCODE_GO_API_KEY",
    "User"
)
if ([string]::IsNullOrWhiteSpace($key)) {
    "API Key 未设置"
} else {
    "API Key 已设置，长度为 $($key.Length)"
}
```

然后在配置中使用：

```toml
env_key = "OPENCODE_GO_API_KEY"
```

设置用户环境变量后需要重启 Codex。仅设置 `$env:OPENCODE_GO_API_KEY` 通常只对当前 PowerShell 进程及其子进程生效。

## 3. OpenCode Go 的 Responses 模型

如果服务商明确说明模型使用 Responses API：

```toml
model = "gpt-5.6-luna"
model_provider = "opencodego_responses"

[model_providers.opencodego_responses]
name = "OpenCode Go (Responses)"
base_url = "https://opencode.ai/zen/go/v1"
wire_api = "responses"
env_key = "OPENCODE_GO_API_KEY"
```

## 4. 同一服务的 Chat Completions 模型

不要把 Chat Completions 模型放到 `wire_api = "responses"` 的 provider 下。为它建立另一个 provider：

```toml
[model_providers.opencodego_chat]
name = "OpenCode Go (Chat Completions)"
base_url = "https://opencode.ai/zen/go/v1"
wire_api = "chat"
env_key = "OPENCODE_GO_API_KEY"
```

切换时同时修改：

```toml
model = "deepseek-v4-flash"
model_provider = "opencodego_chat"
```

## 5. 模型选择器为什么会显示多个模型

当配置包含：

```toml
model_catalog_json = "C:/Users/你的用户名/.codex/models.json"
```

Codex 会从该 JSON 文件读取候选模型。顶部的 `model` 只决定默认选项，不会限制列表中只能显示一个模型。

模型目录条目至少应该包含清晰的 `slug`、`display_name`、`visibility` 和 `supported_in_api`。更完整的字段示例见 `examples/models-entry.template.json`。

重要：把一个模型写进目录，只能让它“可显示/可尝试选择”，不能证明服务商真的支持它。模型 ID、API 协议、Key 权限和模型能力都需要单独确认。

## 6. 修改检查清单

- 先复制 `config.toml` 和 `models.json` 作为备份。
- provider 名称只使用字母、数字、下划线或短横线。
- `model_provider` 必须与 provider 段名称完全一致。
- `base_url` 不要重复拼接 `/responses` 或 `/chat/completions`。
- `wire_api` 必须和服务商实际接口一致。
- 不要将 API Key 写入配置、示例、日志或版本库。
- 修改后完全重启 Codex。
- 先用一个已确认支持的模型做最小调用测试。

