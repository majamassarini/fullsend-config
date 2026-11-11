# fullsend-config

Shared [fullsend](https://github.com/fullsend-ai/fullsend) agent customizations
for every majamassarini repo.

The fleet manifest, `repos.yaml` in the private `majamassarini/homino`, points
each agent's `source:` at a file here by commit URL and sha256. This repo is
public because fullsend fetches those URLs without credentials.

| File | Used by | What it changes |
|---|---|---|
| `harness/triage.yaml` | triage | `TRIAGE_AUTO_CODE=off`: triage never applies `ready-to-code`, so the code agent starts only on a human `/fs-code` or label. Adds `repo-map`. |
| `harness/code.yaml`, `harness/fix.yaml`, `harness/review.yaml` | code, fix, review | Add `repo-map`. |
| `skills/repo-map/` | all four harnesses | Map of the repos and their dependencies, and how to read sibling repos from the sandbox, so agents follow a problem into the repo where it lives and open a linked issue there |

homino layers its own private harnesses on top of these (`.fullsend/<agent>.yaml`
in homino, using these files as their `base:`). They add a private
`homino-config` skill that stays in homino.

To change a file:

1. Change it here and merge.
2. Re-pin its URL (new commit SHA and `sha256`) in homino's `repos.yaml`.
3. Run `fullsend repos install`.

Each harness's `base:` is pinned to the `fullsend-ai/agents` release matching the
fleet's fullsend version (`github.fullsend_ref` in `repos.yaml`). Re-pin it on
every fullsend upgrade with `fullsend agent update`.

Never put secrets here. Reference them as `${VAR}`; they resolve from each
repo's Actions secrets at run time.
