# Showcase — repos standardized with repo-standardizer 🦞

Real repositories that have been standardized by this skill. Each entry links
to the repo and lists what was applied. Send a PR to add yours — one line,
with the before/after notes if you have them.

---

## [repo-standardizer](https://github.com/programmingWTF/repo-standardizer) — this repo (dogfood)

The skill standardizes itself (dogfooding) since v0.1.0:

| Area | Applied |
|---|---|
| Labels | Full v1 baseline: `P0`–`P3`, `impact:*`, type/status/dependencies/security — 23 labels |
| Issue forms | Bug / feature / question YAML forms + `config.yml` |
| PR template | `.github/PULL_REQUEST_TEMPLATE.md` |
| CI | Release workflow: `v*` tag → skill zip + GitHub Release |
| Rulesets | `protect-default-branch`: required review, linear history, no deletions, no fast-forward |
| Docs | Bilingual README, CONTRIBUTING, SECURITY, CODE_OF_CONDUCT, CHANGELOG, VISION, AGENTS/CLAUDE |
| Meta | 10 topics, homepage → ClawHub listing, Discussions on, template=on |

## [cognitive-os](https://github.com/Alastair-Jiang/cognitive-os) — research engineering repo

Standardized 2026-08-19 (PR [#2](https://github.com/Alastair-Jiang/cognitive-os/pull/2)):

| Area | Applied |
|---|---|
| Labels | 27 labels, research-themed rating tiers (`🔬 hypothesis → 🧪 prototype → 📊 experiment → ✅ verified → 🏆 milestone`) |
| Issue forms | Bug / feature / question forms (Chinese) |
| PR template | With research-honesty checklist |
| CI | Python 3.10–3.12 × ruff + pytest |
| Docs | CONTRIBUTING / SECURITY / CoC / CLAUDE / AGENTS / LABELS / CHANGELOG (Chinese), README badges |
| Config-as-code | `.github/labels.json` + idempotent apply script for pull-only upstreams |

---

_Repo standardized by repo-standardizer itself — labels, forms, and docs on
this very page are the skill's own output._