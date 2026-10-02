# 07 — Git Standards

## Branch Naming

All branches must follow this format:

```
<type>/JIRA-XX-short-description
```

Types:
```
feature/    — new capability or feature
bugfix/     — fix for a reported defect
refactor/   — structural improvement with no behavior change
test/       — adding or improving tests only
chore/      — build, CI, tooling, dependency updates
docs/       — documentation only
```

Examples:
```
feature/NAI-12-chat-screen-ui
bugfix/NAI-34-fix-streaming-cancellation
refactor/NAI-56-extract-ai-provider-interface
test/NAI-78-add-rag-pipeline-tests
```

Never push directly to `main` or `develop`. All work goes through pull requests.

---

## Commit Messages

Format:
```
JIRA-XX: imperative short description (≤72 chars)
```

Body (optional, separated by blank line):
```
JIRA-XX: imperative short description

Longer explanation of why this change was made, what tradeoffs were
considered, or any non-obvious context. Wrap at 72 characters.
```

Rules:
- Use imperative mood: "add", "fix", "extract", "remove" — not "added", "fixing"
- Subject line ≤ 72 characters
- Reference the Jira ticket in every commit
- One logical change per commit — do not bundle unrelated changes
- Do not commit work-in-progress code to shared branches

Examples:
```
NAI-12: add ChatScreen composable with message list and input
NAI-34: fix streaming flow cancellation on ViewModel clear
NAI-56: extract AIProvider interface to domain layer
```

---

## Pull Requests

Title format mirrors the commit format:
```
JIRA-XX: short imperative description (≤72 chars)
```

PR description must include:
- **Summary** — what this PR does and why
- **Changes** — key files and components changed
- **Testing** — what was tested and how
- **Blocked / Notes** — anything not yet complete or known issues

Rules:
- PRs must be focused — one ticket, one concern
- Keep PRs small enough to review in a single sitting; split large features into sequential PRs
- All CI checks must pass before requesting review
- All CI checks must pass before merging
- At least one approval required before merge (when working in a team context)
- Resolve all review comments before merging — do not dismiss without addressing

---

## What Must Never Be Committed

- API keys, tokens, passwords, or any secret value
- `google-services.json` with production credentials
- Keystore files (`.jks`, `.keystore`)
- `local.properties`
- Generated credentials or certificates
- `.idea/` or other IDE-specific files (already in `.gitignore`)
- Build outputs (`build/`, `.gradle/`, `*.apk`, `*.aab`)

If a secret is accidentally committed: rotate it immediately, then remove it from history. A removed-but-historically-present secret is still compromised.

---

## Commit Hygiene

- Stage specific files — avoid `git add .` which may accidentally include unintended files
- Review the diff before committing: `git diff --staged`
- Keep commits atomic: each commit should build and, where practical, pass tests independently
- Do not use `--no-verify` to bypass pre-commit hooks unless there is a documented reason
- Do not use `--amend` on commits that have already been pushed to a shared branch
- Do not use `git push --force` on shared branches

---

## CI Requirements

The following must pass before any merge to `main` or `develop`:

- Build succeeds (debug and release)
- All unit tests pass
- All instrumented tests pass (or are tagged and run in CI)
- Lint passes with no new errors
- No new security issues introduced (dependency audit where configured)

CI configuration lives in the repository. Do not bypass or disable CI checks to unblock a merge.

---

## Release Tagging

Release tags follow semantic versioning:
```
v<MAJOR>.<MINOR>.<PATCH>
```

Example: `v1.0.0`, `v1.2.3`

Tag releases from `main` only. Tags must be annotated:
```
git tag -a v1.0.0 -m "Release v1.0.0"
```
