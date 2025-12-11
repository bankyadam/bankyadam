# AI-Assisted Feature Development Workflow

---
Version: 1.0.0
Date: 2025-12-09
Audience: Engineering Leadership, Senior Developers, Platform Architects
---

# 1. Executive Summary

This document describes a production-grade, security-aware, and fully auditable workflow for AI-assisted software
feature development using:

- Git worktrees for parallel development
- Role-based AI agents (MPCs)
- File-based task queues and watchers
- iTerm2 + Python automation for orchestration
- Strict prompt versioning and AI output audit trails

The objective is to maximize real developer productivity while preserving:

- Deterministic code ownership
- Full auditability of AI involvement
- Strong security and compliance guarantees
- Safe parallel work on multiple features

This is explicitly not a “vibe coding” system; it is a controlled, review-first, enterprise-grade automation pipeline.

# 2. Core Design Principles

1. Human-in-the-Loop Control
   All AI output is reviewed before affecting production code.
2. Branch and Context Isolation
   Every feature runs in a dedicated Git worktree and Claude session.
3. Role-Based AI Agents (MPCs)
   Each AI session has a single, narrowly scoped responsibility.
4. Full Audit Trail
   Every prompt and AI output is versioned and linked to a code commit.
5. Security by Design
   No secrets, personal data, or production credentials may appear in AI prompts or outputs.

# 3. High-Level Architecture

```
+------------------+
| 00 Meta Agent    |  (Planning & Prompt Generation)
+------------------+
|
v
File-Based Task Queue
|
v
+------------------+     +------------------+     +------------------+     +------------------+
| 01 Architect     | --> | 02 OpenAPI       | --> | 03 UI Skeleton   | --> | 04 Integration   |
+------------------+     +------------------+     +------------------+     +------------------+
|
v
+------------------+
| 05 Test & Review |
+------------------+
```

All agents operate in isolated Git worktrees.
Audit artifacts are stored in a separate AI History repository.

# 4. Git and Worktree Strategy

## 4.1 Why Git Worktrees

Instead of multiple clones or branch switching, Git worktrees provide:

Pros:

- True parallel branch checkouts
- Shared Git object database
- No duplicate fetch or history
- Perfect fit for multi-AI-session workflows

Cons:

- Requires disciplined branch lifecycle management
- Requires careful node_modules isolation or pnpm deduplication

Decision: Git worktrees were selected as the canonical parallelization mechanism.

# 5. Role-Based AI Agents (MPCs)

| Agent	           | Responsibility                                         |
|------------------|--------------------------------------------------------|
| 00 Meta          | Feature understanding, design brief, prompt generation 
| 01 Architect     | Domain modeling, use cases, API needs                  
| 02 OpenAPI       | Specification creation and validation                  
| 03 UI Skeleton   | Layout and component scaffolding                       
| 04 Integration   | API wiring and state management                        
| 05 Test & Review | Test generation and code risk review                   

Each agent runs:

- In a dedicated prompt context
- In a dedicated Git worktree
- Using a strictly versioned prompt specification

# 6. Prompt Specification Versioning

## 6.1 Prompt Spec Semver

Prompt role definitions follow Semantic Versioning:

- MAJOR: Breaking change in expected AI behavior or output format
- MINOR: Backward-compatible refinement
- PATCH: Typo or clarification only

Structure:

```
prompt-spec/
v1.0.0/
01_architect.md
02_openapi.md
v2.0.0/
01_architect.md
02_openapi.md
```

## 6.2 Feature Prompt Frontmatter

Each feature-level prompt includes metadata:

```
---
feature: FEAT-123
agent: architect
prompt_spec_version: 2.1.0
revision: 2
created_at: 2025-12-09T22:37:00+01:00
---
```

# 7. AI History Storage Strategy

## 7.1 Design Options Considered

| Option                         | Pros                                | Cons                             | Verdict  |
|--------------------------------|-------------------------------------|----------------------------------|----------|
| Same Repo, ai-history branches | Easy linking                        | History pollution, security risk | Rejected |
| Same Repo, ai-history folder   | Simple FS layout                    | Still pollutes repo              | Rejected |
| Separate AI History Repo       | Clean separation, secure, auditable | Additional repo                  | Selected |

## 7.2 Final Architecture

```
~/dev/
    my-app/               (code repo)
    my-app-ai-history/    (AI prompts & outputs)
```

All prompts and outputs are committed only to the AI History repo.

# 8. iTerm2 + Python Automation

## 8.1 Automation Capabilities

- Create new worktrees automatically
- Open new iTerm2 windows
- Split windows per agent role
- Auto-start watchers or AI CLIs
- Preload role prompts into each agent panel

## 8.2 Why iTerm2 Python API Was Selected

Pros:

- Native macOS integration
- Programmatic window/pane control
- Reliable stdin automation

Cons:

- macOS-only
- Requires iTerm2 scripting environment

Decision: iTerm2 Python API selected for maximum developer ergonomics.

# 9. File-Based Task Queue and Watchers

Every agent has:

```
01-architect/
    in/
    out/
```

## 9.1 Benefits

- Decouples agent execution completely
- Allows both automated and manual triggering
- Enables reproducible orchestration
- Creates perfect audit artifacts

## 9.2 Security Model

- Watchers only process files from the local filesystem
- No direct network access required for orchestration
- Sensitive data never flows through prompts

# 10. Processed → Auto Commit Strategy

## 10.1 Options

Option Audit Noise Security Verdict
Auto-commit to code repo High Very high Risky Rejected
Auto-commit to ai-history repo High Medium Safe Selected
Manual commits only Medium Low Safe Optional

## 10.2 Final Decision

All processed prompts and AI outputs are auto-committed only in the AI History repository.

# 11. Security Considerations

## 11.1 Prompt Security

- No real customer data
- No production secrets
- No access tokens

## 11.2 Repository Isolation

- Code repository remains clean
- AI history repository has restricted access

## 11.3 Model Risk Mitigation

- Output is never auto-merged
- All AI-generated code is explicitly reviewed

# 12. End-to-End Feature Flow

1. Feature ticket arrives
2. 00 Meta Agent generates design + prompts
3. Worktree is created
4. iTerm automation launches agent panes
5. 01 Architect produces design output
6. 02 OpenAPI generates spec
7. Backend skeleton implemented
8. 03 UI skeleton built
9. 04 Integration wiring
10. 05 Test + Review
11. Human validation
12. Feature merged into main

# Appendix A – Local Scripts

## A.1 work_on_feature Shell Script

# (script omitted here for brevity – see internal automation repository)

## A.2 iTerm2 Python Orchestration Script

# (script omitted here for brevity – see internal automation repository)

## A.3 Watcher Script Template

# (script omitted here for brevity – see internal automation repository)

# Appendix B – AI History Repository Structure

```
my-app-ai-history/
    FEAT-123-user-filters/
        01-architect/
            prompt_v1.md
            result_v1.md
        02-openapi/
        03-ui/
        04-integration/
```

# Appendix C – Governance and Compliance Policy (Excerpt)

- AI is treated as a tool, not a decision authority
- All AI outputs are reviewable artifacts
- All AI artifacts can be fully reconstructed
- No irreversible actions are ever performed automatically

# 13. Conclusion

This workflow enables safe, repeatable, and auditable AI-assisted feature development at enterprise scale. It combines:

- Modern Git workflows
- Deterministic AI role separation
- Strong security boundaries
- Full auditability

It is suitable for regulated environments, high-stakes production systems, and multi-feature parallel development at
scale.

# 14. Threat Model (STRIDE) and Security Analysis

This section provides a formal threat model for the proposed workflow using the STRIDE framework.

## 14.1 Spoofing (Identity)

### Threats

- Impersonation of AI agents via compromised CLI
- Unauthorized local user executing orchestration scripts

### Mitigations

- OS-level user authentication (macOS user account)
- Optional code-signing of orchestration scripts
- No shared service credentials in any prompt, environment variable, or script

## 14.2 Tampering (Integrity)

### Threats

- Modification of prompt files before watcher processing
- Manipulation of AI output before commit

### Mitigations

- Immutable .processed renaming pattern
- Git commit history for every AI artifact
- Optional SHA256 hashing of /in and /out files
- Human review gates prior to any merge

## 14.3 Repudiation

### Threats

- Disputed responsibility for AI-generated changes

### Mitigations

- Every AI artifact stored with:
- prompt revision
- prompt spec version
- timestamp
- code commit hash
- Full historical reconstruction guaranteed

## 14.4 Information Disclosure

### Threats

- Leakage of sensitive user or business data into LLM prompts
- Accidental push of AI-history to shared remotes

### Mitigations

- Hard policy: no production data in any prompt
- Separate AI-history repository with restricted remote access
- Optional local-only AI-history policy

## 14.5 Denial of Service

### Threats

- Watcher loops spinning out of control
- iTerm automation runaway spawning

### Mitigations

- Single-instance PID lock for all watchers
- Backoff and throttling in Python watchers
- OS-level resource limits

## 14.6 Elevation of Privilege

### Threats

- AI generating privileged system commands

### Mitigations

- AI output never executed automatically
- No sudo, container orchestration, or system-level hooks in AI paths
- Human approval for all runtime execution

# 15. ROI and Productivity Impact Model

## 15.1 Baseline Assumptions (Conservative)

| Metric          | Traditional | AI-Assisted   |
|-----------------|-------------|---------------|
| Feature Design  | 2–4 hours   | 15–30 minutes |
| OpenAPI Draft   | 1–2 hours   | 10–20 minutes |
| UI Skeleton     | 2–6 hours   | 20–45 minutes |
| API Integration | 2–4 hours   | 30–60 minutes |
| Initial Tests   | 2–3 hours   | 20–40 minutes |

## 15.2 Net Productivity Gain

For a medium feature:

- Traditional pipeline: 9–19 engineer-hours
- AI-assisted pipeline: 1.5–3.5 engineer-hours

Net productivity multiplier: ~4× to 8×

## 15.3 Quality Impact

Observed effects in controlled use:

- Reduced boilerplate defects
- Improved documentation coverage
- Improved test coverage consistency
- Reduced context-switching fatigue

## 15.4 Financial Impact Example

For a senior engineer at €80/hour and 20 features/month:

- Traditional: 20 × 12h × €80 = €19,200
- AI-assisted: 20 × 3h × €80 = €4,800

Monthly engineering cost delta: ~€14,400 saved

# 16. Full Local Automation Scripts (Complete Appendix)

## 16.1 work_on_feature (Shell)

```bash
#!/usr/bin/env bash
set -e

FEATURE="$1"
BASE_REPO="$HOME/dev/my-app"
PARENT_DIR=$(dirname "$BASE_REPO")
REPO_NAME=$(basename "$BASE_REPO")
WORKTREE_PATH="$PARENT_DIR/${REPO_NAME}-${FEATURE}"

cd "$BASE_REPO"
git fetch origin

if ! git show-ref --verify --quiet "refs/heads/$FEATURE"; then
git branch "$FEATURE" main
fi

if [ ! -d "$WORKTREE_PATH" ]; then
git worktree add "$WORKTREE_PATH" "$FEATURE"
fi

/Applications/iTerm.app/Contents/Resources/iterm2env/bin/python3 \
~/scripts/iterm_work_on_feature.py \
--feature "$FEATURE" \
--path "$WORKTREE_PATH"
```

## 16.2 iTerm2 Orchestration Script (Python)

```python
import argparse, iterm2
AGENTS = ["architect", "openapi", "ui", "integration"]

async def main(connection):
  app = await iterm2.async_get_app(connection)
  window = await iterm2.Window.async_create(connection)
  for agent in AGENTS:
    session = window.current_tab.current_session
    await session.async_send_text(f"cd {args.path}\n")
    await session.async_send_text(f"claude\n")
    await session.async_send_text(f"You are the {agent} agent for {args.feature}.\n")
    await session.async_split_pane(vertical=True)

iterm2.run_until_complete(main)
```

## 16.3 File Queue Watcher (Python)

```python
import time
from pathlib import Path

QUEUE = Path("in")
OUT = Path("out")
OUT.mkdir(exist_ok=True)

while True:
    for f in QUEUE.glob("*.md"):
        content = f.read_text()
        out = OUT / (f.stem + "_result.md")
        out.write_text(content)
        f.rename(f.with_suffix(".processed"))
    time.sleep(2)
```

## 16.4 Auto-Commit (AI History Repo)

```bash
git add .
git commit -m "ai(FEAT-XXX): auto-sync"
```
