---
name: pi-model-config
description: 维护 pi 的自定义 provider/模型配置（~/.pi/agent/models.json、settings.json 模型相关项）：同步网关模型列表、新增 provider、查模型规格、配置 thinking 级别。用户说“更新 <provider> models”“配置 <provider>”“模型规格/thinking 级别/默认思考等级”时使用。
---

# pi 模型配置维护

## 固定约定

- 配置文件：`~/.pi/agent/models.json`；每个模型的 `name` 与 `id` 保持一致。
- 凭据全部内联在各 provider 的 `apiKey`；`auth.json` 保持为空（`{}`）。
- 自定义 provider 统一 `"api": "openai-completions"` + `"compat": {"supportsDeveloperRole": false}`。
- 全局默认思考级别在 `~/.pi/agent/settings.json` 的 `defaultThinkingLevel`（当前 `max`）。
- 改配置前先确认实际在用的 pi 可执行文件与版本（`which -a pi`、`pi --version`）；机器上有多份安装时以 PATH 中生效的那个为准。
- **修改 models.json 必须用「校验过的完整 JSON 整文件替换」**：先构造并校验完整内容，再覆盖写入；不要做可能失败的零散局部编辑——文件一旦损坏，pi 起不来，也就无法再用 pi 修复。

## provider 定位

- 用户说更新哪个 provider 就更新哪个：从 models.json 精确匹配给出的名字，key 从对应配置里现读，不要抄写到别处。
- 匹配不到就直接回复「Provider <name> 不存在」，并列出 models.json 现有的 provider，**不要猜测、不要映射**（不要自行把某个名字对应到其它 provider）。
- 只有在用户明确要求新增 provider 时才创建新 block。

## 更新 provider 模型列表

1. 拉取该 provider 的 `<baseUrl>/models`（带其 `apiKey`），与 models.json 里的列表对比。
2. 新模型查规格（见下节优先级），已下线的模型直接删。
3. 直接编辑 models.json（保持 2 空格缩进），改完用 JSON 解析校验（如 `python3 -m json.tool ~/.pi/agent/models.json`）。
4. 汇报：新增/移除、规格来源、已知坑；重开 `/model` 生效。

## 规格查询优先级

1. **OpenRouter**：`https://openrouter.ai/api/v1/models`，按模型名匹配（id 大小写/前缀可能不同，如 fireworks 的 `accounts/fireworks/models/deepseek-v4p1-flash` = OR 的 `deepseek/deepseek-v4.1-flash`）。关注 `architecture.input_modalities`、`context_length`、`top_provider.max_completion_tokens`、`reasoning.supported_efforts`。
2. **models.dev**：页面 `https://models.dev/models/<lab>/<model>/`（Context / Output limit / Input 三项），或 `https://models.dev/api.json`。
3. **pi 官方目录**：官网 `https://pi.dev/models`，API `https://pi.dev/api/models`（结构 `{provider: {modelId: {...}}}`，字段与 models.json 同构，含官方 `thinkingLevelMap`）。官网的 `contextWindow`/`maxTokens` 是直连官方 provider 的值（如 openai gpt-6-sol 默认 272000 是短上下文定价档），代理渠道上限以渠道为准。
4. 都查不到 → 问用户，**不要凭惯例猜**。

## thinking 级别

- pi 的 thinking 级别：`off/minimal/low/medium/high/xhigh/max`。
- `thinkingLevelMap` 语义：字符串 = 该级别发送给上游的值；`null` = 隐藏该级别；**`xhigh`/`max` 必须显式给出字符串才会出现在 UI**；`off` 给字符串则关闭时发送该值（如 `"none"`）。
- 超出支持范围会**自动向下 clamp 到最近可用级别**（如 max → high），不报错。

## 新增 provider

1. 在 models.json 加 block（`baseUrl` / `api` / `apiKey` / `compat` / `models`，name=id）。
2. 按用户给的模型列表补齐规格与 `thinkingLevelMap`。
3. 校验 JSON 并汇报。
