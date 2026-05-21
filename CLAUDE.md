# CLAUDE.md — Resume Workspace

## Identity

This workspace belongs to a senior game client developer with 20+ years of experience.
Primary language for all output: **Korean**.

---

## Workspace Purpose

Resume and portfolio management workspace.
`_modules/project_modules.md` is the single source of truth — a library of project cards covering the full career history.

Core workflow:

1. Check `_workspace/CURRENT_TARGET.md` for active applications and workflow state.
2. Use `/apply-workflow <slug>` as the single entry point for any new application.
3. Read `_modules/project_modules.md` only at Phase 3 (card selection) — not upfront.
4. Draft using `_base/` templates. Output goes to `_output/{slug}/`.

---

## Directory Structure

```
resume-workspace/
├── CLAUDE.md
├── _modules/                        ← (private) project card library — Single Source of Truth
│   ├── project_modules.md           ← main material library (index + summaries)
│   ├── module_a.md                  ← project detail card
│   ├── module_b.md                  ← project detail card
│   └── module_c.md                  ← project detail card
├── _base/
│   ├── career_basic.html            ← career document base template
│   └── resume_basic.html            ← resume base template
├── _output/                         ← (private) company-specific outputs
│   └── {slug}/
│       ├── {slug}_{doc_type}.html
│       └── _context.md              ← JD 분석 + 카드 선택 근거 + Workflow State
├── _corpus/                         ← JD 아카이브 (향후 사용)
├── _workspace/                      ← 지원 상태 추적
│   ├── CURRENT_TARGET.md            ← 현재 활성 지원 건 + 상태
│   └── past_jd_analyses/            ← 완료 JD 아카이브
└── .claude/
    ├── rules/
    │   └── writing-style.md         ← tone, expression rules, HTML rules
    ├── skills/
    │   ├── resume-review.md         ← interviewer-perspective review (sub-routine)
    │   ├── resume-build.md          ← how to build a company-specific resume
    │   └── doc-consistency.md       ← cross-document fact validation
    └── commands/
        ├── apply-workflow.md        ← JD → 지원 완료 end-to-end workflow
        ├── module-card.md           ← how to write/update module cards
        └── git-commit.md            ← logical commit grouping
```

---

## Writing Principles (summary)

Full rules in `.claude/rules/writing-style.md`.

- Narrative flow: problem → judgment → outcome
- Figures: before/after format when available (e.g., 9시간 → 1시간)
- Tone: noun-form endings, no exaggeration, no standalone duration as achievement
- Troubleshooting: must include how the root cause was identified, not just what was fixed
- Output naming: `_output/{company}/{company}_{doc_type}.html`

---

## Session Behavior

- Always read `_modules/project_modules.md` before starting any resume task.
- Do not invent achievements or add information not present in the module cards.
- When module cards need updating, flag it explicitly — never edit silently.
- For unfamiliar tasks, check `.claude/skills/` first.
- Company-specific resumes go to `_output/` only — never overwrite `_base/` files.