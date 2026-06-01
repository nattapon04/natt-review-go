<div align="center">

# 🔍 go-review

**A Claude Code skill that reviews your Go code like a senior developer**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![Go](https://img.shields.io/badge/Go-1.18+-00ADD8?logo=go)](https://golang.org)

</div>

---

## What it does

Paste your Go code (or point to a file/diff) and Claude will respond as a **senior Go developer** doing a real review — not a rubber stamp.

The skill covers:

| Dimension | What it checks |
|---|---|
| **Idiomatic Go & Formatting** | Error handling, naming, interface design, context propagation, gofmt/goimports, doc comments |
| **Performance** | Goroutine leaks, unnecessary allocations, mutex contention |
| **Security** | SQL injection, race conditions, unsafe usage, unvalidated input |
| **Change Impact** | What other files/callers might break from your change |

The review focuses on **what changed** and **what that change affects** — not a full project scan. Output is a **structured report**. Claude will then ask which findings you want fixed — nothing is changed without your confirmation.

---

## Installation

### 1. Clone this repo

```bash
git clone https://github.com/nattapon04/natt-review-go.git
```

### 2. Add to Claude Code settings

Edit `~/.claude/settings.json`:

```json
{
  "skills": [
    "/path/to/natt-review-go"
  ]
}
```

<details>
<summary>Windows path example</summary>

```json
{
  "skills": [
    "C:\\Users\\yourname\\path\\to\\natt-review-go"
  ]
}
```

</details>

### 3. Restart Claude Code

The skill is now active.

---

## Usage

Just ask naturally — no special command needed:

```
"review this Go code"
"is this idiomatic Go?"
"check if my change breaks anything"
"can you review my PR?"   ← works on diffs with .go files
```

Or paste Go code directly and ask *"what do you think?"*

---

## Example output

<details>
<summary>Click to expand sample report</summary>

```
## Go Code Review

### Summary
The service layer is mostly solid but has a critical goroutine leak in the
background worker and several places where errors are silently swallowed.
Not production-ready as-is.

### Findings

1. **[CRITICAL] Performance — Goroutine leak in StartWorker**
   - Where: `worker.go:42`
   - Problem: The goroutine started here has no exit condition if ctx is cancelled.
     This will accumulate goroutines over time and eventually OOM the process.
   - Suggestion: Select on ctx.Done() inside the loop and return when it fires.

2. **[MAJOR] Idiomatic Go — Error swallowed in SaveUser**
   - Where: `service/user.go:88`
   - Problem: `if err != nil { log.Println(err) }` logs the error but continues
     execution, silently corrupting state downstream.
   - Suggestion: Return the error to the caller with context using fmt.Errorf.

3. **[MINOR] Naming — Redundant package prefix**
   - Where: `model/user.go:12`
   - Problem: `user.UserID` → should be `user.ID`

### Cross-file Impact
- `handler/user_handler.go` — calls SaveUser; will not receive the new error
- `repository/user_repo.go` — interface change requires updating the test mock

### What's Done Well
- Context is correctly propagated through all public functions
- Table-driven tests in user_test.go are well-structured
- Struct fields use named initialization throughout

---
ต้องการให้แก้ข้อไหนบ้างครับ? (เช่น "แก้ข้อ 1, 2")
```

</details>

---

## Customization

Edit [`SKILL.md`](SKILL.md) to add project-specific rules, adjust severity thresholds, or change the report language.

---

## License

MIT
