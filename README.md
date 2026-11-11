# fullsend-config

Shared [fullsend](https://github.com/fullsend-ai/fullsend) agent customizations
for every majamassarini repo.

The fleet manifest, `repos.yaml` in the private `majamassarini/homino`, points
each agent's `source:` at a file here by commit URL and sha256. This repo is
public because fullsend fetches those URLs without credentials.

| File | Used by | What it changes |
|---|---|---|
| `harness/triage.yaml` | triage | `TRIAGE_AUTO_CODE=off`: triage never applies `ready-to-code`, so the code agent starts only on a human `/fs-code` or label. Adds `repo-map` and `bug-root-cause`, and a 20-minute budget (upstream: 10). |
| `harness/code.yaml`, `harness/fix.yaml`, `harness/review.yaml` | code, fix, review | Add `repo-map`. |
| `skills/repo-map/` | all four harnesses, and local Claude Code | Map of the repos and their dependencies, and how to read sibling repos, so agents follow a problem into the repo where it lives and open a linked issue there |
| `skills/bug-root-cause/` | triage | For bug reports: trace each symptom into the code (in any repo) before deciding, and file the issue in the repo where the cause is (`prerequisites` or `split`), since the coder can't |
| `.claude-plugin/` | local Claude Code | Makes this repo a Claude Code plugin marketplace, so the skills in `skills/` also load in local Claude Code sessions |

homino layers its own private harnesses on top of these (`.fullsend/<agent>.yaml`
in homino, using these files as their `base:`). They add homino's private
skills, which stay in homino.

## Using the skills in local Claude Code

homino's `.claude/settings.json` registers this marketplace and enables the
plugin, so Claude Code offers to install it when you trust the folder. In
any other repo, install it once with:

```
/plugin marketplace add majamassarini/fullsend-config
/plugin install fullsend-config@fullsend-config
```

The plugin has no `version`, so Claude Code updates it to the latest commit
on `/plugin marketplace update`. Its skills are namespaced, e.g.
`fullsend-config:repo-map`.

To change a file:

1. Change it here and merge.
2. Re-pin its URL (new commit SHA and `sha256`) in homino's `repos.yaml`
   and in homino's `.fullsend/{triage,code,fix}.yaml`. A skill change needs
   the new commit SHA too: fullsend reads the skill from the same commit as
   the harness.
3. Run `fullsend repos install`.

Each harness's `base:` is pinned to the `fullsend-ai/agents` release matching the
fleet's fullsend version (`github.fullsend_ref` in `repos.yaml`). Re-pin it on
every fullsend upgrade with `fullsend agent update`.

Never put secrets here. Reference them as `${VAR}`; they resolve from each
repo's Actions secrets at run time.
