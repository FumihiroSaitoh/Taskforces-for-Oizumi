# CLAUDE.md

This file guides Claude Code (claude.ai/code) when working in this repository.

## What this repository is

This repository is **not an application codebase**. It has no source code, build
system, package manifest, or tests. It exists solely to hold a set of **custom
Claude Code skills** (`.claude/skills/`) used by the repo owner for medical
office / clinic workflows and general Claude Code productivity ("Taskforces
for Oizumi" — a personal/team toolkit, currently described in the README as a
"Claude code test").

Because there is no application to build or run, most standard software
engineering workflows (install, build, lint, test, deploy) do not apply here.
When asked to "develop" in this repo, the actual unit of work is almost always
**adding, editing, or refining a skill** under `.claude/skills/`.

## Repository structure

```
.
├── README.md                              # One-line repo description
├── CLAUDE.md                               # This file
└── .claude/
    └── skills/
        ├── grill-me/SKILL.md               # Adversarial plan/design interview skill
        ├── referral-letter/SKILL.md        # Japanese medical referral letter generator
        └── subagent-review/SKILL.md        # 3-agent parallel code review orchestrator
```

There are no other directories, no CI configuration, and no dependency
manifests as of this writing.

## The skills

Each skill is a single `SKILL.md` file with YAML frontmatter (`name`,
`description`) followed by the skill's instructions in the body. Skills are
invoked either automatically (Claude Code matches the `description` against
user intent) or explicitly via `/skill-name`. All three existing skills are
written **entirely in Japanese**, targeting a Japanese-speaking clinical/dev
user — keep new skills consistent with this unless told otherwise.

### `grill-me`
Interviews the user exhaustively about a plan or design, one question at a
time, until shared understanding is reached — walking each branch of the
decision tree and proposing a recommended answer alongside every question.
Triggered by phrases like "grill me" or requests to stress-test a plan.
Instructs Claude to actually explore the codebase before answering
codebase-answerable questions, rather than guessing.

### `referral-letter` (診療情報提供書作成)
Generates a Japanese medical referral letter (紹介状 / 診療情報提供書) from CSV
data exported from an electronic medical record (電子カルテ). This is the most
structurally detailed skill — it enforces a strict two-step workflow:

1. **Step 1 (analysis + confirmation)**: parse the CSV's column structure,
   list all diagnoses found, and explicitly ask the user (a) which diagnosis
   is the referral's primary reason and (b) whether the column interpretation
   is correct. Document generation must not start before this confirmation.
2. **Step 2 (document generation)**: produce the letter in a fixed section
   order (紹介病名 / 紹介目的 / 合併症・併存症 / 既往歴・既往疾患 / 依頼内容 /
   その他 / 結び / 現在の処方), in polite written Japanese (です・ます調), with
   precise rules per section — e.g. 依頼内容 must open with a one-sentence
   core reason for referral before the chronological narrative, has a
   length budget calibrated to case complexity (800–2000字), and has an
   explicit list of forbidden content (no treatment suggestions to the
   receiving specialist, no follow-up instructions, no prognosis
   predictions). Missing data must be marked `〔データなし〕`, never inferred.

When editing this skill, preserve these hard constraints — they encode
professional/medical-communication norms (a referring physician does not
tell the specialist how to treat, and does not fabricate missing chart data).

### `subagent-review`
Orchestrates a 3-way parallel code review using three role-played senior
agents: 可読性 (readability), 設計 (architecture), and セキュリティ (security,
OWASP Top 10-oriented). Workflow:

1. Determine scope: `--full` reviews the whole codebase; no argument reviews
   `git diff`; if the diff is empty, ask the user before falling back to
   `--full`.
2. Infer the product's domain/stack from `CLAUDE.md`, dependency manifests,
   and directory layout, and confirm the inference with the user in one line.
3. Launch all three review agents **in parallel** (each individually
   disable-able via `--no-readability` / `--no-design` / `--no-security`).
4. Merge results into a Japanese report: overall verdict (良好 / 要改善 /
   要大幅改善), points of disagreement between agents, a deduplicated
   priority table (高/中/低 with file:line), then each agent's full report.

## Conventions for AI assistants working here

- **Default language for skill content is Japanese.** Keep new/edited
  `SKILL.md` bodies in Japanese to match the existing skills unless the user
  asks for something else.
- **Skill file format**: YAML frontmatter with `name` and `description` only
  (no `tools` or other fields currently used), followed by the instruction
  body in Markdown. The `description` is what Claude Code's skill matcher
  uses to decide when to auto-invoke a skill — write it to include concrete
  Japanese trigger phrases (e.g. 「grill me」「紹介状を書いて」), the way the
  existing three skills do.
- **Do not add application scaffolding speculatively.** There is currently no
  package.json, build tooling, or test framework. Don't introduce one unless
  the user asks for an actual application to be built — this repo's purpose
  today is skill authoring.
- **Medical content requires care.** `referral-letter` produces clinical
  documents intended for real patient referrals. Do not loosen its
  data-fidelity rule (only CSV-sourced facts, no inference/fabrication) or
  its prohibition on offering treatment advice to the receiving physician —
  these aren't stylistic choices, they reflect what's professionally/
  ethically appropriate for a referral letter.
- **No CI/build/test commands exist.** Don't invent lint/test/build commands
  in documentation or instructions — verify by reading the actual skill file
  changes, since there's nothing to execute.

## Git

- Default branch: `main`.
- Commit history so far is minimal (initial commit + one skill-addition PR);
  there's no enforced commit message convention beyond being descriptive.
