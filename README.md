# Resume Workspace — AI 협업 이력서 관리 시스템

> 경력 데이터를 카드 단위로 보관하고, 지원 공고에 맞춰 재조합해 문서 4종을 생성하는 워크스페이스.

AI에게 이력서를 "써 달라"고 지시하는 방식이 아니라, AI가 매 세션 동일한 원본·규칙·절차를 따르도록 **시스템으로 고정**한 구조다. 코퍼스(RAG) → 파생 지식(wiki) → 서브에이전트 오케스트레이션의 세 레이어로 구성한다.

---

## 해결하는 문제

| 지원마다 반복되던 문제 | 이 시스템의 대응 |
| --- | --- |
| 회사가 바뀔 때마다 이력서를 처음부터 다시 작성 | 경력을 카드로 정제해 두고, JD에 맞는 카드만 재선택 |
| 세션이 바뀌면 AI 출력의 문체·수치 표기·구조가 달라짐 | 작문 규칙·카드 규격을 `.claude/rules/`에 명문화해 주입 |
| 문서를 4종 만들면 수치·서술이 서로 어긋남 | 문서 세트 간 사실 충돌을 감사 단계로 내장 |
| 강점·정체성을 세션마다 다시 분석 | `_wiki/`에 집계본을 만들어 두고 근거 레이어로 참조 |
| JD 접수부터 지원 완료까지 절차가 매번 즉흥적 | `/apply-workflow` 단일 플로우로 Phase 0~6 고정, 중단 시 재개 |
| AI가 카드에 없는 성과를 지어냄 | 비허구 원칙 명시 — 카드 데이터 밖 서술 금지 |
| 세션이 끊기면 맥락 소실 | `CURRENT_TARGET.md` → `_context.md` → `_wiki/` → `_corpus/` 순 선행 학습 |

---

## 설계 판단

프롬프트를 다듬는 대신 구조를 고정하는 쪽을 택했다. 네 가지 판단이 나머지 구성을 결정한다.

**Single Source of Truth**
경력 원본이 여러 회사 출력물에 흩어지면 어느 쪽이 사실인지 판정할 수 없다. 그래서 `_corpus/`를 유일한 원본으로 두고, 회사별 문서는 전부 파생물로 규정했다. 출력물만 고치고 원본을 두는 행위를 구조적으로 금지한다.

**규칙 명문화**
"좋은 문장을 써줘"는 검증할 수 없는 지시다. 그래서 서술 구조(문제 → 판단 → 결과), 수치 표기(`Before → After`), 금지 표현, 마크업 규약을 `.claude/rules/`에 코드처럼 문서화했다. AI 출력이 규칙 위반인지 아닌지를 사람이 판정할 수 있게 된다.

**컨텍스트 격리 서브에이전트**
분석·정제·빌드·검증을 한 스레드에서 처리하면 코퍼스를 반복 로드하고 환각 표면적이 넓어진다. 그래서 역할별 서브에이전트로 분리하고, 각 에이전트에는 메인 스레드가 **선별한 카드만** 주입한다. 읽기전용 에이전트는 보고만 반환하고, 실제 파일 수정 승인 게이트는 메인 스레드에 중앙화한다.

**소스와 생성물 분리**
페이지 번호·목차·슬라이드 번호를 사람이 세면 문서를 고칠 때마다 어긋난다. 그래서 문서는 Markdown으로만 쓰고 `tools/build_doc.py`가 HTML·PDF·TXT를 계산해 만든다. CSS는 `_template/assets/` 한 곳에 두고, 생성물 직접 수정은 금지한다.

---

## 한눈에 보는 흐름

```
채용공고 입수
    │
    ▼  Phase 1  Fit Check              → jd-analyst (읽기전용)
       JD 요구 추출 + Fit 리포트 → 지원 여부 Gate
    │
    ▼  Phase 2  문서 범위 확인
       resume / career / portfolio / coverletter 중 선택
    │
    ▼  Phase 3  카드 선별              → jd-analyst
       _corpus/ 1회 로드 → 매칭 근거 제시 → 사용자 승인
    │
    ▼  Phase 4  문서 빌드              → doc-builder (문서당 1, 병렬)
       _output/{slug}/{slug}_{doc_type}.md → build_doc.py → 생성물
    │
    ▼  Phase 5  품질 검증 (병렬)
       interviewer-reviewer  면접관 시점 리뷰 → High / Medium / Low
       consistency-auditor   다문서 사실 충돌 감사 → Fact conflict
    │  수정은 메인 스레드가 승인 후 수행
    ▼  Phase 6  마무리
       CURRENT_TARGET.md 갱신 · JD 아카이브 · _context.md 상태 기록
```

코퍼스 로드는 Phase 3에서 한 번만 일어나고, 이후 에이전트에는 선택된 카드 내용만 전달된다.

별도 파이프라인으로 `/ingest`(소스 문서 → 카드 정제)가 있고, 카드가 바뀌면 `/wiki-update`가 자동 체인되어 `_wiki/`를 재컴파일한다.

---

## 결과물

회사(slug)마다 문서 4종을 만든다. `.md`가 소스이고 나머지는 전부 생성물이다.

| 문서 | 소스 | 생성물 | 용도 |
| --- | --- | --- | --- |
| 이력서 | `{slug}_resume.md` | `.html` (`--pdf`) | A4 1~2장 압축 |
| 경력기술서 | `{slug}_career.md` | `.html` (`--pdf`) | 분량 제한 없는 상세 서술 |
| 포트폴리오 | `{slug}_portfolio.md` | `.html` (`--pdf`) | A4 landscape 슬라이드 |
| 자기소개서 | `{slug}_coverletter.md` | `.txt` | 채용 폼 붙여넣기 |

```bash
pip install -r tools/requirements.txt
python tools/build_doc.py _output/{slug}/{slug}_career.md --pdf
```

출력 종류는 파일명이 아니라 프론트매터 `doc_type:`이 결정한다(`career` / `resume` / `portfolio` / `coverletter`). `--pdf`는 playwright가 필요하다. 프로젝트 번호·페이지 번호·목차·타임라인 좌표는 렌더러가 계산하므로 소스에 쓰지 않는다.

> 지면 구성을 보여줄 산출물 스크린샷 자리. 저장소에 이미지 파일이 없어 링크는 두지 않았다.

---

## 시스템 구성

| Layer | Path | Role |
| --- | --- | --- |
| RAG / Corpus | `_corpus/` | 경력 카드 라이브러리. SSOT |
| Wiki Layer | `_wiki/` | 카드 파생 집계 — 정체성·강점·경력 아크·관계도·기술 맵 |
| RAG / History | `_workspace/past_jd_analyses/` | 과거 JD 분석 아카이브. 유사 공고 패턴 재활용 |
| Active State | `_workspace/CURRENT_TARGET.md` | 지원 현황·집중 직군·다음 액션 |
| App Context | `_output/{slug}/_context.md` | 회사별 JD 요약·카드 선택 근거·세션 노트 |
| Behavioral Memory | `.claude/rules/` | 문체·마크업 규칙, 카드 작성 규격 |
| Subagents | `.claude/agents/` | 역할별 위임 + 컨텍스트 격리 |
| Procedures | `.claude/skills/` | 내부 서브 프로시저 |
| Commands | `.claude/commands/` | 사용자 직접 호출 슬래시 커맨드 |
| Renderer | `tools/build_doc.py` | `.md` → `.html`(+PDF) / `.txt` |

```
resume/
├── CLAUDE.md          워크스페이스 규칙·레이어 정의 (AI 세션 진입점)
├── _corpus/           카드 7장 — 허브 인덱스 project_modules.md + 상세 카드 6장
├── _wiki/             파생 집계 5종 + 레이어 규칙 README (/wiki-update 로만 재생성)
├── _workspace/        CURRENT_TARGET.md + past_jd_analyses/ (JD 분석 7건)
├── _template/         템플릿 4종 + assets/ CSS 4개 (수정 금지)
├── _output/{slug}/    회사별 문서 — .md 소스 · _context.md · 생성물(gitignore)
├── _portfolio/        코드 포트폴리오 샘플 (TurnBasedCombat_Portfolio.cs)
├── tools/             build_doc.py 렌더러 + requirements.txt
└── .claude/           agents 6 · commands 4 · skills 5 · rules 2
```

`_template/assets/`는 `base.css`(타이포·페이지·인쇄 공통) + `career.css` · `resume.css` · `portfolio.css`. portfolio는 landscape 규칙이 base와 충돌해 단독 로드한다.

---

<details>
<summary><strong>컴포넌트 레퍼런스</strong> — agents · commands · skills · rules</summary>

각 파일이 절차의 원본이다. 아래는 색인이며, 상세는 링크한 파일에만 존재한다.

### agents/ — 서브에이전트 6종

| 에이전트 | 역할 | 스코프 | 모델 | 위임 지점 |
| --- | --- | --- | --- | --- |
| [`jd-analyst`](.claude/agents/jd-analyst.md) | JD 요구 추출·Fit 판단·카드 매칭 → Fit Report 반환 | 읽기전용 | sonnet | apply-workflow Phase 1·3 |
| [`corpus-curator`](.claude/agents/corpus-curator.md) | 소스 문서 → card-spec 규격 카드로 정제 (드래프트 → 승인 후 Write) | Write (`_corpus/`까지) | opus | `/ingest` |
| [`doc-builder`](.claude/agents/doc-builder.md) | 선별 카드 → 문서 소스 1종 조립 후 빌드 | Write (`_output/`) | sonnet | Phase 4 (문서 2종+ 병렬) |
| [`interviewer-reviewer`](.claude/agents/interviewer-reviewer.md) | 시니어 면접관 페르소나로 규격 위반·과장 탐지, 심각도 분류 | 읽기전용 | opus | Phase 5 |
| [`consistency-auditor`](.claude/agents/consistency-auditor.md) | 다문서 간 수치·사건·재직기간 사실 충돌 감사 | 읽기전용 | sonnet | Phase 5 (문서 2종+) |
| [`technical-writer`](.claude/agents/technical-writer.md) | README 등 최상위 문서 작성·개편. 파일·git 히스토리 검증 후 서술 | Write (`README.md`만) | opus | "README 정리" 요청 |

### commands/ — 슬래시 커맨드 4종

| 커맨드 | 역할 |
| --- | --- |
| [`/apply-workflow`](.claude/commands/apply-workflow.md) | JD 접수 → 지원 완료까지 Phase 0~6 단일 플로우. 중단 시 재개 |
| [`/ingest`](.claude/commands/ingest.md) | 소스 문서 → corpus 카드 정제. `/wiki-update` 자동 체인 |
| [`/wiki-update`](.claude/commands/wiki-update.md) | `_corpus/` → `_wiki/` 5개 파일 재컴파일. 단방향, 쓰기 권한 독점 |
| [`/git-commit`](.claude/commands/git-commit.md) | `git diff`를 논리 그룹으로 분류 후 구조화 커밋 메시지 제안 |

### skills/ — 내부 서브 프로시저 5종

| 스킬 | 역할 |
| --- | --- |
| [`resume-build`](.claude/skills/resume-build.md) | `resume`·`career` 소스 작성 절차 (JD 분석 → 카드 선택 → 승인 → 작성 → 빌드 → 검토) |
| [`portfolio-build`](.claude/skills/portfolio-build.md) | 포트폴리오 소스 작성 절차. 슬라이드 타입별 meta 규약 |
| [`coverletter-build`](.claude/skills/coverletter-build.md) | 자기소개서 소스 작성 절차. JD 도메인별 도입부·사례 선택 |
| [`resume-review`](.claude/skills/resume-review.md) | 면접관 페르소나 리뷰 기준·심각도 분류 |
| [`doc-consistency`](.claude/skills/doc-consistency.md) | 문서 세트 간 사실 충돌(필수 수정) / 표현 차이(선택 수정) 판정 |

### rules/ — 행동 규칙 2종

| 규칙 | 역할 |
| --- | --- |
| [`writing-style`](.claude/rules/writing-style.md) | 서술 구조·명사체 종결·수치 `Before → After`·금지 표현·md 마크업 규약 |
| [`card-spec`](.claude/rules/card-spec.md) | 카드 6섹션 고정 구조 (요약 / 기술 스택 / 핵심 설계 / 트러블슈팅 / 성과 / 동기화 트리거) |

</details>

---

*Powered by [Claude Code](https://claude.ai/code) — Anthropic*
