---
name: repo-standardizer
description: Standardize a GitHub repository end-to-end — issue forms, PR template, label taxonomy, CI workflows, CODEOWNERS, rulesets, and docs. Use when creating a new repository or professionalizing an existing one.
metadata:
  openclaw:
    requires:
      bins: [gh, git]
---

# GitHub Repo Standardizer

Detect a repository's current state, then apply a professional baseline:
issue forms, PR template, label taxonomy, CI, CODEOWNERS, rulesets, docs.
**Idempotent** — safe to re-run; fills gaps and reconciles drift without
duplicating or clobbering.

## When to use

- User says "standardize / tidy up / professionalize this repo"
- A new repo was just created and needs templates, labels, CI, and rules from day one
- A repo looks bare: no templates, no labels, no CI, no branch protection

## Preflight (mandatory, in order)

### 1. Authenticate — check gh login first

```bash
gh auth status 2>/dev/null || echo "NOT_LOGGED_IN"
```

- **Logged in** → continue; print `gh api user -q .login` so the user knows which account will act.
- **Not logged in** → STOP. Tell the user (do not guess):
  1. Run `gh auth login` (web/device flow), **or**
  2. Export a token: `export GH_TOKEN=ghp_xxx` — needs `repo`, `workflow` (for CI files) and, for org repos, `admin:org` (or `admin:repo_hook` / org membership admin) scopes.
  - Never paste tokens into chat, logs, or files. If the user pastes a token in chat, advise them to revoke it and re-issue.
  - If `gh auth login` is impossible in this environment (headless), suggest `gh auth login --with-token` reading from a file the user created.

Verify the acting account can write to the target:

```bash
# user repos: no extra check needed beyond token scopes
# org repos: must be a member/admin of the org
gh api "orgs/ORG/memberships/$(gh api user -q .login)" -q .role 2>/dev/null || echo "NO_ORG_ACCESS"
```

- `admin`/`member` → OK. `404` → stop and ask the user to add the account to the org first.

### 2. Detect repository type

```bash
gh repo view OWNER/REPO --json name,owner,visibility,defaultBranchRef,isArchived,isFork \
  -q '{name:.name, ownerType:.owner.type, visibility:.visibility, defaultBranch:.defaultBranchRef.name, archived:.isArchived, fork:.isFork}'
```

| Field | Meaning | Consequence |
|---|---|---|
| `ownerType` | `User` = personal, `Organization` = org | Org repos can also use org-level rulesets; both support repo-level rulesets |
| `visibility` | `PUBLIC` / `PRIVATE` / `INTERNAL` | Private: skip public-facing docs pressure, keep CI secrets minimal; public: README badges + CONTRIBUTING/SECURITY matter |
| `archived` / `fork` | Read-only / fork | **Skip write modules**; report why |

If the repo was not explicitly named by the user, confirm before touching an
org or private repository.

### 3. Audit current state

```bash
gh label list --repo OWNER/REPO --limit 200
gh api repos/OWNER/REPO/contents/.github -q '.[].path' 2>/dev/null || echo "no .github dir"
gh api repos/OWNER/REPO/contents/.github/workflows -q '.[].name' 2>/dev/null || echo "no workflows"
gh api repos/OWNER/REPO/rulesets -q '.[] | {name:.name, enforcement:.enforcement}' 2>/dev/null || echo "no rulesets"
gh api repos/OWNER/REPO/branches -q '.[].name' 2>/dev/null
for f in README.md CONTRIBUTING.md SECURITY.md LICENSE .gitignore; do
  gh api "repos/OWNER/REPO/contents/$f" -q .name 2>/dev/null || echo "missing: $f"
done
```

### 4. Detect test framework (for CI module)

Check for these signals (first match wins):

```bash
gh api repos/OWNER/REPO/contents/package.json -q .name 2>/dev/null   # node → templates/ci-node.yml
gh api repos/OWNER/REPO/contents/pyproject.toml -q .name 2>/dev/null # python → templates/ci-python.yml
gh api repos/OWNER/REPO/contents/go.mod -q .name 2>/dev/null         # go → templates/ci-go.yml
gh api repos/OWNER/REPO/contents/Cargo.toml -q .name 2>/dev/null     # rust → templates/ci-rust.yml
```

No signal → propose the generic CI (or ask the user whether CI is wanted at all).

## Workflow

1. **Preflight** (above). If auth or access fails, stop with a clear message.
2. **Dry-run plan** — show the user a concise table of what will be created/updated/skipped. Get confirmation for: rulesets, branch deletion/protection changes, org-level changes, and anything destructive.
3. **Apply modules** (each idempotent; run in this order).
4. **Verify** — re-query and print an `applied / skipped / failed` checklist.

### Module A — Labels (design first, then idempotent upsert)

**Step 1 — Profile the project** (adjust the taxonomy, never copy blindly):

- **Audience / language region** → decides the rating-label style:
  - Chinese-community project → 王者荣耀-style tiers work well
    (`tier: 青铜` … `tier: 王者`)
  - International project → use universal grades instead
    (`grade: S/A/B/C/D` or `★`–`★★★★★`)
  - Anything else → define a rating scheme that fits the project
- **分工小组 (teams)**: if the repo has an explicit division of labor
  (CODEOWNERS, CONTRIBUTING, a team list in docs) → add one `team: *` label
  per group (e.g. `team: frontend`, `team: algorithm`). No team list → skip.
- **Project type** (library / app / coursework / org-infra) → decide whether
  `dependencies`, `security`, `docs` etc. categories are needed.

**Step 2 — Compose categories** (baseline in `templates/labels.json`,
extend or trim as needed):

| Category | Labels | When |
|---|---|---|
| Type (always) | `bug` `enhancement` `documentation` `question` `help wanted` `good first issue` | always |
| Priority | `priority: critical/high/medium/low` | recommended |
| Status | `status: in progress` `blocked` `wontfix` `ready to merge` `merged` | recommended |
| Rating | tier/grade labels, gradient | per project (Step 1) |
| Team | `team: <group>` one per group | only if a division of labor exists |

**Step 3 — Color rules (mandatory):**

- **Diverse palette**: colors must be rich and varied — the whole label set
  should look like a palette, not a monochrome block. Even within one
  category, spread the hues (e.g. priority labels: red / orange / yellow /
  green, or four clearly different hues).
- **Semantic hints (not hard mappings)**: `ready to merge` / `merged` / done
  → greens (never gray or red); `wontfix` → gray; `in progress` → blue.
  Everything else: pick colors that look good together and match the label's
  meaning loosely — but prefer variety over strict one-meaning-one-color.
- **Rating labels need a gradient** (tiers/grades progress in color, e.g.
  gray → bronze → silver → gold, or dark→light).
- Neighboring labels must be distinguishable. Forbidden: all-one-color,
  adjacent duplicates, or colors that contradict the label content.

**Step 4 — Idempotent upsert.** GitHub has **no** `PUT /labels/{name}`
endpoint. Upsert = check existence (`GET /labels/{name}`), then
`POST /labels` (create) or `PATCH /labels/{name}` (update). Works for both
map-form and array-form `labels.json`:

```bash
R="repos/OWNER/REPO"
jq -c 'if type == "array" then .[] else to_entries[] | {name: .key} + .value end' templates/labels.json | while read -r l; do
  name=$(echo "$l" | jq -r .name); color=$(echo "$l" | jq -r .color); desc=$(echo "$l" | jq -r .description)
  enc=$(python3 -c "import urllib.parse,sys;print(urllib.parse.quote(sys.argv[1]))" "$name")
  if gh api "$R/labels/$enc" >/dev/null 2>&1; then
    gh api -X PATCH "$R/labels/$enc" -f name="$name" -f color="$color" -f description="$desc" --silent && echo "label updated: $name"
  else
    gh api -X POST "$R/labels" -f name="$name" -f color="$color" -f description="$desc" --silent && echo "label created: $name"
  fi
done
```

- URL-encode label names (spaces, slashes).
- To reconcile drift (deleted manual labels), show the diff and ask before removing labels that are already in use.

### Module B — Issue forms + config

Create `.github/ISSUE_TEMPLATE/` with `config.yml` plus one YAML form per
template (bug / feature / question). Push via a commit:

```bash
mkdir -p .github/ISSUE_TEMPLATE
cp templates/issue-form-*.yml templates/config.yml .github/ISSUE_TEMPLATE/
git add .github/ISSUE_TEMPLATE && git commit -m "chore: add issue forms" && git push
```

- If templates already exist, diff them; only overwrite identical or clearly
  stale files (ask first if the user may have customized them).
- If there is no git clone, clone first (`gh repo clone OWNER/REPO`), edit, push.

### Module C — PR template

```bash
mkdir -p .github
cp templates/PR_TEMPLATE.md .github/PULL_REQUEST_TEMPLATE.md
git add .github/PULL_REQUEST_TEMPLATE.md && git commit -m "chore: add PR template" && git push
```

### Module D — CI workflow

Pick the workflow from the framework detection (templates/ci-*.yml). Write to
`.github/workflows/ci.yml`, commit, push. Keep existing workflows; only add
`ci.yml` if none exists.

- Note: pushing workflow files requires a token with the `workflow` scope; if
  the push is rejected with 403, tell the user their token lacks `workflow`.

### Module E — Branch rules

Prefer **rulesets** (modern) over legacy branch protection:

```bash
# list existing
gh api repos/OWNER/REPO/rulesets -q '.[].name'
# create (example: protect default branch)
gh api -X POST repos/OWNER/REPO/rulesets --input - <<'EOF'
{
  "name": "protect-default-branch",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": {"include": ["refs/heads/DEFAULT_BRANCH"], "exclude": []}
  },
  "rules": [
    {"type": "pull_request", "parameters": {"required_approving_review_count": 1, "dismiss_stale_reviews_on_push": true, "require_code_owner_review": false, "require_last_push_approval": true, "required_review_thread_resolution": true}},
    {"type": "required_linear_history"},
    {"type": "deletion"},
    {"type": "non_fast_forward"},
    {"type": "required_signatures"}
  ]
}
EOF
```

- Idempotency: if a ruleset with the same name exists, update it with
  `PUT repos/OWNER/REPO/rulesets/{id}` — **full replace**, include the complete
  body (name, enforcement, conditions, rules, bypass_actors). There is no PATCH
  for rulesets (PATCH returns 404).
- Optional admin bypass: add `"bypass_actors": [{"actor_id": 5,
  "actor_type": "RepositoryRole", "bypass_mode": "always"}]` (id 5 = admin)
  so maintainers can push directly to the protected branch; non-admins still
  go through pull requests.
- `pull_request` parameters are **all required** in current API versions:
  `required_approving_review_count`, `dismiss_stale_reviews_on_push`,
  `require_code_owner_review`, `require_last_push_approval`,
  `required_review_thread_resolution`. Omitting any → HTTP 422.
- `target: "branch"` + `ref_name.include: refs/heads/<default>`; also offer
  `"tag"` rules if tags matter.
- Org repos: optionally offer org-level rulesets (`/orgs/{org}/rulesets`).

### Module F — Docs

- `README.md`: if missing or bare, generate one from `templates/README.md`
  (badges, install, usage, modules table). Keep the user's existing content if
  it is already substantive — only append a badges block.
- `CONTRIBUTING.md`, `SECURITY.md`: copy from templates if missing.
- `LICENSE`: ask the user which license (default MIT) before creating.
- `CHANGELOG.md`: create with `# Changelog` header only if missing.

## Verification

```bash
gh label list --repo OWNER/REPO --limit 200 | wc -l
gh api repos/OWNER/REPO/contents/.github/ISSUE_TEMPLATE -q '.[].name' 2>/dev/null
gh api repos/OWNER/REPO/contents/.github/PULL_REQUEST_TEMPLATE.md -q .name 2>/dev/null
gh api repos/OWNER/REPO/contents/.github/workflows/ci.yml -q .name 2>/dev/null
gh api repos/OWNER/REPO/rulesets -q '.[] | {name:.name, enforcement:.enforcement}'
```

Report a final table: `module | status (applied/skipped/failed) | note`.

## Rules of thumb

- **Idempotent**: every module can run twice with the same result.
- **Never clobber user content**: diff first, ask before overwriting customized files.
- **Auth first**: no token, no action — tell the user how to log in, never guess.
- **Dry-run before destructive ops**: rulesets, branch rules, label deletion, visibility changes.
- **Confirm scope**: org/private repos and anything the user didn't explicitly name.

## Templates

All templates live in `templates/`:
`labels.json`, `config.yml`, `issue-form-bug.yml`, `issue-form-feature.yml`,
`issue-form-question.yml`, `PR_TEMPLATE.md`, `ci-node.yml`, `ci-python.yml`,
`ci-go.yml`, `ci-rust.yml`, `CODEOWNERS`, `CONTRIBUTING.md`, `SECURITY.md`,
`README.md`.
