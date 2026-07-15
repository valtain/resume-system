# Resume Workspace: Jang Tae-beom

## Identity

- **User**: Senior Game Client Developer (20+ Years, Neowiz)
- **Primary Language**: Korean

## Core Rules

- **SSOT**: `_corpus/project_modules.md` (모든 작업 전 반드시 선행 학습)
- **Flow**: Module Card 선택 → `_template/` 템플릿 적용 → `_output/{company}/*.md` 작성 → `tools/build_doc.py` 빌드
- **Source vs Artifact**: 4종 문서(resume·career·portfolio·coverletter) 모두 `.md` 가 소스.
  생성물은 `.html`(resume/career/portfolio) · `.txt`(coverletter) · `.pdf`.
  **생성물 직접 수정 금지** — md 를 고치고 다시 빌드한다. CSS 는 `_template/assets/` 한 곳에만 둔다.
  페이지 번호·슬라이드 번호·목차·타임라인 좌표는 렌더러가 계산한다.
- **Constraint**: 허구 내용 작성 금지. 카드 데이터 내에서만 서술.
- **Safety**: `_template/` 파일 수정 절대 금지.
- **Wiki**: `_wiki/`는 corpus 파생 집계본. 참조만 가능. `/wiki-update` 없이 직접 수정 금지.

## Workspace Architecture (AI-Native Layers)

| Layer | Path | Role |
| ----- | ---- | ---- |
| RAG / Corpus | `_corpus/` | 전체 경력 카드 라이브러리. 검색·선별 시 참조. SSOT. |
| Wiki Layer | `_wiki/` | corpus 카드에서 파생된 지식 집계. 정체성 태그·강점 맵·경력 아크·프로젝트 관계도. `/wiki-update`로만 재생성. |
| RAG / History | `_workspace/past_jd_analyses/` | 과거 JD 분석 아카이브. 유사 공고 작업 시 패턴 재활용. |
| Active State | `_workspace/CURRENT_TARGET.md` | 현재 지원 현황·집중 직군·다음 액션. 세션 간 상태 유지. |
| App Context | `_output/{company}/_context.md` | 회사별 JD 요약·카드 선택 근거·세션 노트. 세션 재개 시 선행 학습. |
| Behavioral Memory | `.claude/rules/` | 문체·출력 규칙. 카드 작성 규격. AI 행동 기준. |
| Procedures | `.claude/skills/` | 내부 서브 프로시저 (resume-build, resume-review, doc-consistency). |
| Commands | `.claude/commands/` | 사용자 직접 호출 진입점 슬래시 커맨드. |
| Output | `_output/{company}/` | 회사별 문서. `.md` 가 소스, `.html`/`.txt`/`.pdf` 는 생성물. |
| Templates | `_template/` | 문서 템플릿(`*_basic.md`) + 스타일(`assets/*.css`). 수정 금지. |
| Renderer | `tools/build_doc.py` | `.md` → `.html`(+`--pdf`) / `.txt`. 번호·목차 자동 계산. |

## Directory Structure

- `_corpus/`: 원천 데이터 (Project Library / RAG Corpus)
- `_template/`: 문서 템플릿 + `assets/` CSS (Read-only)
- `_output/`: 최종 결과물 (Company-specific)
- `tools/`: `build_doc.py` 렌더러 (`pip install -r tools/requirements.txt`)
- `_workspace/`: 워크스페이스 레벨 상태 및 참조 문서
- `.claude/rules/`: 문체 규칙(writing-style.md) + 카드 작성 규격(card-spec.md)
- `.claude/skills/`: 내부 서브 프로시저 (resume-build, resume-review, doc-consistency)

## Session Strategy

- 세션 시작 시: `_workspace/CURRENT_TARGET.md` → `_output/{company}/_context.md` → `_wiki/identity.md` + `_wiki/strengths.md` (존재 시) → `_corpus/` 순으로 선행 학습.
- 신규 회사 작업 시: `_output/{company}/_context.md` 생성 후 `resume-build` 진행.
- 세션 종료 시: 지원 상태 변경 있으면 `CURRENT_TARGET.md` 업데이트 제안.
- 작업 완료 후 동기화 트리거 발생 시 카드 업데이트 제안.
- 카드 작성/수정 시: `/ingest` 사용. wiki-update 자동 체인됨.
