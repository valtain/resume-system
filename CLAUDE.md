# CLAUDE.md — Resume Workspace

## Identity

This workspace belongs to a senior game client developer with 20+ years of experience.
Primary language for all output: **Korean**.

---

## Workspace Purpose

Resume and portfolio management workspace.
`_modules/project_modules.md` is the single source of truth — a library of project cards covering the full career history.

Core workflow:
1. Read `_modules/project_modules.md` before any resume task.
2. Select relevant cards based on the job description provided.
3. Draft using `_base/` templates. Output goes to `_output/`.

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
│   └── {company}/
│       └── {company}_{doc_type}.html
└── .claude/
    ├── rules/
    │   └── writing-style.md         ← tone, expression rules, HTML rules
    ├── skills/
    │   ├── module-card.md           ← how to write/update module cards
    │   ├── resume-build.md          ← how to build a company-specific resume
    │   └── doc-consistency.md       ← cross-document fact validation
    └── commands/
        ├── resume-review.md         ← interviewer-perspective review
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