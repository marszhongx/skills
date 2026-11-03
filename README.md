# skills

个人 agent skill 集合，布局遵循 [skills.sh](https://skills.sh) 约定：`skills/<name>/SKILL.md`，可直接被 `npx skills` 发现。

## 安装

```bash
# 装全部（当前项目）
npx skills add marszhongx/skills --all

# 装单个，全局可用
npx skills add marszhongx/skills --skill pi-model-config -g

# 先看看仓库里有什么
npx skills add marszhongx/skills --list
```

装完更新用 `npx skills update`，卸载用 `npx skills remove`。`-a` 可指定 agent（如 `-a claude-code`、`-a pi`），不指定则交互选择或按已检测到的 agent 安装。

## 目录

| skill | 用途 |
| --- | --- |
| `pi-model-config` | 维护 pi 的自定义 provider/模型配置（`~/.pi/agent/models.json`、`defaultThinkingLevel`）：同步网关模型列表、新增 provider、查模型规格、配 thinking 级别 |

## 本地开发

```bash
npx skills add ./ --list          # 校验发现结果
npx skills add ./ --skill <name> -y --copy   # 本地试装
```
