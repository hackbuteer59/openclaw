<!--
  Sync Impact Report
  ===========================
  Version change: 0.0.0 → 1.0.0 (MAJOR: initial ratification)
  Modified principles: N/A (initial creation)
  Added sections:
    - Core Principles (5 principles)
    - Technology Constraints
    - Development Workflow
    - Governance
  Removed sections: N/A
  Templates requiring updates:
    - .specify/templates/plan-template.md ✅ (Constitution Check section
      already references "[Gates determined based on constitution file]";
      no changes needed — plan generation will read this constitution)
    - .specify/templates/spec-template.md ✅ (no constitution-specific
      references; compatible as-is)
    - .specify/templates/tasks-template.md ✅ (phase structure is generic;
      compatible as-is)
  Follow-up TODOs: none
-->

# OpenClaw Constitution

## Core Principles

### I. Security-First Defaults

All features MUST ship with secure defaults. Risky paths MUST be
explicit and operator-controlled. The operator trust model documented
in `SECURITY.md` is authoritative:

- One gateway instance = one trusted operator boundary.
- Exec behavior defaults to host; sandbox mode is opt-in but
  MUST be clearly surfaced during onboarding.
- Credentials MUST never be committed, logged, or exposed in
  CLI output unless the operator explicitly requests it.
- New network-facing surfaces MUST be reviewed against the
  OWASP Top 10 before merge.

**Rationale**: OpenClaw runs on personal devices with real
credentials. Security failures erode the core value proposition
of a local-first assistant.

### II. Channel Universality

Every user-facing feature MUST consider **all** built-in and
extension messaging channels. When refactoring shared logic
(routing, allowlists, pairing, command gating, onboarding, docs),
all channels MUST be enumerated and tested or explicitly marked
as out-of-scope with justification.

- Core channels: WhatsApp, Telegram, Slack, Discord, Google Chat,
  Signal, iMessage, BlueBubbles, IRC, WebChat.
- Extension channels: Microsoft Teams, Matrix, Feishu, LINE,
  Mattermost, Nextcloud Talk, Nostr, Synology Chat, Tlon,
  Twitch, Zalo, Zalo Personal, voice-call.
- Adding a new channel MUST include updates to `.github/labeler.yml`,
  matching GitHub labels, docs, and status/configuration UI surfaces.

**Rationale**: OpenClaw's differentiation is broad channel reach.
A feature that works on one channel but silently breaks another
degrades the product for every user on that channel.

### III. Plugin-Over-Core

Core MUST stay lean. Optional capability SHOULD ship as a plugin
(extension) or ClawHub skill rather than being added to core.

- New skills MUST be published to ClawHub first; core skill
  additions require a strong product or security justification.
- Plugin-only dependencies MUST live in the extension's own
  `package.json`, never in the root `package.json`.
- `workspace:*` MUST NOT appear in plugin `dependencies`
  (breaks `npm install`); use `devDependencies` or
  `peerDependencies` for `openclaw`.

**Rationale**: A lean core improves startup time, reduces attack
surface, and keeps the dependency tree manageable for users who
install globally via npm.

### IV. TypeScript Discipline

All production code MUST be TypeScript (ESM, strict mode).

- `any` MUST be avoided; `@ts-nocheck` MUST NOT be added.
- Oxlint + Oxfmt MUST pass (`pnpm check`) before merge.
- Files SHOULD stay under ~700 LOC; split/refactor when larger.
- No prototype mutation for sharing class behavior; use explicit
  inheritance or composition.
- Dynamic imports MUST NOT mix `await import("x")` and static
  `import ... from "x"` for the same module in production paths.
- Existing utilities (formatting, tables, progress, theme) MUST
  be reused; duplicating helpers is prohibited.

**Rationale**: Strict typing catches integration bugs early.
Consistent style reduces review friction and accelerates
onboarding for new contributors.

### V. Test Coverage Gate (NON-NEGOTIABLE)

All logic changes MUST be covered by tests. The project enforces
V8 coverage thresholds of **70%** for lines, branches, functions,
and statements.

- Framework: Vitest. Test files MUST be colocated (`*.test.ts`).
- `pnpm build && pnpm check && pnpm test` MUST pass before push.
- E2E tests use `*.e2e.test.ts`; live tests gated behind
  `CLAWDBOT_LIVE_TEST=1` or `LIVE=1`.
- Coverage regressions below thresholds MUST NOT be merged.

**Rationale**: The gateway handles real messages on real channels.
Untested code paths risk sending malformed or duplicate messages
to users' personal contacts.

## Technology Constraints

- **Runtime**: Node 22+ (Bun supported for dev/scripts execution).
- **Package manager**: pnpm (keep `pnpm-lock.yaml` in sync).
  Bun patching MUST stay in sync when touching deps/patches.
- **Build**: `pnpm build` (tsdown → `dist/`). Type-check: `pnpm tsgo`.
- **Lint/Format**: Oxlint + Oxfmt (`pnpm check`, `pnpm format`).
- **Tests**: Vitest with V8 coverage. Max 16 test workers.
- **Carbon dependency**: MUST NOT be updated (frozen).
- **Patched dependencies** (`pnpm.patchedDependencies`): MUST use
  exact versions (no `^`/`~`). New patches require explicit approval.
- **CLI progress/spinners**: use `src/cli/progress.ts` only.
- **Terminal tables/colors**: use `src/terminal/table.ts` and
  `src/terminal/palette.ts` only.

## Development Workflow

- **One PR = one issue/topic.** Do not bundle unrelated changes.
- **PR size limit**: PRs over ~5,000 changed lines are reviewed
  only in exceptional circumstances.
- **Commits**: use `scripts/committer "<msg>" <file...>`.
  Concise, action-oriented messages (e.g., `CLI: add verbose flag`).
- **Pre-commit**: `prek install` runs the same checks as CI.
- **Quality gates before merge**:
  1. `pnpm build` succeeds (no `[INEFFECTIVE_DYNAMIC_IMPORT]` warnings).
  2. `pnpm check` succeeds (lint + format).
  3. `pnpm test` succeeds (coverage thresholds met).
- **Anti-redundancy**: before creating any utility or helper,
  search existing code. Import from the canonical source; never
  create re-export wrapper files.
- **Docs**: use Mintlify conventions (root-relative links, no
  `.md`/`.mdx` extensions). Keep docs generic — no personal
  hostnames, device names, or real phone numbers.
- **Changelog**: user-facing changes only. Append new entries to
  the **end** of the target section in the active version block.

## Governance

This constitution is the authoritative source of project-wide
engineering principles. It supersedes ad-hoc conventions except
where `AGENTS.md` provides more specific operational guidance
(in which case `AGENTS.md` takes precedence for that scope).

- **Amendments** MUST be documented with a version bump, rationale,
  and migration plan when principles are added or removed.
- **Versioning** follows semantic versioning:
  - MAJOR: principle removal or backward-incompatible redefinition.
  - MINOR: new principle or materially expanded guidance.
  - PATCH: clarifications, wording, typo fixes.
- **Compliance review**: all PRs and code reviews SHOULD verify
  alignment with these principles. Complexity beyond minimum
  requirements MUST be justified in the PR description.
- **Final authority**: the Benevolent Dictator (project lead) holds
  final decision on amendments and exceptions.
- **Runtime guidance**: `AGENTS.md` at the repository root contains
  operational details (build commands, commit workflow, release
  process). This constitution defines _why_; `AGENTS.md` defines
  _how_.

**Version**: 1.0.0 | **Ratified**: 2026-03-14 | **Last Amended**: 2026-03-14
