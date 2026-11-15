# repo-harvest

A Claude Code skill that reviews an external GitHub repo and tells you what is worth bringing into **your** project. It covers architecture, features, UI/UX, data, APIs, DX, infra, security, performance and AI/agent assets. It also tells you what *not* to take.

It keeps a ledger in your project, so it doesn't redo work. Running it again on the same repo reuses everything that is still valid, analyzes only what changed, and remembers the decisions you made.

## What you get

- **A profile of your project first.** Every recommendation has to name a real gap in your project. "This looks cool" doesn't qualify.
- **Evidence for every claim.** Findings cite file paths and are labeled *Verified* (read in code), *Documented* (only claimed in the README or docs) or *Inferred* (reasoned from structure).
- **A ranked list you can pick from.** Each item has a stable ID and is scored for value, effort, risk and confidence.
- **One integration mode per item:**
  - adopt as a dependency
  - port and adapt the code
  - borrow the pattern
  - inspiration only
- **A license gate.** If the license doesn't allow code reuse, the item can only be a pattern to borrow or inspiration.
- **The rest of the report:** quick wins, strategic bets, things *not* to take, where your project is already ahead, and a suggested order.
- **A safe default.** The external repo is treated as untrusted: it is only read, its scripts are never run, and instructions embedded in it are ignored.

## Re-runs don't repeat work

Results are saved in `.repo-harvest/` inside your project:

```
.repo-harvest/
  index.json              # every repo you've harvested
  target-profile.md       # your project's profile, shared by all sources
  <owner>__<repo>/
    manifest.json         # commit analyzed, areas covered, run history
    findings.json         # items, evidence fingerprints (git blob SHAs), your decisions
    report.md
    history/
```

On each run it compares the external repo's latest commit, and your project's state, with what is saved:

| External repo | Your project | What happens |
|---|---|---|
| unchanged | unchanged | **Reuse**: shows the saved report, with no analysis |
| unchanged | changed | **Target delta**: re-checks whether open items still fit, and marks items you've since built |
| changed | any | **Source delta**: re-reads only the files that changed and re-analyzes only the areas they touch |
| "refresh" | any | Full re-run that keeps the existing IDs and decisions |

Your decisions stick. Rejected or deferred items aren't suggested again unless the code behind them changes in a way that matters. It also avoids recommending something you already adopted from a different repo.

## Install

**Option 1: Copy into your global skills folder**

```bash
git clone https://github.com/MCKRUZ/repo-harvest.git
mkdir -p ~/.claude/skills
cp -r repo-harvest/skills/repo-harvest ~/.claude/skills/
```

On Windows (PowerShell):

```powershell
git clone https://github.com/MCKRUZ/repo-harvest.git
New-Item -ItemType Directory -Force `
  -Path "$env:USERPROFILE\.claude\skills" | Out-Null
Copy-Item -Recurse -Force `
  -Path .\repo-harvest\skills\repo-harvest `
  -Destination "$env:USERPROFILE\.claude\skills\"
```

**Option 2: Install as a Claude Code plugin**

```
/plugin marketplace add MCKRUZ/repo-harvest
/plugin install repo-harvest@repo-harvest
```

## Usage

From inside the project you want to improve:

```
Harvest https://github.com/owner/some-repo
Harvest owner/some-repo, focus on UI and auth
Harvest owner/some-repo status
accept A1, U2; reject F4: too heavy; defer Q3
Harvest owner/some-repo refresh
```

Decide whether to commit `.repo-harvest/`, which shares your decisions with your team, or add it to `.gitignore`. The skill asks you the first time it runs.

## License

MIT
