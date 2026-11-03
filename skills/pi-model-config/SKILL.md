---
name: pi-model-config
description: Maintain pi's custom provider and model configuration in ~/.pi/agent/models.json and the thinking-level settings. Sync a gateway's model list, add a provider, look up model specs, set thinking levels. Use when the user says "update <provider> models" or "configure <provider>", or asks about model specs, thinking levels, or the default thinking level.
---

# pi model config maintenance

## Conventions

- Config file: `~/.pi/agent/models.json`. Every model's `name` matches its `id`.
- API keys live inline in each provider's `apiKey`. `auth.json` stays `{}`.
- Custom providers use `"api": "openai-completions"` with `"compat": {"supportsDeveloperRole": false}`.
- The global default thinking level is `defaultThinkingLevel` in `~/.pi/agent/settings.json` (currently `max`).
- Confirm which pi binary is in use before editing: `which -a pi` and `pi --version`. With several installs on the machine, go with the one on PATH.
- Rewrite `models.json` as a complete file that has been parsed and validated first. Build the whole content, check it, then overwrite. A partial edit that fails halfway leaves a broken file, pi will not start, and pi is the tool you would use to repair it.

## Choosing the provider

- Update exactly the provider the user names. Match its name in `models.json` and read the key from that block; do not copy the key anywhere else.
- No match: reply "Provider <name> does not exist", list the providers that `models.json` has, and stop. Do not guess at a near match or map one name onto another.
- Create a new block only when the user asks for a new provider.

## Updating a provider's model list

1. Fetch `<baseUrl>/models` with that provider's `apiKey` and diff it against `models.json`.
2. Look up specs for the new models (next section). Delete models the gateway no longer serves.
3. Edit `models.json` with 2-space indent, then validate the JSON (`python3 -m json.tool ~/.pi/agent/models.json`).
4. Report what was added and removed, where the specs came from, and any known issues. Reopen `/model` for the change to take effect.

## Where to find specs

Check these in order.

1. OpenRouter: `https://openrouter.ai/api/v1/models`. Match by name, since the id may differ in case or prefix (fireworks serves `accounts/fireworks/models/deepseek-v4p1-flash`, which OpenRouter lists as `deepseek/deepseek-v4.1-flash`). Take `architecture.input_modalities`, `context_length`, `top_provider.max_completion_tokens`, and `reasoning.supported_efforts`.
2. models.dev: the page `https://models.dev/models/<lab>/<model>/` (Context, Output limit, Input), or `https://models.dev/api.json`.
3. pi's own catalog: `https://pi.dev/models` and `https://pi.dev/api/models`, structured as `{provider: {modelId: {...}}}` with the same fields as `models.json`, including the official `thinkingLevelMap`. Its `contextWindow` and `maxTokens` describe a direct connection to the official provider (openai gpt-6-sol defaults to 272000 for the short-context price tier), so a proxy channel's own limits win.
4. Nothing found: ask the user. Do not guess from convention.

## Thinking levels

- pi offers `off`, `minimal`, `low`, `medium`, `high`, `xhigh`, `max`.
- In `thinkingLevelMap`, a string is the value sent upstream, `null` hides the level, and `xhigh` and `max` appear in the UI only when they map to a string. A string on `off` is sent while thinking is disabled, for example `"none"`.
- A level the model does not support clamps down to the nearest one that works, without an error (`max` becomes `high`).

## Adding a provider

1. Add a block to `models.json` with `baseUrl`, `api`, `apiKey`, `compat`, and `models`, keeping `name` equal to `id`.
2. Fill in specs and `thinkingLevelMap` for the models the user listed.
3. Validate the JSON and report back.
