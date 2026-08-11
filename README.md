# 🦞 GitHub Repo Standardizer

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![GitHub release](https://img.shields.io/github/v/release/programmingHLS/repo-standardizer)](https://github.com/programmingHLS/repo-standardizer/releases)
[![GitHub issues](https://img.shields.io/github/issues/programmingHLS/repo-standardizer)](https://github.com/programmingHLS/repo-standardizer/issues)
[![GitHub pull requests](https://img.shields.io/github/issues-pr/programmingHLS/repo-standardizer)](https://github.com/programmingHLS/repo-standardizer/pulls)
[![GitHub stars](https://img.shields.io/github/stars/programmingHLS/repo-standardizer)](https://github.com/programmingHLS/repo-standardizer/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/programmingHLS/repo-standardizer)](https://github.com/programmingHLS/repo-standardizer/network)
[![GitHub contributors](https://img.shields.io/github/contributors/programmingHLS/repo-standardizer)](https://github.com/programmingHLS/repo-standardizer/graphs/contributors)

> **An OpenClaw skill that turns any bare GitHub repository into a professional one — issue forms, PR template, label taxonomy, CI, rulesets, and docs — with one command.**

Stop pasting the same wall of prompts every time you start a new project. This skill detects the current state of a repository and applies a professional baseline **idempotently**: run it on a fresh repo, or run it on an old repo to fill in what's missing. It never clobbers your content.

---

## ✨ Features

| Module | What it does |
|---|---|
| 🏷️ **Labels** | Idempotent upsert of a curated 16-label taxonomy (`type:*`, `priority:*`, `status:*`, `good first issue`, `help wanted`...) |
| 📋 **Issue forms** | YAML issue forms for bug / feature / question + `config.yml` |
| 🔀 **PR template** | Structured pull request template with checklist |
| 🤖 **CI workflows** | Auto-detects your test framework (Node / Python / Go / Rust) and adds a matching GitHub Actions workflow |
| 🛡️ **Rulesets** | Branch protection: required PR review, linear history, no deletions, required signatures |
| 📚 **Docs** | README badges, `CONTRIBUTING.md`, `SECURITY.md`, `CHANGELOG.md`, `CODEOWNERS`, optional MIT license |
| ✅ **Verification** | Re-queries GitHub and prints an `applied / skipped / failed` checklist |

## 🧠 Safety by design

- **Auth-first**: checks `gh auth status` before anything; if you're not logged in, it tells you exactly how to log in (or provide a token). No token → no action.
- **Repo-aware**: detects personal vs. organization repos and `public` vs. `private` visibility, and adapts what it does.
- **Idempotent**: run it twice, get the same result. Existing labels are updated, not duplicated; existing templates are diffed, not overwritten.
- **Dry-run first**: shows a plan and asks for confirmation before destructive operations (rulesets, branch rules, deletions).

## 📦 Installation

```bash
# via OpenClaw
openclaw skills install git:programmingHLS/repo-standardizer@main

# or via the skills CLI
npx skills add programmingHLS/repo-standardizer
```

Requires: `gh` CLI (authenticated), `git`, and `jq` + `python3` for label encoding.

## 🚀 Usage

Just ask your agent:

```text
Standardize the repo programmingHLS/repo-standardizer
```

Or run modules selectively:

```text
Add issue forms and a PR template to my repo
Fix the labels on this repo to match the standard taxonomy
Protect the main branch with a ruleset requiring review
```

<details>
<summary><b>What happens under the hood?</b></summary>

1. **Preflight** — verify login (`gh auth status`), detect repo type
   (`User` vs `Organization`, `PUBLIC` vs `PRIVATE`), audit current state.
2. **Plan** — show a table of what will be created / updated / skipped.
3. **Apply** — run modules in order, each idempotent.
4. **Verify** — re-query GitHub and report the final checklist.

</details>

## 📁 Repository layout

```text
.
├── SKILL.md               # The skill itself
├── templates/             # Reusable templates
│   ├── labels.json        # Label taxonomy (color + description)
│   ├── issue-form-*.yml   # Bug / feature / question forms
│   ├── config.yml         # Issue template config
│   ├── PR_TEMPLATE.md     # Pull request template
│   ├── ci-*.yml           # Node / Python / Go / Rust workflows
│   ├── CODEOWNERS         # Default owners file
│   ├── CONTRIBUTING.md    # Contribution guide
│   ├── SECURITY.md        # Security policy
│   └── README.md          # README skeleton for new repos
├── LICENSE
└── README.md
```

## 🤝 Contributing

PRs welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for conventions.
Found a bug? Use the [bug report form](https://github.com/programmingHLS/repo-standardizer/issues/new?template=bug.yml).

## 📄 License

[MIT](LICENSE) © 2026 [programmingWTF](https://github.com/programmingWTF) & [LiGuiyu-AI](https://github.com/LiGuiyu-AI) — the friendly lobster 🦞
