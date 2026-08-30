# Swift Engineering Guidelines

Generic engineering rules for AI coding agents working on Swift projects. This file is copied verbatim across projects — keep it project-agnostic. Project-specific facts (build commands, ticket team, branch policy) live in each repo's `AGENTS.md`.

## Goals

- Correctness
- Idiomatic code
- Maintainability
- Simplicity, avoid over engineering
- Developer ergonomics
- References to documentation or sources should be included inline as comments
- Interfaces should be similar in style to Apple's interfaces; follow the Swift API Design Guidelines (https://www.swift.org/documentation/api-design-guidelines/)
- Prefer modern frameworks and libraries over legacy versions for new code (Swift 6 concurrency over Dispatch, SwiftUI over AppKit or UIKit). Changing an existing idiom is governed by "Apple-framework idioms" below, not by this preference.
- Code should follow recommended and standard practices from Apple. For Swift, SwiftUI, AppKit, UIKit, Foundation, Swift Concurrency, ARC, memory management, lifecycle, threading, and API-usage questions, do not rely on memory for "recommended" or "standard" practice. Verify against Apple documentation first, then cite the specific source in the proposal or code comments when relevant.
- For each behavior or UX finding (and for each behavior decision in a new feature), name the closest Apple-API precedent — `URLSession`, `NWConnection`, `NotificationCenter`, etc. — and note whether the proposed or current behavior diverges. Divergence from the closest analogue is a strong signal that a "doc gap" is actually a behavior bug, and that a "minor UX quirk" is actually surprising friction.

## Requirements

- Swift 6 projects MUST build in Swift 6 language mode with full strict concurrency, with no warnings.
- As much code as possible should have test coverage.
- Do not propose a plan based on inferred convention when the choice between two correct implementations depends on Apple guidance. Verify first, then choose the simplest implementation consistent with that guidance.

## Apple-framework idioms

- Removing, updating or replacing an Apple-framework idiom (NotificationCenter, KVO, NSNotification, delegate, target/action, Combine, `@Observable`, dispatch queues) is an ARCHITECTURE change, not an implementation detail. Treat it as a refactor: surface it explicitly, get approval, and do NOT bundle it into a bug fix or feature ticket.
- When fixing a bug in code that uses a Cocoa idiom, the default is to PRESERVE the idiom and fix the specific defect. Removing the idiom requires:
  1. Quoting Apple guidance that recommends against it for this use case, OR
  2. Explicit user approval to do an architectural refactor.
- The Pre-v1 "cleaner end state now" rule does NOT authorize removing Cocoa idioms. It applies to internal API shape, not framework-level patterns.

## Bug fix vs refactor

- Before proposing an implementation, classify the change explicitly:
  - **bug fix** — minimal change, preserves existing architecture
  - **feature** — additive, preserves existing architecture
  - **refactor** — explicitly labeled, gated on separate approval
- If you find yourself proposing a refactor while solving a bug, STOP and SPLIT: propose the surgical fix, file the refactor as a separate ticket, let the user decide.
- "Architectural cleanup" is never a bonus deliverable bundled into a fix.
- The existence of a test does not endorse the behavior it covers. A test proves a path works; it does not prove that path is the intended contract. When triaging a finding bracketed by a test, classify the behavior separately from the test — ask whether an embedder would expect the tested behavior, not whether the test passes.

## Process

- Take an incremental approach, outlining the plan and steps before executing to allow me to indicate if we want to proceed and how far to progress.
- When I point out a problem with your approach, STOP. Describe your proposed fix and wait for confirmation before proceeding.
- Before executing a ticket, re-validate the change classification (bug fix vs feature vs refactor vs doc) from first principles. Do NOT inherit the classification from the ticket body or its source analysis — analysis-time triage can be wrong, and execution is the second chance to catch it. If the re-validation disagrees with the ticket body, STOP and surface the disagreement before proposing a plan.
- **Apple doc pages are JS-rendered** (as of June 2026) and frequently come back empty via plain web fetch. Verify Apple-framework behavior through a source that renders them (a docset tool) or the SDK headers under `…/<Framework>.framework/Headers/`; quote the abstract/declaration when citing.
- When you cannot verify official documentation through available tools, STOP and ask me. Do not substitute blog posts, forum threads, Stack Overflow, or empirical experimentation for official documentation. Say "I can't access the docs — can you check, should I search for secondary sources, or how else would you like to proceed?"
- When you can verify something yourself with a safe, non-destructive command (e.g. curl, checking a running service), do it directly instead of asking the user to run it.

## Code

- All code should be written to be correct, readable, obvious, and maintainable.
- Prioritize maintainability over 'cleverness' or extreme performance.
- Note any bad or outdated coding patterns.
- Code should compile without warnings.

## Testing

- Critical codepaths should be tested.
- When adding new tests, ensure they actually INCREASE code coverage. Do NOT add tests that don't increase code coverage.
- Test code should compile without warnings.
- It is acceptable to add tests that fail as a way of ensuring certain code paths are executed, but a failing test may only land disabled, with a follow-up ticket to fix the behavior and enable it. Never land an enabled, failing test.
- Only coverage of the framework or app matters; coverage of the test code is excluded.

## Documentation

- Document the code as necessary, targeting a junior level engineer.
- Inline documentation is welcome, especially on public methods, interfaces, structures, etc.
- Utilize platform specific documentation standards.
- Changes that only affect documentation do NOT require build and test phases.

### Version references in comments

- Do NOT put release-version numbers (`v0.1.0`, `pre-v0.1.0`, etc.) in source comments or DocC. They are unenforced, duplicate the authoritative source (git tag / `Package.swift` / `README.md`), and decay silently — a comment that says `v0.1.0` is wrong the moment `v0.1.1` ships.
- The version-of-record is the git tag and `README.md`. Keeping the README's version line current is part of cutting a release.
- To mark behavior as provisional, use intent prose ("currently", "for now") plus a Linear ticket for the lift condition. A ticket ID is a durable anchor; a version number is not.
- The pre-1.0 fail-fast policy may be referenced as "pre-1.0 fail-fast policy" (a documented phase with a defined end), never as a specific release number.
- Genuine availability history (post-1.0) goes in the compiler-checked `@available` attribute, not free-text comments.

## Context

- Context for the project will be maintained in various .md files included in the project and the ticket tracker.
- When explicitly requested, create .md files to accurately reflect the current state of the project: ANALYSIS, FEATURE, PLAN, etc. When creating such a file, add a suffix with the model and thinking level — for example ANALYSIS_Opus-4.6-high.md or ANALYSIS_gpt-5-codex-medium.md.
- In analysis documents, frame each behavior or UX finding as a question that surfaces the design choice — e.g., "Is requiring X before Y the right contract, or should the API handle it?" — rather than a statement that locks the current behavior. Documentation, naming, or pure-bug findings (parser crashes, leaks, races) may remain as statements; only behavior and UX findings need the question framing.
- Recommend the creation of FEATURE and PLAN files when capturing significant insight.
- If an existing ANALYSIS/FEATURE/PLAN file is directly relevant to the work being changed, update it to keep it current.
- Bugs, improvements and new features should be captured in Linear tickets, recording the source agent (see "Issue tracking").
- To "archive" a file means to move it to the `Attic/` directory.

## Pre v1.0.0 rules (for apps and frameworks still in initial development)

- Prioritize 'correct' or 'standard' interfaces as defined by Apple.
- Prefer the cleaner end state now.
- Do NOT preserve the public API for compatibility.
- Do not add shims, aliases, migrations, or compatibility layers unless I explicitly ask for them.
- fatalErrors are acceptable in pursuit of failing early, often and loudly while developing the project.
- If a project does NOT have an explicit version tag, assume it is still in initial development.

## Source control

- NEVER git commit to a repo without asking for explicit permission, no exceptions. Treat commit, push, and ticket updates as separate steps, each with its own approval.
- When ready to commit: show me the staged files, the proposed commit message, and ask "Ready to commit?" — then wait for my explicit approval before running git commit.
- In non-interactive runs (cloud tasks, CI, scheduled agents) nobody can grant approval: never commit; stop at the gate and report instead.
- Changes may be happening in another context or terminal so check git status before making any assumptions.
- The abbreviation CPU stands for "Commit, Push and Update Tickets". This essentially means: the proposed change looks good, commit it to the branch, push the branch upstream and update any related tickets. Do not use this abbreviation in your responses, but I may.
- The abbreviation EPRT stands for "Evaluate and Propose a Resolution for Ticket". This should be followed by a ticket id. Do not use this abbreviation in your responses, but I may.

### Branch

- If we are working in mainline branches in repos that we own, it is acceptable to commit directly to the repo.
- If we are working in forks and will need to open a PR for review, create a branch based off the ticket we are executing on if it exists.

## Issue tracking

- We use Linear for issue tracking, accessed via the Linear MCP server. The team and project for each repo are named in its `AGENTS.md`.
- If you are unable to contact the Linear MCP service, inform the user.
- Every ticket created or updated records the **source agent** — the model and thinking level, e.g. `Opus-4.6-high` or `gpt-5-codex-medium`.
- Findings, decisions, and follow-ups belong in Linear — not scattered through code comments.
- Before saying a ticket is blocked by another ticket, verify the blocker’s current Linear status and, if relevant, inspect the current codebase for the landed behavior.

### Linear defaults

When I ask you to list, triage, evaluate, update, prioritize, comment on, or
otherwise interact with Linear tickets, default to open/uncompleted issues
only.

Exclude completed, canceled, and archived issues unless I explicitly ask for
historical, closed, completed, canceled, archived, or all tickets.

Before bulk-updating Linear issues, state the filter being used and the count
of matching issues.

  ## Concurrent worktree sessions

  Multiple agents may be working in this same git worktree at the same time.

  - Never assume existing staged or unstaged changes were made by the current
  agent.
  - Treat all pre-existing changes as user/other-agent work unless this session
  explicitly made them.
  - When evaluating the working directory for a ticket, distinguish:
    - changes present in the working directory/index
    - changes made by this session
    - changes that appear related to other tickets
  - Do not stage, unstage, revert, amend, commit, or combine changes from other
  sessions without explicit approval.
  - If staged changes include multiple ticket IDs, report that fact and ask how
  to split or proceed. Do not propose a combined commit unless explicitly asked.
  - Before proposing a commit, list the staged files and ticket IDs visible in
  the diff, and call out any changes not made by this session.
