# Grok Build policy

DeepSeek Harness 插件。当前模型 id 包含 `grok` 时，在系统提示词里追加一段工作规则。换成其他模型后，这段规则不会出现。

规则改编自 [xai-org/grok-build](https://github.com/xai-org/grok-build)（Apache-2.0）的默认代理提示词，只保留能在 Harness 里独立成立的部分：

- 危险操作先看可逆性和影响范围
- 明确的可逆工作在当前回合做完，结论要有工具输出
- 临时文件放在系统临时目录，用完删除
- 说话方式、Markdown 格式
- 改了网页界面就在浏览器里验证

它不替换 Harness 的身份句和工具说明，也不携带 Grok Build 的工具名、记忆目录或 `apply_patch` 协议。

## 安装

在要启用的 profile 里安装这个目录：

```sh
dsh plugin --profile web add /absolute/path/to/grok-build-policy
```

## 配置

`cordis.patch.yml` 里有两个字符串，匹配时不区分大小写：

| 字段 | 默认 | 作用 |
|---|---|---|
| `modelPattern` | `grok` | 模型 id 需要包含的子串。留空则不按模型过滤。 |
| `providerPattern` | 空 | 提供方 id 需要包含的子串。留空则不限制提供方。 |

两个都留空时，这段规则不会注入。
