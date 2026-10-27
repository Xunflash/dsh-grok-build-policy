# Grok Build policy

[中文](#中文) · [English](#english)

## 中文

DeepSeek Harness 插件。当前模型 id 包含 `grok` 时，在系统提示词里追加一段工作规则。换成其他模型后，这段规则不会出现。

规则改编自 [xai-org/grok-build](https://github.com/xai-org/grok-build)（Apache-2.0）的默认系统提示词，只保留能在 Harness 里独立成立的部分。原文在：

[`crates/codegen/xai-grok-agent/templates/prompt.md`](https://github.com/xai-org/grok-build/blob/main/crates/codegen/xai-grok-agent/templates/prompt.md)

运行时由 [`PromptContext::render_with_renderer`](https://github.com/xai-org/grok-build/blob/main/crates/codegen/xai-grok-agent/src/prompt/context.rs) 渲染。同目录还有两份模板，本插件没有采用：`apply_patch_prompt.md`（Codex 模式）和 `subagent_prompt.md`（子代理）。

追加的内容是一段 `<model_work_policy>`，里面有六块：

- 危险操作先看可逆性和影响范围
- 明确的可逆工作在当前回合做完，结论要有工具输出
- 临时文件放在系统临时目录，用完删除
- 说话方式
- Markdown 格式
- 改了网页界面就在浏览器里验证

它不替换 Harness 的身份句和工具说明，也不携带 Grok Build 的工具名、记忆目录或 `apply_patch` 协议。

### 安装

把下面命令里的 `web` 换成要启用的 profile 名称。安装进该 profile 的全部会话。

从 npm 安装：

```sh
dsh plugin --profile web add grok-build-policy
```

从 GitHub 安装：

```sh
dsh plugin --profile web add github:Xunflash/grok-build-policy
```

改源码时从本机目录安装：

```sh
dsh plugin --profile web add /absolute/path/to/grok-build-policy
```

装好后检查这一行是否在组合结果里：

```sh
dsh --profile web --dump-config
```

输出中应有 `id: grok-build-policy`。Web profile 开了配置热重载时，下一次模型请求就会用上；没有热重载时，重启这个 profile。

### 配置

匹配在每次组装系统提示词时进行，不区分大小写。`modelPattern` 和 `providerPattern` 都是子串。

| 字段 | 默认 | 作用 |
|---|---|---|
| `modelPattern` | `grok` | 模型 id 需要包含的子串。留空则不按模型过滤。 |
| `providerPattern` | 空 | 提供方 id 需要包含的子串。留空则不限制提供方。 |

两个都留空时，这段规则不会注入。默认配置下，`grok-4` 和 `grok-4.7` 会匹配，`deepseek-v4` 不会。

要改已安装的 profile，在该 profile 的 `cordis.patch.yml` 末尾覆盖这一行。`config` 会被整段替换，所以两个字段都要写：

```yaml
- id: grok-build-policy
  name: grok-build-policy
  config:
    modelPattern: grok
    providerPattern: ""
```

### 卸载

```sh
dsh plugin --profile web remove grok-build-policy
```

### 许可

[Apache License 2.0](LICENSE)。工作规则改编自同样以 Apache-2.0 授权的 grok-build，出处写在 [NOTICE](NOTICE)。

## English

A DeepSeek Harness plugin. While the selected model id contains `grok`, it appends a block of working rules to the system prompt. The block is omitted for every other model.

The rules are adapted from the default system prompt of [xai-org/grok-build](https://github.com/xai-org/grok-build) (Apache-2.0). Only the parts that stand on their own inside Harness are kept. The source template is:

[`crates/codegen/xai-grok-agent/templates/prompt.md`](https://github.com/xai-org/grok-build/blob/main/crates/codegen/xai-grok-agent/templates/prompt.md)

[`PromptContext::render_with_renderer`](https://github.com/xai-org/grok-build/blob/main/crates/codegen/xai-grok-agent/src/prompt/context.rs) renders that template. The same directory also contains `apply_patch_prompt.md` (Codex mode) and `subagent_prompt.md` (subagents). This plugin does not use those two.

The appended block is `<model_work_policy>`. It has six parts:

- Treat an action according to how reversible it is and who it affects.
- Do clear, reversible work in the current turn, and support claims of completion with tool output.
- Keep scratch files in the operating system's temporary directory and delete them when finished.
- Communication.
- Markdown formatting.
- Verify web interface changes in the browser.

It does not replace the Harness identity line or its tool instructions. It also does not carry Grok Build tool names, memory directories, or the `apply_patch` protocol.

### Install

Replace `web` with the profile that should receive the plugin. Installation applies to every session in that profile.

Install from npm:

```sh
dsh plugin --profile web add grok-build-policy
```

Install from GitHub:

```sh
dsh plugin --profile web add github:Xunflash/grok-build-policy
```

Install from a local checkout while changing the source:

```sh
dsh plugin --profile web add /absolute/path/to/grok-build-policy
```

Confirm the row is in the composed profile:

```sh
dsh --profile web --dump-config
```

The output should contain `id: grok-build-policy`. On the Web profile, configuration reload is enabled, so the next model request picks the plugin up. Without reload, restart that profile.

### Configuration

Matching runs on every system-prompt assembly and is case-insensitive. `modelPattern` and `providerPattern` are substrings.

| Field | Default | Effect |
|---|---|---|
| `modelPattern` | `grok` | Substring the model id must contain. Empty skips the model check. |
| `providerPattern` | empty | Substring the provider id must contain. Empty allows every provider. |

When both are empty, the rules are not injected. With the defaults, `grok-4` and `grok-4.7` match, and `deepseek-v4` does not.

To change an installed profile, append an override to that profile's `cordis.patch.yml`. `config` is replaced as a whole, so write both fields:

```yaml
- id: grok-build-policy
  name: grok-build-policy
  config:
    modelPattern: grok
    providerPattern: ""
```

### Remove

```sh
dsh plugin --profile web remove grok-build-policy
```

### License

[Apache License 2.0](LICENSE). The working rules are adapted from grok-build, which uses the same license. Attribution is in [NOTICE](NOTICE).
