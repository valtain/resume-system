# Resume Workspace: Jang Tae-beom

## Identity

- **User**: Senior Game Client Developer (20+ Years, Neowiz)
- **Primary Language**: Korean

## Core Rules

- **SSOT**: `_corpus/project_modules.md` — 허브 인덱스(24KB). 상세는 같은 폴더의
  개별 카드(`zoneflow.md` 등)로 링크된다. 카드 선별 시점에 1회만 로드하고,
  이후 서브에이전트에는 선택된 카드 내용만 주입한다 (전량 재로드 금지).
- **Slug**: `_output/{slug}/` 폴더명이 곧 파일 prefix. 소문자 영문·구분자 없음
  (`linegames`, `line_games` 아님). 같은 회사 재지원은 기존 slug 재사용.
- **Flow**: Module Card 선택 → `_template/` 템플릿 적용 → `_output/{slug}/*.md` 작성 → `tools/build_doc.py` 빌드
- **Build**: `python tools/build_doc.py _output/{slug}/{slug}_{type}.md [--pdf]`
  출력 종류는 파일명이 아니라 프론트매터 `doc_type:` 이 결정. `--pdf` 는 playwright 필요.
  md 마크업·meta 블록 키·페이지 타입 전체 스펙은 `tools/build_doc.py` 상단 docstring 이 원본.
- **미이관 문서**: md 소스 없이 `.html`/`.txt` 만 추적 중인 과거 회사 문서가 있다
  (`.gitignore` 예외). 수정 요청 시 생성물을 직접 고치지 말고 md 로 먼저 이관한 뒤
  `git rm --cached` 로 생성물을 떼어낸다.
- **Source vs Artifact**: 4종 문서(resume·career·portfolio·coverletter) 모두 `.md` 가 소스.
  생성물은 `.html`(resume/career/portfolio) · `.txt`(coverletter) · `.pdf`.
  **생성물 직접 수정 금지** — md 를 고치고 다시 빌드한다. CSS 는 `_template/assets/` 한 곳에만 둔다.
  페이지 번호·슬라이드 번호·목차·타임라인 좌표는 렌더러가 계산한다.
- **Constraint**: 허구 내용 작성 금지. 카드 데이터 내에서만 서술.
- **Safety**: `_template/` 파일 수정 절대 금지.
- **Wiki**: `_wiki/`는 corpus 파생 집계본. 참조만 가능. `/wiki-update` 없이 직접 수정 금지.
- **README**: 이 시스템 자체를 보여주는 포트폴리오 문서. 직접 편집 대신 `technical-writer`
  에이전트에 위임한다 — 개수·경로·확장자는 반드시 파일로 검증 후 기재.

## Workspace Architecture (AI-Native Layers)

| Layer | Path | Role |
| ----- | ---- | ---- |
| RAG / Corpus | `_corpus/` | 전체 경력 카드 라이브러리. 검색·선별 시 참조. SSOT. |
| Wiki Layer | `_wiki/` | corpus 카드에서 파생된 지식 집계. 정체성 태그·강점 맵·경력 아크·프로젝트 관계도. `/wiki-update`로만 재생성. |
| RAG / History | `_workspace/past_jd_analyses/` | 과거 JD 분석 아카이브. 유사 공고 작업 시 패턴 재활용. |
| Active State | `_workspace/CURRENT_TARGET.md` | 현재 지원 현황·집중 직군·다음 액션. 세션 간 상태 유지. |
| App Context | `_output/{slug}/_context.md` | 회사별 JD 요약·카드 선택 근거·세션 노트. 세션 재개 시 선행 학습. |
| Behavioral Memory | `.claude/rules/` | 문체·출력 규칙. 카드 작성 규격. AI 행동 기준. |
| Subagents | `.claude/agents/` | 역할별 위임 + 컨텍스트 격리. 분석·빌드는 주입받은 카드만 사용하고, 수정 승인 게이트는 메인 스레드가 보유. |
| Procedures | `.claude/skills/` | 내부 서브 프로시저 (resume-build, coverletter-build, portfolio-build, resume-review, doc-consistency). |
| Commands | `.claude/commands/` | 사용자 직접 호출 진입점 슬래시 커맨드. |
| Output | `_output/{slug}/` | 회사별 문서. `.md` 가 소스, `.html`/`.txt`/`.pdf` 는 생성물. |
| Templates | `_template/` | 문서 템플릿(`*_basic.md`) + 스타일(`assets/*.css`). 수정 금지. |
| Renderer | `tools/build_doc.py` | `.md` → `.html`(+`--pdf`) / `.txt`. 번호·목차 자동 계산. |

## File Inventory

- `tools/`: `build_doc.py` 렌더러 (`pip install -r tools/requirements.txt`)
- `.claude/rules/`: 문체 규칙(writing-style.md) + 카드 작성 규격(card-spec.md)
- `.claude/skills/`: 내부 서브 프로시저 (resume-build, coverletter-build, portfolio-build, resume-review, doc-consistency)
- `.claude/agents/`: 서브에이전트 (jd-analyst, corpus-curator, doc-builder, interviewer-reviewer, consistency-auditor, technical-writer)
- `_portfolio/`: 코드 포트폴리오 샘플 (`TurnBasedCombat_Portfolio.cs`)

## Session Strategy

- 세션 시작 시: `_workspace/CURRENT_TARGET.md` → `_output/{slug}/_context.md` → `_wiki/identity.md` + `_wiki/strengths.md` (존재 시) → `_corpus/` 순으로 선행 학습.
- 신규 회사 작업 시: `_output/{slug}/_context.md` 생성 후 `resume-build` 진행.
- 세션 종료 시: 지원 상태 변경 있으면 `CURRENT_TARGET.md` 업데이트 제안.
- 작업 완료 후 동기화 트리거 발생 시 카드 업데이트 제안.
- 카드 작성/수정 시: `/ingest` 사용. wiki-update 자동 체인됨.
- 문서 2종 이상 동시 작업 시: `.claude/agents/` 로 위임해 컨텍스트 격리 (apply-workflow Phase 4·5 기준).
