# Label Guide

Every label in this repository, what it means, and how it ranks. Labels use
**emoji prefixes** for at-a-glance scanning — the emoji signals the category
and/or meaning. Write labels in the repo's primary language.

## Priority (P0 → P3)

| Label | Meaning |
|---|---|
| `🔴 P0` | Emergency: data loss, security bypass, crash loop, unusable core |
| `🟠 P1` | High: blocks planned work, needs attention soon |
| `🟡 P2` | Medium: normal priority |
| `🟢 P3` | Low: nice to have |

## Impact (severity)

| Label | Meaning |
|---|---|
| `impact: security` | Security boundary, credentials, authz, sandbox, sensitive data |
| `impact: data-loss` | Loses, corrupts, or drops user/session/config data |
| `impact: availability` | Crash, hang, restart loop, or process outage |

## Rating labels (low → high)

| Rank | Label | Meaning |
|---|---|---|
| 1 | `rating: <emoji> <name>` | lowest tier — describe what it means |
| 2 | `rating: <emoji> <name>` | ... |
| 3 | `rating: <emoji> <name>` | ... |
| ... | ... | highest tier — describe what it means |

## Other labels

| Label | Meaning |
|---|---|
| `🐛 bug` | Something isn't working as expected |
| `✨ enhancement` | New feature or request |
| `📚 documentation` | Improvements or additions to documentation |
| `✅ status: ready to merge` | Approved and ready to merge |
| ... | ... |

> Maintained by repo-standardizer — keep in sync whenever labels change.
