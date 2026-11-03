# skills

Agent skills I maintain, in the layout skills.sh expects, so `npx skills` can install them.

## Install

```bash
npx skills add marszhongx/skills --list                        # see what is in the repo
npx skills add marszhongx/skills --skill pi-model-config -g    # one skill, available in every project
npx skills add marszhongx/skills --all                         # everything, into the current project
```

Add `-a <agent>` to target a specific agent, for example `-a claude-code` or `-a pi`. Without it the CLI asks, or installs to the agents it detects. `npx skills update` pulls newer versions and `npx skills remove` uninstalls.

## Skills

| Skill | What it does |
| --- | --- |
| `pi-model-config` | Keeps pi's custom provider and model config current in `~/.pi/agent/models.json`, plus `defaultThinkingLevel` in settings: syncing a gateway's model list, adding a provider, looking up specs, setting thinking levels |

## Working on a skill

```bash
npx skills add ./ --list                             # check what the CLI discovers
npx skills add ./ --skill pi-model-config -y --copy  # local test install
```
