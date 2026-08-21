# use-codex

使用 Codex 时积累的配置笔记、排错经验和可复用 skill。

本仓库的第一份教程解决一个常见问题：如何让 Codex 接入不同的模型提供商，并在多个模型之间切换。示例使用 OpenCode Go，但方法同样适用于其他兼容服务。

## 快速理解

Codex 的模型配置可以拆成四层：

```text
model             = 具体模型 ID
model_provider    = 使用哪一个提供商配置
base_url          = API 基础地址
wire_api          = 请求协议（例如 responses）
```

`model` 只选择模型，`model_provider` 决定请求发往哪里。模型选择器显示哪些选项，则由 `model_catalog_json` 指向的模型目录决定。

## OpenCode Go：Responses API 示例

在 `config.toml` 中：

```toml
model = "gpt-5.6-luna"
model_provider = "opencodego_responses"

[model_providers.opencodego_responses]
name = "OpenCode Go (Responses)"
base_url = "https://opencode.ai/zen/go/v1"
wire_api = "responses"
env_key = "OPENCODE_GO_API_KEY"
```

先在 Windows 用户环境变量中保存密钥：

```powershell
[Environment]::SetEnvironmentVariable(
    "OPENCODE_GO_API_KEY",
    "在这里填入你的真实 API Key",
    "User"
)
```

不要把真实 Key 写进 Git、README、截图或聊天记录。

## 不同协议要分开配置

同一个 provider 只能有一个 `wire_api`。如果一个服务同时提供 Responses 和 Chat Completions，定义两个 provider：

```toml
[model_providers.opencodego_chat]
name = "OpenCode Go (Chat Completions)"
base_url = "https://opencode.ai/zen/go/v1"
wire_api = "chat"
env_key = "OPENCODE_GO_API_KEY"
```

使用 Responses 模型：

```toml
model = "gpt-5.6-luna"
model_provider = "opencodego_responses"
```

使用 Chat Completions 模型：

```toml
model = "deepseek-v4-flash"
model_provider = "opencodego_chat"
```

模型名称能显示在选择器中，不代表 API 一定支持它。调用前要确认模型 ID、协议、权限和上下文能力。

## 模型目录

`model_catalog_json` 指向的 JSON 文件决定模型选择器中的候选项。新增条目只能扩展界面目录，不能让服务商获得它本来不支持的模型。

建议先复制现有模型条目，再按服务商的真实能力修改 `slug`、显示名称、上下文长度、推理等级和工具能力。Codex 更新时可能重新生成模型目录，因此手工修改前应备份。

## 推荐排错顺序

1. 检查 `model_provider` 是否与 `[model_providers.<名称>]` 一致。
2. 检查 `base_url` 是否是 API 基础地址，而不是完整请求地址。
3. 检查 `wire_api` 是否和服务商协议一致。
4. 检查 `model` 是否是服务商真实的模型 ID。
5. 检查环境变量是否存在，并完全重启 Codex。
6. 如果模型能显示但调用失败，优先怀疑协议或模型 ID，而不是模型目录。

详细说明见：

- [模型提供商配置教程](docs/model-provider-configuration.md)
- [Codex 模型提供商配置 skill](skills/codex-model-provider/SKILL.md)
- [配置示例](examples/config.toml.example)

