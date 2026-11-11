---
name: repo-map
description: >-
  Map of the majamassarini home-automation repositories and how they depend
  on each other. Use it whenever an issue, bug or change may involve code
  outside the current repository, to find where the root cause or the fix
  really lives and to read that code.
---

# majamassarini repo map

The home-automation system is split across several repositories that
together behave like one codebase. A symptom in one repo often has its root
cause in another. Before concluding, check the repos the code depends on.

## Repositories

| Repo | Source root | What it is |
|---|---|---|
| `homino` (private) | `automate-home/my-home/`, `deploy/` | The deployed home: YAML configuration of appliances, performers and schedulers, plus Docker/Ansible deployment. Uses all the packages below. |
| `automate-home` | `home/` | Core engine: appliance state machines, events, performers, schedulers, protocol abstractions. |
| `automate-knx-plugin` | `knx_plugin/` | KNX protocol plugin for automate-home. |
| `automate-home-assistant-plugin` | `home_assistant_plugin/` | Home Assistant plugin (e.g. Philips Hue lights). |
| `automate-sonos-plugin` | `soco_plugin/` | Sonos plugin. |
| `automate-lifx-plugin` | `lifx_plugin/` | LIFX plugin. |
| `knx-stack` | `knx_stack/` | KNX protocol stack: encodes and decodes KNX messages. |
| `lifx-lib` | `lifx/` | LIFX LAN protocol library. |
| `automate-ws` | `ws/` | Web UI. |
| `automate-prometheus-exporter` | `prometheus_exporter/` | Reads appliance state changes from Redis and pushes metrics to Prometheus (Grafana dashboards). |

## Dependencies (A → B means A uses B)

```
homino ─→ automate-home, every plugin, automate-ws, automate-prometheus-exporter
automate-knx-plugin            ─→ automate-home, knx-stack
automate-lifx-plugin           ─→ automate-home, lifx-lib
automate-home-assistant-plugin ─→ automate-home
automate-sonos-plugin          ─→ automate-home
automate-ws                    ─→ automate-home, automate-knx-plugin, automate-lifx-plugin, automate-sonos-plugin
automate-prometheus-exporter   ─→ automate-home (via Redis appliance state, not a Python import)
```

Where a problem usually lives:
- Wrong behavior of an appliance type (states, transitions, events): `automate-home`.
- A device not receiving or sending the right protocol messages: the plugin
  for that protocol, then its protocol library (`knx-stack`, `lifx-lib`).
- One specific appliance misconfigured (wrong type, address, performer): `homino`.
- Metrics missing or wrong in Grafana: `automate-prometheus-exporter`, then
  the appliance state it reads from `automate-home`.
- Web UI: `automate-ws`.

## Reading another repo's code

On a developer machine, the repos are usually checked out side by side
(for example `../automate-home` next to `homino`): read them there directly.

In a fullsend sandbox only the current repo is checked out. All repos except
`homino` are public: read them through the GitHub API with `gh`. Don't clone
them.

```bash
# search across all the repos
gh search code --owner majamassarini '<symbol or text>'
# list a directory, then read a file
gh api repos/majamassarini/automate-home/contents/home/appliance/light
gh api repos/majamassarini/automate-home/contents/home/appliance/light/__init__.py --jq .content | base64 -d
```

`homino` is private: from another repo's sandbox you can't read it, so
don't try.

## When the cause is in another repo

A fullsend agent can only change the repository it runs in. If the root
cause, or part of the fix, is in another repo:

1. Say so explicitly, naming the repo, file and function, with evidence.
2. Use the cross-repo issue mechanism to open a linked issue in that repo,
   describing the root cause there. That repo's own agents take it from
   there.
3. Fix here only what belongs here.

Working locally with a person, ask them instead whether to change the other
checkout directly or to open the issue.
